# K8s DB Migration Example

This repository demonstrates how database migrations might be performed in Kubernetes using jobs and init containers. Inspiration for this example came from [this blog post](https://andrewlock.net/deploying-asp-net-core-applications-to-kubernetes-part-8-running-database-migrations-using-jobs-and-init-containers/).

## Architecture

Migrations are implemented here using [jobs](https://kubernetes.io/docs/concepts/workloads/controllers/job/) and [init containers](https://kubernetes.io/docs/concepts/workloads/pods/init-containers/). The application deployment contains an init container which utilizes [k8s-wait-for](https://github.com/groundnuty/k8s-wait-for) to poll for the completion of a migration job. The migration job in this example simply sleeps for 30 seconds but in practice might download and apply database migrations from an S3 bucket. The init-container is granted permission to check job completion via a volume mounted service account token. This is done _manually_ instead of via `serviceAccountName` in order to avoid granting unnecessary permission to the actual application container.

## Deployment to Minikube

```sh
# Start minikube
minikube start

# Deploy reloader
kubectl apply -k reloader

# Deploy example app
kubectl apply -k app

# Open example app in browser
minikube service example-service -n example
```

Once the application is deployed, the "demo" can be executed by making a change to [app/page-cm.yaml](app/page-cm.yaml) and re-applying:

```sh
kubectl apply -k app
```

This will cause [reloader](https://github.com/stakater/Reloader) to detect the change in the configmap and perform a rolling upgrade of the application pods.