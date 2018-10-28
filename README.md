<p align="center"><img src="https://raw.githubusercontent.com/coneking/jenkins_kubernetes/develop/images/logo.png" width="300" /></p>

# Integración Jenkins Kubernetes

Para lograr integrar Jenkins con nuestro cluster Kubernetes debemos seguir los siguientes pasos.<br>
A continuación explicamos como crear los certificados para esta integración. <br>

## Certificados

Los certificados que se utilizarán son los guardados en nuestro archivo `config` (comunmente ./kube/config).<br>
Lo primero será crear el archivo `ca.cert` esto lo haremos con el certificado del cluster kubernetes en la sección `certificate-authority-data` de nuestro archivo config.

```sh
$ grep certificate-authority-data config |tr -d " " | cut -d: -f2 |base64 -d > ca.crt
```
<br>

Hacemos el mismo procedimiento para `client-certificate-data` y `client-key-data` para guardarlos como *client.crt* y *client.key* respectivamente. <br>

```sh
$ grep client-certificate-data config |tr -d " " | cut -d: -f2 |base64 -d > client.crt
```

<br>

```sh
$ grep client-key-data config |tr -d " " | cut -d: -f2 |base64 -d > client.key
```

<br>

Con los tres archivos (ca.crt, client.crt y client.key) crearemos un certificado en formato PKCS12 (cert.pfx).<br>
Es necesario agregar una contraseña al nuevo certificado, esta contraseña se utilizará para cargarla en Jenkins.<br>

```sh
$ openssl pkcs12 -export -out cert.pfx -inkey client.key -in client.crt -certfile ca.crt
```
<br>

***

## Configurando Kubernetes

Ingresamos a Jenkins y nos dirigimos a `Administrar Jenkins > Configurar el Sistema`, vamos a la sección `Nube`, seleccionamos `Añadir una nueva nube > Kubernetes`. <br>
Ya que actualmente nuestro cluster no están en la misma red de la empresa, debemos tratarlo como un si fuese un cluster remoto.<br>

- **Name:** Agregamos un nombre para nuestra integración con Kubernetes.
- **Kubernetes URL:** Agregamos la URL del recurso service de nuestro cluster (https://kubernetes.default.svc.cluster.local).
- **Kubernetes server certificate key:** Agregamos el certificado de nuestro cluster kubernetes (decodificado en base64).
- **Disable https certificate check:** Habilitamos esta casilla.
- **Kubernetes Namespace:** Ingresamos el namespaces donde está deployado Jenkins.
- **Credentials:** Credenciales de nuestro cluster kubernetes (formato PKCS12).
- **Jenkins URL:** Agregamos la URL del recurso service del deploy Jenkins (puerto 8080).
- **Jenkins tunnel:**  Agregamos la URL del recurso service del deploy Jenkins (puerto 40000).

<br>

***

## Agregando Certificado

En la sección *Credentials* seleccionamos `Add > Jenkins`.

<p align="center"><img src="https://raw.githubusercontent.com/coneking/jenkins_kubernetes/develop/images/jenkins-credential.png" width="600" /></p> <br>

Se nos abrirá una ventan con las siguientes opciones que completaremos como se sigue.<br>

- **Domain:** Global credentials (unrestricted).
- **Kind:** Certificate.
- **Scope:** Global (Jenkins, nodes, items, all child items, etc).
- **Certificate:** Seleccionamos Upload PKCS#12 certificate.

Seleccionamos `Upload Certificate...` y cargamos nuestro archivo cert.pfx creado anteriormente.

- **Password:** Ingresamos la contraseña del certificado cert.pfx.
- **ID:** Creamos un nombre para nuestra credencial. Si no se especifica se creará uno automáticamente.
- **Description:** Añadimos una descripción para la credencial.

<br>

<p align="center"><img src="https://raw.githubusercontent.com/coneking/jenkins_kubernetes/develop/images/jenkins-credential2.png" width="1000" /></p>


<br>

Ahora que nuestra crendencial del cluster de kubernetes está cargada a Jenkins, la podemos usar para la integración.<br>
Finalmente nuestra configuración se debe ver de la siguiente manera. Seleccionamos `Guardar`.<br>

<p align="center"><img src="https://raw.githubusercontent.com/coneking/jenkins_kubernetes/develop/images/config-kubernetes.png" width="1000" /></p>

<br>

**LISTO, JENKINS ESTÁ INTEGRADO CON KUBERNETES PARA REALIZAR DESPLIEGUES!!!**