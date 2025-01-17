Id Máquina Virtual: 3.2.7-49.47
IP Máquina: 10.11.49.47
IP Máquina Compañero: 10.11.49.48
Usuario: lsi 
## Información básica de la máquina

2 tarjetas de red: ens33 10.11.49.47 y ens34 10.11.51.47

SSH por puerto 22
HTTP por puerto 80
HTTPS por puerto 443
NTP por puerto 123
rsyslog por puerto 514

comandos básicos:
```shell
ps #muestra procesos
cd #acceso a directorios
ls #muestra los elementos en el directorio
apt update #actualización de repositorios
apt install #instala paquete a determinar
apt upgrade #instala y actualiza paquetes
apt dist-upgrade #incluye dependencias
apt clean #limpia archivos basura resultantes de uso de comandos apt
apt remove #desinstala paquete a determinar
apt remove --purge #borra también ficheros de configuración
apt-cache search #Muestra paquetes relacionados con una cadena de caracteres
reboot #reinicia el ordenador
exit #se sale de la conexión ssh
```

## Parte 1
##### a) _Configure su máquina virtual de laboratorio con los datos proporcionados por el profesor. Analice los ficheros básicos de configuración (interfaces, hosts, resolv.conf, nsswitch.conf, sources.list, etc.)_

En primer lugar nos conectamos por primera vez a la máquina con el usuario lsi, y cambiamos la contraseña con:

```shell
passwd lsi
```

Y hacemos lo propio con

```shell
su
#Metemos la contraseña
passwd
```

Después eliminamos avahi-daemon.service, avahi-daemon.socket y ModemManager.service, para eliminar vulnerabilidades a costa de perder facilidades de red.

A continuación configuramos nuestra dirección estática para evitar DHCP spoofing en **/etc/network/interfaces**:

```bash
auto lo ens33 ens34
iface ens33 inet static
address 10.11.49.47
netmask 255.255.254.0
broadcast 10.11.49.255
network 10.11.48.0
gateway 10.11.48.1
iface ens34 inet static
address 10.11.51.47
netmask 255.255.254.0
broadcast 10.11.51.255
network 10.11.50.0
```

Al buscar información sobre los archivos de configuración básica obtenemos lo siguiente:

> /etc/hosts relaciona nombres de host a IPs de forma local (alternativa local a DNS)
> 
> /etc/network/interfaces contiene la configuración de las interfaces de red local
> 
> /etc/resolv.conf para la resolución de nombres de dominio a IPs mediante DNS, e indica direcciones de servidores DNS
> 
> /etc/nsswitch.conf determina qué fuentes y en qué orden busca determinada información (nombres de host, usuarios, contraseñas...)
> 
> /etc/apt/sources.list contiene los repositorios de los que se obtienen los paquetes de la distribución de la máquina

##### b) *¿Qué distro y versión tiene la máquina inicialmente entregada?. Actualice su máquina a la última versión estable disponible.*

Ejecutamos el comando:

```shell
lsb_release -a #muestra la versión del SO
```

o:

```shell
hostnamectl
```

Comprobamos que el Sistema operativo es un Debian 10 (Buster) desactualizado, por lo que debemos actualizar el sistema con:

```shell
apt update
apt upgrade
```

Al aparecer la pantalla de selección de partición en la que instalar el GRUB (gestor de arranque)

*aqui va una foto*

Seleccionamos /deb/sda y continuamos.

Ahora que tenemos la última versión de Debian 10, es cuestión de cambiar los enlaces de sources.list, ejecutar el comando:

```shell
apt full-upgrade
```

hacer:

```shell
apt upgrade
```

y:

```shell
reboot
```

Para actualizar a Debian 11 y Debian 12.

Para comprobar que ha funcionado hacemos

```shell
lsb_release -a
```

##### c) *Identifique la secuencia completa de arranque de una máquina basada en la distribución de referencia (desde la pulsación del botón de arranque hasta la pantalla de login). ¿Qué target por defecto tiene su máquina?. ¿Cómo podría cambiar el target de arranque?. ¿Qué targets tiene su sistema y en qué estado se encuentran?. ¿Y los services?. Obtenga la relación de servicios de su sistema y su estado. ¿Qué otro tipo de unidades existen?.*

