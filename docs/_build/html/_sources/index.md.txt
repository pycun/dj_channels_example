# Tutorial para Chat usando Django Channels

## Requerimientos

```
El siguiente ejemplo fue realizado en un Servidor con Ubuntu 20.04
```

Se usaron los siguientes paquetes

- python 3.8
```
Para este ejemplo no fue necesario la instalación ya que viene instalado con Ubuntu 20.04
```
- virtualenv
```
En caso de no estar instalado, podemos instalarlo con:

    sudo apt install python3-virtualenv
```
- git
```
Para este ejemplo no fue necesario la instalación ya que viene instalado con Ubuntu 20.04
```
- redis-server 5.0.7
```
En caso de no estar instalado, podemos instalarlo con:

    sudo apt install redis-server
```
- supervisor
```
En caso de no estar instalado, podemos instalarlo con:

    sudo apt install supervisor
```

## Instalación

**Paso 1:** Crear carpeta donde estará nuestro entorno y proyecto
```
    cd ~
    mkdir github
```
**Paso 2:** Bajar proyecto
```
    cd ~/github
    git clone https://github.com/luissalgado9/dj_channels_example.git
```

**Paso 3:** Crear entorno con Python 3.8 y activar entorno
```
    cd ~/github
    virtualenv env_dj_channels_example -p3.8
    source env_dj_channels_example/bin/activate
```
**Paso 4:** Instalar requirements del proyecto
```
    cd ~/github/dj_channels_example
    pip install -r requirements
```
**Paso 5:** Aplicar migraciones
```
    cd ~/github/dj_channels_example
    python manage.py migrate
```
**Paso 6:** Crear logs para Asgi
```
    cd ~/github/dj_channels_example
    mkdir .logs
    cd .logs
    touch asgi.log
```
**Paso 7:** Configurar Supervisor
```
    cd /etc/supervisor/conf.d
    sudo nano dj_channels.conf
```
Pegamos el siguiente contenido
```
; ==================================
;  celery ASGI
; ==================================

[fcgi-program:asgi]
# TCP socket used by Nginx backend upstream
socket=tcp://0.0.0.0:8000

# Directory where your site's project files are located
directory=/home/ubuntu/github/dj_channels_example/

# Each process needs to have a separate socket file, so we use process_num
# Make sure to update "mysite.asgi" to match your project name
command=/home/ubuntu/github/env_dj_channels_example/bin/daphne -u /run/daphne/daphne%(process_num)d.sock --endpoint fd:fileno=0 --access-log - --proxy-headers dj_channels_example.asgi:application

# Number of processes to startup, roughly the number of CPUs you have
numprocs=4

# Give each process a unique name so they can be told apart
# process_name=asgid
process_name=asgi%(process_num)d

# Automatically start and recover processes
autostart=true
autorestart=true

# Choose where you want your log to go
stdout_logfile=/home/ubuntu/github/dj_channels_example/.logs/asgi.log
redirect_stderr=true
```

**Paso 8:** Crear archivo de Configuración de Daphne
```
    cd ~
    sudo mkdir /run/daphne/
    sudo chmod -R 777 /run/daphne/
    touch /usr/lib/tmpfiles.d/daphne.conf
    sudo nano /usr/lib/tmpfiles.d/daphne.conf
```
Pegamos lo siguiente
```
d /run/daphne 0755
```

**Paso 7:** Actualizar supervisor
```
    sudo supervisorctl reread
    sudo supervisorctl update
```