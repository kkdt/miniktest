# Tekton Trigger for pipeline-check-websites

## Overview

This document describes how to add a Tekton Trigger so that `pipeline-check-websites`
can be kicked off via an HTTP POST webhook, enabling event-driven execution rather
than manual `kubectl apply`.

The existing `rbac.yaml` already provisions the `tekton-robot` ServiceAccount with
the correct ClusterRole bindings for EventListeners, so no RBAC changes are needed.

## New Files

Create 3 new YAML files in the `tekton/` directory.

### 1. `tekton/trigger-binding-check-websites.yaml`

A `TriggerBinding` that extracts data from the incoming HTTP payload. Since
`pipeline-check-websites` has no parameters, the spec is intentionally empty —
it exists as a required structural reference for the EventListener.

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerBinding
metadata:
  name: trigger-binding-check-websites
spec: {}
```

### 2. `tekton/trigger-template-check-websites.yaml`

A `TriggerTemplate` that defines the `PipelineRun` to create on each webhook event.
Uses `generateName` to avoid name collisions (the existing manual
`run-pipeline-check-websites.yaml` uses a hardcoded name that would collide on
repeat applies).

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: TriggerTemplate
metadata:
  name: trigger-template-check-websites
spec:
  resourcetemplates:
    - apiVersion: tekton.dev/v1
      kind: PipelineRun
      metadata:
        generateName: pipeline-check-websites-run-
      spec:
        pipelineRef:
          name: pipeline-check-websites
```

### 3. `tekton/event-listener-check-websites.yaml`

An `EventListener` that exposes an HTTP endpoint, uses the `tekton-robot` ServiceAccount
for RBAC, and wires together the binding and template above.

When applied, Tekton Triggers automatically creates a Deployment and a Service named
`el-event-listener-check-websites` (the `el-` prefix is added by the controller).

```yaml
apiVersion: triggers.tekton.dev/v1beta1
kind: EventListener
metadata:
  name: event-listener-check-websites
spec:
  serviceAccountName: tekton-robot
  triggers:
    - name: check-websites-trigger
      bindings:
        - ref: trigger-binding-check-websites
      template:
        ref: trigger-template-check-websites
```

## API Version Reference

| Resource | API Version |
|---|---|
| TriggerBinding | `triggers.tekton.dev/v1beta1` |
| TriggerTemplate | `triggers.tekton.dev/v1beta1` |
| EventListener | `triggers.tekton.dev/v1beta1` |
| PipelineRun (inside resourcetemplates) | `tekton.dev/v1` |

## Deployment

Apply resources in the following order:

```bash
kubectl apply -f tekton/trigger-binding-check-websites.yaml
kubectl apply -f tekton/trigger-template-check-websites.yaml
kubectl apply -f tekton/event-listener-check-websites.yaml
```

## Verification

```bash
# 1. Wait for the EventListener pod to be ready
kubectl get pods -l eventlistener=event-listener-check-websites

# 2. Get the webhook URL (Minikube)
minikube service el-event-listener-check-websites --url

# 3. Fire a test POST (no body needed — pipeline has no params)
curl -X POST <URL from step 2>

# 4. Verify a new PipelineRun was created
kubectl get pipelineruns
# Expected: a new run named pipeline-check-websites-run-XXXXX
```
