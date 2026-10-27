# Guía Rápida: Configuración de un Servidor DHCP en Linux (ISC DHCP)

![Configuracion DHCP Linux](./configure-dhcp-linux.png)

*Imagen: Como configurar DHCP (Fuente: [linuxconfig.org](https://linuxconfig.org/what-is-dhcp-and-how-to-configure-dhcp-server-in-linux))*

### Esta guía explica cómo **instalar y configurar** un servidor DHCP básico utilizando el paquete `isc-dhcp-server` en distribuciones basadas en Debian/Ubuntu.

---

## Índice

* [1. Instalación del Paquete](#1-instalación-del-paquete-️)
* [2. Configuración de la Interfaz de Red](#2-configuración-de-la-interfaz-de-red-️)
* [3. Configuración del Servidor (`dhcpd.conf`)](#3-configuración-del-servidor-dhcpdconf-️)
* [4. Gestión del Servicio](#4-gestión-del-servicio-)
* [5. Recursos Adicionales](#5-recursos-adicionales-)


<div style="page-break-after: always;"></div>

## 1. Instalación del Paquete ⚙️

Actualiza los repositorios e instala el servicio DHCP ejecutando el siguiente comando en la terminal:

```
Bash

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

```
Bash

sudo nano /etc/default/isc-dhcp-server
```

2. **Localiza la línea `INTERFACESv4` y define la interfaz de red correspondiente (por ejemplo, `eth0` o `enp3s0`)**:

```
Plaintext

INTERFACESv4="eth0"
```

<div style="page-break-after: always;"></div>

## 3. Configuración del Servidor (`dhcpd.conf`) 🛠️

Edita el archivo principal de parámetros para definir las concesiones de IP, la máscara, la puerta de enlace y los servidores DNS.

1. **Abre el archivo de configuración**:

```
Bash

sudo nano /etc/dhcp/dhcpd.conf
```

2. **Añade o adapta la siguiente estructura de red**:

```
Plaintext

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

```
Bash

sudo systemctl restart isc-dhcp-server
```

* **Verificar el estado**:

```
Bash

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
---

💡 *[!NOTA] Si el servicio falla al iniciar, revisa los registros del sistema* (`sudo journalctl -u isc-dhcp-server`) *para detectar posibles errores de sintaxis en el archivo* `dhcpd.conf`.

<div style="page-break-after: always;"></div>

## 5. Recursos Adicionales 💎

* 🔗 [Configuración de un servidor DHCP - Linux](https://www.javiercd.es/posts/servicios/dhcp/instalacion_y_configuracion_de_un_servidor_dhcp_en_linux/dhcp_ipv4/)
* 📖 [Wikipedia: Dynamic Host Configuration Protocol](https://en.wikipedia.org/wiki/Dynamic_Host_Configuration_Protocol)
* 🎦 [YouTube: Instalar y configurar ISC DHCP](https://www.youtube.com/watch?v=tduMWUhAQ0Q)
