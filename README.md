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
Cuando se termine de configurar no va a adar una direccion web que utilisando el ejemplo anterior quedaria "http://192.168.1.1/" y con eso ya estaria terminada la configuracion de PFsense.
<br/>

### <ins>Windows Server</ins>
Al colocar la ISO de Windows Server esta se configura de manera automatica, para evitar esto destildaremos la opcion <ins>Skip Unattended Installation</ins> para poder configurarlo manualmente, colocamos las especificaciones mencionadas en "requisitos del sistema".<br/>
Antes de iniciar la maquina hay que ir a la configuracion y en la opcion de network elegiremos en Adapter1 la opcion de "**Internal Network**"<br/>
Al iniciar la instalacion dejaremos el lenguaje del sistema en ingles, luego elegiremos la opcion "Windows Server 2022 Datacenter Evolution (Desktop Edition)" y para finalizar elegiremos la opcion custom install y elegiremos el drive0 para que finalice la instalacion. Antes de reiniciarse te pedira una contraseña para establecer el administrador la cual es a gusto propio.<br/>
<br/>
Ya con la instalacion finalizada hay que logearse con la cuenta de administrator, ya en el inicio dejamos que se inicie el $Server Manager$ y nos dirigimos a:
* network Conections
  * Click derecho a Properties
    * doble click a Internet Protocol V4<br/>
    
Aca configuramos la direccion IP manualmente con con alguna IP dentro del rango DHCP, colocamos una mascara de /24, la gateway va a ser la ip de la red LAN y el servidor DNS va a ser la misma que como direccion IP, tambien como DNS secundaria se puede poner alguna de las bien conocidas como google:8.8.8.8 o 4.4.8.8.<br/>
<br/>
Para probar que la coneccion es estable se puede ingresar al servicio interno PFsense que te da al finalizar la configuracion (en el ejemplo seria "http://192.168.1.1/") si esta entra quiere decir que la coneccion esta establecida y que la red LAN funciona correctamente.<br/>

### <ins>Windows 10</ins>
Luego de probar el funcionamiento de la Red LAN en Windows Server, realizaremos el mismo proceso de instalacion con Windows 10. Antes de iniciar la maquina hay que ir a la configuracion y en la opcion de network elegiremos en Adapter1 la opcion de "**Internal Network**".<br/>
Y al momento de instalar eligiremos la opcion de windows sin KEY y la opcion de "Win-10 pro".<br/>
<br/>
Ya con la instalacion finalizada hay que logearse, ya en el inicio nos dirigimos a:
* network Conections
  * Click derecho a Properties
    * doble click a Internet Protocol V4<br/>
Aca configuramos la direccion IP manualmente con con alguna IP dentro del rango DHCP diferente a la que le dimos a Win-Server, colocamos una mascara de /24, la gateway va a ser la ip de la red LAN y el servidor DNS va a ser la misma direccion IP que tiene Win-Server.<br/>
<br/>
Para probar que la coneccion es estable se puede ingresar al servicio interno PFsense que te da al finalizar la configuracion (en el ejemplo seria "http://192.168.1.1/") si esta entra quiere decir que la coneccion esta establecida y que la red LAN funciona correctamente.<br/>

### <ins>Linux/Ubuntu</ins>
