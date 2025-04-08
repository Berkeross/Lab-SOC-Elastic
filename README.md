# Lab-SOC-Elastic
Laboratorio soc utilizando ElasticSIEM como base.<br/>
Soc lab using ElasticSIEM as a core siystem.

### DESCRIPCION
Este repositorio cuenta con un SOC elaborado en base a ElasticSIEM, conectado a un Windows Server con AD (Active Directory) y una maquina Windows 10.
Estas 3 maquinas virtuales estan conectadas a una red interna (LAN) usando PFsense, concluyendo con un total de 4 maquinas virtuales conectadas entre ellas.<br/>
A continuacion se encunetra una guia de como implementar este sistema desde 0, aun asi quedan todos los archivos configurados arriba😸.

# GUIDE
En este repositorio voy a explicar paso a paso como descargar, instalar e implementar estas maquinas virtuales y conectarlas entre si para tener un laboratorio SOC/SIEM.

### REQUISITOS DE SISTEMA
Dentro de VirtualBox Utilice 4 maquinas viertuales abiertas simultaneamente de las cuales 2 fueron maquinas windows.
Teniendo en cuentas esto utilice aproximadamente:
* 8GB de Memoria RAM (2GB para cada maquina).
* 6 hilos(GPU), 2 hilos para las maquinas win ademas 1 hilo para Ubuntu y 1 mas para PFsense.
* 50GB de disco para Win-Server, 50GB para Win-10, 10GB para PFsense y 50GB para Linux/Ubuntu.
> [!NOTE]
> Cabe aclarar que no es estricto, si es necesario agregarle mas y disninuirle la canttidad, el funcionamiento va a ser el mismo.

