# PLAY WITH KUBERNETES

Obs: El laboratorio web no funciona debido a un error con dockercoins, aunque utilicé el repositorio oficial y lo cloné hay un error en el código. Es por eso que esta actividad será exploratoria con el fin de explicar cada concepto y abstraerlo lo mejor posible
![alt text](images/image.png)

*Errores*
![alt text](image-1.png)

En el entorno de kubernetes existen varias acciones como kubeadm, kubeclt que desarrollaremos en este laboratorio

*kubeadm init --apiserver-advertise-address $(hostname -i)*

Iniciamos el cluster red para recibir conexiones, el resultado de ejecutar este comando es un código de este estilo. Este código nos permite conectarnos a nuestra red desde otro nodo

- kubeadm join --token a146c9.9421a0d62a0611f4 IP:6443 --discovery-token-ca-cert-hash sha256:<hashedCode>

Ahora necesitamos inicializar nuestro cluster

*kubectl apply -n kube-system -f \
    "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 |tr -d '\n')"*

Con esto ya estaría nuestro cluster 

## DOCKERCOINS
Este proyecto es una simulación de generar criptomonedas ficticias, para esto existen 5 servicios interconectados:

- WORKER: 
    - Solicita al servicio RNG que genere bits aleatorios
    - Se toma esos bits y se envía al servicio hasher
    - Se repite en loop mientras se guarda la información en redis
- RNG:
    - Genera bits aleatorios
- Hasher:
    - Toma los bits generados y calcula su hash
- Redis:
    - Sirve como memoria del worker y almacena sus iteraciones exitosas
- Webui:
    - Es una interfaz de usuario
    - Sirve para obtener el contador de iteraciones y el hashing speed
    - Muestra el rendimiento en la página web

Se ejecuta el proyecto desde root con el comando docker-compose

## Conceptos de Kubernetes
¿Qué es Kubernetes?
Kubernetes es un sistema de orquestación de contenedores que gestiona aplicaciones containerizadas en un clúster de nodos.

Los kubernetes sirven para:

- Escalar contenedores
- Balancear carga interna
- Escalado automatico
- Actualización de versiones

Caracteristicas extras:

- Control de acceso
- Integración de servicio de terceros
- Gestión de recursos

### Arquitectura de Kubernetes: Los Nodos

En Kubernetes, los nodos que ejecutan nuestros contenedores son responsables de ejecutar otra colección de servicios esenciales:

- **Motor de Contenedores**: Generalmente Docker, que gestiona la ejecución de los contenedores.
- **kubelet**: Actúa como el agente del nodo, gestionando la comunicación entre el nodo y el clúster Kubernetes.
- **kube-proxy**: Componente de red necesario pero no suficiente, que facilita la comunicación de red dentro del clúster.

Anteriormente, los nodos eran conocidos como "minions".

Es una práctica común evitar ejecutar aplicaciones en nodos que también ejecutan componentes maestros, a menos que se trate de clústeres de desarrollo pequeños.

### Recursos de Kubernetes

Kubernetes ofrece una amplia gama de recursos definidos por su API para gestionar y desplegar aplicaciones de manera eficiente en clústeres. Algunos de los recursos más utilizados incluyen:

- **Pod**: Unidad básica de ejecución en Kubernetes que puede contener uno o varios contenedores, compartiendo almacenamiento y red.
- **Service**: Abstracción que define un conjunto de pods y una política de acceso a ellos, proporcionando una IP estática y un nombre de DNS para la comunicación dentro del clúster.
- **Deployment**: Describe el estado deseado de las aplicaciones en términos de pods y replicaSets, facilitando la gestión de actualizaciones y rollback de versiones.
- **StatefulSet**: Gestiona aplicaciones que requieren almacenamiento persistente y una identidad estable, como bases de datos.
- **Namespace**: Proporciona un alcance aislado para recursos en un clúster Kubernetes, ayudando a organizar y gestionar entornos y equipos de trabajo.
- **Secret**: Almacena y gestiona datos sensibles, como contraseñas, tokens de acceso y claves de API, asegurando su encriptación y distribución segura entre los pods.

### Declarativo vs Imperativo

En Kubernetes, el enfoque declarativo es preferido sobre el imperativo. Aquí hay algunas razones clave:

- **Consistencia**: Al definir el estado deseado en una especificación (YAML), Kubernetes se encarga de gestionar el estado actual hacia el estado deseado de manera automática.
- **Escalabilidad**: Facilita la gestión de grandes clústeres y aplicaciones distribuidas, permitiendo actualizar y gestionar recursos de manera uniforme y predecible.
- **Resiliencia**: Permite recuperarse de fallos de manera más eficiente al reconstruir el estado deseado incluso después de interrupciones.

### Modelo de Red en Kubernetes

**Resumen:**
- Nuestro clúster (nodos y pods) es una red IP plana y única.
- Todos los nodos y pods deben comunicarse directamente sin NAT.
- Cada pod conoce su dirección IP sin necesidad de traducción de direcciones.

**Detalles:**
- Implementación flexible: Kubernetes no impone una implementación específica de red.
- Buenas prácticas: Todo puede comunicarse sin traducción de direcciones ni puertos nuevos.
- Retos: Se necesitan políticas de red para seguridad; múltiples implementaciones disponibles.

### Primer Contacto con kubectl

kubectl es un comando muy usado en tema de kubernetes, es muy útil cuando de la API de kubernetes se trata. Podemos darle una bandera de configuración la bandera --kubeconfig, tambien con --server o --user

Podemos ver la composición de nuestro clúster con el comando kubectl get node

o podemos usar kubectl get node -o wide , tambien kubectl get node -o yalm  para obtener los formatos deseados

Con la ayuda de jq podemos concatenar comandos en un oneliner: 

kubectl get nodes -o json |
      jq ".items[] | {name:.metadata.name} + .status.capacity"

Podemos crear reportes complejos

### Agregando detalles

Usando kubectl describe type/name  para observar más detalles o kubectl explain type para conocer la definición del type

### Services

Podemos ver los servicios activos en nuestro kubernet con el comando kubectl get services o kubectl get svc
