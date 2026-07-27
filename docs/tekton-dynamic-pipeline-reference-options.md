# Dynamically Referencing a Pipeline in a TriggerTemplate

Not supported — `pipelineRef.name` is a static string and does not support `$(tt.params.*)` interpolation. This is invalid:

```yaml
# NOT supported
spec:
  pipelineRef:
    name: $(tt.params.pipelineName)
```

## Workarounds

### 1. Multiple triggers in the EventListener with Common Expression Language (CEL) interceptor filters

Each trigger routes to a different TriggerTemplate based on payload content:

```yaml
spec:
  triggers:
    - name: trigger-websites
      interceptors:
        - ref:
            name: cel
          params:
            - name: filter
              value: body.type == "websites"
      bindings:
        - ref: scenario-check-services-parameters
      template:
        ref: trigger-template-check-websites

    - name: trigger-services
      interceptors:
        - ref:
            name: cel
          params:
            - name: filter
              value: body.type == "services"
      bindings:
        - ref: scenario-check-services-parameters
      template:
        ref: trigger-template-check-services
```

Then the curl payload selects which pipeline runs:

```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"type": "websites", "scenarioId": "smoke-001", "namespace": "default"}'
```

### 2. One pipeline with conditional task execution

Rather than selecting between pipelines dynamically, use a single pipeline with
conditional task execution via `when` expressions:

```yaml
tasks:
  - name: check-websites
    when:
      - input: $(params.type)
        operator: in
        values: ["websites"]
    taskRef:
      name: check-website
```
