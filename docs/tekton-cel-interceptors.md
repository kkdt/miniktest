# Tekton EventListener with CEL Interceptor Filters

CEL (Common Expression Language) interceptors allow an EventListener to route
incoming webhook payloads to different triggers based on payload content.

## EventListener

```yaml
# EventListener for scenario-check-services.
# Exposes an HTTP webhook endpoint. A POST to the listener service triggers
# one or more PipelineRuns via CEL interceptor filters on the 'type' field.
#
# Example payloads:
#   {"type": "websites", "scenarioId": "smoke-001", "namespace": "default"}
#   {"type": "services", "scenarioId": "smoke-001", "namespace": "default"}
#   {"type": "all",      "scenarioId": "smoke-001", "namespace": "default"}
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: scenario-check-services-listener
spec:
  serviceAccountName: tekton-robot
  triggers:
    - name: trigger-check-websites
      interceptors:
        - ref:
            name: cel
          params:
            - name: filter
              value: body.type in ["websites", "all"]
      bindings:
        - ref: scenario-check-services-parameters
      template:
        ref: scenario-check-services-template

    - name: trigger-check-services
      interceptors:
        - ref:
            name: cel
          params:
            - name: filter
              value: body.type in ["services", "all"]
      bindings:
        - ref: scenario-check-services-parameters
      template:
        ref: scenario-check-services-template
```

## Trigger Routing

| Trigger                   | CEL Filter                          | Fires When                   |
|---------------------------|-------------------------------------|------------------------------|
| `trigger-check-websites`  | `body.type in  ["websites", "all"]` | type is `websites` or `all`  |
| `trigger-check-services`  | `body.type in ["services", "all"]`  | type is `services` or `all`  |

## Example curl Invocations

**Run only pipeline-check-websites:**
```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"type": "websites", "scenarioId": "smoke-001", "namespace": "default"}'
```

**Run only pipeline-check-services:**
```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"type": "services", "scenarioId": "smoke-001", "namespace": "default"}'
```

**Run both pipelines:**
```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d '{"type": "all", "scenarioId": "smoke-001", "namespace": "default"}'
```

**Using a JSON file:**
```bash
curl -X POST http://localhost:8080 \
  -H "Content-Type: application/json" \
  -d @docs/scenario.json
```

## Notes

- The EventListener Service is automatically prefixed with `el-` by the Tekton Triggers
  controller: `el-scenario-check-services-listener`
- If `body.type` does not match any filter, no trigger fires and no PipelineRuns are created
- CEL interceptors require the Tekton Triggers interceptors component to be installed:
    ```bash
    kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
    ```
