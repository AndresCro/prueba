#Instalación de BackupPC
##Paso 1
Descargar el paquete backuppc.
apt-get -y install backuppc

Durante la descarga pedirá el tipo de servidor web, en este caso Apache.

Se puede dejar la contraseña o se puede cambiar con
htpasswd /etc/backuppc/htpasswd backuppc

Configuración de BackupPC.
BackupPC necesita acceso sin contraseña a un usuario del servidor cliente donde se requiere sacar los respaldos mediante una conexión ssh.

Creación de las llaves.
su - backuppc 
ssh-keygen (se da enter en las tres opciones)

Mediante la herramienta Rsync va a lograr realizar el respaldo, por lo tanto necesita ser habilitada en cliente como en la configuración.

Es necesario indicar como va a ingresar BackupPC, en este caso mediante el usuario respaldos y con la herramienta Rsync.

Editor /etc/backuppc/config.pl

Se agregan las siguientes líneas:
$Conf{RsyncClientCmd} = '$sshPath -q -x -l respaldos $host $rsyncPath $argList+';
$Conf{RsyncClientRestoreCmd} = '$sshPath -q -x -l respaldos $host $rsyncPath $argList+';

Y se comentan estas otras (para no perder la estructura).

$Conf{RsyncClientCmd} = '$sshPath -q -x -l root $host $rsyncPath $argList+';
$Conf{RsyncClientRestoreCmd} = '$sshPath -q -x -l root $host $rsyncPath $argList+';

Se reinicia el servicio backuppc.
service backuppc restart

###Del lado del cliente.
Se crea el usuario respaldos.
	useradd -m respaldos (puede ser cualquier nombre)
passwd respaldos

BackupPC necesita acceso directo, por lo tanto copiará la llave en respaldos para ingresar de forma automática sin contraseñas.
ssh-copy-id respaldos@ip
ssh respaldos@ip

Instalación de rsync del cliente
apt-get -y install rsync

Configuración del usuario respaldos del cliente para rsync
visudo
	respaldos ALL=NOPASSWD: /usr/bin/rsync
	
##Paso 2
Entrar en un navegador a
	IP del servidor/backuppc 
	storage.proyecto-becarios-cert-2019.ml/backuppc
	
Ingresar con el usuario backuppc y la contraseña de la preconfiguración o modificada por htpasswd

Edit hosts -> hosts
	Borrar el host local
	Añadir los hosts y usuarios (host | usuario)
	directory.proyecto-becarios-cert-2019.ml | respaldos
		database.proyecto-becarios-cert-2019.ml | respaldos
dns.proyecto-becarios-cert-2019.ml | respaldos 

Edit hosts -> Xfer
	Se especifica el método rsync
 
Edit hosts -> Schedule
	Se agrega a dos respaldos como mínimo en
	FullKeepCnt 2
 
Edit hosts -> Server
	Se modifica el WakeupSchedule a cada media hora agregando después de cada número, el medio.
	1, 1.5, 2, 2.5, 3, 3.5 , …, 23.5
 
Hosts -> Select a host
	Se selecciona el host a realizar el primer backup.
	
Edit Config (del host seleccionado) -> Xfer
	RsyncShareName 
		Insert /la_ruta_a_respaldear
		Insert /home/respaldos
	Include/Exclude
		Cambiar el usuario por el usuario con el que está en el servidor cliente.
 
Iniciar con Start Full Backup y con Browse Backups se visualiza los archivos copiados.

En el servidor dentro de en el directorio /var/lib/backuppc/(host)/ se hace un bind -o mount hacia /srv/backup/(host).

Instalar certificados SSL

Encender el módulo SSL de apache
	a2enmod ssl
	
Cambiar las líneas SSL por default de apache por los certificados de Let’s Encrypt

Editor  /etc/apache2/sites-available/default-ssl.conf 
	SSLCertificateFile /srv/ssl/cert.pem
 	SSLCertificateKeyFile /srv/ssl/privkey.pem
	SSLCertificateChainFile /srv/ssl/chain.pem
	SSLCACertificatePath /srv/ssl/
	SSLCACertificateFile /srv/ssl/fullchain.pem
			
Poner la página SSL de apache
		a2ensite default-ssl
		
Reiniciar Apache
		service apache2 reload
		
Redirigir el tráfico http a https

Configurar el archivo de la página http por default de apache

Editor  /etc/apache2/sites-available/000-default.conf 

	ServerName storage.proyecto-becarios-cert-2019.ml
	<Location />
	Redirect permanent / https://storage.proyecto-becarios-cert-2019.ml/
	</Location>
	
Reiniciar Apache
	service apache2 reload
