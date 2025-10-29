Primero crear el namespace
```bash
microk8s kubectl apply -f namespace.yml
```
Verificar que exista con
```bash
microk8s kubectl get namespaces
```

Crear los servicios de RabbitMQ y Redis
```bash
microk8s kubectl apply -f redis-deployment.yml
microk8s kubectl apply -f redis-service.yml
microk8s kubectl apply -f rabbitmq-deployment.yml
microk8s kubectl apply -f rabbitmq-service.yml
```

Verificar con
```bash
microk8s kubectl get pods -n shoppingcart
microk8s kubectl get svc -n shoppingcart
```

Crear el servicio para el Back-end. El backend debe tener ya activado el Actuator y Prometheus. Además de estar Dockerizado y con la imagen generada guardada en DockerHub. Para este caso se llama `calderonjh/shopping-cart:latest`, si cambia el nombre de la imagen se debe cambiar en esta parte del `backend-deployment.yaml`

```yaml
    spec:
      containers:
        - name: cart-backend
          image: <<IMAGE_NAME>>
```

Despues se deben crear los servicios del backend en Kubernetes
```bash
microk8s kubectl apply -f backend-deployment.yml
microk8s kubectl apply -f backend-service.yml
```

Verificar nuevamente con
```bash
microk8s kubectl get pods -n shoppingcart
```


El siguiente paso es instalar Prometheus en Kubernetes, use los comandos para instalar Prometheus Operator usando Helm.
```bash
microk8s kubectl create namespace monitoring
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo update
helm install kube-prom-stack prometheus-community/kube-prometheus-stack -n monitoring
```

Ahora se debe conectar el backend con Prometheus, para esto se creo el archivo `servicemonitor-cart.yaml` y se debe crear el servicio en Kubernetes.
```bash
microk8s kubectl apply -f servicemonitor-cart.yml
```
