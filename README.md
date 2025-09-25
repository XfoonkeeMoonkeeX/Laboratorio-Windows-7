    
# Laboratorio de Pentesting Metodológico sobre Windows 7 SP1

![Windows 7 Logo](https://i.imgur.com/gQYyC8p.png)

Este repositorio documenta el proceso completo de un pentest realizado a una máquina virtual Windows 7 Professional SP1 (64-bit), desde el reconocimiento inicial hasta la obtención de privilegios de `SYSTEM`.

**Disclaimer:** *Todas las actividades documentadas aquí se realizaron en un entorno de laboratorio controlado y aislado, con máquinas de mi propiedad. Este proyecto tiene fines exclusivamente educativos para estudiar y comprender las fases de un ciberataque y las vulnerabilidades de sistemas sin parches. No intentes replicar estas acciones en sistemas que no te pertenezcan o para los cuales no tengas permiso explícito.*

---

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

  

Resultado: Se identificó al host 192.168.1.13 como nuestro objetivo, confirmado por su dirección MAC asociada a VirtualBox (PCS Systemtechnik GmbH).
