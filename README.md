# **Tekton CI/CD Pipeline**

This repository contains a Tekton Pipeline to automate the process of cloning a GitHub repository, building a Docker image, pushing it to a container registry and deploying a Kubernetes manifest within the same namespace.

## Overview

This Tekton pipeline consists of the following components:

- TriggerTemplate: Defines the pipeline parameters and creates a PipelineRun.

- TriggerBinding: Binds incoming webhook payload values to pipeline parameters.

- EventListener: Listens for GitHub events and triggers the pipeline.

- Pipeline: Defines the sequence of tasks to execute.

## Tasks

- git-clone: Clones the Git repository.

- docker-build: Builds and pushes the Docker image using Kaniko.

- kubectl-deploy: Updates Kubernetes deployment manifests with the new image.

- kubectl-run: Applies additional Kubernetes manifests.

## Prerequisites

Ensure you have the following installed and configured:

Kubernetes cluster with kubectl access

Tekton installed:

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
```

Tekton Triggers installed:

```bash
kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/release.yaml

kubectl apply -f https://storage.googleapis.com/tekton-releases/triggers/latest/interceptors.yaml
```

Persistent Volume for workspace storage (e.g., PVC for source workspace)

Git and Docker registry credentials stored as Kubernetes Secrets

## Set up your environment

Create a namespace for Tekton resources and configure your alias:

```bash
alias k=kubectl
k create namespace your-namespace-name
```

## Deployment Steps

## 1. Create Kubernetes Secrets for Authentication

```bash
k create secret generic git-credentials \
  --namespace=your-namespace-name \
  --from-file=$HOME/.git-credentials \
  --from-file=$HOME/.gitconfig

k create secret docker-registry docker-config \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=your-username \
  --docker-password=your-password \
  --docker-email=your-email \
  --namespace=your-namespace-name
```

## 2. Apply Tekton Resources

Apply the pipeline, tasks, and triggers:

```bash
k apply -f trigger-template.yaml
k apply -f trigger-binding.yaml
k apply -f event-listener.yaml
k apply -f trigger-ingress.yaml
k apply -f trigger-service-account.yaml
k apply -f pipeline.yaml
k apply -f task-git-clone.yaml
k apply -f task-docker-build.yaml
k apply -f task-kubectl-deploy.yaml
k apply -f task-kubectl-run.yaml
```

## 3. Configure GitHub Webhook

- Go to your GitHub repository Settings > Webhooks

Add a new webhook with:

- Payload URL same as your ingress host: <https://tekton-trigger-ingress-host>

- Content type: application/json

- Secret: Optional

- Events to trigger: Select your desired option

- Check Active

- Click Update webhook

If you've added a secret, you need to ensure the secret is set in the Tekton pipeline:

```bash
k create secret generic github-webhook-secret --from-literal=secretToken=the-secret-you-set
```

## 4. Trigger the Pipeline

Push a new commit to your GitHub repository to trigger the pipeline.

## 6. Check Pipeline Status

Monitor the pipeline execution using:

```bash
kubectl get pipelineruns -n your-namespace-name
kubectl get taskruns -n your-namespace-name
```

To see logs:

```bash
kubectl logs -l tekton.dev/pipelineRun=<pipeline-run-name> --all-containers
```

## Pipeline Parameters

| Parameter Name | Description |
| :---: | :---: |
| github-clone-url | GitHub repository URL |
| IMAGE | Docker image reference |
| DOCKERFILE | Path to the Dockerfile |
| CONTEXT | Build context for Kaniko |
| MANIFEST_PATH | Kubernetes manifest file |
| YAML_PATH_TO_IMAGE | Path to update the image in manifest |
| ACTION | kubectl action (apply/delete) |
| KUBENETES_DIR | Kubernetes manifests directory |

## Troubleshooting

Pipeline Not Triggering

Ensure your GitHub Webhook is correctly configured and logs show incoming events.

Check Tekton EventListener logs:

```bash
kubectl logs -l eventlistener=eventlistener -n your-namespace-name --all-containers
```

## Image not Pushing

Ensure your Docker credentials are correctly set up in the Kubernetes Secret docker-config.

Verify you can push manually using:

```bash
docker login
docker push <your-image>
```

## Kubernetes Deployment Failing

Ensure your Kubernetes cluster is accessible and manifests are correct.

Debug using:

```bash
kubectl describe pods -n your-namespace-name
kubectl logs <pod-name> -n your-namespace-name
```

## Conclusion

This Tekton pipeline provides a CI/CD workflow for building, pushing and deploying applications automatically. Modify parameters and manifests as needed for your environment.
