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
