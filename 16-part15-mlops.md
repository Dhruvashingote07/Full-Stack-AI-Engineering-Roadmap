# Part 15: MLOps

## Chapter 1: Introduction to MLOps

MLOps (Machine Learning Operations) is a set of practices that combines ML, DevOps, and Data Engineering to deploy and maintain ML systems in production reliably and efficiently. It applies DevOps principles to the ML lifecycle, enabling teams to automate, monitor, and govern ML workflows at scale.

### The ML Lifecycle

Data Collection → Data Validation → Feature Engineering → Model Training → Model Evaluation → Model Deployment → Model Monitoring → Retraining Loop

### Why MLOps Matters

- **Reproducibility**: Ensures experiments can be reproduced exactly
- **Collaboration**: Enables data scientists and engineers to work together
- **Automation**: Automates the ML pipeline from data to deployment
- **Governance**: Tracks model versions, data lineage, and decisions
- **Scaling**: Deploys and manages models at scale across environments
- **Monitoring**: Detects data drift, concept drift, and model degradation

### MLOps Maturity Levels

| Level | Name | Characteristics |
|-------|------|-----------------|
| 0 | No MLOps | Manual, script-based, no tracking |
| 1 | DevOps but no MLOps | Automated CI/CD, manual ML steps |
| 2 | Automated Training | Experiment tracking, pipeline automation |
| 3 | Automated Deployment | Continuous training, model registry, A/B testing |
| 4 | Full MLOps | Auto-retraining, drift detection, full governance |

> 💡 **Pro Tip:** Don't aim for Level 4 MLOps from day one. Start with experiment tracking (Level 1), add pipeline automation (Level 2), then gradually introduce model registry and deployment automation. Most teams never need Level 4 — Level 2 or 3 is sufficient for 90% of ML projects.

---

## Chapter 2: MLflow — Experiment Tracking

MLflow is an open-source platform for managing the ML lifecycle. Its tracking component allows you to log parameters, metrics, and artifacts from ML experiments.

### Core Concepts

- **Experiments**: Logical grouping of runs
- **Runs**: Single execution of a model training script
- **Parameters**: Input configurations (e.g., learning rate, batch size)
- **Metrics**: Evaluation results (e.g., accuracy, loss)
- **Artifacts**: Output files (e.g., model files, plots, data)

### Setting Up MLflow

```bash
pip install mlflow
mlflow server --host 0.0.0.0 --port 5000
```

### Tracking an Experiment

```python
import mlflow
import mlflow.sklearn
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

mlflow.set_tracking_uri("http://localhost:5000")
mlflow.set_experiment("iris-classification")

with mlflow.start_run(run_name="rf-v1"):
    mlflow.log_param("n_estimators", 100)
    mlflow.log_param("max_depth", 10)

    model = RandomForestClassifier(n_estimators=100, max_depth=10)
    model.fit(X_train, y_train)

    predictions = model.predict(X_test)
    accuracy = accuracy_score(y_test, predictions)

    mlflow.log_metric("accuracy", accuracy)
    mlflow.sklearn.log_model(model, "model")
    mlflow.log_artifact("confusion_matrix.png")
```

### Autologging

```python
mlflow.autolog()
model = RandomForestClassifier(n_estimators=100)
model.fit(X_train, y_train)
```

### Nested Runs

```python
with mlflow.start_run(run_name="parent-run"):
    mlflow.log_param("parent_param", 42)
    with mlflow.start_run(run_name="child-run", nested=True):
        mlflow.log_param("child_param", 24)
        mlflow.log_metric("child_metric", 0.95)
```

### MLflow Tracking UI

The MLflow UI provides experiment comparison with parallel coordinates plots, run detail pages with all logged data, artifact browser, and model registration from the UI.

### Best Practices for Experiment Tracking

- Use descriptive experiment names
- Log all hyperparameters before training
- Log metrics at each epoch for time-series analysis
- Tag runs with metadata (e.g., dataset version, environment)
- Use `mlflow.start_run(run_name="descriptive-name")`
- Log dataset hashes for reproducibility

> **Common Mistake**: Forgetting to set the tracking URI leads to runs being logged locally instead of the shared server.

> ✅ **Best Practice:** Log dataset hashes along with model parameters. When a model degrades in production, you can trace exactly which data version it was trained on. Combine with `mlflow.log_input()` in newer versions for first-class dataset tracking.

---

## Chapter 3: MLflow Models and Model Registry

### MLflow Models

MLflow provides a standard format for packaging ML models that can be used with various deployment tools.

```python
mlflow.sklearn.log_model(model, "model")
loaded_model = mlflow.sklearn.load_model("runs:/<run-id>/model")
predictions = loaded_model.predict(X_test)
```

### Model Flavors

| Flavor | Description |
|--------|-------------|
| `python_function` | Generic Python function |
| `sklearn` | scikit-learn models |
| `pytorch` | PyTorch models |
| `tensorflow` | TensorFlow models |
| `onnx` | ONNX models |
| `keras` | Keras models |
| `spark` | Spark MLlib models |
| `lightgbm` | LightGBM models |
| `xgboost` | XGBoost models |