### MAQUINAS
Para virtualizar estas 4 maquinas virtuales se requiere de un virtualizador, en esta guia voy a utilizar **VirtualBox**. pero se puede usar calquier otro.
Se van a ejecutar 4 maquinas virtuales en total utilizando, estas van a ser:
- Una maquina virtual con [PFsese](https://www.pfsense.org/download/)
- Una maquina virtual con [Windows Server 2022](https://www.microsoft.com/es-es/evalcenter/download-windows-server-2022)
- Una maquina virtual con [Windows 10](https://www.microsoft.com/es-es/software-download/windows10)
- Una maquina virtual con [Linux Ubuntu](https://ubuntu.com/download/desktop)

### RED
Dentro de VirtualBox utilizaremos 2 tipos de redes:
* Una es la red WAN para dar internet.
* Y la red lan que viene por defcto en VirtualBox.

## INSTALACION Y CONFIGURACION DE MAQUINAS

### <ins>PFsense</ins>
Luego de descargar PFsense, hay que darme al boton de "nuevo" en Virtual box, se eleje el archivo $.iso$ y lo colocamos como "Tipe: Linux" y "Version: other Linux x64".<br/>
Antes de iniciar la maquina hay que darle <ins>click derecho</ins> y ir a configuracion, dentro de configuracion vamos a la pestaña <ins>network</ins> donde hay que habilitar el <ins>adapter2</ins>, donde vamos a cambiar la opcion WAN por INTERNAL NETWORK y en opciones avanzadas en **promiscuous mode** habilitamos ALLOW ALL y le damos a accept.<br/>
<br/>
Ahora iniciamos PFsese y dentro de la instalacion aceptaremos todo hasta llegar a la **configuracion de red** donde elejiremos cual sera la $red WAN$ y la $red LAN$, la red WAN sera la red NAT y LA red LAN sera la red del <ins>adapter2</ins>. luego de eso elejimos el disco donde instalar el sistema y elejimos reboot.<br/>
<br/>
Luego de que se reinicie nos dirigimos a $devices$ luego a $optical units$ y destildamos la opcion donde esta la <ins>.iso</ins> de PFsense, luego reiniciamos el sistema.<br/>
Con el sistema y instalado esperamos a que cargue del todo (esto puede tomar aprox 1 minuto o 2), luego de que se termine la carga deverian aparecer 16 opciones.<br/>
<br/>
Para que PFsense detecte cual es cada red se asignan manualmente de esta manera:
* Elegimos la opcion 1.
* En la opcion de VLANs damos simplemente enter
* Cuando pida identificar la red WAN hay que elegir la opcion que este configurado en Vbox como Adapter1 
* Cuando pida identificar la red LAN hay que elegir la opcion que este configurada en Vbox como Local Network que en este caso seria Adapter2
</br>
> [!NOTE]
> Teniendo en cuenta como identifica las redes PFsense se puede definir que Adapter1=em0 - Adapter2=em1 - Adapter3=em2 - Adapter4=em3. Siempre teniendo en cuenta que estamos utilizando Virtual Box. 
<br/>
Cuando se termine de configurar hay que ir a la opcion 2 donde:<br/>

* Seleccionaremos la red lan con una opcion numerica.
* Luego elegiremos la IP de la LAN (Ej: 192.168.1.1).
* Luego la mascara que sera de 24.
* En la Opcion de ingresar una IP Gatewat simplemente se le da al boton enter.
* En la opcion de ingresar una IP V6 se da enter nuevamente.
* Y cuando se solicite elegir si queremos DHCP para la red LAN damos "Y"
* El rango IP para el DHCP empesaria (utilizando el ejemplo anterior) con 192.168.1.2 a 192.168.1.10, luego damos enter y esperamos a que se guarden los cambios.
<br/>
Cuando se termine de configurar no va a adar una direccion web que utilisando el ejemplo anterior quedaria "http://192.168.1.1/" y con eso ya estaria terminada la configuracion de PFsense.</br>
</br>

> [!IMPORTANT]
> para logearse en la pagina de PFsense utilice Login: Admin y Password: pfsense.

### <ins>Windows Server</ins>
Al colocar la ISO de Windows Server esta se configura de manera automatica, para evitar esto destildaremos la opcion <ins>Skip Unattended Installation</ins> para poder configurarlo manualmente, colocamos las especificaciones mencionadas en "requisitos del sistema".<br/>
Antes de iniciar la maquina hay que ir a la configuracion y en la opcion de network elegiremos en Adapter1 la opcion de "**Internal Network**".<br/>
Al iniciar la instalacion dejaremos el lenguaje del sistema en ingles, luego elegiremos la opcion "Windows Server 2022 Datacenter Evolution (Desktop Edition)" y para finalizar elegiremos la opcion custom install y elegiremos el drive0 para que finalice la instalacion. Antes de reiniciarse te pedira una contraseña para establecer el administrador la cual es a gusto propio.<br/>
<br/>
Ya con la instalacion finalizada hay que logearse con la cuenta de administrator, ya en el inicio dejamos que se inicie el $Server Manager$ y nos dirigimos a:
* network Conections
  * Click derecho a Properties
    * doble click a Internet Protocol V4<br/>
    
Aca configure la direccion IP manualmente con con alguna IP dentro del rango DHCP, coloque una mascara de /24, la gateway va a ser la ip de la red LAN y el servidor DNS va a ser la misma quecoloque como direccion IP, tambien como DNS secundaria se puede poner alguna de las bien conocidas como google:8.8.8.8 o 4.4.8.8.<br/>
<br/>
Para probar que la coneccion es estable se puede ingresar al servicio interno PFsense que te da al finalizar la configuracion (en el ejemplo seria "http://192.168.1.1/") si esta entra quiere decir que la coneccion esta establecida y que la red LAN funciona correctamente.<br/>
<br/>
> [!TIP]
> para mas comodidad a la hora de indentificar las maquinas, se puede cambiar el nombre dirigendote al buscador: About your PC -> Rename This PC y cambiarlo a por ejemplo (WIN-AD), esto nos va a servir mas adelante.

Una vez conectado hay que crear un Servidor dedicado de **Active Directory** para esto vamos al $Server Manager$ y nos dirigimos a:
* Add Roles and Features.
  * Next tres veces seguidas.
    * Buscamos y tildamos "Active Directory Domain Service".
      * Add Features.
        * Siguiente tres veces mas.
          * Install.
<br/>
Esperamos a que se termine de intalar y "cerrar". Luego seleccionamos el vanderin que se encuentra en la parte superior izquierda y dirigirnos a la opcion de "Promote this server to a domain controller", Los siguentes pasos son:<br/>
<br/>

* Selecciona "Add New Forest"
  * En "Root Domain Name" coloca un nombre de dominio como "soc.lab" o "prueba.local" y luego al boton "next".
    * elegimos una contraseña y la colocamos, luego repetimos en "Confirm Password" y le damos a Next.
      * Dejar la parte DNS por defecto y Next.
        * El NetBIOS lo escribimos en fisico como en un papel par recordarlo y Next.
          * Las siguientes dos opciones las dejamos por defecto y Next.
            * Le damos a instalar y reiniciamos cuando se le solicite.
<br/>
Una vez reiniciado el Active Directory quedara activo.

### <ins>Windows 10</ins>
Luego de probar el funcionamiento de la Red LAN en Windows Server, realizaremos el mismo proceso de instalacion con Windows 10. Antes de iniciar la maquina hay que ir a la configuracion y en la opcion de network elegiremos en Adapter1 la opcion de "**Internal Network**".<br/>
Al momento de instalar dejaremos el lenguaje del sistema en ingles, la opcion de windows sin KEY y la opcion de "Win-10 pro".<br/>
<br/>
Ya con la instalacion finalizada hay que logearse, ya en el inicio nos dirigimos a:
* network Conections
  * Click derecho a Properties
    * doble click a Internet Protocol V4<br/>

Aca configure la direccion IP manualmente con con alguna IP dentro del rango DHCP diferente a la que le dio a Win-Server, coloque una mascara de /24, la gateway va a ser la ip de la red LAN y el servidor DNS va a ser la misma direccion IP que tiene Win-Server.<br/>
Para probar que la coneccion es estable se puede ingresar al servicio interno PFsense que te da al finalizar la configuracion (en el ejemplo seria "http://192.168.1.1/") si puede acceder quiere decir que la coneccion esta establecida y que la red LAN funciona correctamente.<br/>
<br/>
Una vez conectado a la red nos vamos a dirigir a:<br/>

* Settings.
  * About your PC.

cambiaremos el nombre a uno mas mas facil de recordar y <ins>reinicie la PC</ins>. Una vez reiniciado nos dirigimos al mismo lugar y busca la opcion "Advanced System Settings", allí elegimos la opcion de cambiar nombre de dominio, y en la opcion "Member of Domain:" escribimos el "**NetBIOS**" que guarmamos anteriormente, una vez colocado le solicitara un nombre y contraseña de ese dominio y va a utilizar la cuenta de administrador que uso para logearse a Windows Server. Una vez dentro del Dominio le da a "ok" y reinicie cuando se le solicite.
De esta forma estara dentro del dominio y conectado a Windows server.

### <ins>Linux/Ubuntu</ins>
Una vez active Directory (Windows Server) este conectado con la maquina win10 hay que crear la zona de monitoreo SIEM. Para esto crea la maquina virtual con la configuracion indicada en "Requisitos del Sistema". Antes de iniciar la maquina hay que ir a la configuracion y en la opcion de network elegiremos en Adapter1 la opcion de "**Internal Network**".<br/>
SIgue la instalacion y al ingresar al desktop configura la red. La misma se configura en el boton en la parte superior izquierda dirigiendote a:<br/>

* Wired Connected.
  * Wired Settings.
    * Dirigete a la "tuerca"
      * Dirigete a la pestaña "IPv4"
<br/>
Aqui configure la direccion IP manualmente con con alguna IP dentro del rango DHCP diferente a la que le dio a Win-Server y Win10, coloque una mascara de /24, la gateway va a ser la ip de la red LAN y el servidor DNS va a ser la misma direccion IP que tiene Win-Server.<br/>
Para probar que la coneccion es estable se puede ingresar al servicio interno PFsense que te da al finalizar la configuracion (en el ejemplo seria "http://192.168.1.1/") si puede acceder quiere decir que la coneccion esta establecida y que la red LAN funciona correctamente.<br/>
<br/>

### ElasticSIEM
Para INstalar **ElasticSIEM** se requiere de 3 sistemas lso cuales serias java, ekastic y kibana respectivamente. Estos se instalan en la terminal de linux con con los siguientes comandos.
Instalamos Primero Java con el comando:

	sudo apt update && sudo apt upgrade -y
	sudo apt install -y openjdk-17-jdk
 
Despues descargamos e instalamos elastic con el comando:

	wget https://artifacts.elastic.co/downloads/elasticsearch/elasticsearch-8.9.0-amd64.deb
	sudo dpkg -i elasticsearch-8.9.0-amd64.deb

Y habilitamos amobos sistemas, primero elastic:

	sudo systemctl enable elasticsearch
	sudo systemctl start elasticsearch
 
> [!TIP]
> Para verificar si el prigrama corre correctamente se puede utilizar: 	"[sudo systemctl status elasticsearch]".
<br/>

Despues de instalar Elastic hay que <ins>configurarlo</ins>. este se encuenctran en la ruta "[/etc/elasticsearch/elasticsearch.yml]" y se pueden editar utilizando el comando $nano$:

	sudo nano /etc/elasticsearch/elasticsearch.yml

 Dentro del archivo de configuracion, busca las siguientes lineas:

 	#network.host: "192.168.0.1"
  	#http.port: 9200
   	#xpack.security.http.ssl:
	  enabled: true
    #xpack.security.transport.ssl:
	  enabled: true

Estas lineas las vas a configuar copiando estas lineas indicadas del archivo de [Configuración Elastic](Documentación/Configuración-Elastic.txt).<br/>
<br/>
Una vez configruado, se utiliza el comando "[sudo systemctl restart elasticsearch]" para resetear el preograma, el cual tarda uno algunos segundos.
Una vez reiniciado el prigrama, en el buscador predetermiinado del sistema colocamos "</ins>http://localhost:9200</ins>". al ingresar nos solicitara usuario y contraseña, el cual se consigue con el siguiente comando.

 	sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u elastic

Al utilizarlo en la terminal te dejara un lanea que indica "New value: TU_CONTRASEÑA_ELASTIC". Esta contraseña guardala en un archivo ".txt" en el escritorio, y al recarlar la url en los espacios de texto se debe llenar con:<br/>
User: elastic<br/>
Password : TU_CONTRASEÑA_ELASTIC<br/>
<br/>
La pagina te mostrara tu nombre de sistema y algunos datos que indican que esta en funcionamiento.<br/>
Una vez terminada la instalacion de elastic descarmamos e instalamos Kibana con el comando:

	wget https://artifacts.elastic.co/downloads/kibana/kibana-8.9.0-amd64.deb
	sudo dpkg -i kibana-8.9.0-amd64.deb

Luego de descargarlo se activa el programa con:

	sudo systemctl enable kibana
	sudo systemctl start kibana

Una vez activo el programa se debe cambiar la contraseña de kibana para acceder a elastic. Para esto utilizamos el comando:

	sudo /usr/share/elasticsearch/bin/elasticsearch-reset-password -u kibana_system
 
Dejara un lanea que indica "New value: TU_CONTRASEÑA_KIBANA". esta contraseña tambien se debe guardar en el archivo .txt para usar mas adelante.<br/>
Ya con la contraseña se pasa a configurar kibana utiliza el comando "[sudo nano /etc/kibana/kibana.yml]" y buscas las lineas:

	#server.port: 5601
 	#server.host: ""
  	#server.publicBaseUrl:
   	#elasticsearch.hosts: ["http://localhost:9200"]
    #elasticsearch.username: "kibana_system"
	#elasticsearch.password: "pass"

Estas se deben configurar copiando estas lineas indicadas del archivo de [Configuración Kibana](Documentación/Configuración-Kibana.txt).<br/>




