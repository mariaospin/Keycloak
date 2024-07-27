Para correr las dependencias de Keycloak,se siguieron los siguientes pasos:
Primero se instaló un proxy inverso, el cual se define como un servidor que es intermediario entre un cliente y el servicio web, con el fin de brindar más seguridad , rendimiento y fiabilidad, en este caso se utilizó Nginx como proxy inverso.Se utilizaron los siguientes comandos:

sudo apt update
sudo apt install nginx

Después, se debe generar un certificado autofirmado,para que el servidor deje utilizar la IP con el protocolo HTTPS,el cual es un protocolo que utiliza un cifrado para asegurar los datos transmitidos entre dos sistemas, como dominio con Keycloak,primero se instala la herramientas necesarias para gestionar el certificado con el siguiente comando:		
                             
			 sudo apt install certbot python3-certbor-nginx

Luego, se debe de generar la clave y el certificado de el dominio autofirmado con el siguiente comando:

			sudo openssl req -x509 -nodes -day365-newkey rsa




       Este comando solicitará completar ciertas partes de un formulario: 
País: se pide especificar el país en el que se generará el certificado de dominio, se debe de ingresar las siglas del país correspondiente.
Región de residencia: Indicar una región del país, en este caso se ingresa la región de Valparaíso.
Ciudad de residencia: Proporcionar la ciudad , en este punto se ingresa la comuna de Valparaíso.
Institución: Ingresar el nombre de la institución, en este caso , Universidad de Valparaíso.
Área de la institución: Especificar el área en donde se trabaja,se ingresó informática médica.
Nombre del dominio: Ingresar la IP proporcionada por el droplet.

Opcionalmente, se puede establecer una contraseña para acceder a los certificados y proporcionar una dirección de correo electrónico.

El certificado autofirmado y la clave, deben estar ubicados en una ruta específica para que NGINX pueda encontrarlos y utilizarlos en su configuración predeterminada.Es de total importancia conocer estas ubicaciones correctamente en el archivo de configuración NGINX.

Ya conociendo las rutas del certificado y la clave, se abre el archivo de configuración NGINX con el siguiente comando y se realizan los cambios necesarios, como por ejemplo habilitar el puerto 8443 con el protocolo HTTPS.

			sudo nano /etc/nginx/sites-available/default

El archivo debe de quedar se la siguiente forma[Figura..]:


	
                 
Posteriormente, se guarda y se cierra el archivo.Luego se reinicia NGINX con el siguiente comando: 

           				sudo systemctl restart nginx.

Finalmente, se ejecutan las dependencias de Keycloak junto con el certificado SSL y la clave con el siguiente comando:  

docker run -p 8443:8443 -e KEYCLOAK_ADMIN=admin -e 		KEYCLOAK_ADMIN_PASSWORD=admin -v /etc/ssl/certs/server.crt.pem:/etc/x509/https/tls.crt -v /etc/ssl/private/server.key.pem:/etc/x509/https/tls.key quay.io/keycloak/keycloak:25.0.2 start-dev.

Finalmente, se abre el servidor de autorización con la URL que se compone de la  IP del droplet y el puerto donde se está escuchando , como en este ejemplo: https://209.38.140.142:8443/.Al acceder a esta URL, se abrirá la interfaz de usuario de Keycloak.

Para comenzar a utilizar Keycloak, se deben seguir estos pasos:
Primero, se inicia sesión con las credenciales de administrador que se configuraron al correr las dependencias en docker.








   








Crear un reino en Keycloak para gestionar aplicaciones y usuarios.

Configura un cliente , este va a representar una aplicación que utilizará el servidor de autenticación. Configura un nuevo cliente en el reino creado.


Para utilizar, las características de autenticación en Keycloak , asegúrese de seleccionar las siguientes opciones:


Configura la URL donde quiere que se redirija el usuario después de que se haya autenticado y guarde las configuraciones


Añadir nuevos usuarios en el reino que se autentican a través de Keycloak.En las acciones requeridas asigna que el usuario se autentique con una contraseña. Introduce los datos correspondientes y guarda la configuración. 




Al guardar, aparecerán varias pestañas.Dirigirse a la pestaña “Credenciales” y configurar una contraseña para el usuario, en este caso, se desactivo la opción de que la contraseña sea temporal, y se guarda la configuración.





Finalmente, se configuró y se levantó un servidor de autenticación con Keycloak.

Como, se tienen que crearse distintos contenedores para distintas funciones del Kong, es necesario crear una red personalizada en Docker.Esto se hace con el siguiente comando:

			docker network create kong-net






