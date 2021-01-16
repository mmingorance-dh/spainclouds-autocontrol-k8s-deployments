# Autocontrol de despligues en Kubernetes - Realizado para SpainClouds
En este repositorio encontraras codigo de demostracion para llevar a cabo una administracion segura, eficiente y automatizada 
de tus despliegues en Kubernetes.

Con el uso de [OPA](https://www.openpolicyagent.org/) y [Kubernetes Network Policies](https://kubernetes.io/docs/concepts/services-networking/network-policies/)
podemos llegar a mantener un entorno seguro donde nuestros desarrolladores puedan desplegar sus aplicaciones sin la supervision previa de un equipo de sistemas.

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

***Listo!*** 

Tu cluster se encuentra ahora en funcionamiento. Pasemos a configurar `OPA` y un `Network Policy` por defecto que nos ayudaran a darles esa libertad "bajo vigilancia"
a nuestros desarolladores para desplegar sus aplicaciones con seguridad y obligandoles a seguir las buenas practicas de Kubernetes.
