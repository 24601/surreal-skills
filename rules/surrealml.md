# SurrealML -- Machine Learning Models in SurrealDB

SurrealML is the SurrealDB project for storing, versioning, and executing
machine learning models inside the database. Models are uploaded as `.surml`
artifacts (a packaging format that bundles ONNX or PyTorch state, metadata,
and an inference signature), addressable as namespaced functions, and
invokable from any SurrealQL statement -- including inside permissions
predicates, futures, and live queries.

> The artifact extension `.surml` is fixed; the upstream toolchain (`surrealml`
> Python SDK, `surreal ml` CLI subcommand) handles compilation from native
> framework formats.

---

## When to Use SurrealML

| Use case | Why in-database fits |
|----------|----------------------|
| Real-time inference colocated with the data being scored | Zero round-trip; one SurrealQL statement scores + persists the result |
| Embeddings generated on insert (BEFORE write events) | Trigger a model from a `DEFINE EVENT` and store the vector in the same record |
| Deterministic feature pipelines | Models live next to the data they consume; no skew between training and serving stores |
| Multi-tenant model serving with permission scoping | DEFINE PERMISSIONS on the model function applies row-level rules to inference |
| Audit + lineage requirements | Models are versioned objects in the DB, queryable by `INFO` |

If you serve a single model behind a high-QPS HTTP API with no DB coupling,
keep the model in a dedicated serving runtime (Triton, Ray Serve, BentoML).
SurrealML's value is the *colocation*.

---

## Supported Frameworks

| Source format | Path into SurrealDB | Notes |
|---------------|---------------------|-------|
| PyTorch (`.pt`, `.pth`) | `surrealml.SurMlFile.from_pytorch(...)` | Eager + scripted modules |
| ONNX (`.onnx`) | `surrealml.SurMlFile.from_onnx(...)` | Preferred portable format |
| Scikit-learn | `surrealml.SurMlFile.from_sklearn(...)` | Wraps to ONNX internally |
| TensorFlow / Keras | `surrealml.SurMlFile.from_keras(...)` | Wraps to ONNX internally |
| HuggingFace transformers | `surrealml.SurMlFile.from_hf(...)` | Auto-extracts the inference graph |

All paths land at the same on-disk artifact: a `.surml` file containing the
ONNX graph, normalization stats, version metadata, and a typed input/output
signature.

---

## Authoring a Model Artifact (`.surml`)

```python
from surrealml import SurMlFile, ModelMeta
import torch

class Linear(torch.nn.Module):
    def __init__(self):
        super().__init__()
        self.fc = torch.nn.Linear(4, 1)
    def forward(self, x): return self.fc(x)

model = Linear().eval()
example = torch.randn(1, 4)

surml = SurMlFile.from_pytorch(
    model=model,
    example_input=example,
    meta=ModelMeta(
        name="house_price_v1",
        version="0.1.0",
        description="Linear regression for tutorial",
        inputs=[
            {"name": "sqft",      "dtype": "float32"},
            {"name": "bedrooms",  "dtype": "float32"},
            {"name": "bathrooms", "dtype": "float32"},
            {"name": "age",       "dtype": "float32"},
        ],
        outputs=[{"name": "price", "dtype": "float32"}],
        normalization={"sqft": {"mean": 1850.0, "std": 540.0}},
    ),
)

surml.save("house_price_v1.surml")
```

The `ModelMeta` is mandatory in current toolchain versions; SurrealDB uses it
to validate inference inputs at query time and to populate `INFO FOR DB`
output.

---

## Uploading the Model

### Via SurrealQL

```surql
-- Upload from a base64-encoded .surml blob (suitable for migrations and rollouts)
DEFINE MODEL ml::house_price<0.1.0>(sqft: number, bedrooms: number, bathrooms: number, age: number) -> number
  COMMENT "Linear regression for tutorial"
  CONTENTS $surml_bytes;

-- Inspect uploaded models
INFO FOR DB;       -- includes "models" section
INFO FOR MODEL ml::house_price<0.1.0>;
```

### Via the CLI

```bash
surreal ml import \
  --endpoint http://localhost:8000 \
  --user root --pass root \
  --namespace test --database test \
  --name house_price --version 0.1.0 \
  house_price_v1.surml
```

### Via the Python SDK

```python
from surrealdb import Surreal
from surrealml import SurMlFile

surml = SurMlFile.load("house_price_v1.surml")
with Surreal("ws://localhost:8000/rpc") as db:
    db.signin({"username": "root", "password": "root"})
    db.use("test", "test")
    db.upload_ml(surml)         # uploads as ml::house_price<0.1.0>
```

---

## Calling Models from SurrealQL

Once defined, the model is a callable function under the `ml::` namespace:

