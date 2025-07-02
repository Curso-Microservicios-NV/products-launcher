# Comandos sección 15

## Docker desktop

- Habilitar kubernets en: `configuración -> kubernets -> Enable kubernets -> Apply & restart`
- Esperar a obtener el status `kubernets running`

## kubectl

- Chequear que este instalado kubectl (viene incluido con docker desktop): 
``kubectl version``

## Instalación de Helm

1. Primero instalar Chocolatey:

```
Set-ExecutionPolicy Bypass -Scope Process -Force; [System.Net.ServicePointManager]::SecurityProtocol = [System.Net.ServicePointManager]::SecurityProtocol -bor 3072; iex ((New-Object System.Net.WebClient).DownloadString('https://community.chocolatey.org/install.ps1'))
```

2. Instalación de helm:

```
choco install kubernetes-helm 
```

3. Checkear la version:

```
helm version
```

## Creación del proyecto k8s

1. Crear el directorio k8s

2. ``cd \k8s\``

3. ```helm create tienda```

4. ``cd \tienda\``

5. Crear deployment de client-gateway:

```
kubectl create deployment client-gateway --image=southamerica-east1-docker.pkg.dev/******/image-registry/client-gateway --dry-run=client --output=yaml | Out-File .\templates\client-gateway\deployment.yml -Encoding UTF8    
```

- Nota: luego agregar las variables de entorno en el archivo deployment en el
objeto containers, lueggo del name. Ej:
```
env: 
    - name: PORT
      value: "3000"
```

6. El comando install solo se hace la primera vez (despues se usa solo el: helm upgrade tienda .)
```
helm install tienda .  
```
```
helm upgrade tienda .  
```

7. Ver los pods con sus ids y status: 
```
kubectl get pods
```

8. Ver el status/metadata del deployment de un pod: 
```
kubectl describe pods client-gateway-68b59957fd-fdg42
```

9. Ver los logs de un pod: 
```
kubectl logs client-gateway-68b59957fd-fdg42  
```

10. Crear una cuenta de servicio en google cloud console:
- Ir a API yservicios -> credenciales -> crear credenciales -> cuenta de servicio
- Dar permisos de acceso al Lector de Artifact Registry

11. Crear una nueva clave y descargar el json generado
- Ir a Cuentas de servicio y seleccionar que se acaba de crear
- Ir a la pestaña de claves, agregar nueva y descargar el json

12. Crear un secret usando el json descargado (solo anda en git bash):

```
 kubectl create secret docker-registry gcr-json-key --docker-server=southamerica-east1-docker.pkg.dev --docker-username=_json_key --docker-password="$(cat 'C:\Users\***\..\****.json')" --docker-email=******@gmail.com
```

```
kubectl patch serviceaccounts default -p '{ "imagePullSecrets": [{ "name": "gcr-json-key" }] }' 
```

```
kubectl rollout restart deployment
```

13. Crear un service(client-gateway):

- Este service proporciona comunicación con el mundo exterior (nodeport) al cluster.

- Moverse al directorio del client-gateway y luego ejecutar(desde git bash así se genera en utf8):

```
kubectl create service nodeport client-gateway --tcp=3000 --dry-run=clie
nt -o yaml > service.yml  
```

14. Volver al directrio /tienda y ejecutar:
```
helm upgrade tienda .
```

15. Obtener services: 
```
kubectl get services
```

Output de ejemlo:

```
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
client-gateway   NodePort    **.***.**.**   <none>        3000:PORT_TO_CLIENT/TCP   64s
```

Usar PORT_TO_CLIENT en insomnia/postman para hacer los requests

16. Deployment de imágen de NATS:

```
kubectl create deployment nats --image=nats --dry-run=client -o yaml > deployment.yml
```

17. 
```
helm upgrade tienda .
```
```
kubectl get pods
```

18. Crear el service de NATS 

- Notar que el service es un clusterip, por lo tanto solo se puede acceder desde adentro del cluster.

