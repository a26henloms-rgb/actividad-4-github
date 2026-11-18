# Guía Rápida: Configuración de un Servidor DHCP en Linux (ISC DHCP)

![Configuracion DHCP Linux](./configure-dhcp-linux.png)

*Imagen: Como configurar DHCP (Fuente: [linuxconfig.org](https://linuxconfig.org/what-is-dhcp-and-how-to-configure-dhcp-server-in-linux))*

> Esta guía explica cómo **instalar y configurar** un servidor DHCP básico utilizando el paquete `isc-dhcp-server` en distribuciones basadas en Debian/Ubuntu.

---

## Índice

* [1. Instalación del Paquete](#1-instalación-del-paquete-️)
* [2. Configuración de la Interfaz de Red](#2-configuración-de-la-interfaz-de-red-️)
* [3. Configuración del Servidor (`dhcpd.conf`)](#3-configuración-del-servidor-dhcpdconf-️)
* [4. Gestión del Servicio](#4-gestión-del-servicio-)
* [5. Reservas de Direcciones IP](#5-reservas-de-direcciones-ip-)
* [6. Recomendaciones de seguridad](#6-recomendaciones-de-seguridad-)
* [7. Recursos Adicionales](#7-recursos-adicionales-)


<div style="page-break-after: always;"></div>

## 1. Instalación del Paquete ⚙️

Actualiza los repositorios e instala el servicio DHCP ejecutando el siguiente comando en la terminal:

```Bash
sudo apt update
sudo apt install isc-dhcp-server
```

---

## 2. Configuración de la Interfaz de Red 🛠️

### Archivos clave de ISC DHCP

| Ruta | Propósito |
|------|-----------|
| `/etc/default/isc-dhcp-server` | Define en qué interfaz(es) escucha el servidor (`INTERFACESv4`) |
| `/etc/dhcp/dhcpd.conf` | Archivo principal de configuración: subredes, rangos, opciones y tiempos de concesión |
| `/var/lib/dhcp/dhcpd.leases` | Base de datos con las concesiones de IP activas e históricas |
| `/var/log/syslog` | Registro del sistema donde aparecen las peticiones y errores del servicio |

El servidor DHCP debe saber en qué interfaz de red local debe escuchar las peticiones de los clientes.

1. **Abre el archivo de configuración por defecto**:

```Bash
sudo nano /etc/default/isc-dhcp-server
```

2. **Localiza la línea `INTERFACESv4` y define la interfaz de red correspondiente (por ejemplo, `eth0` o `enp3s0`)**:

```text
INTERFACESv4="eth0"
```

<div style="page-break-after: always;"></div>

## 3. Configuración del Servidor (`dhcpd.conf`) 🛠️

Edita el archivo principal de parámetros para definir las concesiones de IP, la máscara, la puerta de enlace y los servidores DNS.

1. **Abre el archivo de configuración**:

```Bash
sudo nano /etc/dhcp/dhcpd.conf
```

2. **Añade o adapta la siguiente estructura de red**:

```text
# Declaración de autoridad obligatoria
authoritative;

# Definición de la subred y rango de direcciones IP
subnet 192.168.1.0 netmask 255.255.255.0 {
    range 192.168.1.100 192.168.1.200;

    # Puerta de enlace predeterminada
    option routers 192.168.1.1;

    # Servidores DNS (ej. Cloudflare y Google)
    option domain-name-servers 1.1.1.1, 8.8.8.8;

    # Tiempos de concesión en segundos
    default-lease-time 3600;
    max-lease-time 7200;
}
```

### Parámetros principales de `dhcpd.conf`

| Parámetro | Descripción | Ejemplo |
|-----------|-------------|---------|
| `authoritative` | Indica que este servidor es el oficial de la red y responde a todas las peticiones | `authoritative;` |
| `subnet` / `netmask` | Define la red y la máscara sobre la que el servidor ofrece direcciones | `subnet 192.168.1.0 netmask 255.255.255.0` |
| `range` | Intervalo de direcciones IP que se asignarán dinámicamente | `range 192.168.1.100 192.168.1.200;` |
| `option routers` | Puerta de enlace predeterminada para los clientes | `option routers 192.168.1.1;` |
| `option domain-name-servers` | Servidores DNS que usarán los clientes | `option domain-name-servers 1.1.1.1, 8.8.8.8;` |
| `default-lease-time` | Duración normal de la concesión, en segundos | `default-lease-time 3600;` |
| `max-lease-time` | Duración máxima de la concesión, en segundos | `max-lease-time 7200;` |

---

## 4. Gestión del Servicio 📋

Una vez guardada la configuración, aplica los cambios y comprueba el estado del servicio:

* **Reiniciar el servicio**:

```Bash
sudo systemctl restart isc-dhcp-server
```

* **Verificar el estado**:

```Bash
sudo systemctl status isc-dhcp-server
```
### Comandos útiles para gestionar el servicio

| Comando | Función |
|---------|---------|
| `sudo systemctl start isc-dhcp-server` | Inicia el servicio |
| `sudo systemctl stop isc-dhcp-server` | Detiene el servicio |
| `sudo systemctl restart isc-dhcp-server` | Reinicia el servicio y aplica los cambios de configuración |
| `sudo systemctl status isc-dhcp-server` | Muestra el estado actual del servicio |
| `sudo systemctl enable isc-dhcp-server` | Activa el inicio automático al arrancar el sistema |
| `sudo systemctl disable isc-dhcp-server` | Desactiva el inicio automático |
| `sudo journalctl -u isc-dhcp-server` | Consulta los registros del servicio para depurar errores |

### Verificar que el servidor funciona

Después de reiniciar el servicio, comprueba que todo está correcto siguiendo estos pasos:

1. **Valida la sintaxis del archivo de configuración** (no reinicia nada, solo comprueba errores):

```Bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```
<div style="page-break-after: always;"></div>

2. **Comprueba que el servidor escucha en el puerto DHCP** (UDP 67):

```Bash
sudo ss -lunp | grep :67
```

3. **Observa las peticiones de los clientes en tiempo real**:

```Bash
sudo journalctl -u isc-dhcp-server -f
```

4. **Consulta las concesiones activas**:

```Bash
cat /var/lib/dhcp/dhcpd.leases
```

### Mensajes habituales en el registro

| Mensaje | Significado |
|---------|-------------|
| `DHCPDISCOVER` | Un cliente busca un servidor DHCP |
| `DHCPOFFER` | El servidor ofrece una IP al cliente |
| `DHCPREQUEST` | El cliente solicita la IP ofrecida |
| `DHCPACK` | El servidor confirma la asignación |
| `DHCPNAK` | El servidor rechaza la petición del cliente |
| `no subnet declaration` | La interfaz no tiene una `subnet` que coincida en `dhcpd.conf` |

> **Consejo:** pulsa `Ctrl + C` para salir del modo seguimiento (`-f`) de `journalctl`.
---

> 💡 **[!NOTA]** Si el servicio falla al iniciar, revisa los registros del sistema (`sudo journalctl -u isc-dhcp-server`) para detectar posibles errores de sintaxis en el archivo* `dhcpd.conf`.

## 5. Reservas de Direcciones IP 📌

Una **reserva** permite que un equipo concreto (identificado por su dirección MAC) reciba siempre la misma IP, sin dejar de usar DHCP. Es útil para impresoras, servidores o cámaras IP.

1. **Obtén la dirección MAC del cliente** (en el equipo cliente):

```Bash
ip link show
```
<div style="page-break-after: always;"></div>

2. **Añade la reserva dentro del bloque `subnet`** de `/etc/dhcp/dhcpd.conf`:

```text
host impresora {
    hardware ethernet 08:00:27:ab:cd:ef;
    fixed-address 192.168.1.50;
}
```

3. **Comprueba la sintaxis antes de reiniciar**:

```Bash
sudo dhcpd -t -cf /etc/dhcp/dhcpd.conf
```

4. **Reinicia el servicio**:

```Bash
sudo systemctl restart isc-dhcp-server
```

### Parámetros de una reserva

| Parámetro | Descripción | Ejemplo |
|-----------|-------------|---------|
| `host` | Nombre identificativo de la reserva (sin espacios) | `host impresora` |
| `hardware ethernet` | Dirección MAC del cliente | `hardware ethernet 08:00:27:ab:cd:ef;` |
| `fixed-address` | IP que se asignará siempre a ese cliente | `fixed-address 192.168.1.50;` |

### Verificar las concesiones

Para ver las IP asignadas en este momento:

```Bash
cat /var/lib/dhcp/dhcpd.leases
```

> **Importante:** la IP reservada debe estar **fuera del rango dinámico** (`range 192.168.1.100 192.168.1.200`) para evitar conflictos con otros clientes.

---

## 6. Recomendaciones de seguridad 🔒

Dado que el protocolo DHCP opera sin autenticación nativa y asigna parámetros críticos de red (puerta de enlace y DNS), es fundamental aplicar medidas de protección para evitar ataques de suplantación (*DHCP Spoofing*) o denegación de servicio (*DHCP Starvation*):

1. **Restringir la interfaz de escucha (`INTERFACESv4`):**
   Asegúrate de especificar únicamente la interfaz de red local interna en `/etc/default/isc-dhcp-server` (por ejemplo, `INTERFACESv4="eth0"`). Nunca dejes el servidor escuchando en interfaces expuestas a redes públicas o externas (WAN), ya que respondería a peticiones no autorizadas.

2. **Limitar el acceso al puerto UDP 67 mediante cortafuegos:**
   Configura reglas en el firewall del servidor (`ufw`) para permitir el tráfico hacia el puerto **UDP 67** únicamente desde la subred local definida en `dhcpd.conf` (`192.168.1.0/24`) ejecutando: `sudo ufw allow in on eth0 from 192.168.1.0/24 to any port 67 proto udp`

3. **Proteger los archivos de configuración y concesiones:**
   Verifica que `/etc/dhcp/dhcpd.conf` y `/var/lib/dhcp/dhcpd.leases` solo puedan ser modificados por el usuario `root` o el usuario del servicio `dhcpd`, evitando alteraciones no autorizadas en los rangos de IP o en los servidores DNS entregados a los clientes.

4. **Activar DHCP Snooping en los switches de la red:**
   A nivel de infraestructura, habilita *DHCP Snooping* en los switches gestionables para marcar como "puerto de confianza" (*trusted*) únicamente el puerto donde está conectado este servidor Linux. Esto bloquea servidores DHCP falsos (*rogue DHCP*) conectados por usuarios en otros puertos de la red.

5. **Monitorizar el agotamiento del rango dinámico (`range`):**
   Revisa periódicamente los registros con `sudo journalctl -u isc-dhcp-server` y el archivo `/var/lib/dhcp/dhcpd.leases`. Si observas cientos de peticiones `DHCPDISCOVER` con direcciones MAC distintas en pocos segundos, podrías estar sufriendo un ataque de *DHCP Starvation* destinado a agotar el rango `192.168.1.100 - 192.168.1.200`.

> 💡 **[!NOTA]** El paquete `isc-dhcp-server` fue declarado en fin de vida (*End of Life*) por ISC. Para nuevos despliegues en producción, se recomienda planificar la migración hacia **ISC Kea DHCP** (`kea-dhcp4-server`), su sucesor oficial con mejor soporte de seguridad y APIs modernas.

---

## 7. Recursos Adicionales 💎

* 🔗 [Configuración de un servidor DHCP - Linux](https://www.javiercd.es/posts/servicios/dhcp/instalacion_y_configuracion_de_un_servidor_dhcp_en_linux/dhcp_ipv4/)
* 📖 [Wikipedia: Dynamic Host Configuration Protocol](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)
* 🎦 [YouTube: Instalar y configurar ISC DHCP](https://www.youtube.com/watch?v=tduMWUhAQ0Q)

## Información del proyecto 

Esta documentación se ha elaborado utilizando Markdown. 

El control de versiones se realiza mediante Git 
y el repositorio remoto se encuentra alojado en GitHub. 

## Próximas mejoras

- Añadir una sección de configuración de reservas DHCP por dirección MAC.
- Incluir un apartado de resolución de problemas con los errores más comunes y sus soluciones.
- Ampliar la guía con la configuración de un servidor DHCP en alta disponibilidad (failover).
- Añadir capturas de pantalla de cada paso de la instalación y configuración.
