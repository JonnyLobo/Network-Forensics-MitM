# Network-Forensics-MitM
"Laboratorio práctico: Auditoría de red (Capa 2), ejecución controlada de ataque Man-in-the-Middle (ARP Spoofing) y análisis forense de paquetes con Wireshark."
# 🛡️ Análisis Forense y Auditoría de Capa 2: Interceptación MitM (ARP Spoofing)

## 📌 Resumen Ejecutivo
Este proyecto documenta la ejecución, análisis y detección forense de un ataque *Man-in-the-Middle* (MitM) mediante el envenenamiento del protocolo ARP (ARP Spoofing) en un entorno de red local (LAN). El objetivo principal de este laboratorio es demostrar competencias prácticas en **arquitectura de redes, monitoreo de tráfico, análisis de paquetes y estrategias de mitigación (Blue Team)**.

El laboratorio fue ejecutado de forma ética en una infraestructura virtualizada y aislada (Sandboxing) mediante VMware (NAT/Host-Only).

## 🛠️ Entorno de Laboratorio y Herramientas
* **Hipervisor:** Entorno corporativo aislado (VMnet8 NAT - Rango IP `192.168.199.0/24`).
* **Máquina Atacante (Pretesting):** Kali Linux (IP: `192.168.199.133`)
* **Máquina Víctima (Endpoint):** Windows 10/11 (IP: `192.168.199.137`)
* **Enrutador/Gateway Virtual:** (IP: `192.168.199.2`)
* **Herramientas Utilizadas:** `arpspoof` (Suite dsniff), `sysctl` (IP Forwarding), `Wireshark` (Análisis Forense de Red), `cmd/arp` (Auditoría de endpoints).

---

## 🚀 Fases de la Auditoría

### Fase 1: Reconocimiento y Estado Base (Baseline)
Antes de la alteración de la red, se auditó la tabla de memoria ARP del endpoint víctima para establecer la línea base. 

**Evidencia A: Tabla ARP Legítima**
*La IP del enrutador (`.2`) está correctamente mapeada a su dirección MAC física original (`00-50-56-ed-5f-38`). La red opera con normalidad.*
![Evidencia A: Tabla ARP Base](PON_AQUI_LA_IMAGEN_DE_LA_TABLA_LIMPIA.png)

### Fase 2: Ejecución Táctica (Rompiendo la Capa 2)
Se habilitó el enrutamiento a nivel de kernel en el sistema atacante (`net.ipv4.ip_forward=1`) para actuar como un puente invisible. Posteriormente, se inyectaron paquetes ARP falsificados de forma bidireccional.

**Evidencia B: Detección de Anomalía Matemática (El Compromiso)**
*Al re-auditar el endpoint, se detecta el vector de ataque: La IP del Enrutador (`.2`) y la IP del Atacante (`.133`) apuntan a la **misma dirección MAC física**. Esto es criptográfica y físicamente imposible en una red no comprometida.*
![Evidencia B: Tabla ARP Envenenada](PON_AQUI_LA_IMAGEN_DE_LA_TABLA_ENVENENADA.png)

### Fase 3: La Perspectiva de la Víctima (El Cebo)
Con el flujo de tráfico (Norte-Sur) desviado a través de la máquina atacante, se observó la interacción del usuario final.

**Evidencia C: Portal de Autenticación Vulnerable**
*El usuario en el endpoint Windows interactúa con un portal bancario de pruebas e ingresa sus credenciales de autenticación, asumiendo erróneamente que la red local (LAN) no está comprometida y obviando la falta de cifrado SSL/TLS.*
![Evidencia C: Perspectiva de la Víctima](PON_AQUI_LA_IMAGEN_DEL_LOGIN_DE_ALTORO.png)

### Fase 4: Análisis Forense y Extracción (Wireshark)
Se utilizó Wireshark en la máquina atacante para realizar un monitoreo pasivo de los paquetes interceptados.

**Evidencia D: Aislamiento del Tráfico y Filtrado**
*Mediante la aplicación de filtros lógicos estrictos (`ip.src == 192.168.199.137 and http`), se aisló el ruido de la red para identificar exactamente el paquete HTTP POST que contiene el envío del formulario.*
![Evidencia D: Filtro Wireshark](PON_AQUI_LA_IMAGEN_DEL_FILTRO_VERDE.png)

**Evidencia E: Flujo HTTP (Extracción de Credenciales en Texto Plano)**
*Al inspeccionar el flujo TCP (Follow HTTP Stream) del paquete POST identificado, se evidencia la captura exitosa de las credenciales de autenticación enviadas en texto plano.*
![Evidencia E: Credenciales Extraidas](PON_AQUI_LA_IMAGEN_DEL_TEXTO_ROJO_CON_LA_CLAVE.png)

---

## 🛡️ Conclusiones y Remediación (Perspectiva SOC / Blue Team)

Este incidente de laboratorio resalta dos vulnerabilidades críticas en la arquitectura de sistemas clásicos y sus respectivas mitigaciones:

1. **La vulnerabilidad inherente de ARP (Stateless):** El protocolo ARP carece de mecanismos de autenticación por diseño, lo que permite la suplantación.
   * **Mitigación (Redes Corporativas):** Implementar **Dynamic ARP Inspection (DAI)** a nivel de Switches (Capa 2) para validar que los paquetes ARP coincidan con las asignaciones DHCP legítimas, descartando respuestas maliciosas.
   * **Mitigación (Servidores Críticos):** Configurar entradas ARP estáticas a nivel del sistema operativo.

2. **Transmisión de datos en texto plano:** La interceptación demostró el riesgo catastrófico de utilizar protocolos sin cifrar (HTTP).
   * **Mitigación:** Obligar la implementación del protocolo **HTTPS (TLS/SSL)**, el cual cifra la carga útil (Payload) en la Capa de Aplicación. Bajo un entorno TLS correctamente configurado, la interceptación MitM solo obtendría datos ilegibles.

---
*Laboratorio documentado por Jonny Lobo - Analista SOC / Infraestructura O&M.*
