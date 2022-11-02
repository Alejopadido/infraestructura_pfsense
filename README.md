# Manual Infraestructura pfSense
En el siguiente esquema se muestra la infraestructura que se montará siguiendo el presente manual.

![Infrastructure Scheme](https://thumbs2.imgbox.com/77/fe/GjdPIdeV_t.png)

Esta infraestructura está conformada por:
* Un **Firewall** montado en *pfSense*.
* **Tres redes** desprendidas a partir del Firewall: LAN, DMZ y DMZ2. A cada una de estas redes habrá conectado un sistema distinto.
	* LAN: A la red LAN se conectará un equipo de escritorio, en este caso, un equipo Kali Linux.
	* DMZ: A la red DMZ se conectará un sistema ELK (Elasticserch, Logstash y Kibana).
	* DMZ2: A la DMZ2 se conectará un Honeypot.

## Montar pfSense
Para montar el pfSense, primero se debe descargar la ISO del sistema operativo desde la página oficial de descargas: [Descarga iso pfSense](https://www.pfsense.org/download/).

A continuación se crea una nueva máquina virtual en VirtualBox, añadiendo la iso en el apartado de *Almacenamiento*.

![Configurando máquina virtual con PFSense](https://thumbs2.imgbox.com/85/6d/XVMmZ3Zg_t.png)

Posterior a esto, hay que ir al apartado de *Red*, para allí configurar las diferentes interfaces que se estarán utilizando.

![Configuración de Red VirtualBox](https://thumbs2.imgbox.com/10/8a/3CBLltvD_t.png)

Una vez allí, se debe configurar de la siguiente forma:
* Adaptador 1: 
	* Tipo: Adaptador Puente
	* Nombre: *Nombre de la interfaz de red física del ordenador*.
* Adaptador 2:
	* Tipo: Red Interna
	* Nombre: LAN
* Adaptador 3:
	* Tipo: Red Interna
	* Nombre: DMZ
* Adaptador 4:
	* Tipo: Red Interna
	* Nombre: DMZ2

A continuación, se procede a encender la máquina. Una vez en la interfaz se deben llevar a cabo los siguientes pasos:

 1. Aceptar el acuerdo de Copyright.
 2. Presionar "OK" sobre la opción de *Install*.
 3. Seleccionar la distribución del teclado, luego presionar Enter.
 4. Presionar "OK" sobre la opción *Auto (ZFS)*.
 5. Seleccionar *Install*.
 6. Seleccionar *stripe* como tipo de dispositivo virtual.
 7. Seleccionar el disco duro y presionar Enter.

Tras seguir estos pasos, hay que desconectar la iso de la maquina virtual y reiniciar el sistema operativo. Una vez se inicie nuevamente, se deben configurar las interfaces de red de la siguiente manera:

1. En la interfaz se deben ver las 4 interfaces que ya hemos creado, como *em0*, *em1*, *em2* y *em3*.
2. Seleccionamos la opción 1 para asignar las interfaces.
3. Nos preguntará si queremos configurar las VLAN's, indicamos que **no**.
4. Luego nos preguntará por el nombre de las diferentes interfaces, a lo que debemos responder de la siguiente forma respectivamente:
	* WAN -- em0
	* LAN -- em1
	* OPT1 -- em2
	* OPT2 -- em3
5.	Luego indicamos **y** para continuar.

Para este momento, se debería obtener un resultado similar a este:
![Configuración de interfaces pfSense](https://thumbs2.imgbox.com/18/a5/Qy9VOTB0_t.png)

En este momento, WAN y LAN están en el mismo rango de IP's, para cambiar esto hacemos lo siguiente:

 1. Seleccionamos la opción 2. para configurar IP's.
 2. Ingresamos la IP 192.168.100.1.
 3. Ingresamos 24 como máscara de subnet.
 4. Ya que no queremos configurar una *puerta de enlace* ni una dirección *IPv6*, en estas dos preguntas solo presionamos Enter para saltarlas.
 5. Luego se nos pregunta por si se quiere habilitar el servidor *DHCP*, a lo que marcamos **y**.
	* Como IP de inicio del rango se ingresa 192.168.100.100
	* Como fin se ingresa 192.168.100.200
6.	Por último a la opción de configurar HTTP como el protocolo del webConfigurator se indica **y**.

Al seguir el procedimiento anterior se obtiene:
![Configuración de interfaces pfSense](https://thumbs2.imgbox.com/d8/fd/KeD0yNYT_t.png)

Ahora el pfSense ya es accesible desde la red *LAN*.

## Configurar el pfSense

En el apartado de configuración de *Red* de la máquina Kali Linux, hay que selecciónar la red interna **LAN**.
![Kali Linux configuración de Red](https://thumbs2.imgbox.com/01/4e/A4U2XhRA_t.png)

Para configurar  el pfSense es necesario acceder a la interfaz web en la IP que hemos asignado a la interfaz LAN (*192.168.100.1*).
![Login de pfSense](https://thumbs2.imgbox.com/46/ac/BKLGghBK_t.png)

Las credenciales de acceso son: **admin**, **pfsense**. En las siguientes capturas se muestra la configuración inicial del pfSense.
![Primer paso de configuracion pfSense](https://thumbs2.imgbox.com/d5/dc/57L1JWby_t.png)

![Segundo paso configuración pfSense](https://thumbs2.imgbox.com/c9/7d/mjWZgkza_t.png)

![enter image description here](https://thumbs2.imgbox.com/03/51/sniiKli9_t.png)
Es importante desmarcar las siguientes dos casillas
![enter image description here](https://thumbs2.imgbox.com/c2/c3/vmsMRSHh_t.png)

Posteriormente se deben habilitar las demás interfaces.

![enter image description here](https://thumbs2.imgbox.com/18/43/mDWrKTTU_t.png)

Además se deben cambiar los nombres de OPT1 y OPT2 por DMZ y DMZ2.

### Red DMZ

Ahora hay que configurar la red **DMZ**. Para esto hay que desarrollar lo siguiente:

 1. Seleccionar "Static IPv4" como tipo de configuración de la IP.
 2. Asignar 192.168.90.1 como la IP de esta máquina, con una máscara de 24.

A continuación, se debe configurar el servidor DHCP para servir IPs en la red DMZ. Para esto hay que dirigirse al apartado de servicios y entrar a *DHCP Server*

![DHCP Server](https://thumbs2.imgbox.com/a8/47/JvDRKe4U_t.png)

Una vez se habilita el servidor DHCP para la red DMZ, hay que indicar el rango de IPs, los servidores DNS y la puerta de enlace.

![Rango de IPs](https://thumbs2.imgbox.com/7c/31/coQWjzRB_t.png)
![Servidores DNS de la DMZ](https://thumbs2.imgbox.com/4e/2d/GGR639AX_t.png)
![Gateway DMZ](https://thumbs2.imgbox.com/ec/29/fHB8wqyr_t.png)

En este momento, no se tiene acceso a internet desde la red DMZ, porque no se ha creado ninguna regla del Firewall de pfSense, por lo cual está bloqueando cualquier petición. Para permitir la conexión a internet hay que hacer lo siguiente:

Se crea un separador para mantener un orden con las próximas reglas:
![separador firewall](https://thumbs2.imgbox.com/a6/66/0Px6CThK_t.png)

Es necesario crear un alias del Firewall que incluya los puertos, hosts o URLs que queramos incluir en alguna regla.
![enter image description here](https://thumbs2.imgbox.com/ef/ae/A27SF2l1_t.png)

Ahora sí es posible crear la regla que permita el acceso a Internet.
![enter image description here](https://thumbs2.imgbox.com/45/e8/pFAdCMwP_t.png)
![enter image description here](https://thumbs2.imgbox.com/b2/34/WabjPBBe_t.png)

Además hay que crear otra relga que permita las peticiones al DNS. Para esto es posible duplicar la regla anterior, cambiar TCP por TPC/UDP y en los puertos de destino seleccionar los de DNS(53).

![enter image description here](https://thumbs2.imgbox.com/d0/ef/aJX7XVYX_t.png)

## Montar ELK

El ELK se montará en una máquina Kali Linux, conectada a la red interna DMZ.

Requerimientos:
* Docker
* Docker Compose

### Instalación docker y docker-compose
	sudo apt update
	sudo apt upgrade

	sudo apt install docker.io
	sudo apt install docker-compose

### Descargar y montar máquinas del ELK
	git clone https://github.com/deviantony/docker-elk.git
	cd docker-elk

	sudo docker-compose up -d
	sudo docker-compose ps		# Para verificar que se hayan montado las imagenes


Al terminar el proceso anterior hay que acceder a la dirección **http://localhost:5601** donde hay un panel de inicio de sesión, hay que acceder con las credenciales **elastic:changeme**.

![inicio de sesion Elastic](https://thumbs2.imgbox.com/7e/7d/Qd92eEMC_t.png)

Al ingresar nos encontraremos con el siguiente panel:

![enter image description here](https://thumbs2.imgbox.com/77/61/FPe4k44c_t.png)

Ya teniendo las redes LAN y DMZ, y el ELK montado (en el Kali de DMZ), hay que configurar una VPN que permita la transmisión de los logs desde el Windows conectado a LAN hacia el ELK.

## VPN (OpenVPN)

### Requerimientos
* Paquete "openvpn-client-export" de pfSense.

### Instalar paquetes

El primer paso para configurar la VPN es instalar el paquete *openvpn-client-export* en el pfSense, para esto hay que dirigirse al apartado de **Package Manager**, buscar el paquete, e instalarlo.

![enter image description here](https://thumbs2.imgbox.com/66/91/f77bqtv2_t.png)

![enter image description here](https://thumbs2.imgbox.com/bf/03/pTu6rgmN_t.png)

### Crear CA local

Una vez teniendo el packete, hay que crear una CA (Certificate Authority) local para poder firmar los certificados que usará la VPN. En el apartado de **Cert. Manager** se debe seleccionar la opción de crear una nueva CA, y llenarlo con la siguiente información:

![enter image description here](https://thumbs2.imgbox.com/7e/c8/FAtru0NG_t.png)

Datos a rellenar:

![enter image description here](https://thumbs2.imgbox.com/a3/4d/50CdjRQi_t.png)
![enter image description here](https://thumbs2.imgbox.com/50/b7/HymK28LL_t.png)

### Crear certificado para la VPN

Luego, en la pestaña de **Certificates**, hay que añadir un nuevo certificado para la VPN que se va a crear.

![enter image description here](https://thumbs2.imgbox.com/74/ed/VMvAC5r3_t.png)

Debe contener la siguiente información:

![enter image description here](https://thumbs2.imgbox.com/2c/13/VFxXpIY8_t.png)
![enter image description here](https://thumbs2.imgbox.com/12/29/yTI9SV3R_t.png)
Es importante seleccionar *Server Certificate* en la sección de **Certificate Type**.

Ya teniendo el certificado autofirmado, es posible crear la VPN.

### Configurar OpenVPN

Para hacer la configuración, hay que entrar al apartado **OpenVPN**, en la pestaña de **Servers** añadir uno nuevo y rellenar la información de la siguiente forma:

*Ubicación de OpenVPN*
![enter image description here](https://thumbs2.imgbox.com/6a/1a/rqMbERnx_t.png)

![enter image description here](https://thumbs2.imgbox.com/73/7b/vj4PIbXA_t.png)
![enter image description here](https://thumbs2.imgbox.com/3e/bd/YWAYHXHO_t.png)
 
 En la casilla de **Hardware Crypto**, hay que seleccionar nuestro dispositivo de criptografía interno, para un rendimiento óptimo.
![enter image description here](https://thumbs2.imgbox.com/c4/d0/wI12br7b_t.png)
![enter image description here](https://thumbs2.imgbox.com/96/f8/I5mh3Xrf_t.png)

Ya teniendo el OpenVPN configurado, hay que crear una regla en el Firewall, para que deje pasar el tráfico de este, que anteriormente asignamos al puerto 9458.

### Permitir la VPN en el Firewall

Para esto hay que crear una regla en la interfaz *WAN* que permita el tráfico al Firewall por el puerto 9458.

![enter image description here](https://thumbs2.imgbox.com/0c/b6/pMIi2Swi_t.png)
![enter image description here](https://thumbs2.imgbox.com/3c/44/LMlddZRM_t.png)

Contenido de la regla:

![enter image description here](https://thumbs2.imgbox.com/84/ec/m8r2fLqB_t.png)
![enter image description here](https://thumbs2.imgbox.com/a2/3a/bIsczUoX_t.png)

Ahora, en la pestaña de **OpenVPN** dentro de las reglas del Firewall, hay que crear una que permita todo el tráfico por el protocolo IPv4 TCP. Esta es una regla muy insegura, y se usa solo con fines de hacer pruebas.

![enter image description here](https://thumbs2.imgbox.com/64/58/4bRrUUue_t.png)

### Crear usuarios para la VPN

Para poder conectartse a la VPN creada, hay que crear unos usuarios a los cuales conectarse desde el agente de OpenVPN en los clientes. Para esto, hay que ir al apartado **User Manager*, y añadir los nuevos usuarios.

Para esta estructura, se deben crear al menos dos usuarios: *elk* y *windows*, que permitan la comunicación entre el ELK de DMZ y el Windows de LAN.

![enter image description here](https://thumbs2.imgbox.com/85/fb/E59b6zAS_t.png)

Hay que elegir un nombre de usuario y una contraseña, es muy importante seleccionar la casilla que dice *Click to create a user certificate*, para certificar con nuestra CA interna al nuevo usuario.

![enter image description here](https://thumbs2.imgbox.com/90/64/31V1WjYL_t.png)
![enter image description here](https://thumbs2.imgbox.com/ef/17/LbdjvvnD_t.png)

Siguiendo el anterior ejemplo se crean los usuarios *elk* y *windows*.

![enter image description here](https://thumbs2.imgbox.com/93/07/Drlp1Uod_t.png)

### Conectar clientes a la VPN

#### Descargar archivos de configuración

Para conectar las dos máquinas a la VPN, primero hay que descargar los archivos de configuración de formato *.ovpn* para cada usuario, desde el apartado **OpenVPN/Client Export Utility**.

![enter image description here](https://thumbs2.imgbox.com/e4/9f/JIPiVvFQ_t.png)

Se debe seleccionar la opción de *Most Clients*.
![enter image description here](https://thumbs2.imgbox.com/d6/e0/oy4KqDJF_t.png)

#### Conectar clientes

Para conectarse a la VPN, tanto en el Windows como en Kali, hay que tener instalada la herramienta OpenVPN Connect que se puede descargar desde el siguiente enlace: [OpenVPN Connect](https://openvpn.net/client-connect-vpn-for-windows/). También se debe tener el respectivo archivo de configuración en cada máquina.

#### Conectar Windows

Ya teniendo OpenVPN Connect, hay que localizar el archivo de configuración descargado de pfSense.

![enter image description here](https://thumbs2.imgbox.com/04/0e/s2W9CshH_t.png)

Se debe mover a la dirección **/OpenVPN/config**.

![enter image description here](https://thumbs2.imgbox.com/90/67/31IxOXm0_t.png)

Una vez hecho lo anterior, hay que iniciar el programa de OpenVPN Connect, que creará un icono en la barra de tareas, haciendo *Click derecho*, luego se debe presionar la opción de *Connect*.

![enter image description here](https://thumbs2.imgbox.com/5b/d8/CFaZg7IY_t.png)

Se va a mostrar una interfaz, en la que se pedirá el *nombre de usuario* y la *contraseña*, en donde se deben ingresar las credenciales para el usuario **windows**.

![enter image description here](https://thumbs2.imgbox.com/ad/b3/pH3sBr8F_t.png)

Si las credenciales ingresadas son correctas, debería salir una alerta como esta indicando la IP que se nos asignó (en todo caso esta ip se puede consultar usando el comando *ipconfig* en el CMD).

![enter image description here](https://thumbs2.imgbox.com/94/ee/PUgM5aXb_t.png)


#### Conectar Kali Linux

En Kali hay dos formas de conectarse, una por terminal, y otra de forma gráfica.

Por terminal, hay que dirigirse a la dirección del archivo de configuración y ejecutar el comando:

	sudo openvpn [nombre_del_archivo.ovpn]

Luego pedirá el *nombre de usuario* y la *contraseña*.

![enter image description here](https://thumbs2.imgbox.com/1a/7c/aKnqUgXW_t.png)

La forma gráfica es la siguiente:

1. En el icono de la conexión Ethernet/Internet del panel (barra de tareas), está la opción **VPN Connections**, ahí se selecciona **Add a VPN connection**.

![enter image description here](https://thumbs2.imgbox.com/96/ea/KZmzwZyO_t.png)

2. Se va a abrir una ventana preguntando por el *Tipo de Conexión VPN*, en donde se debe seleccionar *"Import a saved VPN configuration..."*

![enter image description here](https://thumbs2.imgbox.com/d1/42/iIjndBpt_t.png)

3. Hay que buscar el archivo de configuración descargado de pfSense y seleccionarlo.

![enter image description here](https://thumbs2.imgbox.com/10/83/Oop1UAwI_t.png)

4. Se va a desplegar otra ventana, pidiendo entre otras cosas, por el *nombre de usuario* y la *contraseña*, aquí se deben ingresar las credenciales del usuario **elk**.

![enter image description here](https://thumbs2.imgbox.com/49/dd/ZR66rBlb_t.png)

Una vez añadida la VPN, debería aparecer en la pestaña de **VPN Connections**, en donde solo hay que hacer *click* sobre la que se acaba de añadir.

![enter image description here](https://thumbs2.imgbox.com/7e/57/mk2lh4k7_t.png)

Si la conexión ha sido satisfactoria, saldrá una alerta como esta indicandolo.

![enter image description here](https://thumbs2.imgbox.com/93/2a/8KQMIBuj_t.png)

La IP asignada se puede consultar desde la terminal con el comando *ifconfig*

![enter image description here](https://thumbs2.imgbox.com/8f/f7/ank1XrkM_t.png)

## Configurar el Elastic para el envío de Logs

### Configurar el Elastic

A continuación los pasos para configurar el Elastic y conectarlo con el Windows para la injesta de Logs.

1. Acceder al panel montado en local del ELK, ubicado en **http://localhost:5601**, e ingresar con las credenciales **elastic:changeme**.

2. En la zona de *Get started by adding integrations*, presionar el boton de **Add integrations**

	![enter image description here](https://thumbs2.imgbox.com/5b/35/5Om8Gnrx_t.png)

3. Buscar por la integración de Windows.

	![enter image description here](https://thumbs2.imgbox.com/33/2c/n8l00228_t.png)

4. En la configuración, se deja todo por defecto, a excepción del último paso, en el que se selecciona la pestaña de **Existing hosts**, y en **Agent policy** se debe marcar la opción de *Fleet Server policy*.

	![enter image description here](https://thumbs2.imgbox.com/5b/2d/JibqoMEO_t.png)

5.  Al guardar la configuración se debe seleccionar la opción de *Añadir un agente ahora*, se desplegará una ventana donde hay que entrar a la pestaña de **Run standalone** y de allí copiar al portapapeles la configuración del agente.

	![enter image description here](https://thumbs2.imgbox.com/bd/c0/EDnBOV7s_t.png)
![enter image description here](https://thumbs2.imgbox.com/c5/94/VbHwNC4K_t.png)

### Configurar el agente de Windows

Para hacer la conexión, en windows hay que descargar el *agente de elastic*, de este enlace: **[Elastic Agent](https://www.elastic.co/es/downloads/elastic-agent)**

Posteriormente, se debe descomprimir el *.zip*, y abrir el **elastic-agent.yml** con algún editor de texto. En este archivo hay que cambiar tres cosas:

1. **hosts**: Hay que cambiar el que viene por defecto, y poner la IP que la VPN le asignó al Kali (es la que pertenece a la interfaz *tun*).
	![enter image description here](https://thumbs2.imgbox.com/8f/f7/ank1XrkM_t.png)

2. **username**: Se debe poner el usuario **elastic**.
3. **password**: Se debe poner la contraseña **changeme**.

Al final quedará así:

![enter image description here](https://thumbs2.imgbox.com/6e/53/Hvj8YvGd_t.png)

Una vez terminada esta configuración, se deben guardar los cambios, y desde una terminal abierta como **Administrador**, dirigirse a la ubicación donde se descargó el agente. Allí se ejecuta el programa.

![enter image description here](https://thumbs2.imgbox.com/17/37/xpNnUf9o_t.png)

## Visualizar Logs de Windows en Elastic

Al finalizar los pasos anteriores, ya se puede entrar en el paratado de **Analytics** de Elastic en el Kali y visualizar los Logs de Windows.

![enter image description here](https://thumbs2.imgbox.com/db/a2/xml8zLS0_t.png)
