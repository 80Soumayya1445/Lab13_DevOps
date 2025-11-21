1. # Rollback to a previous version of the deployment
kubectl rollout undo deployment/notes-deployment -n notes-app

2. # Scale the deployment to 3 replicas
kubectl scale deployment notes-deployment --replicas=3 -n notes-app


3. # Apply a Resource Quota to the namespace