Para target por defecto:

```shell
systemctl get-default
```

Obtenemos que el target por defecto es el graphical.target, lo cual podemos cambiar a multi-user con el comando

```shell
systemctl set-default multi-user.target
```

Para obtener las relaciones:

```shell
systemctl list-dependencies multi-user.target
```

Con éste último comando observamos que existen distintos tipos de elementos:

.service
.target
.socket
.timer
.mount

Para configurar sudo de la máquina, estando en root:

```shell
visudo
```

Y añadimos debajo de root:

```bash
lsi    ALL=(ALL) ALL
```

Y así podemos ejecutar comandos que requieran servicios sin ser root con el comando _sudo_, por ejemplo:

```shell
sudo apt update
#metemos contraseña de lsi cuando la pida
```

##### d) *Determine los tiempos aproximados de botado de su kernel y del userspace. Obtenga la relación de los tiempos de ejecución de los services de su sistema.*

Para obtener los tiempos de botado ejecutamos el comando

```shell
systemd-analyze
```

Para la relación de los tiempos de ejecución de los services empleamos

```shell
systemd-analyze blame
```
##### e) *Investigue si alguno de los servicios del sistema falla. Pruebe algunas de las opciones del sistema de registro journald. Obtenga toda la información journald referente al proceso de botado de la máquina. ¿Qué hace el systemd-timesyncd?*

Para ver los servicios fallados:

```shell
systemctl list-unit-files --type=service --state=failed
```

Para obtener logs del servicio systemd-timesyncd:

```shell
journalctl -u systemd-timesyncd.service
```

En el log observamos que realiza una serie de conexiones a una pool de direcciones para NTP, por lo que concluinos que es un daemon para sincronización del reloj por NTP (Network Time Protocol).
##### f) *Identifique y cambie los principales parámetros de su segundo interface de red (ens34). Configure un segundo interface lógico. Al terminar, déjelo como estaba.*

Para ver la configuración de ens34:

```shell
ifconfig ens34
```

Para cambiar ens34:

```shell
ifconfig ens34 down
ifconfig ens34 mtu 1200 #tamaño máximo paquetes de datos
ifconfig ens34 hw ether 00:1e:2e:b5:18:07 #dirección MAC
ifconfig ens34 10.11.50.51 netmask 255.255.254.0 #dirección ip + máscara
ifconfig ens34 up
```

Para crear un interfaz lógico hacemos un :

```shell
ifconfig ens34:0 192.168.1.1 netmask 255.255.255.0
ifconfig ens34:0 up
```

##### g) *¿Qué rutas (routing) están definidas en su sistema?. Incluya una nueva ruta estática a una determinada red.*

Para acceder a la tabla de enrutamiento usamos:

```shell
netstat -nr
```

Y para añadir y borrar una ruta de la misma:

```shell
ip route add 10.11.53.0/24 via 10.11.48.1
```

```shell
ip route del 10.11.53.0/24 via 10.11.48.1
```

##### h) *En el apartado d) se ha familiarizado con los services que corren en su sistema. ¿Son necesarios todos ellos?. Si identifica servicios no necesarios, proceda adecuadamente. Una limpieza no le vendrá mal a su equipo, tanto desde el punto de vista de la seguridad, como del rendimiento.*

Para consultar los servicios activos de la máquina:

```shell
systemctl list-unit-files --type=service --state=enabled
```

Al final borramos los siguientes servicios:

> apparmor.service -> Módulo de seguridad del kernel (perfiles de seguridad)
> 
> anacron.service -> tareas rutinarias (como cron). No las vamos a usar de momento
> 
> accounts-daemon.service -> Gestión de cuentas de GNOME. Riesgo de seguridad
> 
> avahi-daemon.service -> Simplificación de configuración de red. Abre puertos
> 
> bluetooth.service -> No vamos a usar bluetooth
> 
> cron.service -> tareas rutinarias. No las vamos a usar de momento
> 
> cups.service -> impresoras
> 
> cups-browsed.service -> impresoras
> 
> e2scrub_reap.service -> snapshots. Llama a un servicio que no está en la máquina para arreglar los problemas que detecta
> 
> ModemManager.service -> Gestiones de Modem de banda ancha
> 
> open-vm-tools.service -> virtualización. No vamos a usar mv dentro de una mv
> 
> power-profiles-daemon.service -> perfiles de batería. Está conectado directamente a corriente
> 
> rtkit-daemon.service -> sincronización en tiempo real. Principalmente para audio
> 
> ssa.service -> servicio bad
> 
> switcheroo-control.service -> intercambiar entre dos GPUs
> 
> udisks2.service
> 
> vgauth.service -> también virtualización
> 
> wpa_supplicant.service -> Redes móviles
##### i) *Diseñe y configure un pequeño “script” y defina la correspondiente unidad de tipo service para que se ejecute en el proceso de botado de su máquina.*

