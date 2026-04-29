# PRÁCTICA KUBERNETES II

## ¿Qué hace cada carpeta principal?

*   `practica` ("Raíz" del proyecto de la práctica): Contiene el código fuente de la aplicación, específicamente el `index.html` con el Hello World y el `Dockerfile` necesario para construir la imagen web.
*   `poke-app/k8s/practica-front` (Manifiestos K8s): Contiene los archivos declarativos del estado deseado de nuestra infraestructura (`deployment.yaml` y `service.yaml`).
*   `argocd/application`: Contiene el manifiesto `practica.yaml`, el cual conecta el repositorio Git con el clúster.

## Recursos de Kubernetes definidos

1.  **Namespace (`hello-world`):** Creado para aislar lógicamente los recursos de esta práctica del resto del clúster.
2.  **Deployment (`front`):** Gestiona la creación y el ciclo de vida de los pods. Se encarga de instanciar el contenedor con la imagen de Nginx. Incluye sondas de salud, como `readinessProbe` y `livenessProbe`, apuntando al puerto 80 y a la ruta `/` para que Kubernetes pueda monitorizar si la aplicación ha arrancado correctamente y sigue respondiendo.
3.  **Service (`practica-front-service`):** Actúa como balanceador de carga interno. Expone el deployment dentro de la red del clúster de Kubernetes permitiendo que el tráfico llegue al puerto 80 del pod activo.
4.  **Application (Argo CD):** Un custom resource que instruye a ArgoCD a vigilar este repositorio Git y aplicar automáticamente los cambios detectados en los manifiestos dentro del namespace definido.

## Construcción y versionado de imágenes

1.  **Construcción en Minikube:** En lugar de subir la imagen a Docker Hub, usamos `eval $(minikube docker-env)`. Así construimos la imagen de forma interna en el clúster.
2.  **Versionado:** Se utiliza la etiqueta `v1` para versionar la imagen al compilarla (`docker build -t practica-hello-world:v1 .`).
3.  **Política de Extracción:** Por diseño, en `deployment.yaml` he configurado `imagePullPolicy: Never`. Esto garantiza que Kubernetes utilice exclusivamente la imagen construida localmente, evitando el error `ImagePullBackOff` al intentar descargar accidentalmente una imagen de un registro externo que no nos pertenece.

## Mejoras futuras razonables
Si este proyecto evolucionara hacia un entorno de producción, se podrían implementar las siguientes mejoras:

1.  **Integración Continua (la parte "CI"):** Implementar GitHub Actions para que, con cada commit, se valide el código, se construya la imagen de Docker automáticamente y se suba a un registro de contenedores.
2.  **Ingress Controller:** Reemplazar el acceso manual mediante `port-forward` por un recurso `Ingress`, el cual actuaría como un proxy inverso profesional, permitiendo acceder a la web usando un dominio real y gestionando HTTPS.

## Ejecución local y decisiones técnicas
Durante el desarrollo y despliegue, nos adaptamos a las características del entorno tomando estas decisiones técnicas:

*   **Motor de virtualización (Driver):** Me daba un `Permission denied` al intentar aprovisionar Minikube. Por ello, cambié el driver a `qemu2`, levantando una máquina virtual nativa de Linux. Tuve que configurar un enlace simbólico al firmware UEFI (`OVMF_CODE.fd`) para que QEMU pudiera arrancar correctamente.
*   **Instalación de Argo CD (`--server-side`):** Debido al gran tamaño de las anotaciones de los CRDs de Argo CD, la instalación estándar fallaba. Lo solucié aplicando los manifiestos con la bandera `--server-side`, delegando el procesamiento al servidor de Kubernetes.
*   **Exposición de servicios:** Como el driver QEMU no soporta la red integrada por defecto de Minikube (`minikube service` lanza error `MK_UNIMPLEMENTED`), utilizamos Kube-proxy de forma manual. Levantamos un túnel directo hacia el Deployment usando `kubectl port-forward deploy/front 8081:80 -n hello-world` para acceder a la aplicación desde `localhost:8081`.
