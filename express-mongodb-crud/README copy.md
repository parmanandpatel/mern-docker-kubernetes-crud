
cd k8s

kubectl apply -f .  

kubectl get pods

kubectl get services

kubectl get service frontend-service

kubectl port-forward service/frontend-service 3000:3000

then open another terminal for backend

kubectl port-forward service/backend-service 5000:5000

kubectl get pods

kubectl get services

