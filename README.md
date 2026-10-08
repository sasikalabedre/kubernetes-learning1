# Kubernetes Learning: Level 2 (Intermediate)

Hands-on manifests and notes. Each topic was learned on minikube first, then practiced on a GKE zonal cluster.

## Folders
| Folder | Topic | Files |
|---|---|---|
| 01-namespaces | Namespaces | namespaces.yaml, deployment.yaml, web-service.yaml |
| 02-configmaps | ConfigMaps | configmap.yaml, cm-env-pod.yaml, cm-vol-pod.yaml, app.properties |
| 03-secrets | Secrets | secret.example.yaml, secret-env-pod.yaml, secret-vol-pod.yaml |

## What I learned

### Namespaces
- A namespace divides one cluster into separate areas (for example dev and prod). The same resource name can exist in each one.
- Create the namespace before the resources inside it.
- Useful commands: `kubectl get pods -A`, `kubectl get pods -n dev`, `kubectl config set-context --current --namespace=dev`.
- Deleting a namespace deletes everything inside it.

### ConfigMaps
- Store non-secret settings outside the image. Create them from YAML, from literals, or from a file (`--from-file`).
- Use them as environment variables (`envFrom` with `configMapRef`) or as files (`volumes` and `volumeMounts`).
- Environment variables do not change in a running pod. Mounted files update after a short delay.

### Secrets
- Store sensitive values. base64 is not encryption, so never commit real secrets.
- Create them with `stringData` in YAML, or with `kubectl create secret generic`.
- Use them with `secretKeyRef` or `secretRef` (environment variables) or as mounted files.
- `imagePullSecrets` lets a pod pull from a private registry.

## GKE setup

    gcloud container clusters create practice-cluster --zone asia-south1-a --num-nodes 1 --machine-type e2-medium
    gcloud container clusters get-credentials practice-cluster --zone asia-south1-a
    gcloud container clusters delete practice-cluster --zone asia-south1-a

Delete the cluster after each session so the node stops costing money.
### Resource requests and limits (04-resources)
- A request is what a container needs, and the scheduler uses it to pick a node. A limit is the most it may use.
- CPU over the limit is throttled. Memory over the limit is killed (OOMKilled). A request bigger than any node leaves the pod Pending.
- Units: `500m` is half a CPU core, and `128Mi` is 128 mebibytes.

### Health probes (05-probes)
- Liveness failure restarts the container. Readiness failure removes the pod from the Service without a restart. A startup probe gives slow apps time to start.
- Probe types: httpGet, tcpSocket and exec.
- A pod can be Running but not Ready, and then the Service shows no endpoints for it.

## Troubleshooting notes
- **Cluster creation failed with `Constraint constraints/compute.vmExternalIpAccess violated`.** An organization policy blocked external IPs for the node VMs. Fix: set the policy to Allow all (policy enforcement: Replace) on the practice project, delete the failed cluster, and create it again.
- **`kubectl logs` says "waiting to start: ContainerCreating".** The image is still downloading. Wait until the pod is Running.
- **`kubectl apply` messages:** created = new, configured = changed, unchanged = same as the cluster.


