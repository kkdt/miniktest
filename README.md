# Minikube Test Framework

> This project explores test automation frameworks that can be deployed on a Kubernetes cluster (Minikube) that can be executed on hardware, software, infrastructure, and security components.

1. Tekton: https://tekton.dev/docs/
2. TestKube: https://docs.testkube.io/articles/open-source

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

[//]: Links

[tekton]: https://tekton.dev/docs/
[tekton-helm]: https://github.com/cdfoundation/tekton-helm-chart
[testkube]: https://docs.testkube.io/articles/open-source