### Model Registry

```python
from mlflow.tracking import MlflowClient

client = MlflowClient()
client.create_registered_model("IrisClassifier")
result = mlflow.register_model("runs:/<run-id>/model", "IrisClassifier")
client.transition_model_version_stage(name="IrisClassifier", version=1, stage="Staging")
versions = client.get_latest_versions("IrisClassifier", stages=["Staging"])
```

### Model Stages

| Stage | Purpose |
|-------|---------|
| `None` | Unregistered version |
| `Staging` | Testing and validation |
| `Production` | Live serving |
| `Archived` | Deprecated versions |

### Loading Models by Stage

```python
model = mlflow.pyfunc.load_model(model_uri="models:/IrisClassifier/Production")
model = mlflow.pyfunc.load_model(model_uri="models:/IrisClassifier/2")
```

> 💡 **Pro Tip:** Use model stages (Staging → Production → Archived) to gate deployments. Promote models through environments with validation at each step: Staging for automated tests, Production after manual approval. Never load a model by version number in production — always use `stage="Production"`.

---

## Chapter 4: MLflow Projects

MLflow Projects provide a reusable and reproducible way to package ML code.

### MLproject File

```yaml
name: IrisClassifier
conda_env: conda.yaml
entry_points:
  train:
    parameters:
      n_estimators: {type: int, default: 100}
      max_depth: {type: int, default: 10}
    command: "python train.py --n_estimators {n_estimators} --max_depth {max_depth}"
  evaluate:
    parameters:
      model_uri: {type: string}
    command: "python evaluate.py --model_uri {model_uri}"
  predict:
    parameters:
      model_uri: {type: string}
      input_data: {type: string}
    command: "python predict.py --model_uri {model_uri} --input_data {input_data}"
```

### Running Projects

```bash
mlflow run . -P n_estimators=200 -P max_depth=15
mlflow run . --docker-image python:3.9
mlflow run https://github.com/user/repo.git -P n_estimators=100
```

### Conda Environment

```yaml
name: iris-env
channels:
  - defaults
dependencies:
  - python=3.9
  - pip
  - pip:
    - mlflow>=2.0
    - scikit-learn>=1.0
    - pandas
    - numpy
```

---

## Chapter 5: Kubeflow — Pipelines and Components

Kubeflow is a Kubernetes-native platform for developing, orchestrating, deploying, and running ML workflows.

### Kubeflow Pipelines

```python
from kfp import dsl
from kfp import compiler
import kfp

@dsl.component
def train_model(dataset: str, learning_rate: float = 0.01, epochs: int = 10) -> str:
    import mlflow
    import pandas as pd
    from sklearn.linear_model import LogisticRegression
    with mlflow.start_run():
        mlflow.log_param("learning_rate", learning_rate)
        data = pd.read_csv(dataset)
        model = LogisticRegression(C=learning_rate)
        model.fit(data.drop("target", axis=1), data["target"])
        mlflow.sklearn.log_model(model, "model")
        return mlflow.active_run().info.run_id

@dsl.pipeline(name="Iris Training Pipeline")
def iris_pipeline(dataset: str = "gs://bucket/iris.csv", learning_rate: float = 0.01):
    train_task = train_model(dataset=dataset, learning_rate=learning_rate)
    evaluate_task = evaluate_model(model_uri=train_task.output)

compiler.Compiler().compile(iris_pipeline, "pipeline.yaml")
kfp.Client(host="https://your-kubeflow-host").create_run_from_pipeline_func(
    iris_pipeline, arguments={"dataset": "gs://bucket/iris.csv"}
)
```

### Custom Components

```python
from kfp.dsl import component, InputPath, OutputPath

@component(base_image="python:3.9", packages_to_install=["pandas", "scikit-learn"])
def preprocess_data(input_path: InputPath(str), output_path: OutputPath(str), test_size: float = 0.2):
    import pandas as pd
    from sklearn.model_selection import train_test_split
    data = pd.read_csv(input_path)
    train, test = train_test_split(data, test_size=test_size)
    train.to_csv(f"{output_path}/train.csv", index=False)
    test.to_csv(f"{output_path}/test.csv", index=False)
```

### Reusable Components YAML

```yaml
name: Preprocess Data
description: Split data into train/test sets
inputs:
  - name: input_path
    type: String
  - name: test_size
    type: Float
    default: 0.2
outputs:
  - name: output_path
    type: String
implementation:
  container:
    image: python:3.9
    command:
      - python
      - -u
      - preprocess.py
    args:
      - --input-path
      - inputValue: input_path
      - --output-path
      - outputPath: output_path
```

### KFServing (now KServe)

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: iris-classifier
spec:
  predictor:
    pytorch:
      storageUri: gs://model-bucket/iris-model
      resources:
        limits:
          nvidia.com/gpu: "1"
