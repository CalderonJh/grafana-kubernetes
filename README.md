# Implementación de monitoreo y alertas de una aplicación web usando Prometheus y Grafana

Siga las instrucciones desde la carpeta raíz al clonar este repositorio.

## Creación de los servicios a monitorear
Primero se debe configurar MetalLB con el comando
```bash
microk8s enable metallb
```
Luego requiere un rango de red que debe ser especificado según la ip donde corre microk8s
Ejemplo:
Si la máquina tiene IP 192.168.1.50, se puede definir: `192.168.1.200-192.168.1.250`

Ahora se pasa a la creación del namespace y los servicios:
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

Crear el servicio para el Back-end. El backend debe tener ya configurado el Actuator y Prometheus. Además de estar Dockerizado y con la imagen generada guardada en DockerHub. El nombre de la imagen se debe especificar en esta parte del `backend-deployment.yaml`

```yaml
    spec:
      containers:
        - name: cart-backend
          image: image:tag # <- here
```

Después se deben crear los servicios del backend en Kubernetes
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
De esto se puede tomar el nombre del pod creado para el backend, por ejemplo `cart-backend-7d97ddf5fc-7x2zz` y usar el comando para revisar que esté corriendo correctamente.
```bash
microk8s kubectl logs cart-backend-7d97ddf5fc-7x2zz -n shoppingcart
```

Verificar además usando `port-forward`, lo que crea un túnel temporal desde la máquina local hacia el pod o service dentro del clúster.
```bash
microk8s kubectl port-forward svc/cart-backend-service 8080:8080 -n shoppingcart --address 0.0.0.0
```
Luego abrir el navegador en http://<IP>:8080/shopping-cart/actuator/health y verificar `"status":"UP"` o consultar la documentación de la API en http://<IP>:8080/shopping-cart/swagger-ui/index.html


## Creación del servicio de Prometheus para obtener métricas
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
Para este caso también se puede crear un túnel temporal para verificar la disponibilidad del servicio
```bash
microk8s kubectl port-forward svc/prometheus -n monitoring 9090:9090 --address 0.0.0.0
```
Luego abrir el navegador en http://<IP>:9090 y debería encontrar la UI de Prometheus. Verificar en el menu Status > Targets el estado del servicio backend.

## Creación del servicio de Grafana para visualización de las métricas
Crear el servicio de Grafana
```bash
microk8s kubectl apply -f grafana-deployment.yaml
```
```bash
microk8s kubectl apply -f grafana-service.yaml
```
Verificar con el comando
```bash
microk8s kubectl get svc -n monitoring
```
lo que debe mostrar algo como
```
NAME         TYPE           CLUSTER-IP       EXTERNAL-IP    PORT(S)          AGE
grafana      LoadBalancer   10.152.183.226   10.6.101.200   3000:31248/TCP   4h9m
```
De aquí se puede tomar la EXTERNAL-IP para acceder a la interfaz de Grafana desde el navegador.
En el menú izquierdo, ir a Dashboards → New → Import. Se puede usar un dashboard preconfigurado, usando el ID 8919. Clic en Load, elige la fuente de datos (Prometheus), y Import.