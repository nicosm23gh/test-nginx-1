# test-nginx

¿Qué pasa si no hago el link simbólico entre sites-available y sites-enabled de mi sitio web?

Si no creas el enlace simbólico entre sites-available y sites-enabled, Nginx no va a reconocer el sitio y no lo servirá. Es como si configuraras un sitio web, pero no le dieras la llave para que Nginx lo pueda leer. Lo único que habrías hecho sería crear el archivo de configuración, pero Nginx no lo cargaría, por lo que el sitio no estaría disponible para los usuarios.

La solución es simple: crear el enlace simbólico, lo cual permite que Nginx "activa" tu sitio. Sin este enlace, aunque lo hayas configurado bien, el servidor web no lo podrá servir.

¿Qué pasa si no le doy los permisos adecuados a /var/www/nombre_web?

Si no le das los permisos correctos a los archivos de tu sitio web, básicamente estarás bloqueando el acceso de Nginx a esos archivos. Nginx necesita permisos de lectura (al menos) sobre los archivos del sitio para poder servirlos cuando un usuario los pida. Si no tiene esos permisos, se pueden dar errores como 403 (Forbidden), que es el servidor diciendo que no puede acceder a los archivos, aunque los haya encontrado.

Además, si los permisos no son los adecuados, podrías estar poniéndole en riesgo la seguridad del servidor, permitiendo que otros usuarios accedan o modifiquen los archivos del sitio. Por eso es importante dar los permisos adecuados, donde el propietario del directorio debe ser www-data, que es el usuario con el que Nginx trabaja.

Resumen de la práctica:
En esta práctica hemos configurado un servidor web con Nginx en una máquina virtual Debian a través de Vagrant. Nuestro objetivo era alojar dos sitios web, uno utilizando Git para descargar los archivos y el otro configurado para recibir archivos mediante FTPES (FTP seguro).

Configuración de Nginx:

Clonamos los sitios web desde GitHub y los sincronizamos en las carpetas /var/www/site1 y /var/www/site2.
Configuramos Nginx para que sirviera ambos sitios, asegurándonos de crear los archivos de configuración necesarios en /etc/nginx/sites-available/ y habilitándolos con enlaces simbólicos en /etc/nginx/sites-enabled/. Sin este paso, Nginx no serviría los sitios.
Configuración de FTPES:

Instalamos y configuramos vsftpd para habilitar FTPES en el puerto 21, permitiendo que los archivos de site2 se transfirieran de forma segura a través de FTP.
Creamos un usuario FTP y le otorgamos los permisos adecuados para que solo pueda acceder a los archivos de site2.
Problemas encontrados:
Uno de los principales problemas fue olvidarnos de crear el enlace simbólico entre sites-available y sites-enabled, lo cual hizo que uno de los sitios no estuviera disponible. También, al principio no configuramos correctamente los permisos de los archivos de los sitios, lo que causó que Nginx no pudiera acceder a ellos y aparecieran errores 403. Una vez corregidos estos detalles, todo funcionó correctamente.

En resumen, la práctica nos enseñó cómo configurar un servidor web completo con Nginx, cómo manejar los permisos y cómo habilitar FTP seguro para la transferencia de archivos.