```

> **Best Practice**: Use Kubeflow Pipelines for complex multi-step workflows but prefer lighter-weight solutions for simpler use cases.

---

## Chapter 6: Apache Airflow — DAGs and Operators

Apache Airflow is a platform to programmatically author, schedule, and monitor workflows.

### Core Concepts

- **DAG**: A collection of tasks with dependencies
- **Operator**: Defines a single task
- **Sensor**: Waits for an external event
- **Scheduler**: Triggers tasks when dependencies are met
- **Executor**: Defines how tasks are run
- **XCom**: Cross-communication between tasks

### Basic DAG Definition

```python
from datetime import datetime, timedelta
from airflow import DAG
from airflow.operators.python import PythonOperator
from airflow.operators.bash import BashOperator

default_args = {
    "owner": "ml-team",
    "depends_on_past": False,
    "email_on_failure": True,
    "retries": 3,
    "retry_delay": timedelta(minutes=5),
    "start_date": datetime(2024, 1, 1),
}

with DAG(
    dag_id="ml_training_pipeline",
    default_args=default_args,
    schedule_interval="@daily",
    catchup=False,
    tags=["ml", "training"],
) as dag:

    def extract_data(**context):
        execution_date = context["execution_date"]
        return {"record_count": 10000}

    def preprocess(**context):
        ti = context["ti"]
        record_count = ti.xcom_pull(task_ids="extract_data")
        return {"processed_records": record_count["record_count"]}

    def train_model(**context):
        ti = context["ti"]
        processed = ti.xcom_pull(task_ids="preprocess")
        return {"model_path": "/models/model_v1.pkl"}

    extract_task = PythonOperator(task_id="extract_data", python_callable=extract_data, provide_context=True)
    preprocess_task = PythonOperator(task_id="preprocess", python_callable=preprocess, provide_context=True)
    train_task = PythonOperator(task_id="train_model", python_callable=train_model, provide_context=True)
    backup_task = BashOperator(task_id="backup_data", bash_command="gsutil cp data.csv gs://bucket/backups/")

    extract_task >> preprocess_task >> train_task
    preprocess_task >> backup_task
```

### Operators

```python
from airflow.operators.python import BranchPythonOperator

def choose_branch(**kwargs):
    accuracy = kwargs["ti"].xcom_pull(task_ids="evaluate")
    if accuracy > 0.95:
        return "deploy_model"
    return "retrain_model"

branch_task = BranchPythonOperator(task_id="check_accuracy", python_callable=choose_branch, provide_context=True)
```

### Sensors

```python
from airflow.sensors.filesystem import FileSensor
from airflow.providers.http.sensors.http import HttpSensor

file_sensor = FileSensor(task_id="wait_for_data", filepath="/data/raw/new_data.csv", poke_interval=60, timeout=3600)

http_sensor = HttpSensor(
    task_id="wait_for_model_endpoint",
    http_conn_id="http_default",
    endpoint="health",
    response_check=lambda response: response.json()["status"] == "ready",
    poke_interval=30,
)
```

### Hooks

```python
from airflow.providers.postgres.hooks.postgres import PostgresHook

def query_database(**kwargs):
    hook = PostgresHook(postgres_conn_id="ml_db")
    df = hook.get_pandas_df("SELECT * FROM training_data WHERE date = %(date)s", parameters={"date": "2024-01-01"})
    return df.to_csv()
```

### XComs

```python
def push_data(**kwargs):
    ti = kwargs["ti"]
    ti.xcom_push(key="data_path", value="/data/train.csv")

def pull_data(**kwargs):
    ti = kwargs["ti"]
    data_path = ti.xcom_pull(key="data_path", task_ids="push_task")
    print(f"Received path: {data_path}")
```

### Best Practices

- Keep DAGs idempotent
- Use `max_active_runs=1` for sequential pipelines
- Set appropriate retry policies for transient failures
- Use connection pooling for database hooks
- Monitor task durations and set SLA misses

---

## Chapter 7: DVC — Data Version Control

DVC brings version control concepts to ML projects, enabling data and model versioning alongside code.

### Installation and Setup

```bash
pip install dvc
git init
dvc init
```

### DVC Pipelines

```yaml
stages:
  preprocess:
    cmd: python src/preprocess.py data/raw data/processed
    deps:
      - data/raw
      - src/preprocess.py
    params:
      - preprocess.test_size
      - preprocess.random_state
    outs:
      - data/processed/train.csv
      - data/processed/test.csv
  train:
    cmd: python src/train.py data/processed models/model.pkl
    deps:
      - data/processed
      - src/train.py
    params:
      - train.n_estimators
      - train.max_depth
    outs:
      - models/model.pkl
    metrics:
      - metrics/accuracy.json:
          cache: false
  evaluate:
    cmd: python src/evaluate.py models/model.pkl data/processed
    deps:
      - models/model.pkl
      - data/processed
      - src/evaluate.py
    metrics:
      - metrics/eval.json:
          cache: false
```

### Parameters File

```yaml
# params.yaml
preprocess:
  test_size: 0.2
  random_state: 42
train:
  n_estimators: 100
  max_depth: 10
  learning_rate: 0.1
