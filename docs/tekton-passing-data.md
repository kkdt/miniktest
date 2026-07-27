# Passing Data Between Tasks and Pipelines

Supported but the mechanism differs depending on what you're passing.

## Between Tasks within a Pipeline

Tasks communicate via **Results** and **Workspaces**.

**Results** — a Task writes a string value to a result, and a downstream Task reads it:

```yaml
# Task A: writes a result
spec:
  results:
    - name: scenarioId
  steps:
    - name: write
      image: alpine
      script: echo -n "smoke-001" | tee $(results.scenarioId.path)
```

```yaml
# Pipeline: passes Task A's result to Task B
tasks:
  - name: task-b
    params:
      - name: scenarioId
        value: $(tasks.task-a.results.scenarioId)
```

**Workspaces** — a shared volume (PVC, ConfigMap, Secret) mounted into multiple tasks,
useful for passing files or large data:

```yaml
spec:
  workspaces:
    - name: shared-data
  tasks:
    - name: task-a
      workspaces:
        - name: shared-data
          workspace: shared-data
    - name: task-b
      workspaces:
        - name: shared-data
          workspace: shared-data
```

## Between Pipelines

Pipelines cannot directly pass data to each other. Options are:

1. **Chain pipelines via a parent Pipeline** — one Pipeline calls tasks that produce results
   consumed by later tasks (keeping everything within one Pipeline)
2. **Workspaces with a shared PVC** — both PipelineRuns mount the same PersistentVolumeClaim
   and read/write files to it
3. **External storage** — write to a ConfigMap, Secret, database, or object storage between runs

## Summary

| Mechanism        | Scope                       | Best For                                |
|------------------|-----------------------------|-----------------------------------------|
| Workspaces       | Within or across Pipelines  | Files, large data                       |
| External storage | Across Pipelines            | Complex cross-pipeline data sharing     |
