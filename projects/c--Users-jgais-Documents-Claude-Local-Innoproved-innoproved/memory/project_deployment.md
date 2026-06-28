---
name: project-deployment
description: Deployment via SSH auf Hostpoint — SSH-Alias und Zielpfad
metadata:
  type: project
---

Deployment erfolgt via SSH auf Hostpoint.

**SSH-Alias:** `hostpoint` (konfiguriert in `~/.ssh/config`, kein Passwort/Key nötig)
**Zielpfad auf Server:** `~/www/innoproved.ch/`
**Befehl:** `scp <datei> hostpoint:~/www/innoproved.ch/<zielpfad>`

**Why:** Der Alias `hostpoint` ist in der lokalen SSH-Config hinterlegt und enthält Host, User und Key. Kein Passwort notwendig.

**How to apply:** Bei jedem Deployment scp oder rsync mit `hostpoint:~/www/innoproved.ch/` als Ziel verwenden. Unterverzeichnisse (js/, css/pages/, admin/) entsprechend angeben.