```

### Running Pipelines

```bash
dvc repro
dvc repro train
dvc repro --force
dvc dag
dvc status
```

### Experiments

```bash
dvc exp run --set-param train.n_estimators=200
dvc exp run --queue --set-param train.n_estimators=50
dvc exp run --run-all
dvc exp show
dvc exp apply exp-abc123
```

### Remotes

```bash
dvc remote add -d myremote gs://my-bucket/dvc-store
dvc remote add -d s3remote s3://my-bucket/dvc
dvc push
dvc pull
```

### DVC Lock File

```yaml
# dvc.lock
schema: '2.0'
stages:
  preprocess:
    cmd: python src/preprocess.py data/raw data/processed
    deps:
      - path: data/raw
        md5: abc123def456
      - path: src/preprocess.py
        md5: 789ghi012jkl
    params:
      params.yaml:
        preprocess:
          test_size: 0.2
    outs:
      - path: data/processed/train.csv
        md5: 345mno678pqr
```

### Best Practices

- Store `dvc.lock` and `dvc.yaml` in git
- Add `.dvc/cache` to `.gitignore`
- Use descriptive stage names
- Use `dvc gc` to clean up unused cache files

> **Common Mistake**: Tracking large files directly in git. Always use `dvc add` for datasets and models larger than 100MB.

> 💡 **Pro Tip:** Use `dvc exp diff` to compare metrics across experiments and `dvc exp apply` to reproduce the best run exactly. This creates a fully reproducible pipeline where any team member can get the same results with a single command.

---

## Chapter 8: Weights & Biases (WandB)

Weights & Biases is a platform for tracking experiments, optimizing hyperparameters, and managing ML workflows.

### Installation and Setup

```bash
pip install wandb
wandb login
```

### Experiment Tracking

```python
import wandb
from sklearn.ensemble import RandomForestClassifier
from sklearn.metrics import accuracy_score

wandb.init(
    project="iris-classification",
    name="rf-experiment-1",
    config={"n_estimators": 100, "max_depth": 10, "learning_rate": 0.1},
    tags=["baseline", "v1"],
)

config = wandb.config
model = RandomForestClassifier(n_estimators=config.n_estimators, max_depth=config.max_depth)
model.fit(X_train, y_train)

for epoch in range(10):
    loss = train_step()
    accuracy = eval_step()
    wandb.log({"epoch": epoch, "train_loss": loss, "val_accuracy": accuracy})

wandb.log({"test_accuracy": accuracy_score(y_test, model.predict(X_test))})
wandb.sklearn.log_model(model, "iris-rf-model")
wandb.finish()
```

### Sweeps (Hyperparameter Optimization)

```python
import wandb

sweep_config = {
    "method": "bayes",
    "metric": {"name": "val_accuracy", "goal": "maximize"},
    "parameters": {
        "n_estimators": {"values": [50, 100, 200, 500]},
        "max_depth": {"min": 3, "max": 20},
        "learning_rate": {"distribution": "log_uniform", "min": 0.001, "max": 0.1},
    },
    "early_terminate": {"type": "hyperband", "min_iter": 3, "eta": 3},
}

sweep_id = wandb.sweep(sweep_config, project="iris-hpo")

def train():
    with wandb.init() as run:
        config = wandb.config
        model = RandomForestClassifier(n_estimators=config.n_estimators, max_depth=config.max_depth)
        model.fit(X_train, y_train)
        wandb.log({"val_accuracy": accuracy_score(y_test, model.predict(X_test))})

wandb.agent(sweep_id, train, count=50)
```

> ✅ **Best Practice:** Use WandB sweeps with Bayesian optimization for hyperparameter tuning. It's typically 3-5x more sample-efficient than grid search. Set early termination (HyperBand or median stopping) to kill underperforming runs early and focus compute on promising configurations.

### Artifacts

```python
run = wandb.init(project="iris-artifacts")

artifact = wandb.Artifact(name="iris_dataset", type="dataset", metadata={"source": "sklearn", "samples": 150})
artifact.add_file("data/processed/train.csv")
artifact.add_file("data/processed/test.csv")
run.log_artifact(artifact)

model_artifact = wandb.Artifact(name="iris_model", type="model")
model_artifact.add_file("models/model.pkl")
model_artifact.metadata = {"accuracy": 0.95}
run.log_artifact(model_artifact)

artifact = run.use_artifact("iris_dataset:v0")
artifact.download(path="./data")
```

### Integration with Frameworks

```python
# PyTorch Lightning
from pytorch_lightning.loggers import WandbLogger
wandb_logger = WandbLogger(project="lightning-project")
trainer = Trainer(logger=wandb_logger)

# Keras
import wandb
from wandb.keras import WandbMetricsLogger
wandb.init(project="keras-project")
model.fit(x_train, y_train, callbacks=[WandbMetricsLogger(log_freq=5)])
```

### Best Practices

- Use `wandb.Alert` for model degradation notifications
- Use tags and notes for filtering runs
- Set up team projects with shared dashboards
- Organize runs with `group` and `job_type` parameters

---

## Chapter 9: ONNX — Model Conversion and Runtime

ONNX is an open format to represent ML models, enabling interoperability between frameworks.

### PyTorch to ONNX

```python
import torch
import torchvision.models as models

