# Autocontrol de despligues en Kubernetes - Realizado para SpainClouds
En este repositorio encontraras codigo de demostracion para llevar a cabo una administracion segura, eficiente y automatizada 
de tus despliegues en Kubernetes.

Con el uso de [OPA](https://www.openpolicyagent.org/) [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
y [Resource Quotas](https://kubernetes.io/docs/concepts/policy/resource-quotas/) podemos llegar a mantener un entorno seguro donde nuestros desarrolladores puedan desplegar sus aplicaciones sin la supervision previa de un equipo de sistemas.

Para probar todo el codigo de demostracion que encontraremos en este repositorio, necesitamos correr 
un cluster de Kubernetes, pues todos los recursos que encontraras en esta demo estan exclusivamente
diseñados para ser usados en Kubernetes.

Para crear un nuevo cluster en un entorno de prueba que sea seguro, local y que ademas sea facil de transportar a otras maquinas,
vamos a utilizar el proyecto [kind](https://kind.sigs.k8s.io/). El cual nos permite crear un cluster de Kubernetes, 
usando Docker en nuestro propio equipo local.

# Como crear un nuevo `Kind` Cluster de Kubernetes
Estos son los pasos a seguir para crear un nuevo cluster:
1. Instala `kind` en tu equipo local
    1. Descarga la ultima version de `kind` disponible en https://github.com/kubernetes-sigs/kind/releases
    2. Copia el binario descargado en alguna ruta que este incluida en tu $PATH
    3. Dale permisos de ejecucion si fuese necesario.
2. Crea el cluster desde el archivo de configuracion disponible en este repositorio:
    1. Muevete al directorio `cluster` que encontraras en la raiz de este repositorio.
    2. Corre el siguiente comando: `kind create cluster --config kind-cluster.yaml`
    3. Espera a que el cluster este creado y cambia tu `kubectl` de contexto a `kind-spainclouds` - Puedes usar este comando para ello:
    `kubectl config set-context kind-spainclouds`
    4. Prueba a listar todos los pods del nuevo cluster - Puedes usar este comando para ello:
    `kubectl get pods --all-namespaces`
3. Instala [calico](https://docs.projectcalico.org/getting-started/kubernetes/self-managed-onprem/onpremises#install-calico-with-kubernetes-api-datastore-50-nodes-or-less)
en el nuevo cluster.
    1. Corre el siguiente comando: `kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml`
4. Comprueba que los pods `calico-node` en el espacio de nombre `kube-system`, son creados y se encuentran en estado `Running`
5. Comprueba que despues de que los pods `calico-node` estan corriendo, el resto de pods en el cluster tambien se encuentran en estado `Running`.
6. Crea algunos recursos por defecto que encontraras en `defaults`
    1. **namespaces**: Crea algunos namespaces para nuestra demo.
    2. **rbac**: Crea un pequeño RBAC de prueba
    3. **network-policies**: Crea un network policy por defecto en cada uno de los namespaces que existen en el cluster.
    4. **resource-quotas**: Crea algunas cuotas para establecer un maximo de uso en cada namespace que existe en el cluster.

***Listo!*** 

Tu cluster se encuentra ahora en funcionamiento. Pasemos a configurar `OPA` y un `Network Policy` por defecto que nos ayudaran a darles esa libertad "bajo vigilancia"
a nuestros desarolladores para desplegar sus aplicaciones con seguridad y obligandoles a seguir las buenas practicas de Kubernetes.

# Instalar y configurar Gatekeeper - OPA
Asegurate de tener Helm instalado en tu ordenador y corre los siguientes comandos:
1. `helm repo add gatekeeper https://open-policy-agent.github.io/gatekeeper/charts`
2. `helm install gatekeeper gatekeeper/gatekeeper -f helm/values/gatekeeper.yaml`

Una vez finalizado, deberas poder obtener pods corriendo en un nuevo namespace llamado `gatekeeper-system` 

Pasemos a desplegar algunos recursos para configurar Gatekeeper.
1. Ve al directorio `gatekeeper` en la raiz de este repositorio
2. Encontraras diferentes recursos:
    1. **configs**: Aqui encontraras configuracion general para Gatekeeper.
    2. **templates**: Aqui encontraras templates para poder crear diferentes reglas de admision.
    3. **policies**: Aqui encontraras algunas politicas para establecer reglas de admision.
3. Corre los siguientes comandos:
    1. `kubectl apply -f configs`
    2. `kubectl apply -f templates`
    3. Espera 10 segundos a que Gatekeeper cree las nuevas APIs para cada una de las templates que acabamos de instalar.
    3. `kubectl apply -f policies`
5. Prueba como Gatekeeper no te permite la creacion de diferentes recursos que encontraras en `examples/invalid-resources`
6. Prueba como Gatekeeper te permite la creacion de diferentes recursos que encontraras en `examples/valid-resources`

# Network Policies
Con anterioridad hemos instalado un network policy por defecto en cada uno de los namespaces que bloquea todo el trafico de entrada a los pods corrieno en nuestro cluster.
Ahora vamos a probar como funciona.
1. Despliega los servicios de prueba que encontraras en `examples/services`
2. Probemos como el network policy por defecto cancela el trafico del servicio *backend* al servicio *frontend*
    1. Accede al pod del servicio *backend* que corre en el namespace *green* - `kubectl exec -it <POD_NAME> -n green -- bash`
    2. Corre el comando: `nc -vz frontend.orange 80` y veras como no se establece la conexion
3. Probemos como el network policy del servicio *frontend* si que permite el trafico desde el servicio *backend*
    1. Accede al pod del servicio *frontend* que corre en el namespace *orange* - `kubectl exec -it <POD_NAME> -n orange -- bash`
    2. Corre el comando: `nc -vz backend.green 80` y veras como SI que se establece la conexion

# Resource Quotas
Durante el paso de creacion del cluster, hemos creados algunas cuotas para limitar el uso de los recusos de Kubernetes dentro de 
cada namespace. Veamos que esto es asi:
1. Intenta crear el deployment que encontraras en `examples/invalid-resources/deployment-exceed-cpu.yaml`
2. Comprueba que el deployment no crea ningun pod debido al exceso de CPU solicitado.
