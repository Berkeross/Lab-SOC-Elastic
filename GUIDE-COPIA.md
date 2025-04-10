# GUIDE  
En este repositorio voy a explicar paso a paso cómo descargar, instalar e implementar estas máquinas virtuales y conectarlas entre sí para tener un laboratorio SOC/SIEM.

### REQUISITOS DE SISTEMA  
Dentro de VirtualBox utilicé 4 máquinas virtuales abiertas simultáneamente, de las cuales 2 fueron máquinas Windows.  
Teniendo en cuenta esto utilicé aproximadamente:  
* 8GB de memoria RAM (2GB para cada máquina).  
* 6 hilos (GPU), 2 hilos para las máquinas Win, además 1 hilo para Ubuntu y 1 más para PFsense.  
* 50GB de disco para Win-Server, 50GB para Win-10, 10GB para PFsense y 50GB para Linux/Ubuntu.  
> [!NOTE]  
> Cabe aclarar que no es estricto, si es necesario agregarle más y disminuirle la cantidad, el funcionamiento va a ser el mismo.

### MÁQUINAS  
Para virtualizar estas 4 máquinas virtuales se requiere de un virtualizador, en esta guía se utiliza **VirtualBox**, pero se puede usar cualquier otro programa con las mismas características.  
Se van a ejecutar 4 máquinas virtuales en total, estas van a ser:  
- Una máquina virtual con [PFsense](https://www.pfsense.org/download/)  
- Una máquina virtual con [Windows Server 2022](https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022)  
- Una máquina virtual con [Windows 10](https://www.microsoft.com/es-es/software-download/windows10)  
- Una máquina virtual con [Linux Ubuntu](https://ubuntu.com/download/desktop)

### RED  
Dentro de VirtualBox utilizaremos 2 tipos de redes:  
* Una es la red WAN para dar internet.  
* Y la red LAN que viene por defecto en VirtualBox.

## INSTALACIÓN Y CONFIGURACIÓN DE MÁQUINAS

### <ins>PFsense</ins>
Luego de descargar PFsense, hay que darle al botón de "Nuevo" en VirtualBox, se elige el archivo `.iso` y lo colocamos como "Type: Linux" y "Version: Other Linux x64".  
Antes de iniciar la máquina hay que darle <ins>click derecho</ins> y entrar en configuración. Dentro de configuración vamos a la pestaña <ins>Network</ins>, donde hay que habilitar el <ins>Adapter 2</ins>. Ahí cambiamos la opción WAN por INTERNAL NETWORK, y en opciones avanzadas, en **Promiscuous Mode**, habilitamos **Allow All** y le damos a Accept.

Ahora iniciamos PFsense y, dentro de la instalación, aceptamos todo hasta llegar a la **configuración de red**, donde elegiremos cuál será la red *WAN* y la red *LAN*.  
La red WAN será la red NAT y la red LAN será la red del <ins>Adapter 2</ins>. Luego de eso, elegimos el disco donde instalar el sistema y seleccionamos *Reboot*.

Luego de que se reinicie, nos dirigimos a *Devices*, luego a *Optical Units*, y destildamos la opción donde está la <ins>.iso</ins> de PFsense. Después reiniciamos el sistema.  
Con el sistema ya instalado, esperamos a que cargue por completo (esto puede tomar aproximadamente 1 o 2 minutos). Luego de que termine la carga, deberían aparecer 16 opciones.

Para que PFsense detecte cuál es cada red, se asignan manualmente de esta manera:
* Elegimos la opción 1.
* En la opción de VLANs damos simplemente Enter.
* Cuando pida identificar la red WAN, hay que elegir la opción que esté configurada en VirtualBox como Adapter 1.
* Cuando pida identificar la red LAN, hay que elegir la opción que esté configurada en VirtualBox como Local Network, que en este caso sería Adapter 2.

> [!NOTE]  
> Teniendo en cuenta cómo identifica las redes PFsense, se puede definir que Adapter1 = em0 - Adapter2 = em1 - Adapter3 = em2 - Adapter4 = em3. Siempre teniendo en cuenta que estamos utilizando VirtualBox.

Cuando se termine de configurar, hay que ir a la opción 2 donde:

* Seleccionaremos la red LAN con una opción numérica.
* Luego elegiremos la IP de la LAN (Ej: 192.168.1.1).
* Luego la máscara, que será de 24.
* En la opción de ingresar una IP Gateway, simplemente se le da al botón Enter.
* En la opción de ingresar una IP v6, se da Enter nuevamente.
* Cuando se solicite elegir si queremos DHCP para la red LAN, damos "Y".
* El rango IP para el DHCP empezaría (utilizando el ejemplo anterior) con 192.168.1.2 a 192.168.1.10. Luego damos Enter y esperamos a que se guarden los cambios.

Cuando se termine de configurar, nos va a dar una dirección web que, utilizando el ejemplo anterior, quedaría "http://192.168.1.1/" y con eso ya estaría terminada la configuración de PFsense.

> [!IMPORTANT]  
> Para loguearse en la página de PFsense utilizar:  
> **Login**: admin  
> **Password**: pfsense

### <ins>Windows Server</ins>
Al colocar la ISO de Windows Server, esta se configura de manera automática. Para evitar esto, destildaremos la opción <ins>Skip Unattended Installation</ins> para poder configurarlo manualmente. Colocamos las especificaciones mencionadas en "Requisitos del sistema".  
Antes de iniciar la máquina, hay que ir a la configuración y en la opción de Network elegiremos en Adapter1 la opción de "**Internal Network**".  
Al iniciar la instalación dejaremos el lenguaje del sistema en inglés, luego elegiremos la opción "Windows Server 2022 Datacenter Evolution (Desktop Edition)" y para finalizar elegimos la opción *Custom Install* y seleccionamos el *Drive 0* para que finalice la instalación. Antes de reiniciarse, te pedirá una contraseña para establecer el administrador, la cual es a gusto propio.  

Ya con la instalación finalizada, hay que loguearse con la cuenta de *Administrator*. Ya en el inicio, dejamos que se inicie el $Server Manager$ y nos dirigimos a:
* Network Connections
  * Click derecho en Properties
    * Doble click en Internet Protocol V4

Aquí configuramos la dirección IP manualmente con alguna IP dentro del rango DHCP. Colocamos una máscara de /24, la Gateway va a ser la IP de la red LAN, y el servidor DNS va a ser la misma IP que colocamos como dirección IP. También, como DNS secundaria, se puede poner alguna de las bien conocidas como Google: 8.8.8.8 o 4.4.8.8.  

Para probar que la conexión es estable, se puede ingresar al servicio interno de PFsense que te da al finalizar la configuración (en el ejemplo sería "http://192.168.1.1/"). Si entra, quiere decir que la conexión está establecida y que la red LAN funciona correctamente.  

> [!TIP]  
> Para más comodidad a la hora de identificar las máquinas, se puede cambiar el nombre dirigiéndote al buscador: *About your PC* -> *Rename This PC* y cambiarlo a, por ejemplo, `WIN-AD`. Esto nos va a servir más adelante.

Una vez conectado, hay que crear un servidor dedicado de **Active Directory**. Para esto, vamos al $Server Manager$ y nos dirigimos a:
* Add Roles and Features
  * Next tres veces seguidas
    * Buscamos y tildamos "Active Directory Domain Services"
      * Add Features
        * Next tres veces más
          * Install

Esperamos a que se termine de instalar y luego clic en "Close". Seleccionamos el banderín que se encuentra en la parte superior izquierda y nos dirigimos a la opción "Promote this server to a domain controller". Los siguientes pasos son:

* Seleccionar "Add New Forest"
  * En "Root Domain Name" colocar un nombre de dominio como "soc.lab" o "prueba.local", y luego Next
    * Elegimos una contraseña, la colocamos, repetimos en "Confirm Password" y damos Next
      * Dejar la parte de DNS por defecto y Next
        * El NetBIOS lo escribimos físicamente (en papel) para recordarlo y damos Next
          * Las siguientes dos opciones las dejamos por defecto y Next
            * Damos a Install y reiniciamos cuando se solicite

Una vez reiniciado, el Active Directory quedará activo.

---

### <ins>Windows 10</ins>
Luego de probar el funcionamiento de la red LAN en Windows Server, realizaremos el mismo proceso de instalación con Windows 10. Antes de iniciar la máquina, hay que ir a la configuración y en la opción de Network elegimos en Adapter1 la opción de "**Internal Network**".  
Al momento de instalar, dejamos el lenguaje del sistema en inglés, la opción de Windows sin KEY y seleccionamos "Windows 10 Pro".

Ya con la instalación finalizada, hay que loguearse. Ya en el inicio nos dirigimos a:
* Network Connections
  * Click derecho en Properties
    * Doble click en Internet Protocol V4

Aquí configuramos la dirección IP manualmente con alguna IP dentro del rango DHCP diferente a la que le dimos al Win-Server. Colocamos una máscara de /24, la Gateway será la IP de la red LAN, y el servidor DNS será la misma dirección IP que tiene Win-Server.  

Para probar que la conexión es estable, se puede ingresar al servicio interno de PFsense (por ejemplo, "http://192.168.1.1/"). Si puede acceder, quiere decir que la conexión está establecida y que la red LAN funciona correctamente.  

Una vez conectado a la red, nos dirigimos a:

* Settings
  * About your PC

Cambiaremos el nombre a uno más fácil de recordar y <ins>reiniciaremos la PC</ins>. Una vez reiniciada, nos dirigimos al mismo lugar y buscamos la opción "Advanced System Settings", allí elegimos "Change domain or workgroup name", y en la opción "Member of Domain:" escribimos el "**NetBIOS**" que guardamos anteriormente.  
Una vez colocado, solicitará un usuario y contraseña de ese dominio, y usaremos la cuenta de administrador del Windows Server. Una vez dentro del dominio, damos OK y reiniciamos cuando se solicite.  
De esta forma, estará dentro del dominio y conectado a Windows Server.

---

### <ins>Linux/Ubuntu</ins>
Una vez que Active Directory (Windows Server) esté conectado con la máquina Win10, hay que crear la zona de monitoreo SIEM.  
Para esto, creamos la máquina virtual con la configuración indicada en "Requisitos del Sistema".  
Antes de iniciar la máquina, vamos a la configuración y en la opción de Network elegimos en Adapter1 la opción de "**Internal Network**".  
Sigue la instalación y, al ingresar al escritorio, configuramos la red.  

La misma se configura desde la parte superior derecha:
* Wired Connected
  * Wired Settings
    * Clic en la "tuerca"
      * Ir a la pestaña "IPv4"

Aquí configuramos la dirección IP manualmente con alguna IP dentro del rango DHCP diferente a la de Win-Server y Win10. Colocamos una máscara de /24, la Gateway será la IP de la red LAN, y el servidor DNS será la misma IP que tiene Win-Server.  

Para probar que la conexión es estable, accedemos a "http://192.168.1.1/" (ejemplo). Si accede, la red LAN funciona correctamente.

---

## ElasticSIEM
Para instalar **ElasticSIEM**, se requieren tres sistemas: Java, Elasticsearch y Kibana.  
Estos se instalan en la terminal de Linux con los siguientes comandos.  

Primero, instalamos Java:

```bash
sudo apt update && sudo apt upgrade -y
sudo apt install -y openjdk-17-jdk
```

Después, descargamos e instalamos Elasticsearch:

```bash
wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.9.0-amd64.deb
sudo dpkg -i elasticsearch-8.9.0-amd64.deb
```

Y habilitamos el servicio:

```bash
sudo systemctl enable elasticsearch
sudo systemctl start elasticsearch
```

> [!TIP]  
> Para verificar si el programa corre correctamente, se puede usar:  
> `sudo systemctl status elasticsearch`

Luego de instalar Elastic, hay que <ins>configurarlo</ins>.  
El archivo se encuentra en: `/etc/elasticsearch/elasticsearch.yml`  
Podemos editarlo con:

```bash
sudo nano /etc/elasticsearch/elasticsearch.yml
```

Dentro del archivo de configuración, busca y edita las siguientes líneas (quitando el `#` al inicio):

```yml
network.host: "192.168.0.1"
http.port: 9200
xpack.security.http.ssl.enabled: true
xpack.security.transport.ssl.enabled: true
```

> [!IMPORTANT]  
> Todas las líneas que contienen `#` están comentadas. Quitar el `#` en las líneas que quieras configurar.

Estas configuraciones se copian del archivo `Configuración-Elastic.txt`.

Una vez configurado, reiniciamos el servicio:

```bash
sudo systemctl restart elasticsearch
```

En el navegador del sistema, colocamos `http://localhost:9200`.  
Nos pedirá usuario y contraseña, los cuales conseguimos con el siguiente comando:

```bash
sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic
```

El resultado será una línea indicando:  
`New value: TU_CONTRASEÑA_ELASTIC`

Esta será la contraseña del usuario "elastic".  
Guárdala en un archivo `.txt` en el escritorio.  
Al recargar la URL, coloca:

**User**: elastic  
**Password**: TU_CONTRASEÑA_ELASTIC  

La página mostrará el nombre de sistema y algunos datos que indican que está en funcionamiento.

---

### <ins>Kibana</ins>
Una vez terminada la instalación de Elastic, descargamos e instalamos Kibana con el comando:

	wget https://artifacts.elastic.co/downloads/kibana/kibana-8.9.0-amd64.deb
	sudo dpkg -i kibana-8.9.0-amd64.deb

Luego de descargarlo, se activa el programa con:

	sudo systemctl enable kibana
	sudo systemctl start kibana

Una vez activo el programa, se debe cambiar la contraseña de Kibana para acceder a Elastic. Para esto, utilizamos el comando:

	sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
 
Dejará una línea que indica "New value: TU_CONTRASEÑA_KIBANA", la cual es la contraseña para el usuario kibana_system. Esta contraseña también se debe guardar en el archivo .txt para usar más adelante.<br/>
Ya con la contraseña, se pasa a configurar Kibana. Utiliza el comando "[sudo nano /etc/kibana/kibana.yml]" y busca las líneas a editar:

	#server.port: 5601
 	#server.host: ""
  	#server.publicBaseUrl:
   	#elasticsearch.hosts: ["http://localhost:9200"]
    #elasticsearch.username: "kibana_system"
	#elasticsearch.password: "pass"

Estas se deben configurar copiando las líneas indicadas del archivo de [Configuración Kibana](Documentación/Configuración-Kibana.txt).<br/>
Una vez configurado, se reinician los dos programas:

	sudo systemctl restart elasticsearch
	sudo systemctl restart kibana

Al finalizar, los programas pueden tardar en iniciarse aproximadamente 2 minutos. Luego de ese tiempo tendrás [http://localhost:9200], donde muestra que Elastic está activo, y "[<ins>http://localhost:5601</ins>]", donde tendrás el Elastic de forma gráfica. Luego de ingresar, pulsas el botón que diga "I explore for my own" y tendrás el sistema en completo funcionamiento.
## Sysmon y Winlogbeat
Para poder enviar logs desde Win10 y Win-Server al sistema de Elastic, hay que conectar ambos sistemas utilizando <ins>Sysmon y Winlogbeat</ins>. Ambas configuraciones funcionan igual para los dos sistemas, con lo cual habrá que repetir el proceso por cada uno.

### <ins>Sysmon</ins>
Para este programa, hay que descargarlo desde [https://docs.microsoft.com/en-us/sysinternals/downloads/sysmon]. Una vez descargado, se extrae en la ruta [C:\Sysmon\]. Ya con la carpeta extraída, descargamos la configuración, siendo este el último componente.<br/>
> [!NOTE]
> Este archivo se puede descargar por internet y hay varias versiones hechas por la comunidad, mayormente se encuentran en GitHub.
<br/>

En esta guía se utiliza una configuración descargada utilizando la terminal de PowerShell como **Administrador**, con el siguiente comando:

	Invoke-WebRequest -Uri "https://raw.githubusercontent.com/SwiftOnSecurity/sysmon-config/master/sysmonconfig-export.xml" -OutFile "C:\Sysmon\sysmonconfig.xml"

Con la configuración descargada, se ejecuta el programa utilizando PowerShell como administrador:

	C:\Sysmon\Sysmon64.exe -accepteula -i C:\Sysmon\sysmonconfig.xml

Para verificar que Sysmon esté en funcionamiento, se puede utilizar el siguiente comando. Si este no muestra logs o da algún error, significa que hubo un problema con la instalación:

	Get-WinEvent -LogName "Microsoft-Windows-Sysmon/Operational" -MaxEvents 10 | Format-List

Una vez instalado, es necesario enviar los logs que genere Sysmon a Elastic. Esto se hará con Winlogbeat, el cual se encarga de conectar ambas máquinas virtuales (Win10 y Win-Server) a Elastic.

### <ins>Winlogbeat</ins>
Este programa es compatible con Elastic y se puede descargar desde su página: [https://www.elastic.co/downloads/beats/winlogbeat].  
Una vez descargado el archivo .zip, se descomprime en la ruta [C:\Program Files\Winlogbeat\].  
Dentro de la carpeta, busca el archivo `winlogbeat.yml` y configúralo utilizando el bloc de notas. Las líneas a editar o agregar son las siguientes:

> [!IMPORTANT]  
> Dentro de Windows, así como en Linux, todas las líneas del archivo `.yml` que contengan <ins>"#"</ins> quedarán como configuración predeterminada. A las líneas que se deben editar se les debe quitar el "#".
<br/>

	#winlogbeat.event_logs:
	  - name: Application

	  - name: System

	  - name: Security

	#host: "localhost:5601"
	#hosts: ["localhost:9200"]
	#protocol: "http"
	#username: "elastic"
	#password: "changeme"
	#ssl.verification_mode:

Las líneas a editar o agregar se encuentran en [Configuración Winlogbeat](Documentación/Configuración-Winlogbeat.txt).<br/>
Una vez editado el archivo, ejecutamos PowerShell como administrador y para dirigirse a la carpeta donde está Winlogbeat, utiliza el comando:

	cd 'C:\Program Files\Winlogbeat'

Ya en el directorio, ejecutas:

	.\install-service-winlogbeat.ps1
	Start-Service winlogbeat

Luego de instalar, podemos verificar de dos formas:  
Una es entrando a *Services* y buscando el programa.  
La otra opción es ejecutar en PowerShell como administrador el siguiente comando:

	.\winlogbeat.exe setup -e

Este muestra los logs recogidos por el programa.

### <ins>Conexión con Elastic</ins>
Cuando ambos sistemas operativos tengan instalado Sysmon y Winlogbeat, se deben implementar en ElasticSIEM.
<br/>
Dentro de ElasticSIEM, dirígete a [Menú lateral → Analytics → Discover] y selecciona "<ins>Add Integrations</ins>".  
Luego de agregar esta integración, vas a ver en la pestaña <ins>Discover</ins>, sobre la parte superior izquierda, una pestaña que muestra los **Data View**. En esa pestaña busca el recuadro de "<ins>Winlogbeat-*</ins>".<br/>
`Winlogbeat-*` muestra los logs de los sistemas Windows en una sola pestaña. En esta se ve una [TimeLine](Capturas/Captura-Logs-1.png) en la cual se va mostrando la cantidad de [alertas](Capturas/Captura-Logs-2.png) que muestra Winlogbeat a lo largo del tiempo.<br/>
Por otro lado, dirígete a [Menú lateral → Security → Alerts], donde se encuentra la información de estas [alertas](Capturas/Captura-Alertas-1.png), en las cuales se [filtran y categorizan](Capturas/Captura-Alertas-2.png) para una mejor detección.<br/>
Contando con estas dos funciones, más otras que ya vienen integradas en el paquete ElasticSIEM, es posible hacer pruebas de ataques simulados.

## Simulación de ataques

Teniendo un entorno SIEM funcional, es momento de comprometerlo para comprobar sus funcionalidades. Para esto se va a utilizar <ins>**BadBlood**</ins> para comprometer Windows Server y <ins>**Atomic Red Team**</ins> para comprometer Windows 10.

### <ins>BadBlood</ins>
BadBlood es un malware sencillo de descargar e instalar. Para realizar la descarga, ingresa al link de [GitHub](https://github.com/davidprowe/BadBlood), descarga el .zip y extrae el contenido en cualquier ubicación.<br/>
Una vez extraído, ingresa como administrador a PowerShell y ejecuta los siguientes comandos:

	cd .\Downloads\BadBlood-Master\BadBlood-Master\

Luego de ingresar a la carpeta del malware, este se ejecuta con:

	.\invoke-BadBlood.ps1

Con esto aparecerá un pequeño párrafo explicativo como guía para ejecutarlo. Una vez ejecutado, deja que la magia suceda (esto puede tomar varios minutos).<br/>
Una vez terminado el proceso, en ElasticSIEM, en la pestaña de *Discover*, deberían haberse registrado varios logs y eventos relacionados con la creación de usuarios, roles y eventos en Win-Server.
### <ins>Atomic Red Team</ins>
Atomic Red Team es un programa un poco más complejo de instalar y ejecutar. Primero es necesario desactivar Windows Defender, para esto hay que dirigirse a [<ins> Menú del buscador → Settings → Update & Security → Windows Defender → Virus & Threat Protection → Virus & Threat Protection Settings → Manage Settings </ins>]. En esta pestaña desactiva todas las opciones. Una vez desactivado, el último paso es ejecutar el comando:

	Set-MpPreference -DisableRealtimeMonitoring $true

Después, para que PowerShell pueda instalar las dependencias, necesita el módulo ".yaml", el cual se instala con: 

	Install-Module powershell-yaml -Force -Scope CurrentUser
	Import-Module powershell-yaml

Con el antivirus completamente desactivado, se deben descargar dos carpetas. La primera es [atomic-red-team-master](https://github.com/redcanaryco/atomic-red-team.git) y la segunda es [invoke-atomicredteam-master](https://github.com/redcanaryco/invoke-atomicredteam.git), ambas se encuentran en GitHub descargando el archivo .zip.<br/>
Antes de extraerlos se debe crear una carpeta con nombre "AtomicRedTeam" en la carpeta "C:\" (disco local). Estos archivos descargados se deben descomprimir en la carpeta AtomicRedTeam, allí debería quedar:
	
 	C:\AtomicRedTeam\
	    └── atomic-red-team-master\
	    ├    └── atomic-red-team-master\
	    ├    	├── .devcontainer\
	    ├    	├── atomics\
     	    ├		└── ...\	
	    └── invoke-atomicredteam-master\
	         	└── invoke-atomicredteam-master\
	         		├── .git\
	    			├── docker\
	         		└── ...
 
Como se puede ver, ambos archivos al extraerlos crean una carpeta del mismo nombre dentro de su propia carpeta con el contenido necesario. Para evitar problemas con los comandos que se deben ejecutar, se debe quitar la carpeta intermedia con nombre repetido para tener una ruta más corta, además de mover la carpeta "\atomics" a la ruta "C:\AtomicRedTeam\". Esto debería quedar así:

  	C:\AtomicRedTeam\
	    ├── atomic-red-team-master\
     	    ├	 └── .devcontainer\
	    ├	 └── ...
	    ├── atomics\
     	    ├	 └── indexes\
	    ├	 └── ...
	    └── invoke-atomicredteam-master\
	         └── .git\
	         └── ...

Una vez las carpetas estén colocadas correctamente, se debe instalar. En este repositorio se explicará el paso a paso de esta versión. En caso de encontrarte con un error no dudes en consultar con la [Wiki](https://github.com/redcanaryco/invoke-atomicredteam/wiki) de Atomic Red Team. Primero se debe ingresar a PowerShell como administrador y ejecutar:

	powershell -exec bypass

Luego de que se recargue PowerShell se debe ejecutar:

	Install-Module -Name invoke-atomicredteam,powershell-yaml -Scope CurrentUser

Finalizada la instalación del módulo, se debe verificar que se instaló correctamente con el código:

	Get-Module

Si al ejecutarlo aparece "Invoke-AtomicRedTeam" en la lista de módulos quiere decir que el programa se instaló satisfactoriamente. Para poder simular un ataque es necesario ingresar un código. Este código es el que se usa para categorizar ataques en la organización ["MITRE ATT&CK"](https://attack.mitre.org/matrices/enterprise/).<br/>

> [!NOTE]  
> El link que está atado a MITRE ATT&CK lleva a un cuadro con todos los ataques categorizados. Si a estos les pasas el mouse por encima verás un código como "T1548" o "T1016". Estos son los códigos utilizados para lanzar ataques con Atomic Red Team.
<br/>

Para iniciar un ataque se debe utilizar el comando `Invoke-AtomicTest` seguido de un código de ataque. También se le puede agregar tags al final de la línea como por ejemplo `-ShowDetailsBrief`, que muestra una lista de funciones del ataque en concreto, o `-CheckPrereqs`, el cual muestra una lista de requisitos para que se pueda ejecutar el ataque (mayormente siempre se cumplen los requisitos, ya que son programas que vienen incluidos con Windows).<br/>
Para poder generar logs, ejecuta algunos ataques, los cuales tendrían que salir en ElasticSIEM. También mencionar que, en caso de algún error en el proceso, se puede verificar utilizando el [video de YT](https://www.youtube.com/watch?v=_xW3fAumh1c) que se utilizó para hacer la parte de Atomic en esta guía.

## Pasos finales y práctica
Luego de tener ejecutados ambos malwares en los sistemas operativos, siendo BadBlood en Win-Server y algún ataque a elección en Atomic Red Team, se va a tener acceso a gran cantidad de logs generados que envía Winlogbeat a ElasticSIEM. Con estos se puede hacer el seguimiento para verificar cómo se desarrolla un malware en un entorno controlado y cuáles son sus paso a paso para lograr infectar un Sistema Operativo.<br/>
<br/>
Dentro de la parte **práctica**, se puede explorar el sistema Elastic para conocer todas sus funciones. A modo personal, para realizar esta guía le solicité a **<ins>ChatGPT</ins>** que me dé desafíos en los cuales me indicara qué malware ejecutar en Atomic Red Team y responder una serie de preguntas.<br/>
<br/>
<br/>
<br/>
Con un sistema SIEM como Elastic se pueden realizar gran cantidad de acciones con las cuales se puede aprender con cada prueba que se hace. Este proyecto no es una solución definitiva, sino un entorno para practicar y mejorar en el proceso. Si llegaste hasta acá, o si te sirve de inspiración para construir tu propio laboratorio SOC, significa que valió la pena, ya que lo que me mueve es la curiosidad constante y las ganas de seguir aprendiendo.