model = models.resnet18(pretrained=True)
model.eval()

dummy_input = torch.randn(1, 3, 224, 224)

torch.onnx.export(
    model,
    dummy_input,
    "resnet18.onnx",
    export_params=True,
    opset_version=17,
    do_constant_folding=True,
    input_names=["input"],
    output_names=["output"],
    dynamic_axes={"input": {0: "batch_size"}, "output": {0: "batch_size"}}
)
```

### TensorFlow to ONNX

```bash
pip install tf2onnx
```

```python
import tensorflow as tf
import tf2onnx

model = tf.keras.models.load_model("model.h5")
spec = (tf.TensorSpec((None, 224, 224, 3), tf.float32, name="input"),)
model_proto, _ = tf2onnx.convert.from_keras(model, input_signature=spec, opset=17, output_path="model.onnx")
```

### ONNX Runtime Inference

```python
import onnxruntime as ort
import numpy as np

session = ort.InferenceSession("resnet18.onnx", providers=['CUDAExecutionProvider', 'CPUExecutionProvider'])

input_name = session.get_inputs()[0].name
output_name = session.get_outputs()[0].name

input_data = np.random.randn(1, 3, 224, 224).astype(np.float32)
outputs = session.run([output_name], {input_name: input_data})
predictions = outputs[0]
```

### ONNX Runtime Optimization

```python
import onnxruntime as ort
from onnxruntime.quantization import quantize_dynamic, QuantType

sess_options = ort.SessionOptions()
sess_options.graph_optimization_level = ort.GraphOptimizationLevel.ORT_ENABLE_ALL
sess_options.optimized_model_filepath = "optimized_model.onnx"

session = ort.InferenceSession("model.onnx", sess_options, providers=['CUDAExecutionProvider'])

quantize_dynamic("model.onnx", "model_quantized.onnx", weight_type=QuantType.QUInt8)
```

### ONNX Model Validation

```python
import onnx
from onnx import checker

model = onnx.load("model.onnx")
checker.check_model(model)
print(f"IR version: {model.ir_version}")
print(f"Graph name: {model.graph.name}")
for input_tensor in model.graph.input:
    print(f"Input: {input_tensor.name}, shape: {input_tensor.type.tensor_type.shape}")
```

> 💡 **Pro Tip:** ONNX Runtime can be 2-5x faster than native PyTorch/TensorFlow inference due to graph optimizations, operator fusion, and quantization. For production serving, convert models to ONNX and use ORT with the appropriate execution provider (CUDA for GPU, OpenVINO for Intel CPU).

---

## Chapter 10: TensorRT — Optimization and Deployment

NVIDIA TensorRT is an SDK for high-performance deep learning inference.

### FP16 Quantization

```python
import tensorrt as trt

def build_engine_fp16(onnx_file_path, engine_file_path):
    logger = trt.Logger(trt.Logger.WARNING)
    builder = trt.Builder(logger)
    network = builder.create_network(1 << int(trt.NetworkDefinitionCreationFlag.EXPLICIT_BATCH))
    parser = trt.OnnxParser(network, logger)

    with open(onnx_file_path, 'rb') as f:
        if not parser.parse(f.read()):
            for error in range(parser.num_errors):
                print(parser.get_error(error))
            return None

    config = builder.create_builder_config()
    config.set_flag(trt.BuilderFlag.FP16)
    config.set_memory_pool_limit(trt.MemoryPoolType.WORKSPACE, 1 << 30)

    serialized_engine = builder.build_serialized_network(network, config)
    with open(engine_file_path, 'wb') as f:
        f.write(serialized_engine)
    return serialized_engine
```

### TensorRT Inference

```python
import tensorrt as trt
import numpy as np

class TensorRTInference:
    def __init__(self, engine_file_path):
        logger = trt.Logger(trt.Logger.WARNING)
        runtime = trt.Runtime(logger)
        with open(engine_file_path, 'rb') as f:
            self.engine = runtime.deserialize_cuda_engine(f.read())
        self.context = self.engine.create_execution_context()

    def infer(self, input_data):
        # Set tensor addresses and execute
        self.context.set_tensor_address("input", input_data.ctypes.data)
        self.context.execute_v2([])
        return output
