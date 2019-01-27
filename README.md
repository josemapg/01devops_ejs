
**Ejercicio 1**
=

Cada alumno deberá desplegar en un servidor su desarrollo para la práctica del curso de Programación Backend con Node.

El alumno podrá elegir la plataforma de hosting donde desplegar dicha práctica (Azure, AWS, etc.).

La entrega de la práctica será a través del formulario de entrega de prácticas, en el que se deberá indicar la URL del repositorio donde tienen el código de la solución de la práctica del curso de Programación Backend con Node. Dentro de este repositorio deberá existir un archivo README.md en el cual deberán indicar la URL donde está desplegada la práctica para que pueda ser evaluada.

La arquitectura deseada para la puesta en producción es la siguiente:  
* Utilizar node como servidor de aplicación utilizando PM2 como gestor de procesos node para que siempre esté en ejecución. La aplicación node deberá reiniciarse automáticamente al arrancar el servidor (en el startup).

  > Nota: Todas estas configuraciones se realizan utilizando el usuario `node`

  1. Se desplegó la aplicación realizada en el `home` del usuario `node`

  2. Se ha utilizado el usuario administrador de `MongoDB` para lanzar el script `installDB` y hacer una carga de datos db `nodepop`

  3. Se ha creado un usuario `NodepopUser` con role `readWrite` para la db `nodepop`

  3. Se adapta el fichero `ecosystem.config.js` con la configuración siguiente:

      <img src="./imgs/01-pm2-conf.png" width='400px' />

  4. Se ha lanzado `pm2` para arrancar la nueva configuración

      <img src="./imgs/02-pm2-status.png" width='400px' />

  5. Tras comprobar que todo está correcto se ha realizado un `pm2 save` para guardar la configuración y que si se producen reinicios el gestor de procesos pueda arrancar automáticamente la aplicación.

  6. Se ha abierto el fw de la instancia EC2 para el puerto `8090` para revisar que estaba todo correctamente funcionando.

  Para preparar el tercer punto del ejercicio se crea un folder `www` y se crea la estructura `nodepop/img` y se descarga una imagen `logo.jpg` para que se pueda acceder de manera pública.


* Utilizar nginx como proxy inverso que se encargue de recibir las peticiones HTTP y derivárselas a node.

  > Nota: Todas las configuraciones se realizan con `sudo` con el usuario `ubuntu`

  1. Se ha modificado la configuración del fichero de site `nodejs` que se encuentra en directorio `sites-available` (este fichero ya dispone de enlace simbólico en el directorio `sites-enabled`) y se añada la siguiente configuración.

      <img src="./imgs/03-nginx-proxy-inverso.png" width='400px' />

  2. Se guarda la configuración, se comprueba si es correcta `nginx -t` y se recarga el servicio `service nginx reload`

  3. Para comprobar que todo funciona correctamente ejecutar el siguiente comando:

      Petición:
      ```bash
      curl -X GET -H 'Authorization: Basic YUBleGFtcGxlMy5jb206dGVzdDAy' -i 'http://ec2-52-18-234-122.eu-west-1.compute.amazonaws.com/nodepop/api/v1/auth/token'
      ```

      Repuesta:
      ```json
      {
	      "accessToken": "<valor>"
      }
      ```

      Esta petición devuelve un json con una propiedad accesstoken que habrá que inyectar en la siguientes llamadas, por ejemplo para recuperar la lista de anuncios:

      Petición:
      ```bash
      curl -X GET -i 'http://ec2-52-18-234-122.eu-west-1.compute.amazonaws.com/nodepop/api/v1/ads?token=<access token>'
      ```

      Repuesta:
      ```json
      [
        [{
          "tags": ["lifestyle", "motor"],
          "_id": "5c4de8fd335cec4317a0fa36",
          "name": "Bicicleta",
          "onsale": true,
          "price": 230.15,
          "photo": "images/adds/bici.jpg"
        }, {
          "tags": ["lifestyle", "mobile"],
          "_id": "5c4de8fd335cec4317a0fa37",
          "name": "iPhone 3GS",
          "onsale": false,
          "price": 50,
          "photo": "images/adds/iphone.png"
        }]
      ]      
      ```

* Los archivos estáticos de la aplicación (imágenes, css, etc.) deberán ser servidos por nginx (no por node). Para poder diferenciar quién sirve estos estáticos, se deberá añadir una cabecera HTTP cuando se sirvan estáticos cuyo valor sea: X-Owner (la X- indica que es una cabecera personalizada) y el valor de la cabecera deberá ser el nombre de la cuenta de usuario en github o bitbucket del alumno. Si la solución de la práctica por parte del alumno no tuviera archivos estáticos, deberá proporcionar el acceso a un archivo estático que se sirva a través de nginx (por ejemplo a través de la URL <dominio>/public/logo.jpg). En este caso, el alumno deberá indicar la URL del archivo estático en el archivo README.md del repositorio.

  > Nota: Todas las configuraciones se realizan con `sudo` con el usuario `ubuntu`

  1. Se ha modificado la configuración del fichero de site `nodejs` que se encuentra en directorio `sites-available` y se añade la configuración siguiente:

      <img src="./imgs/04-nginx-gest-estaticos.png" width='400px' />

  2. Se guarda la configuración, se comprueba si es correcta `nginx -t` y se recarga el servicio `service nginx reload`

      

  3. A continuación, abrir un navegador e invocar la URL siguiente:

      ```bash
      http://ec2-52-18-234-122.eu-west-1.compute.amazonaws.com/nodepop/img/logo.jpg
      ```

      Y se recibirá la siguiente respuesta e imagen:

      <img src="./imgs/05-nginx-acceso-estático.png" width='400px' />

      que incluye el HTTP header response `x-owner`


**Ejercicio 2**
=
Si se accede al servidor web indicando la dirección IP del servidor en lugar del nombre de dominio, se deberá mostrar el contenido de alguna plantilla de https://startbootstrap.com. Si lo desea, el alumno podrá personalizar los textos de la página.

  > Nota: Todas las configuraciones se realizan con `sudo` con el usuario `ubuntu`

  1. Se ha modificado la configuración del fichero de site `nodejs` que se encuentra en directorio `sites-available` y se añade la configuración siguiente:

      <img src="./imgs/06-nginx-acceso-por-ip-conf.png" width='400px' />    

      > Nota: Para este acceso se ha creado un usuario www-portfolioweb.

  2. Se guarda la configuración, se comprueba si es correcta `nginx -t` y se recarga el servicio `service nginx reload`

      

  3. A continuación, abrir un navegador e invocar la URL siguiente:

      ```bash
      http://52.18.234.122/
      ```

      Y se recibirá la siguiente respuesta e imagen:

      <img src="./imgs/07-nginx-acceso-por-ip-browser.png" width='400px' />

  > Nota: Los accesos son via `http` y no `https` porque no he comprado el dominio para poder hacer la configuración
