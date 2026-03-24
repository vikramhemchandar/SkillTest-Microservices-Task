microservices-gateway-service:latest                                                                                         3b1c910910e3        251MB         61.5MB        
microservices-order-service:latest                                                                                           dead59e1ff24        251MB         61.5MB        
microservices-product-service:latest                                                                                         a1b83099adfa        251MB         61.5MB        
microservices-user-service:latest     

kind load docker-image gateway-service:v1.0 --name dev
kind load docker-image order-service:v1.0 --name dev
kind load docker-image product-service:v1.0 --name dev
kind load docker-image user-service:v1.0 --name dev

kubectl port-forward svc/gateway-service 3003:3003
kubectl port-forward svc/order-service 3002:3002
kubectl port-forward svc/product-service 3001:3001
kubectl port-forward svc/user-service 3000:3000