# Pocketbase https://pocketbase.io/
Pocketbase es un BaaS (Backend as a Service) escrito en GO que genera un ejecutable y trabaja con una base de datos SQLite que genera un archivo en local por lo que su despliegue en un VPS es muy fácil. La documentación del proyecto es excelente y, aunque aún está en desarrollo y no tiene versión 1.0, se puede utilizar para proyectos propios y/o sencillos en producción. También hay opciones gratuitas (en principio) y muy sencillas (2 minutos) de host en la nube https://pockethost.io/ pero a mi siempre me gusta tener el control, tanto de coste como de espacio.

Aquí aprenderás como instalar el ejecutable Pocketbase en un VPS y mantenerlo en ejecución como un servicio propio del sistema, por lo que si tu VPS se reinicia por el motivo que sea el servicio se levantará automáticamente.

## Actualiza paquetes en tu VPS:
```
sudo apt update && sudo apt upgrade -y
```

## Instala wget y unzip (si no los tienes):
```
sudo apt install wget unzip
```

## Descarga y ejecuta Pocketbase en tu VPS:
Ve a https://pocketbase.io/docs/ y copia el enlace de descarga de tu sistema operativo, en nuestro caso Linux. Después en tu VPS utiliza:
```
sudo wget enlace-de-descarga
```

Se habrá descargado un .zip con el programa. Ahora crea un directorio (para mantener las cosas organizadas) y descomprime el contenido allí con unzip:
```
mkdir pocketbased
sudo unzip nombre-del-archivo-de-pocketbase.zip -d ./pocketbased
cd pocketbased
```

Una vez en el directorio verás que hay 3 archivos, entre ellos el ejecutable de pocketbase. Desde este directorio es desde donde se ejecuta el programa con el comando:
```
./pocketbase serve
```

Este comando lo que hace es levantar un servidor web con el proyecto. Si no indicas nada después de " serve " se levanta por defecto en el localhost:8090 y también se puede indicar un dominio, como haremos al final de la guía. Comprueba que funciona:

![image](https://github.com/user-attachments/assets/12b2bbea-73e6-4b35-b72d-18f7c5969c25)

En este punto ya podrías entrar a a localhost:8090/_/ , crear tu admin user y contraseña, y empezar a usar esta maravilla de software.

## Configurando como servicio del sistema:
Una vez comprobado que funciona el programa, corta la ejecución en terminal con Ctrl+C. Lo ideal ahora sería ir a nuestro proveedor de DNS y crear un record A que apunte a nuestro servidor con nuestro dominio/subdominio. Si lo haces con Cloudflare tendrás automáticamente conexión https con tu servicio (recuerda que pocketbase levanta un servidor web al ejecutarse).

Después navega a /lib/systemd/system y crea un archivo de configuración con nano llamado pocketbase.service. Dentro indica lo siguiente:
```
[Unit]
Description = pocketbase

[Service]
Type           = simple
User           = root
Group          = root
LimitNOFILE    = 4096
Restart        = always
RestartSec     = 5s
StandardOutput = append:/pocketbased/output.log   #poner la ruta que toque y borrar este comentario
StandardError  = append:/pocketbased/errors.log   #poner la ruta que toque y borrar este comentario
ExecStart      = /pocketbased/pocketbase serve MIDOMINIO.com   #poner la ruta y el dominio/subdominio que toque y borrar este comentario

[Install]
WantedBy = multi-user.target
```

Como hemos modificado los archivos del sistema, tenemos que reiniciar el daemon:
```
sudo systemctl daemon-reload
```
y levantar el servicio:
```
sudo systemctl start pocketbase.service
```

Y ya tenemos listo el servicio en segundo plano, configurado para que se reinicie cada vez que se encienda el VPS.