Creamos en /home/lsi/Escritorio/servicescript/memoryctrl.sh con el siguiente contenido:

```sh
#!/bin/bash

THRESHOLD=50.00

FECHA_ACTUAL=$(date +"%Y-%m-%d %H:%M:%S")

PERCENT_USED=$(df -h / | awk 'NR==2 {print $5}' | sed 's/%//')

if (( $(echo "$PERCENT_USED > $THRESHOLD" | bc -l) )); then
	echo "${FECHA_ACTUAL}: porcentaje de memoria usada ($PERCENT_USED%) excede el 50%" >> /usr/local/bin/disk_space_report.txt
else
	echo "${FECHA_ACTUAL}: porcentaje de memoria usada ($PERCENT_USED%) aceptable" >> /usr/local/bin/disk_space_report.txt
fi
```

y le damos permisos de ejecución:

```shell
chmod +x /home/lsi/Escritorio/servicescript/memoryctrl.sh
```

Luego en /etc/systemd/system/memoryctrl.service:

```sh
[Unit]
Description= Checks memory used is not over 50%
After=basic.target

[Service]
Type=oneshot
ExecStart=/home/lsi/Escritorio/servicescript/memoryctrl.sh
RemainAfterExit=yes

[Install]
WantedBy=multi-user.target
```

Y hacemos un enable:

```shell
systemctl enable memoryctrl.service
```

##### j) *Identifique las conexiones de red abiertas a y desde su equipo.*

Ejecutamos uno de los siguientes comandos:

```shell
netstat -nea #solo conexiones
```

```shell
netstat -l #conexiones + sockets
```

```shell
ss -a #todas las conexiones
```

```shell
ss -6 #para conexiones ipv6
```

```shell
ss -4 #para conexiones ipv4
```
##### k) *Nuestro sistema es el encargado de gestionar la CPU, memoria, red, etc., como soporte a los datos y procesos. Monitorice en “tiempo real” la información relevante de los procesos del sistema y los recursos consumidos. Monitorice en “tiempo real” las conexiones de su sistema.*

Para acceder a los **procesos y recursos del sistema**:

```shell
top #recomendado
```

o:

```shell
systemd-cgtop #printea toda la tabla constantemente
```

Y para monitorizar la red:

```shell
iftop
```

```shell
iptraf #no recomendado, muy complicado
```

```shell
netstat -netuac #hace prácticamente lo mismo
```

##### l) *Un primer nivel de filtrado de servicios los constituyen los tcp-wrappers. Configure el tcp-wrapper de su sistema (basado en los ficheros hosts.allow y hosts.deny) para permitir conexiones SSH a un determinado conjunto de IPs y denegar al resto. ¿Qué política general de filtrado ha aplicado?. ¿Es lo mismo el tcp-wrapper que un firewall?. Procure en este proceso no perder conectividad con su máquina. No se olvide que trabaja contra ella en remoto por ssh.*

En /etc/hosts.deny ponemos:

```script
ALL: ALL: twist
```

y en /etc/hosts.allow:

```script
sshd: 127.0.0.1, 10.11.49.48, 10.11.51.48 #localhost e ip compañero
sshd: 10.20.32.0/21 #EDUROAM
sshd: 10.30.0.0/16 #VPN
sshd: ::1 #localhost ipv6
sshd: 2002:0a0b:3130::1 #ipv6 compañero
```

##### m) *Existen múltiples paquetes para la gestión de logs (syslog, syslog-ng, rsyslog). Utilizando el rsyslog pruebe su sistema de log local. Pruebe también el journald.*

