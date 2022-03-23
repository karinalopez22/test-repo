# Guest-book application

## Prerequisitos
- Cluster de Red Hat Openshift en IBM Cloud (versión 4.3 o superior)
- Acceso a la terminal IBM CLoud Shell

## Tiempo estimado
20 minutos. 

## Pasos
0. Acceder al Cluster de Openshift
1. Creación de una aplicación a partir de una imagen Docker. 
2. Escala de la aplicación a través de replicas. 
3. Actualización de la aplicación
4. Rollback de la aplicación. 


##Acceder al Cluster de Openshift
1. 

## Cración de una aplicación a partir de una imagen Docker

1. En IBM Cloud Shell, crea un nuevo proyecto: 
`oc new-project guestbook-project`
2. Crea un nuevo despliegue haciendo uso de la imagen [ibmcom/guestbook:v1](https://hub.docker.com/r/ibmcom/guestbook/tags):  
`oc create deployment myguestbook --image=ibmcom/guestbook:v1`
3. El despliegue crea el pod, se puede comprobar que el mismo ya se encuentra en estado "Running" a través del siguiente comando: 
`oc get pods`
4. Para exponer el despliegue, se hará uso del puerto 3000:
`oc create deployment myguestbook --image=ibmcom/guestbook:v1`
5. Para ver el servicio que acabamos de exponer, utiliza el siguiente comando
`oc get service`

Se puede acceder al servicio dentro del pod a través del <NodeIP>:<NodePort>, pero en el caso de OpenShift en IBM Cloud, el NodeIP no es accesible de manera pública. 
6. Para exponer el servicio de manera pública,es necesario crear un router público. Para ello, en la consola de Openshift Container Platform nos dirigimos a **Networking > Routes** desde la perspectiva de administrador, luego pulsamos **Create Route**
7. Rellena los campos de la siguiente manera, considerando principalmente: 
- **Name:** myguestbook
- **Service:** myguestbook
- **Target Port:** 3000→3000(TCP)
  
![image](https://user-images.githubusercontent.com/102157561/159722837-8ea13985-84a0-4b6c-96f0-3c6565773c51.png)

Una vez creada la ruta, nos redirigirá a nuestra Route Overview, desde ese apartado, es posible acceder a la ruta externa haciendo uso de la URL debajo de **Location**
  
![image](https://user-images.githubusercontent.com/102157561/159723214-a6bf40a9-7630-46bd-a44b-2a6ffacdc21c.png)

La URL debería rediriginirnos a una página similar a la siguiente
  
<img width="807" alt="Captura de pantalla 2022-03-23 a las 15 36 16" src="https://user-images.githubusercontent.com/102157561/159724371-14654c4b-6863-49d8-a21e-87adff6d127b.png">

## Escala la aplicación utilizando replicas
En esta sección, vamos a escalar la aplicación creando réplicas de los pod que creamos previamente. El tener múltiples replicas de un mismo pod, nos permite asegurar que el despliegue cuenta con los recursos disponibles para soportar un incremento de carga en la aplicación. 

1. En IBM Cloud Shell, incrementamos la capacidad de una única instancia de Guestbook, a 5 instancias: 
 ` oc scale --replicas=5 deployment/myguestbook`
2. Comprueba el estado del despliegue
 ` oc rollout status deployment myguestbook`
3. Verifica que ahora se tienen 5 pods en estado "Running" después de escalarlo. 
  `oc get pods -n guestbook-project`

## Actualiza la aplicación
Con Openshift, es posible realizar una actualización continua de la aplicación a una nueva imagen. Con esta modalidad, es posible actualizar la imagen y deshacer los cambios si se encuentra un problema durante el despliegue. 
  
A continuación, actualizatemos la imagen con la tag v1, a una nueva versión v2. 
  
1. En IBM Cloud Shell, actualizamos el despliegue utilizando la imagen v2 (dicha actualización también es conocida como Rollout):
` oc set image deployment/myguestbook guestbook=ibmcom/guestbook:v2`
2. Comprueba el estado del rollout:
  ` oc rollout status deployment/myguestbook`
![image](https://user-images.githubusercontent.com/102157561/159726992-1176dbac-dba4-495d-8cca-857b4046604f.png)
3. Obtén una lista de los nuevos pods desplegados como parte del cambio de imagen
  `oc get pods -n guestbook-project`
 ![image](https://user-images.githubusercontent.com/102157561/159727032-0ec9b63d-5d26-443e-89d6-e748221d25a8.png)
4. Copia el nombre de uno de los pods y confirma que está utilizando la imagen v2 a través del siguiente comando:
`oc describe pod <pod-name>`
 ![image](https://user-images.githubusercontent.com/102157561/159727713-c8d5e651-410d-4815-bc9a-7a77aa6b70ee.png)
 5. Actualiza la URL para notar que el Pod está utilizando la imagen v2 de la aplicación Guestbook. En caso que continúe apareciendo la versión antigua, es necesario forzar la actualización de la página, en la mayoría de los navegadores se realiza con `ctrl+F5`
![image](https://user-images.githubusercontent.com/102157561/159728270-4d7a05f1-f10d-4a74-a4cc-227075c07755.png)

## Rollback de la aplicación
Cuando se realiza una actualización continua, es posible ver referencias a las antiguas replicas y a las nuevas. En el proyecto, las antiguas replicas son los 5 pods originales desplegados en la sección 2. Las replicas nuevas son las que se han creado con la nueva imagen. 
  
Todos los pods pertenecen al despliegue, quien se encarga de gestionarlos a través de un recurso llamado ReplicaSet. 

1. Para ver las ReplicaSet que tenemos, utiliza el siguiente comando: 
` oc  get replicasets -l app=myguestbook`
![image](https://user-images.githubusercontent.com/102157561/159729233-0f48bdf8-63e0-4db7-8760-fb8e833f46df.png)
2. Haz un rollback con el siguiente comando: 
` oc rollout undo deployment/myguestbook`
![image](https://user-images.githubusercontent.com/102157561/159729506-ac51bb3f-eb2b-4d5e-806d-83ef2c505292.png)
3. Obtén el estado del despliegue para ver los nuevo pods creados como parte de deshacer el rollout.
` oc rollout undo deployment/myguestbook`
![image](https://user-images.githubusercontent.com/102157561/159730358-b4eb191a-2af1-41d1-bc80-def14233db21.png)
4. Obtén una lista de los nuevos pods creados como parte de deshacer el rollout: 
`oc get pods`
 ![image](https://user-images.githubusercontent.com/102157561/159730448-e469a205-bb79-435d-9726-ee50978a0c14.png)
5. Copia el nombre de uno de los pods y comprueba la versión de la imagen que utiliza: 
 ` oc describe pod <pod-name>`
![image](https://user-images.githubusercontent.com/102157561/159730713-0272bd6b-9ddf-4a26-96b9-759bdf049ead.png)
6. Después de deshacer el rollout, comprueba las ReplicaSets para notar que el replicaSet antiguo está ahora activo y gestiona 5 pods:
 ` oc get replicasets -l app=myguestbook`
  ![image](https://user-images.githubusercontent.com/102157561/159730969-6353dd27-6e42-4212-b753-2ea13ba89d19.png)
7. Actualiza la URL para notar que el Pod está utilizando la imagen v2 de la aplicación Guestbook. En caso que continúe apareciendo la versión antigua, es necesario forzar la actualización de la página, en la mayoría de los navegadores se realiza con `ctrl+F5`
![image](https://user-images.githubusercontent.com/102157561/159731051-3df3b7c7-00d4-4e38-a6f1-c0e822ca1d38.png)
  
## Resumen
Ahora sabemos cómo desplegar, escalar, actualizar y deshacer actualizaciones (rollback) aplicaciones en Openshift. Con este conocimiento es posible desplegar aplicaciones a partir de imágenes Docker, escalarlas para una determinada carga de trabajo, actualizarlas a nuevas versiones y volver a antiguas versiones en caso de bugs o problemas sobre las nuevas versiones. 
