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
#Contraseña
passwd
```

Después eliminamos avahi-daemon.service, avahi-daemon.socket y ModemManager.service, para eliminar vulnerabilidades a costa de perder facilidades de red.

A continuación configuramos nuestra dirección estática para evitar DHCP spoofing en /etc/hosts/interfaces

Al buscar información sobre los archivos de configuración básica obtenemos lo siguiente:

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

y hacer:

```shell
apt upgrade
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

Para configurar sudo de la máquina:

```shell
sudo visudo
```

Y añadimos debajo de root:

```sh
lsi    ALL=(ALL) ALL
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

En el log observamos que realiza una serie de conexiones a una pool de direcciones para NTP, por lo que concluinos que es un daemon para sincronización del reloj por NTP (Network Time Protocol)
##### f) *Identifique y cambie los principales parámetros de su segundo interface de red (ens34). Configure un segundo interface lógico. Al terminar, déjelo como estaba.*

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

> apparmor.service:
> anacron.service
> accounts-daemon.service
> avahi-daemon.service
> bluetooth.service
> cron.service
> cups.service
> cups-browsed.service
> e2scrub_reap.service
> ModemManager.service
> open-vm-tools.service
> power-profiles-daemon.service
> rtkit-daemon.service
> ssa.service
> switcheroo-control.service
> udisks2.service
> vgauth.service
> wpa_supplicant.service
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
netstat -neta #solo conexiones
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

##### l) *Un primer nivel de filtrado de servicios los constituyen los tcp-wrappers. Configure el tcp-wrapper de su sistema (basado en los ficheros hosts.allow y hosts.deny) para permitir conexiones SSH a un determinado conjunto de IPs y denegar al resto. ¿Qué política general de filtrado ha aplicado?. ¿Es lo mismo el tcp-wrapper que un firewall?. Procure en este proceso no perder conectividad con su máquina. No se olvide que trabaja contra ella en remoto por ssh.*

En /etc/hosts.deny ponemos:

```script
ALL: ALL: twist
```

y en /etc/hosts.allow:

```script
sshd: 127.0.0.1, 10.11.49.48, 10.11.51.48
sshd: 10.20. #EDUROAM
sshd: 10.30.9.151 #VPN
```

##### m) *Existen múltiples paquetes para la gestión de logs (syslog, syslog-ng, rsyslog). Utilizando el rsyslog pruebe su sistema de log local. Pruebe también el journald.*

En primer lugar instalamos rsyslog:

```shell
apt install rsyslog
```


##### n) *Configure IPv6 6to4 y pruebe ping6 y ssh sobre dicho protocolo. ¿Qué hace su tcp-wrapper en las conexiones ssh en IPv6? Modifique su tcp-wapper siguiendo el criterio del apartado h). ¿Necesita IPv6?. ¿Cómo se deshabilita IPv6 en su equipo?*

Para crear el túnel editamos el archivo /etc/hosts/interfaces y añadimos:

```shell
auto ________________ 6to4

------
------  #datos de las direcciones estáticas
------
iface 6to4 inet6 v4tunnel
address 2002:0a0b:312f::1
netmask 16
endpoint any
local 10.11.49.47
```

## Parte 2

##### a) *En colaboración con otro alumno de prácticas, configure un servidor y un cliente NTPSec básico.*

##### b) *Cruzando los dos equipos anteriores, configure con rsyslog un servidor y un cliente de logs.*

##### c) *Haga todo tipo de propuestas sobre los siguientes aspectos.: ¿Qué problemas de seguridad identifica en los dos apartados anteriores?. ¿Cómo podría solucionar los problemas identificados?*

##### d) *En la plataforma de virtualización corren, entre otros equipos, más de 200 máquinas virtuales para LSI. Como los recursos son limitados, y el disco duro también, identifique todas aquellas acciones que pueda hacer para reducir el espacio de disco ocupado.*

##### e) *Instale el SIEM splunk en su máquina. Sobre dicha plataforma haga los siguientes puntos:*
1. Genere una query que visualice los logs internos del splunk
2. Cargué el fichero /var/log/apache2/access.log y el journald del sistema y visualícelos.
3. Obtenga las IPs de los equipos que se han conectado a su servidor web (pruebe a generar algún tipo de gráfico de visualización), así como las IPs que se han conectado un determinado día de un determinado mes.
4. Trate de obtener el país y región origen de las IPs que se han conectado a su servidor web y si posible sus coordenadas geográficas.
5. Obtenga los hosts origen, sources y sourcestypes.
6. ¿Cómo podría hacer que splunk haga de servidor de log de su cliente