En primer lugar instalamos rsyslog:

```shell
apt install rsyslog
```

Con journald podemos obtener los logs de un servicio mediante:

```shell
journalctl -u _____.service
```

Y el log de la bios usando:

```shell
journalctl -b
```

##### n) *Configure IPv6 6to4 y pruebe ping6 y ssh sobre dicho protocolo. ¿Qué hace su tcp-wrapper en las conexiones ssh en IPv6? Modifique su tcp-wapper siguiendo el criterio del apartado h). ¿Necesita IPv6?. ¿Cómo se deshabilita IPv6 en su equipo?*

Para crear el túnel editamos el archivo **/etc/hosts/interfaces** y añadimos:

```shell
auto ________________ 6to4

------
------  #datos de las direcciones estáticas (los otros interfaces)
------
iface 6to4 inet6 v4tunnel
address 2002:0a0b:312f::1
netmask 16
endpoint any
local 10.11.49.47
```

Para deshabliltar IPv6, en **/etc/sysctl.conf** añadimos:

```bash
net.ipv6.conf.all.disable_ipv6 = 1
net.ipv6.conf.default.disable_ipv6 = 1
net.ipv6.conf.lo.disable_ipv6 = 1
```

y aplicamos los cambios con sysctl -p (lo mismo con los valores a 0 o las líneas comentadas para habilitar de nuevo)

*Nota: en caso de temer problemas de buffer a la hora de desactivar/activar IPv6, con IPv6 activada, se hace lo siguiente:*

*ifdown 6to4 (debería mostrar que la interfaz está disponible pero sin configurar)*

*ip tun del 6to4 (borramos la interfaz desconfigurada)*

*ifconfig -a (comprobamos que ya no aparece)*

## Parte 2

##### a) *En colaboración con otro alumno de prácticas, configure un servidor y un cliente NTPSec básico.*

Sustituimos ntp por ntpsec:

```shell
apt install ntpdate ntpsec
```

y comprobamos que el servicio esté activo:

```shell
systemctl enable ntpsec
systemctl start ntpsec
systemctl status ntpsec
```

A continuación editamos el archivo **/etc/ntpsec/ntp.conf**

Configuración del servidor:

