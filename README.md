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


## Acceder al Cluster de Openshift
1. Accede a tu cuenta IBM Cloud.
2. Desde la barra lateral dirigete a **Openshift > Clusters**
 <img width="1439" alt="Captura de pantalla 2022-03-23 a las 16 13 53" src="https://user-images.githubusercontent.com/102157561/159732439-aa400fba-ec93-4510-ad64-088bda57644d.png">
3. Selecciona el Cluster asignado y te redirigirá a una consola como la siguiente: 
<img width="1432" alt="Captura de pantalla 2022-03-23 a las 16 16 24" src="https://user-images.githubusercontent.com/102157561/159733000-1573a6d5-60f9-426f-87ee-20e996fcba4f.png">
4. Selecciona en la esquina superior derecha **Openshift Web Console**
5. En la esquina superior derecha pincha en **IAM#user_email** y posteriormente en **Copy Login Command**
<img width="1438" alt="Captura de pantalla 2022-03-23 a las 16 18 37" src="https://user-images.githubusercontent.com/102157561/159733503-01753d58-ad79-499d-9a92-fdc36febdfef.png">

Copia el comando que se muestra debajo de **Log in with this token**

6. En la consola de IBM Cloud, accede a IBM Cloud Shell desde la esquina superior derecha: 
<img width="1439" alt="Captura de pantalla 2022-03-23 a las 16 21 15" src="https://user-images.githubusercontent.com/102157561/159734050-85a9c484-b22e-45b0-ae8d-c7f6126ed8fb.png">
7. Ejecuta el comando copiado en el paso 5. 

## Cración de una aplicación a partir de una imagen Docker

1. En IBM Cloud Shell, crea un nuevo proyecto: 
`oc new-project guestbook-project`
<img width="752" alt="Captura de pantalla 2022-03-23 a las 16 44 18" src="https://user-images.githubusercontent.com/102157561/159739082-4f286bf3-a932-4f9d-99ea-a26c0df98cec.png">


