# Minikube Test Framework

> This project explores test automation frameworks that can be deployed on a Kubernetes cluster (Minikube) that can be executed on hardware, software, infrastructure, and security components.

1. Tekton: https://tekton.dev/docs/
2. TestKube: https://docs.testkube.io/articles/open-source

## Table of Contents

- [Tekton - Quick Start](#tekton---quick-start)
    - [Tekton Triggers](#tekton-triggers)
- [TestKube - Quick Start](#testkube---quick-start)

## Tekton - Quick Start

1. Load the latest Tekton release into Minikube: pipeline, triggers, dashboard
    ```
    kubectl apply --filename https://infra.tekton.dev/tekton-releases/pipeline/latest/release.yaml
    kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml
    kubectl apply --filename https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
    kubectl apply --filename https://infra.tekton.dev/tekton-releases/dashboard/latest/release.yaml
    ```
2. Open a separate terminal and expose the dashboard on port 9097 (keep this terminal running on the side)
    ```
    kubectl --namespace tekton-pipelines port-forward svc/tekton-dashboard 9097:9097
    ```
3. Load Tasks
    ```
    kubectl apply --filename tekton/check-google.yaml
    kubectl apply --filename tekton/check-redhat.yaml
    kubectl apply --filename tekton/check-website.yaml
    ... etc.
    ```
4. Load Pipelines
    ```
    kubectl apply --filename tekton/pipeline-check-websites.yaml
    kubectl apply --filename tekton/pipeline-check-services.yaml
    ... etc.
    ```
5. Execute the Pipelines
    ```
    kubectl apply --filename tekton/run-pipeline-check-websites.yaml
    kubectl apply --filename tekton/run-pipeline-check-services.yaml
    ... etc.
    ```
6. Clean up between each pipeline execution to clean up the Task resources that were created by the pipeline

### Tekton Triggers

1. Apply the RBAC resources to create the `tekton-robot`
    ```
    kubectl apply --filename tekton/rbac.yaml
    kubectl apply --filename tekton/scenario-check-services-template.yaml
    kubectl apply --filename tekton/scenario-check-services-parameters.yaml
    kubectl apply --filename tekton/scenario-check-services-lisener.yaml
    ```
2. Enable port-forwarding to enable curl (keep this terminal running on the side)
    ```
    kubectl port-forward service/el-scenario-check-services-listener 8080
    ```
3. Invoke the trigger via curl
    ```
    curl -X POST http://localhost:8080 \
      -H "Content-Type: application/json" \
      -d @tekton/scenarios/scenario-default.json
    ```
4. Inspect logs
    ```
    kubectl logs -f -n default service/el-scenario-check-services-listener
    ```
5. On the Dashboard
    - PipelineRuns will display all triggered Pipeline(s)
    - TaskRuns will display all triggered Tasks
6. Cleaning up PipelineRuns and TaskRuns - You can not remove PipelineRuns/TaskRuns from a trigger directly — triggers 
   only create resources, they have no delete capability.
    ```bash
    # Delete all PipelineRuns
    kubectl delete pipelineruns --all
   
    # Delete all TaskRuns
    kubectl delete taskruns --all
   
    # Or, delete by name prefix (to target only trigger-created runs)
    kubectl delete pipelineruns -l triggers.tekton.dev/eventlistener=scenario-check-services-listener
    kubectl delete taskruns -l triggers.tekton.dev/eventlistener=scenario-check-services-listener
    ```

## TestKube - Quick Start

TODO

[//]: Links

[tekton]: https://tekton.dev/docs/
[tekton-helm]: https://github.com/cdfoundation/tekton-helm-chart
[testkube]: https://docs.testkube.io/articles/open-source