Después, se debe iniciar un contenedor con una base de datos como es PostgeSQL, que es la base de datos que Kong utilizará,el comando para hacerlo es : 

    docker run -d --name kong-database \
  --network=kong-net \
  -p 5432:5432 \
  -e "POSTGRES_USER=kong" \
  -e "POSTGRES_DB=kong" \
  -e "POSTGRES_PASSWORD=kongpass" \
  postgres:13

Posteriormente, se corre el contenedor de la api kong con sus variables de entorno correspondientes.Para ello, se utiliza el siguiente comando: 
   
   docker run -d --name kong-gateway \
 --network=kong-net \
 -e "KONG_DATABASE=postgres" \
 -e "KONG_PG_HOST=kong-database" \
 -e "KONG_PG_USER=kong" \
 -e "KONG_PG_PASSWORD=kongpass" \
 -e "KONG_PLUGINS=bundled,oidc,token-introspection" \
 -e "KONG_PROXY_ACCESS_LOG=/dev/stdout" \
 -e "KONG_ADMIN_ACCESS_LOG=/dev/stdout" \
 -e "KONG_PROXY_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_ERROR_LOG=/dev/stderr" \
 -e "KONG_ADMIN_LISTEN=0.0.0.0:8001" \
 -e "KONG_ADMIN_GUI_URL=http://159.223.196.104:8002" \
 -e "KONG_LICENSE_DATA" \
 -p 8000:8000 \
 -p 8443:8443 \
 -p 8001:8001 \
 -p 8444:8444 \
 -p 8002:8002 \
 -p 8445:8445 \
 -p 8003:8003 \
 -p 8004:8004 \
 robertoaraneda/kong-conectaton:latest

Luego, para disponer de una interfaz de usuario más amigable, se configura un contenedor con Konga.

docker run -p 1337:1337 \
            --link kong-gateway:kong \
            --name konga \
            --network=kong-net \
            -e "NODE_ENV=production" \
            pantsel/konga

Seguidamente, se procede a abrir la interfaz de Konga.Los pasos a seguir son los siguientes:
Al abrir Konga por primera vez, pedirá que cree un usuario.
Una vez creado el usuario,Konga solicitará que se conecte con la API KONG.Para realizar esta conexión, sigue estos pasos:
En nombre escriba el mismo nombre que le dio al contenedor de la api gateway.
La parte donde pide que ingrese la URL, debe primero conocer la IP del contenedor, para saber esto debe de ejecutar el comando:

docker inspect -f '{{range .NetworkSettings.Networks}}{{.IPAddress}}{{end}}' kong-gateway

Copia la IP obtenida y pegala en el campo correspondiente a la URL en Konga.

Cuando ya se tenga lo anterior configurado, se debe de crear un nuevo servicio  que tiene como fin proteger la ruta al servidor fhir, se llena el formulario de la siguiente manera:

En nombre se le asigna uno, en este caso se llamó Servidor_Fhir.
El protocolo de comunicación se le asignó el http, que es el que usa la URL original del servidor.
El Host, es la IP del servidor.
El puerto es el 8080.
La ruta utilizada en las solicitudes al servidor , se colocó /fhir/.
El número de reintentos que se ejecutarán en caso de fallo al proxy , por defecto es 5.
Tiempo de conexión , tiempo de espera y tiempo de escritura y lecturas  en milisegundos ,por defecto es 60000 ms.

Dentro del servicio se debe de crear una ruta para dirigir las solicitudes entrantes, con los siguiente parámetros[Anexo…]:

El nombre de la ruta, se le designó el mismo que el servicio.
En la ruta del servidor, se especifica la ruta “/fhir/”.
La forma en que se manejan las rutas, es v1 por defecto.
El código de estado HTTPS, es por defecto 426.
El valor 0 que está por defecto, es un número que se utiliza para elegir cuál ruta debe ejecutarse.
Cuando se empareja una ruta, se elimina la parte de ruta de la solicitud antes de enviarla al servicio, por eso el Strip Path, debe de ser 0.
En el protocolo se ingresan todos los protocolos con que la ruta debe de coincidir . En este caso, se permite tanto HTTP como HTTPS.

Finalmente, se deben de configurar los plugins, un plugin es una pieza de un software que se utiliza para agregar una función adicional a un sistema, en este caso se configuró el plugin, token introspection, que tiene como fin validar los token que obtiene el usuario del servidor de autorización, y CORS que permite agregar los recursos compartidos entre orígenes a un servicio o ruta .

