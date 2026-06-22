# Laboratorio de Diagnóstico de Red y Seguridad (Metodología SOC)

## Descripción del Proyecto
Este proyecto consiste en el despliegue de un entorno virtualizado y controlado (Home Lab) diseñado para realizar auditorías de vulnerabilidades, análisis de tráfico de red y simulación de escenarios de respuesta a incidentes, alineándose con las metodologías utilizadas en un Centro de Operaciones de Seguridad (SOC).

---

## Fase 1: Arquitectura e Infraestructura del Entorno
Para garantizar la seguridad y evitar que el tráfico malicioso afecte a la red local o externa, todo el entorno se ha configurado de manera aislada dentro de un hipervisor.

### Detalle del Hardware y Software Base:
* **Sistema Anfitrión (Host):** Windows (32 GB RAM)
* **Hipervisor:** VMware Workstation Pro
* **Configuración de Red:** Modo **NAT** (Red virtual privada y aislada)

### Inventario de Máquinas Virtuales y Direccionamiento:
| Máquina | Rol / Función | Sistema Operativo | Dirección IP |
| :--- | :--- | :--- | :--- |
| **Máquina Atacante** | Simulación de adversario (Red Team) | Kali Linux | `192.168.118.128` |
| **Máquina Víctima** | Auditoría y monitoreo (Blue Team) | Metasploitable 2 | `192.168.118.129` |

### Validación de Conectividad (Ping Test)
Se verificó la comunicación interna entre ambos sistemas ejecutando un test de ICMP desde la máquina atacante hacia la víctima, obteniendo un **0% de pérdida de paquetes**, lo que confirma el correcto funcionamiento del canal de red aislado.

---

## ⚔️ Fase 2: Reconocimiento Automatizado y Análisis de Superficie de Exposición

Para identificar de manera eficiente los puntos de entrada potenciales en la máquina objetivo, se desarrolló un script de automatización en Bash (`escaneo.sh`) encargado de ejecutar análisis iterativos utilizando **Nmap**.

### 🛠️ Automatización con Bash Scripting
El script realiza un descubrimiento veloz de puertos abiertos y, de forma inmediata, ejecuta categorías de scripts de seguridad de Nmap sobre los puertos activos para mapear versiones y configuraciones inseguras de manera precisa.

### 📊 Matriz de Vulnerabilidades Identificadas

| Puerto / Protocolo | Servicio Detectado | Versión de Software | Nivel de Riesgo | Impacto Potencial / Hallazgo |
| :--- | :--- | :--- | :--- | :--- |
| **21 / TCP** | FTP | `vsftpd 2.3.4` |  **Crítico** | Acceso total al sistema mediante un *Backdoor* conocido en esta versión de software. Se permite el inicio de sesión anónimo (`Anonymous login`). |
| **22 / TCP** | SSH | `OpenSSH 4.7p1 Debian` |  Medio | Potencial enumeración de usuarios del sistema y ataques de fuerza bruta dirigidos. |
| **23 / TCP** | Telnet | `Linux telnetd` |  Alto | Interceptación de tráfico de red. Al no poseer cifrado nativo, las credenciales viajan expuestas en texto plano. |
| **25 / TCP** | SMTP | `Postfix smtpd` |  Medio | Enumeración remota de casillas de correo internas y usuarios válidos del sistema operativo. |
| **445 / TCP** | SMB | `Samba smbd 3.0.20-Debian` |  **Crítico** | Ejecución Remota de Código (RCE) sin autenticación previa (CVE-2007-2447). El firmado de mensajes de seguridad está deshabilitado. |
| **3306 / TCP** | MySQL | `MySQL 5.0.51a-3ubuntu5` |  Alto | Exposición directa del motor de base de datos, propenso a filtraciones o inyecciones de comandos estructurados. |
| **8180 / TCP** | HTTP | `Apache Tomcat/Coyote 1.1` |  Alto | Acceso a paneles de gestión expuestos, vulnerable al despliegue de artefactos maliciosos (`.war`) mediante credenciales por defecto. |

### Diagnóstico Técnico de Hallazgos Críticos

1. **Vulnerabilidad de Cadena de Suministro en vsFTPd 2.3.4:** El servidor de archivos FTP contiene una puerta trasera histórica. Si un atacante intenta autenticarse utilizando una secuencia específica de caracteres en el nombre de usuario, el sistema abre de manera automática una consola interactiva con privilegios máximos en el puerto 6200.
2. **Ejecución de Comandos Remotos en Samba 3.0.20:** Esta implementación de SMB cuenta con una falla crítica en la interpretación de los scripts de mapeo de nombres de usuario. Permite inyectar comandos de Linux directamente en los paquetes de red, logrando control total sobre el servidor víctima de manera inmediata.