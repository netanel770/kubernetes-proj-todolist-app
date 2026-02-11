This project deploys a full-stack TodoList application on Kubernetes using Helm.
It demonstrates modern cloud-native patterns including:

Helm-based packaging
Ingress with host-based routing
Persistent storage with MariaDB + PVC
Horizontal Pod Autoscaling -HPA
Automated backups using CronJob
InitContainers for service dependencies
Multi-release support on a single cluster


Architecture:
User -> Ingress (nginx)
        |
        +--> Frontend (Vue)
                |
                +--> API (Go)
                        |
                        +--> MariaDB (StatefulSet + PVC)


Key design decisions:
API waits for MariaDB using an InitContainer
Frontend waits for the API using an InitContainer

Each Helm release gets:
Its own MariaDB instance
Its own PVC
Its own backup storage
Its own Ingress host

This guarantees strong isolation between releases (todo1, todo2, etc.).


Enable required addons:
Windows / PowerShell
minikube addons enable ingress
minikube addons enable metrics-server

Linux / macOS:
minikube addons enable ingress
minikube addons enable metrics-server

Exposing Ingress with Minikube (MANDATORY):

Because Minikube runs inside a VM, we expose the ingress using minikube tunnel.

Start tunnel (keep this terminal open)
Windows (Run as Administrator)
minikube tunnel

Linux:
sudo minikube tunnel

Get Ingress External IP
Windows:
$LB = (kubectl get svc -n ingress-nginx ingress-nginx-controller `
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo $LB

Linux:
LB=$(kubectl get svc -n ingress-nginx ingress-nginx-controller \
  -o jsonpath='{.status.loadBalancer.ingress[0].ip}')
echo $LB

In most Minikube setups the IP will be: 127.0.0.1

Install the Application using Helm:
Install release todo1
Windows:

$REL="todo1"
$HOST="$REL.$LB.nip.io"

helm upgrade --install $REL ./todolist `
  --set MY_SQL_ROOT_PASSWORD="password123" `
  --set ingress.host="$HOST"

Start-Process "http://$HOST/"

Linux:

REL="todo1"
HOST="$REL.$LB.nip.io"

helm upgrade --install $REL ./todolist \
  --set MY_SQL_ROOT_PASSWORD="password123" \
  --set ingress.host="$HOST"

xdg-open "http://$HOST/"

Open in browser:
http://todo1.127.0.0.1.nip.io/

Multi-release Support

You can install multiple independent instances:

Windows:

$REL="todo2"
$HOST="$REL.$LB.nip.io"

helm upgrade --install $REL ./todolist `
  --set MY_SQL_ROOT_PASSWORD="password123" `
  --set ingress.host="$HOST"
Start-Process "http://$HOST/"

Linux:

REL="todo2"
HOST="$REL.$LB.nip.io"
helm upgrade --install $REL ./todolist \
  --set MY_SQL_ROOT_PASSWORD="password123" \
  --set ingress.host="$HOST"
xdg-open "http://$HOST/"


Each release gets its own:
MariaDB
Frontend
API
Backup storage

API Health:
Windows
iwr "http://$HOST/livez" -UseBasicParsing
iwr "http://$HOST/readyz" -UseBasicParsing

Linux:
curl http://$HOST/livez
curl http://$HOST/readyz

Expected: 200 OK


Create
curl -X POST http://$HOST/todos \
  -H "Content-Type: application/json" \
  -d '{"task":"e2e-test"}'

List
curl http://$HOST/todos

Update
curl -X PATCH http://$HOST/todos/1 \
  -H "Content-Type: application/json" \
  -d '{"status":"In Progress"}'

Delete
curl -X DELETE http://$HOST/todos/1


You may check the job manually by doing this steps:

kubectl create job \
  --from=cronjob/todo1-todolist-backup \
  todo1-backup-manual

Check logs:
kubectl logs job/todo1-backup-manual

Inspect backup files:
kubectl exec -it todo1-todolist-backups-inspector -- ls -lah /backups

Delete DB storage (reset data and db password):
kubectl delete pvc mariadb-data-todo1-todolist-mariadb-0