![ntp-server-1](https://github.com/user-attachments/assets/3e87cfea-cc9f-496b-af5a-fa6b8a657c05)
![ntp-server-2](https://github.com/user-attachments/assets/dd167009-4d9c-457b-904c-8c18929fcb86)

Configuración del cliente:

![ntp-client-1](https://github.com/user-attachments/assets/3bbd764d-1a0b-420d-b414-8aed9744c2b2)
![ntp-client-2](https://github.com/user-attachments/assets/496e6c5b-bdd5-4939-84bb-bf6b58cec4ed)

Y reiniciamos el servicio:

```shell
systemctl restart ntpsec.service
```

Para comprobar los servidores ntp a los que la máquina está conectada usamos el comando:

```shell
ntpq -p
```

Está conectado al servidor si hay un asterisco al lado de la ip.

Y para alterar y comprobar la fecha de la máquina:

```shell
date
date --set "YYYY:MM:DD HH:mm:ss"
```

##### b) *Cruzando los dos equipos anteriores, configure con rsyslog un servidor y un cliente de logs.*

Teniendo rsyslog instalado y activo.

Configuración del servidor:

![rsyslog-server-1](https://github.com/user-attachments/assets/37bb282a-8945-43b0-8751-52b5709c136d)
![rsyslog-server-2](https://github.com/user-attachments/assets/52271b6f-4e7a-4cdd-8a69-6be59b45159c)

Configuración del cliente:

![rsyslog-client-1](https://github.com/user-attachments/assets/bcb192d8-0e70-4e0a-b0ec-d93707b63a04)
![rsyslog-client-2](https://github.com/user-attachments/assets/b15f7630-998c-4c3a-9395-40afa39a4b7f)
*Nota: descomentar las 2 lineas bajo el comentario "Implementación propuesta en clase" o la linea bajo el comentario "Server" y las 4 lineas antes de "RULES".*

Tras editar el archivo **/etc/rsyslog.conf** y guardar los cambios, se reinicia el servicio:

```shell
systemctl restart rsyslog.service
```

<strong><u><span style="font-size: 24px;">Muy Importante</span></u></strong>: Al estar desconectado del servidor (o el mismo no estar en pie), los mensajes de log se deben guardar en una cola que se manda al servidor una vez vuelva a estar operativo. <strong><u>Esos logs deben aparecer en el archivo aunque se hayan realizado en un momento en el que el servidor no estuviese activo</u></strong>**

Para comprobar el funcionamiento de la cola primero desactivamos el servicio, luego el socket (como el socket tiene una dependencia con el servicio los desconectamos a la vez <strong>systemclt stop syslog syslog.socket</strong>) y mientras el servidor está desconectado, el cliente generará logs. A la hora de conectarlos, primero iniciamos el socket <strong>systemctl start syslog.socket</strong> y luego el servicio <strong>systemctl start syslog</strong>, con esto los logs que se enviaron durante la desconexión deberían aparecer en el servidor (comando tail para comprobarlo en tiempo real).

##### c) *Haga todo tipo de propuestas sobre los siguientes aspectos.: ¿Qué problemas de seguridad identifica en los dos apartados anteriores?. ¿Cómo podría solucionar los problemas identificados?*

Entre los posibles problemas de los servidores tenemos los siguientes:  Autenticación (Man In the Middle)  Falta de cifrado de los datos enviados  Configuración incorrecta puede generar vulnerabilidades (sobreexposición, port scanning, envío de informes falsos, DoS...)  Puerto conocido en ambos casos (NTPsec: 123, rsyslog: 514)

##### d) *En la plataforma de virtualización corren, entre otros equipos, más de 200 máquinas virtuales para LSI. Como los recursos son limitados, y el disco duro también, identifique todas aquellas acciones que pueda hacer para reducir el espacio de disco ocupado.*

Para obtener el espacio de disco:

```shell
df -h
```

Para limpiar espacio:

```shell
apt clean
apt --purge autoremove
```

Borramos man:

```shell
apt remove --purge man-db
apt remove --purge dbhelper
```

También borramos libreoffice, los kernels viejos, las versiones antiguas de python y los logs repetidos en /var/log/
##### e) *Instale el SIEM splunk en su máquina. Sobre dicha plataforma haga los siguientes puntos:*

1. Genere una query que visualice los logs internos del splunk
2. Cargué el fichero /var/log/apache2/access.log y el journald del sistema y visualícelos.
3. Obtenga las IPs de los equipos que se han conectado a su servidor web (pruebe a generar algún tipo de gráfico de visualización), así como las IPs que se han conectado un determinado día de un determinado mes.
4. Trate de obtener el país y región origen de las IPs que se han conectado a su servidor web y si posible sus coordenadas geográficas.
5. Obtenga los hosts origen, sources y sourcestypes.
6. ¿Cómo podría hacer que splunk haga de servidor de log de su cliente

Instalamos los paquetes necesarios:

```shell
apt install curl
apt install apache2
```

Y desempaquetamos el paquete que descargamos del teams de la asignatura de la versión más reciente de Splunk

A continuación editamos el archivo **/opt/splunk/etc/system/default/server.conf**:

```bash
minspacefree=5000 #cambiar por 500
```

Al final ejecutamos lo siguiente:

```shell
/opt/splunk/bin/splunk enable boot-start #para que se inicie en arranque, o
/etc/init.d/splunk start #para iniciar el servicio solo
```

Por otro lado, en un navegador conectado mediante la VPN, nos conectamos en:

```navegador
http://10.11.49.47:8000
```

Cargamos en Splunk **/var/log/syslog** y **/var/log/apache2/access.log**

Algunos comandos que podemos emplear son:

```splunk
source="/var/log/apache2/access.log" | table field1 | iplocation field1 | geostats count
```

para obtener la localización.

```splunk
source="/var/log/apache2/access.log" | sourcetype=access_combined | stats count by clientip
```

para obtener las ips conectadas al servidor.

```splunk
sourcetype=access_combined | where _time>=strptime("2024-03-01 00:00:00", "%Y-%m-%d %H:%M:%S") AND _time<=strptime("2024-03-01 23:59:59", "%Y-%m-%d %H:%M:%S") | stats count by clientip
```

para las ips conectadas en un día y hora concretos.