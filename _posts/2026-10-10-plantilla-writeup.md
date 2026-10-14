---
title: "Plantilla de writeup (bórrame y úsame de base)"
description: Estructura sugerida para publicar writeups de máquinas en este blog.
date: 2026-10-10 22:00:00 -0300
categories: [Writeups, Plantilla]
tags: [plantilla, htb, linux]
pin: false
math: false
mermaid: false
---

> Este post es solo una **plantilla**. Copia este archivo, cámbiale el nombre a `AAAA-MM-DD-nombre-de-la-maquina.md` dentro de `_posts/` y reemplaza el contenido.
{: .prompt-tip }

## Resumen

Una o dos frases con la cadena de ataque completa, por ejemplo:

`Enumeración web → credenciales expuestas → RCE → escalada de privilegios`

| Dato | Valor |
| --- | --- |
| Plataforma | HTB / THM / otra |
| Sistema operativo | Linux / Windows |
| Dificultad | Easy / Medium / Hard |

## Reconocimiento

Explica qué escaneaste y **por qué**. Los comandos van en bloques de código:

```bash
nmap -p- --min-rate 5000 -sS -n -Pn <IP> -oG allPorts
```

Describe qué encontraste: puertos, servicios, versiones y qué te llamó la atención.

## Enumeración

Cuenta el razonamiento. Los lectores aprenden más de *por qué* probaste algo que del comando en sí.

## Acceso inicial

Cómo conseguiste entrar y qué fallos permitieron el acceso.

## Escalada de privilegios

Qué enumeraste en el sistema y cuál fue la vía hacia el máximo privilegio.

## Lecciones aprendidas y mitigación

- Qué error de configuración o de código lo hizo posible.
- Cómo se corregiría o detectaría (esto es lo que aporta valor desde el lado Red Team).

## Notas de formato de Chirpy

Para destacar avisos puedes usar:

> Información útil para el lector.
{: .prompt-info }

> Algo que conviene tener en cuenta.
{: .prompt-warning }

Para una imagen, guárdala en `assets/img/` y refiérela así: `![texto alternativo](/assets/img/captura.png)`.