```

### Layer Fusion

TensorRT automatically fuses layers:
- Conv + BN + ReLU → ConvWithBNReLU
- Conv + Add + ReLU → ConvAddReLU
- FC + Softmax → FCWithSoftmax
- Multi-head attention → Fused MHA

### Performance Benchmarks

| Precision | Memory | Throughput | Accuracy (ResNet-50) |
|-----------|--------|------------|----------------------|
| FP32 | 1x | 1x | 76.13% |
| FP16 | 0.5x | 2x | 76.13% |
| INT8 | 0.25x | 4x | 75.91% |

---

## Chapter 11: Triton Inference Server

NVIDIA Triton Inference Server simplifies model deployment across multiple frameworks.

### Model Repository Structure

```
model_repository/
├── resnet50/
│   ├── 1/
│   │   └── model.onnx
│   └── config.pbtxt
├── bert/
│   ├── 1/
│   │   └── model.py
│   ├── 2/
│   │   └── model.pt
│   └── config.pbtxt
```

### Model Configuration

```protobuf
name: "resnet50"
platform: "onnxruntime_onnx"
max_batch_size: 64
input [
  { name: "input", data_type: TYPE_FP32, dims: [3, 224, 224] }
]
output [
  { name: "output", data_type: TYPE_FP32, dims: [1000] }
]
dynamic_batching {
  preferred_batch_size: [8, 16, 32]
  max_queue_delay_microseconds: 100
}
instance_group [
  { count: 2, kind: KIND_GPU, gpus: [0, 1] }
]
```

### Running Triton Server

```bash
docker pull nvcr.io/nvidia/tritonserver:23.12-py3
docker run --gpus all -p 8000:8000 -p 8001:8001 -p 8002:8002 \\
  -v /path/to/model_repository:/models \\
  nvcr.io/nvidia/tritonserver:23.12-py3 \\
  tritonserver --model-repository=/models
```

### Client Inference

```python
import tritonclient.http as httpclient
import numpy as np

client = httpclient.InferenceServerClient(url="localhost:8000")
print(client.is_server_live())

input_data = np.random.randn(1, 3, 224, 224).astype(np.float32)
input_tensor = httpclient.InferInput("input", input_data.shape, "FP32")
input_tensor.set_data_from_numpy(input_data)

result = client.infer("resnet50", [input_tensor])
output = result.as_numpy("output")
```

### Model Ensembles

```protobuf
name: "image_pipeline"
platform: "ensemble"
max_batch_size: 8
ensemble_scheduling {
  step [
    { model_name: "preprocess", model_version: 1,
      input_map: { key: "INPUT_IMAGE", value: "image" },
      output_map: { key: "PREPROCESSED", value: "preprocessed_image" } },
    { model_name: "resnet50", model_version: 1,
      input_map: { key: "input", value: "preprocessed_image" },
      output_map: { key: "output", value: "raw_predictions" } },
  ]
}
```

---

## Chapter 12: FastAPI for ML Deployment

FastAPI is a modern web framework for building APIs with Python.

### Basic App

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import numpy as np
import joblib
import uvicorn

app = FastAPI(title="ML Model API", version="1.0.0")
model = None

@app.on_event("startup")
async def load_model():
    global model
    model = joblib.load("models/model.pkl")

class PredictionInput(BaseModel):
    features: list[float]

class PredictionOutput(BaseModel):
    prediction: int
    probability: float

@app.get("/health")
async def health():
    return {"status": "healthy"}

@app.post("/predict", response_model=PredictionOutput)
async def predict(input_data: PredictionInput):
    try:
        features = np.array(input_data.features).reshape(1, -1)
        prediction = model.predict(features)[0]
        probabilities = model.predict_proba(features)[0]
        return PredictionOutput(prediction=int(prediction), probability=float(np.max(probabilities)))
    except Exception as e:
        raise HTTPException(status_code=500, detail=str(e))
```

### Running the Server

```bash
pip install fastapi uvicorn python-multipart
uvicorn main:app --host 0.0.0.0 --port 8000 --reload
uvicorn main:app --host 0.0.0.0 --port 8000 --workers 4 --loop uvloop
```

### Dependency Injection

```python
from fastapi import Depends

class MLModel:
    def __init__(self, model_path: str):
        self.model = joblib.load(model_path)

    async def predict(self, features: np.ndarray) -> np.ndarray:
        return self.model.predict(features)

async def get_model() -> MLModel:
    return MLModel("models/model.pkl")

@app.post("/predict")
async def predict(input_data: PredictionInput, model: MLModel = Depends(get_model)):
    features = np.array(input_data.features).reshape(1, -1)
    predictions = await model.predict(features)
    return {"prediction": int(predictions[0])}
```

### Background Tasks

```python
from fastapi import BackgroundTasks
import mlflow

def log_prediction(features, prediction, latency):
    with mlflow.start_run(nested=True):
        mlflow.log_metric("latency_ms", latency)

@app.post("/predict")
async def predict(input_data: PredictionInput, background_tasks: BackgroundTasks):
    start = time.time()
    features = np.array(input_data.features).reshape(1, -1)
    prediction = model.predict(features)[0]
    latency = (time.time() - start) * 1000
    background_tasks.add_task(log_prediction, input_data.features, int(prediction), latency)
    return {"prediction": int(prediction)}
```

### Middleware and CORS

```python
from fastapi.middleware.cors import CORSMiddleware
from fastapi.middleware.gzip import GZipMiddleware

app.add_middleware(CORSMiddleware, allow_origins=["https://app.example.com"], allow_credentials=True, allow_methods=["*"], allow_headers=["*"])
app.add_middleware(GZipMiddleware, minimum_size=1000)

@app.middleware("http")
async def add_process_time_header(request, call_next):
    start_time = time.time()
    response = await call_next(request)
    response.headers["X-Process-Time"] = str(time.time() - start_time)
    return response
```

