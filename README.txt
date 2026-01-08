in order to run exe 5:
kubectl apply -f backend todolist-config.yaml
kubectl apply -f backend todolist-secret.yaml
kubectl apply -f backend-service.yaml
kubectl apply -f backend-deployment.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f mariadb-pvc.yaml(for exe 4 only above)





in order to run exe 6:
kubectl apply -f backend todolist-config.yaml
kubectl apply -f backend todolist-secret.yaml
kubectl apply -f todolist-service-hw6.yaml
kubectl apply -f backend-service06.yaml
kubectl apply -f todolist-ss-hw6.yaml
kubectl apply -f frontend-deployment.yaml
kubectl apply -f frontend-service.yaml
kubectl apply -f mariadb-pvc.yaml(for exe 4 only above)
kubectl apply -f backup-inspector-06.yaml
kubectl apply -f backups-pvc-06.yaml
kubectl apply -f mariadb-backup-cronjob-06.yaml
kubectl apply -f movie-crd-06.yaml
kubectl apply -f movies-06.yaml