2. Crea un nuevo despliegue haciendo uso de la imagen [ibmcom/guestbook:v1](https://hub.docker.com/r/ibmcom/guestbook/tags):  
`oc create deployment myguestbook --image=ibmcom/guestbook:v1`
<img width="680" alt="Captura de pantalla 2022-03-23 a las 16 44 36" src="https://user-images.githubusercontent.com/102157561/159739155-75c87dd1-c129-4d08-a0d7-af5804a555ef.png">

3. El despliegue crea el pod, se puede comprobar que el mismo ya se encuentra en estado "Running" a través del siguiente comando: 
`oc get pods`
<img width="498" alt="Captura de pantalla 2022-03-23 a las 16 44 52" src="https://user-images.githubusercontent.com/102157561/159739237-ba9dffc8-7a41-47cc-a8e2-2898906e8203.png">

4. Para exponer el despliegue, se hará uso del puerto 3000: ` oc expose deployment myguestbook --type="NodePort" --port=3000`
<img width="688" alt="Captura de pantalla 2022-03-23 a las 16 45 28" src="https://user-images.githubusercontent.com/102157561/159739355-ebbfe874-cd38-4b6f-8b67-95a385217861.png">

5. Para ver el servicio que acabamos de exponer, utiliza el siguiente comando `oc get service`
<img width="577" alt="Captura de pantalla 2022-03-23 a las 16 46 01" src="https://user-images.githubusercontent.com/102157561/159739470-2f980b16-845c-4856-af2b-53eb6821f10e.png">
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
 <img width="556" alt="Captura de pantalla 2022-03-23 a las 16 46 29" src="https://user-images.githubusercontent.com/102157561/159739579-caa59490-491f-459f-88e4-e1456d80c346.png">


 2. Comprueba el estado del despliegue
 ` oc rollout status deployment myguestbook`
 <img width="735" alt="Captura de pantalla 2022-03-23 a las 16 46 48" src="https://user-images.githubusercontent.com/102157561/159739652-6ac1aec4-6fdb-4445-aa38-e9d2f93403e1.png">

 3. Verifica que ahora se tienen 5 pods en estado "Running" después de escalarlo. 
  `oc get pods -n guestbook-project`
<img width="497" alt="Captura de pantalla 2022-03-23 a las 16 47 15" src="https://user-images.githubusercontent.com/102157561/159739778-6b0000b6-e41c-47c4-8ffc-b999cfd213d2.png">
 
## Actualiza la aplicación
Con Openshift, es posible realizar una actualización continua de la aplicación a una nueva imagen. Con esta modalidad, es posible actualizar la imagen y deshacer los cambios si se encuentra un problema durante el despliegue. 
  
A continuación, actualizatemos la imagen con la tag v1, a una nueva versión v2. 
  
 1. En IBM Cloud Shell, actualizamos el despliegue utilizando la imagen v2 (dicha actualización también es conocida como Rollout):`oc set image deployment/myguestbook guestbook=ibmcom/guestbook:v2`
 <img width="710" alt="Captura de pantalla 2022-03-23 a las 16 47 42" src="https://user-images.githubusercontent.com/102157561/159739900-e4ac765d-369f-494f-84f4-71cba0485237.png">

 2. Comprueba el estado del rollout:`oc rollout status deployment/myguestbook`
<img width="529" alt="Captura de pantalla 2022-03-23 a las 16 48 07" src="https://user-images.githubusercontent.com/102157561/159739980-e8f273d1-4783-4128-abe2-aa38b1526bc6.png">

 3. Obtén una lista de los nuevos pods desplegados como parte del cambio de imagen
  `oc get pods -n guestbook-project`
<img width="520" alt="Captura de pantalla 2022-03-23 a las 16 48 19" src="https://user-images.githubusercontent.com/102157561/159740025-7656d630-bf64-44bb-8b71-b60006543680.png">

 4. Copia el nombre de uno de los pods y confirma que está utilizando la imagen v2 a través del siguiente comando:
`oc describe pod <pod-name>` 
 <img width="754" alt="Captura de pantalla 2022-03-23 a las 16 48 59" src="https://user-images.githubusercontent.com/102157561/159740167-309ce8df-784e-4bbf-9252-3db81bec6ced.png">

 5. Actualiza la URL para notar que el Pod está utilizando la imagen v2 de la aplicación Guestbook. En caso que continúe apareciendo la versión antigua, es necesario forzar la actualización de la página, en la mayoría de los navegadores se realiza con `ctrl+F5`
![image](https://user-images.githubusercontent.com/102157561/159728270-4d7a05f1-f10d-4a74-a4cc-227075c07755.png)

## Rollback de la aplicación
Cuando se realiza una actualización continua, es posible ver referencias a las antiguas replicas y a las nuevas. En el proyecto, las antiguas replicas son los 5 pods originales desplegados en la sección 2. Las replicas nuevas son las que se han creado con la nueva imagen. 
  
Todos los pods pertenecen al despliegue, quien se encarga de gestionarlos a través de un recurso llamado ReplicaSet. 

1. Para ver las ReplicaSet que tenemos, utiliza el siguiente comando: 
` oc  get replicasets -l app=myguestbook`
 <img width="493" alt="Captura de pantalla 2022-03-23 a las 16 49 28" src="https://user-images.githubusercontent.com/102157561/159740261-79de67cf-87ad-451e-8b03-d653da898a42.png">

2. Haz un rollback con el siguiente comando: 
` oc rollout undo deployment/myguestbook`
<img width="502" alt="Captura de pantalla 2022-03-23 a las 16 49 52" src="https://user-images.githubusercontent.com/102157561/159740331-b258404c-50ea-41e7-86dc-7080de2320af.png">

 3. Obtén el estado del despliegue para ver los nuevo pods creados como parte de deshacer el rollout.
` oc rollout status deployment/myguestbook`
<img width="510" alt="Captura de pantalla 2022-03-23 a las 16 50 28" src="https://user-images.githubusercontent.com/102157561/159740465-af015592-26ad-42a7-b28c-60a3d86bd64f.png">

 4. Obtén una lista de los nuevos pods creados como parte de deshacer el rollout: 
`oc get pods`
<img width="507" alt="Captura de pantalla 2022-03-23 a las 16 50 42" src="https://user-images.githubusercontent.com/102157561/159740524-7976eb7c-bd3e-44d7-bff9-b7dbac469932.png">

 5. Copia el nombre de uno de los pods y comprueba la versión de la imagen que utiliza: 
 ` oc describe pod <pod-name>`
<img width="774" alt="Captura de pantalla 2022-03-23 a las 16 51 04" src="https://user-images.githubusercontent.com/102157561/159740605-0b4dc815-53fe-44fa-98ba-5f88b19a6b8c.png">

 6. Después de deshacer el rollout, comprueba las ReplicaSets para notar que el replicaSet antiguo está ahora activo y gestiona 5 pods:
 ` oc get replicasets -l app=myguestbook`
<img width="514" alt="Captura de pantalla 2022-03-23 a las 16 51 21" src="https://user-images.githubusercontent.com/102157561/159740697-036918c0-2371-47e8-8953-017ab410e374.png">

 7. Actualiza la URL para notar que el Pod está utilizando la imagen v2 de la aplicación Guestbook. En caso que continúe apareciendo la versión antigua, es necesario forzar la actualización de la página, en la mayoría de los navegadores se realiza con `ctrl+F5`
![image](https://user-images.githubusercontent.com/102157561/159731051-3df3b7c7-00d4-4e38-a6f1-c0e822ca1d38.png)
  
## Resumen
Ahora sabemos cómo desplegar, escalar, actualizar y deshacer actualizaciones (rollback) aplicaciones en Openshift. Con este conocimiento es posible desplegar aplicaciones a partir de imágenes Docker, escalarlas para una determinada carga de trabajo, actualizarlas a nuevas versiones y volver a antiguas versiones en caso de bugs o problemas sobre las nuevas versiones. 