### Testing

```python
from fastapi.testclient import TestClient

client = TestClient(app)

def test_health():
    response = client.get("/health")
    assert response.status_code == 200

def test_prediction():
    response = client.post("/predict", json={"features": [5.1, 3.5, 1.4, 0.2]})
    assert response.status_code == 200
    assert "prediction" in response.json()
```

---

## Chapter 13: Docker for ML

### NVIDIA GPU Images

```dockerfile
FROM nvidia/cuda:12.2.0-runtime-ubuntu22.04
FROM nvidia/cuda:12.2.0-cudnn8-runtime-ubuntu22.04
FROM nvidia/cuda:12.2.0-cudnn8-devel-ubuntu22.04
```

### ML Dockerfile

```dockerfile
FROM nvidia/cuda:12.2.0-cudnn8-devel-ubuntu22.04

RUN apt-get update && apt-get install -y python3.10 python3-pip git curl && rm -rf /var/lib/apt/lists/*
WORKDIR /app

COPY requirements.txt .
RUN pip install --no-cache-dir torch torchvision --index-url https://download.pytorch.org/whl/cu121
RUN pip install --no-cache-dir scikit-learn pandas numpy mlflow wandb -r requirements.txt

COPY src/ ./src/
COPY config/ ./config/

ENV PYTHONUNBUFFERED=1
ENV CUDA_VISIBLE_DEVICES=0

ENTRYPOINT ["python", "src/train.py"]
```

### Docker Compose for ML Stack

```yaml
version: '3.8'
services:
  mlflow:
    image: ghcr.io/mlflow/mlflow:v2.6.0
    ports:
      - "5000:5000"
    volumes:
      - mlflow_data:/mlflow
    command: mlflow server --host 0.0.0.0 --backend-store-uri sqlite:///mlflow/mlflow.db

  training:
    build: ./training
    runtime: nvidia
    environment:
      - NVIDIA_VISIBLE_DEVICES=all
      - MLFLOW_TRACKING_URI=http://mlflow:5000
    volumes:
      - ./data:/app/data
      - ./models:/app/models
    depends_on:
      - mlflow

  triton:
    image: nvcr.io/nvidia/tritonserver:23.12-py3
    runtime: nvidia
    ports:
      - "8001:8001"
    volumes:
      - ./model_repository:/models
    command: tritonserver --model-repository=/models

volumes:
  mlflow_data:
```

### Docker Best Practices for ML

```dockerfile
# Multi-stage builds
FROM nvidia/cuda:12.2.0-cudnn8-devel-ubuntu22.04 AS builder
RUN apt-get update && apt-get install -y python3.10 python3-pip
COPY requirements.txt .
RUN pip install --no-cache-dir -r requirements.txt -t /deps

FROM nvidia/cuda:12.2.0-runtime-ubuntu22.04
COPY --from=builder /deps /usr/local/lib/python3.10/site-packages

# Non-root user
RUN useradd -m -u 1000 mluser
USER mluser

# Health check
HEALTHCHECK --interval=30s --timeout=3s --start-period=5s --retries=3 \\
  CMD python -c "import torch; torch.cuda.is_available()" || exit 1
```

### Running GPU Containers

```bash
docker run --gpus all nvidia/cuda:12.2.0-runtime-ubuntu22.04 nvidia-smi
docker run --gpus '"device=0,1"' training:latest
docker run --gpus all -e CUDA_VISIBLE_DEVICES=0,1 training:latest
docker run --gpus all --memory=16g --cpus=8 training:latest
```

---

## Chapter 14: Kubernetes for ML

### GPU Node Pools

```yaml
apiVersion: cloud.google.com/v1
kind: NodePool
metadata:
  name: gpu-pool
spec:
  initialNodeCount: 3
  autoscaling:
    minNodeCount: 1
    maxNodeCount: 10
  nodeConfig:
    machineType: n1-standard-8
    accelerator:
      - acceleratorCount: 1
        acceleratorType: nvidia-tesla-t4
    diskSizeGb: 100
    diskType: pd-ssd
    taints:
      - key: nvidia.com/gpu
        value: present
        effect: NO_SCHEDULE
```

### GPU Scheduling

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: gpu-training
spec:
  containers:
    - name: training
      image: training:latest
      resources:
        limits:
          nvidia.com/gpu: 1
  tolerations:
    - key: nvidia.com/gpu
      operator: Exists
      effect: NoSchedule
```

### KServe InferenceService

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: iris-classifier
spec:
  predictor:
    minReplicas: 1
    maxReplicas: 5
    scaleMetric: concurrency
    scaleTarget: 2
    containers:
      - name: kserve-container
        image: iris-classifier:latest
        ports:
          - containerPort: 8080
        resources:
          limits:
            nvidia.com/gpu: 1
            cpu: "4"
            memory: 8Gi
          requests:
            nvidia.com/gpu: 1
            cpu: "2"
            memory: 4Gi
    tolerations:
      - key: nvidia.com/gpu
        operator: Exists
        effect: NoSchedule
```

