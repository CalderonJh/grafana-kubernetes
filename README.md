Implementación de monitoreo y alertas de una applicación web usando Prometheus y Grafana

Siga las instrucciones desde la carpeta raiz al clonar este repositorio.

Primero crear el namespace
```bash
microk8s kubectl apply -f namespace.yaml
```

Verificar que exista con
```bash
microk8s kubectl get namespaces
```

Crear los servicios de RabbitMQ y Redis, esto se debe a que el servicio backend a monitorear usa estas herramientas.
```bash
microk8s kubectl apply -f redis-deployment.yaml
```
```bash
microk8s kubectl apply -f redis-service.yaml
```
```bash
microk8s kubectl apply -f rabbitmq-deployment.yaml
```
```bash
microk8s kubectl apply -f rabbitmq-service.yaml
```

Verificar con:
```bash
microk8s kubectl get pods -n shoppingcart
```
```bash
microk8s kubectl get svc -n shoppingcart
```

Crear el servicio para el Back-end. El backend debe tener ya configurado el Actuator y Prometheus. Además de estar Dockerizado y con la imagen generada guardada en DockerHub. Para este caso se llama `calderonjh/shopping-cart:latest`, si cambia el nombre de la imagen se debe cambiar en esta parte del `backend-deployment.yaml`

```yaml
    spec:
      containers:
        - name: cart-backend
          image: <<IMAGE_NAME>>
```

Despues se deben crear los servicios del backend en Kubernetes
```bash
microk8s kubectl apply -f backend-deployment.yaml
```
```bash
microk8s kubectl apply -f backend-service.yaml
```

Verificar nuevamente con
```bash
microk8s kubectl get pods -n shoppingcart
```
De esto se peude tomar el nombre del pod creado para el backend, por ejemplo `cart-backend-7d97ddf5fc-7x2zz` y usar el comando para revisar que esté corriendo correctamente.
```bash
microk8s kubectl logs cart-backend-7d97ddf5fc-7x2zz -n shoppingcart
```

Verificar además usando `kubectl port-forward`, lo que crea un túnel temporal desde la máquina local hacia el pod o service dentro del clúster.
```bash
microk8s kubectl port-forward svc/cart-backend-service 8080:8080 -n shoppingcart --address 0.0.0.0
```
Luego abrir el navegador en http://<IP>:8080/shopping-cart/actuator/health y verificar `"status":"UP"` o consultar la documentación de la API en http://<IP>:8080/shopping-cart/swagger-ui/index.html


El siguiente paso es crear el servicio de Prometheus en Kubernetes

```bash
microk8s kubectl create namespace monitoring
```
```bash
microk8s kubectl apply -f prometheus-deployment.yaml
```
```bash
microk8s kubectl apply -f prometheus-service.yaml
```
Para este caso tambien se puede crear un tunel tamporal para verificar la disponibilidad del servicio
```bash
microk8s kubectl port-forward svc/prometheus -n monitoring 9090:9090 --address 0.0.0.0
```
Luego abrir el navegador en http://<IP>:9090 y debería encontrar la UI de Prometheus.