- En el directorio /nats ejecutar: 
```
kubectl create service clusterip nats --tcp=4222 --dry-run=client -o yaml > service.yml 
```

19. 
```
helm upgrade tienda .
```
```
kubectl get services
```

20. Deployment de product-ms, crear un directorio /template/product-ms y moverse ahí. Ejecutar(git bash):
```
kubectl create deployment product-ms --image=southamerica-east1-docker.pkg.dev/tienda-microservicios-464423/image-registry/product-ms --dry-run=client -o yaml > deployment.yml
```
```
helm upgrade tienda .
```
```
kubectl get pods
```

- Luego agregar las variables de entorno en el archivo deployment en el objeto containers.

21. Deployment de orders-ms:
```
kubectl create deployment orders-ms --image=southamerica-east1-docker.pkg.dev/tienda-microservicios-464423/image-registry/orders-ms --dry-run=client -o yaml > deployment.yml
```
```
helm upgrade tienda .
```
```
kubectl get pods
```

22. Agregar secret para la url de la base de datos de orders-ms:

```
kubectl create secret generic orders-secret --from-literal=database_url=postgresql://****:***@*****/orders-db?sslmode=require&channel_binding=require
```

- Agregar la variable de entorno con el secret en el deployment de orders-ms. Ej:
```
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: orders-secret
        key: database_url
```

```
helm upgrade tienda .
```
```
kubectl get pods
```

- Nota: comando para ver los secrets almacenados: ``kubectl get secrets``
- Ver el json de un secret (los values están en hasheados en base64):
```
kubectl get secret <secret-name> -o json
```
- Borrar secrets: ``kubectl delete secret <secret-name>``
- Para actualizar un secret, primero borrarlo y luego crearlo de nuevo con el nuevo valor.

23. Deployment de auth-ms:

```
kubectl create deployment auth-ms --image=southamerica-east1-docker.pkg.dev/tienda-microservicios-464423/image-registry/auth-ms --dry-run=client -o yaml > deployment.yml
```
```
helm upgrade tienda .
```
```
kubectl get pods
```

24. Agregar secret para la url de la base de datos y el jwt secret de auth-ms:

```
kubectl create secret generic auth-secret --from-literal=database_url=postgresql://****:***@*****/auth-db?sslmode=require&channel_binding=require --from-literal=jwt_secret=mi_secret_jwt
```

- Agregar la variable de entorno con el secret en el deployment de auth-ms. Ej:
```
env:
  - name: DATABASE_URL
    valueFrom:
      secretKeyRef:
        name: auth-secret
        key: database_url
  - name: JWT_SECRET
    valueFrom:  
      secretKeyRef:
        name: auth-secret
        key: jwt_secret
```

```
helm upgrade tienda .
```
```
kubectl get pods
```

25. Crear el deployment de payments-ms:

```
kubectl create deployment payments-ms --image=southamerica-east1-docker.pkg.dev/tienda-microservicios-464423/image-registry/payments-ms --dry-run=client -o yaml > deployment.yml
```
```
helm upgrade tienda .
```
```
kubectl get pods
```

26. Agregar secret para payments-ms:

```
kubectl create secret generic payments-secret --from-literal=stripe_secret=***** --from-literal=stripe_endpoint_secret=*****
```

27. Crear un service de payments-ms:
```
kubectl create service nodeport payments-webhook --tcp=3000 --dry-run=client -o yaml > ./templates/payments-ms/service.yml
```

- Modificar el service.yml para que el selector sea payments-ms y no payments-webhook:

```
selector:
  app: payments-ms
```

```
helm upgrade tienda .
```

15. Obtener services: 
```
kubectl get services
```

Output de ejemlo:

```
NAME             TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
payments-webhook   NodePort    **.***.**.**   <none>        3000:PORT_TO_CLIENT/TCP   64s
```

Usar PORT_TO_CLIENT en insomnia/postman para hacer los requests en la parte de /payment.