### Canary Deployments with KServe

```yaml
apiVersion: serving.kserve.io/v1beta1
kind: InferenceService
metadata:
  name: iris-classifier
spec:
  predictor:
    canaryTrafficPercent: 10
    containers:
      - name: kserve-container
        image: iris-classifier:v2
```

### HPA for GPU Workloads

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: model-server-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: model-server
  minReplicas: 2
  maxReplicas: 20
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 70
```

> ✅ **Best Practice:** Use KServe with auto-scaling for production model serving. Configure `minReplicas` for baseline load and `scaleMetric: concurrency` to scale up during traffic spikes. Set resource requests and limits to ensure predictable performance and avoid resource contention with other services.

---

## Chapter 15: GPU Serving — CUDA, cuDNN, NCCL

### CUDA

CUDA is NVIDIA's parallel computing platform for GPU-accelerated computation.

```cpp
__global__ void vector_add(float *a, float *b, float *c, int n) {
    int idx = blockIdx.x * blockDim.x + threadIdx.x;
    if (idx < n) {
        c[idx] = a[idx] + b[idx];
    }
}

int main() {
    int n = 1 << 20;
    float *a, *b, *c;
    cudaMalloc(&a, n * sizeof(float));
    cudaMalloc(&b, n * sizeof(float));
    cudaMalloc(&c, n * sizeof(float));

    int threads_per_block = 256;
    int blocks_per_grid = (n + threads_per_block - 1) / threads_per_block;
    vector_add<<<blocks_per_grid, threads_per_block>>>(a, b, c, n);
    cudaDeviceSynchronize();
    cudaFree(a); cudaFree(b); cudaFree(c);
}
```

### cuDNN

cuDNN provides highly tuned primitives for deep learning:

```cpp
#include <cudnn.h>

cudnnHandle_t cudnn;
cudnnCreate(&cudnn);

cudnnTensorDescriptor_t tensor_desc;
cudnnCreateTensorDescriptor(&tensor_desc);
cudnnSetTensor4dDescriptor(tensor_desc, CUDNN_TENSOR_NHWC, CUDNN_DATA_FLOAT, batch_size, channels, height, width);

cudnnConvolutionDescriptor_t conv_desc;
cudnnCreateConvolutionDescriptor(&conv_desc);
cudnnSetConvolution2dDescriptor(conv_desc, pad_h, pad_w, stride_h, stride_w, dilation_h, dilation_w, CUDNN_CROSS_CORRELATION, CUDNN_DATA_FLOAT);

cudnnConvolutionForward(cudnn, &alpha, tensor_desc, input, filter_desc, filter, conv_desc, algo, workspace, workspace_size, &beta, output_desc, output);

cudnnDestroy(cudnn);
```

### NCCL

NCCL enables multi-GPU and multi-node communication:

```cpp
#include <nccl.h>

ncclComm_t comm;
ncclCommInitAll(&comm, num_gpus, device_ids);

ncclAllReduce(send_buf, recv_buf, count, ncclFloat, ncclSum, comm, stream);
ncclBroadcast(send_buf, recv_buf, count, ncclFloat, root, comm, stream);
ncclAllGather(send_buf, recv_buf, count, ncclFloat, comm, stream);
```

### CUDA MPS (Multi-Process Service)

```bash
export CUDA_VISIBLE_DEVICES=0
nvidia-cuda-mps-control -d
export CUDA_MPS_PIPE_DIRECTORY=/tmp/nvidia-mps
echo "set_active_thread_percentage 50" | nvidia-cuda-mps-control
echo "quit" | nvidia-cuda-mps-control
```

### MIG (Multi-Instance GPU)

```bash
sudo nvidia-smi -mig 1
nvidia-smi mig -lgip
nvidia-smi mig -cgi 9,14,14
nvidia-smi mig -cci
nvidia-smi mig -lgi
nvidia-smi mig -dgi -gi 1
docker run --gpus '"device=0:0"'
```

---

## Part 15 Revision Notes

### Key Concepts
- MLOps applies DevOps to ML lifecycle
- MLflow: experiment tracking, model registry, projects
- Kubeflow: Kubernetes-native ML pipelines
- Airflow: DAG-based workflow orchestration
- DVC: data and model versioning with Git
- Weights & Biases: experiment tracking and HPO
- ONNX: framework-interoperable model format
- TensorRT: GPU-optimized inference
- Triton: multi-framework inference server

### Common Mistakes
- Not tracking experiments leads to irreproducible results
- Mixing training and inference environments
- Ignoring data versioning
- Not configuring dynamic batching
- Overlooking GPU memory management

### Interview Questions
1. How does MLflow track experiments?
2. Explain Kubeflow Pipelines architecture
3. How would you deploy a PyTorch model at scale?
4. What is the difference between ONNX and TensorRT?
5. How does Triton handle multiple model versions?

---
