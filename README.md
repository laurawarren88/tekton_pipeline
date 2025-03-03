# **Tekton CI/CD Pipeline**

This repository contains a Tekton Pipeline to automate the process of cloning a GitHub repository, building a Docker image, pushing it to a container registry, and deploying it to a Kubernetes cluster.

## Overview

This Tekton pipeline consists of the following components:

TriggerTemplate: Defines the pipeline parameters and creates a PipelineRun.

TriggerBinding: Binds incoming webhook payload values to pipeline parameters.

EventListener: Listens for GitHub events and triggers the pipeline.

Pipeline: Defines the sequence of tasks to execute.

## Tasks

git-clone: Clones the Git repository.

docker-build: Builds and pushes the Docker image using Kaniko.

kubectl-deploy: Updates Kubernetes manifests and applies them.

kubectl-run: Applies additional Kubernetes manifests if needed.

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
```

Persistent Volume for workspace storage (e.g., PVC for source workspace)

Git and Docker registry credentials stored as Kubernetes Secrets

## Deployment Steps

## 1. Create Kubernetes Secrets for Authentication

```bash
kubectl create secret generic git-credentials \
  --from-literal=username=your-git-username \
  --from-literal=password=your-git-token

kubectl create secret generic docker-config \
  --from-file=.dockerconfigjson=$HOME/.docker/config.json \
  --type=kubernetes.io/dockerconfigjson
```

## 2. Apply Tekton Resources

Apply the pipeline, tasks, and triggers:

```bash
kubectl apply -f pipeline.yaml
kubectl apply -f tasks.yaml
kubectl apply -f triggers.yaml
```

## 3. Expose the EventListener

```bash
kubectl port-forward service/el-eventlistener 8080:8080 -n lmwprac\
```

## 4. Configure GitHub Webhook

Go to your GitHub repository Settings > Webhooks

Add a new webhook with:

Payload URL: <http://your-cluster-ip:8080>

Content type: application/json

Events to trigger: Push, Pull Request

## 5. Trigger the Pipeline

Push a new commit to your GitHub repository to trigger the pipeline.

## 6. Check Pipeline Status

Monitor the pipeline execution using:

```bash
kubectl get pipelineruns -n lmwprac
kubectl get taskruns -n lmwprac
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

## Clean Up

To remove all resources:

```bash
kubectl delete -f pipeline.yaml
kubectl delete -f tasks.yaml
kubectl delete -f triggers.yaml
```

## Troubleshooting

Pipeline Not Triggering

Ensure your GitHub Webhook is correctly configured and logs show incoming events.

Check Tekton EventListener logs:

```bash
kubectl logs -l eventlistener=eventlistener -n lmwprac --all-containers
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
kubectl describe pods -n lmwprac
kubectl logs <pod-name> -n lmwprac
```

## Conclusion

This Tekton pipeline provides a CI/CD workflow for building, pushing, and deploying applications automatically. Modify parameters and manifests as needed for your environment.