```surql
-- Direct invocation
RETURN ml::house_price<0.1.0>(2400, 4, 2.5, 12);

-- Pull features from a record
SELECT
  *,
  ml::house_price<0.1.0>(sqft, bedrooms, bathrooms, age) AS predicted_price
FROM listing
WHERE city = "Phoenix";

-- Use in a computed field
DEFINE FIELD predicted_price ON TABLE listing
  VALUE ml::house_price<0.1.0>(sqft, bedrooms, bathrooms, age);

-- Use in a permission rule (deny low-confidence rows)
DEFINE TABLE listing PERMISSIONS
  FOR select WHERE ml::quality_classifier<1.0.0>(features) > 0.8;

-- Use in a BEFORE-write event to populate embeddings
DEFINE EVENT compute_embedding ON TABLE document WHEN $event = "CREATE" THEN {
  UPDATE $value SET embedding = ml::sentence_encoder<2.0.0>(content);
};
```

Versioning is part of the call site: `ml::name<version>` lets multiple
versions coexist in the same database. Drop a version with `REMOVE MODEL
ml::name<version>;`.

---

## Storage and Indexing

Models live in the database's `models` namespace, persisted to whichever
storage engine the DB is using (memory / RocksDB / SurrealKV / TiKV). For
TiKV-backed clusters, large models are sharded the same way as oversize
record values; consult `rules/deployment.md` for storage-engine sizing.

Embedding outputs typically feed a vector index. Combine with
`rules/vector-search.md`:

```surql
DEFINE TABLE document SCHEMAFULL;
DEFINE FIELD content   ON TABLE document TYPE string;
DEFINE FIELD embedding ON TABLE document TYPE array<float>
  VALUE ml::sentence_encoder<2.0.0>(content);
DEFINE INDEX idx_doc_embedding ON TABLE document FIELDS embedding
  HNSW DIMENSION 384 DIST COSINE;
```

Inserts now compute the embedding server-side; clients never see the model
weights.

---

## Permissioning Model Calls

Models are first-class definable objects, so they accept the same `PERMISSIONS`
and `COMMENT` blocks as tables and functions:

```surql
DEFINE MODEL ml::pricing_model<2.0.0>(features: array<number>) -> number
  PERMISSIONS FOR select WHERE $auth.role IN ["pricing_team", "admin"]
  COMMENT "Internal pricing -- not for customer queries";
```

Combined with row-level permissions, this means model inference inherits the
same trust boundary as any other DB read; an unprivileged user cannot invoke
an internal model, even via a custom Surrealism extension that wraps it.

---

## Versioning and Rollouts

Treat model upgrades as schema changes -- manage them through SurrealKit
rollouts (`rules/surrealkit.md`):

```toml
# database/rollouts/20260503150000__upgrade_pricing_model.toml
[expand]
sql = """
DEFINE MODEL ml::pricing_model<2.1.0>(features: array<number>) -> number
  CONTENTS $surml_bytes;
"""

[contract]
sql = """
REMOVE MODEL ml::pricing_model<2.0.0>;
"""
```

The expand phase introduces the new version while keeping the old one usable;
the contract phase removes the old version after the application cuts over.
This pattern matches `rules/data-modeling.md` migration recipes.

---

## Surrealism Versus SurrealML

| Need | Pick |
|------|------|
| Deploy a trained model and call it from queries | SurrealML |
| Implement a custom analyzer, tokenizer, or arbitrary Rust logic | Surrealism (`rules/surrealism.md`) |
| Need both | Use Surrealism to call the SurrealML model with custom pre/post-processing |

SurrealML is purpose-built for ML inference; Surrealism is the general WASM
extension surface. They compose -- a Surrealism module can invoke
`ml::name<version>(...)` like any SurrealQL function.

---

## Operational Considerations

| Concern | Recommendation |
|---------|----------------|
| Memory pressure | ONNX models load into RAM at call time; size the host accordingly. Use `surreal start --user-mem-limit ...` to bound it. |
| Cold-start latency | First call after server start triggers ONNX runtime init. Precompute embeddings in a `DEFINE EVENT` rather than at query time for low-tail-latency reads. |
| GPU acceleration | The reference Surreal binary is CPU-only. Compile with the `gpu` feature for CUDA/Metal builds (advanced; see upstream notes). |
| Model size | TiKV recommended for >500MB artifacts; RocksDB and SurrealKV handle them but reads pull the full blob into memory. |
| Backup | `surreal export` includes models. Verify with `surreal import` to a clean instance. |

---

## Cross-References

- `rules/vector-search.md` -- where SurrealML embeddings most often land
- `rules/data-modeling.md` -- computed fields, BEFORE-write events
- `rules/surrealism.md` -- Rust-based extensions (composes with SurrealML)
- `rules/surrealkit.md` -- rollout-managed model upgrades
- `rules/security.md` -- permissioning model calls
- `rules/langchain.md` -- when you want a Python LangChain pipeline to use
  SurrealML server-side embeddings instead of OpenAI / HuggingFace
- `rules/sdks.md` -- `upload_ml` and friends across SDKs
