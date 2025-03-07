## Administración de Redes Linux

# Entregable 3: Gestión de Usuarios

**`[GITT]` `[Sem 3.2]` `[LNA]` `(Spring 2025)`**

**Ingeniería en Tecnologías de Telecomunicación** | *Universidad Carlos III de Madrid*

---

## Introducción

Este documento contiene el registro del desarrollo de la actividad, incluyendo
las instrucciones principales, las decisiones, y los resultados.

## Ejercicio 1. Configuración de claves SSH

En mi caso, este ejercicio es innecesario, ya que la autenticación por SSH a
los laboratorios de telemática ya está configurada de forma extensa. Sin
embargo, por completitud, se detallará el proceso de configuración.

### 1.1. Generación de la clave

El primer paso es la generación de claves. Se puede hacer de forma interactiva
o con un solo comand. Para este ejemplo, se hará de forma interactiva, con el
archivo por defecto y la *passphrase* '1234' (aunque no se vea). El proceso es
el siguiente:

```txt
$ ssh-keygen
Generating public/private rsa key pair.
Enter file in which to save the key (/home/alonso_uni/.ssh/id_rsa):
Enter passphrase (empty for no passphrase): 
Enter same passphrase again:
Your identification has been saved in /home/alonso_uni/.ssh/id_rsa
Your public key has been saved in /home/alonso_uni/.ssh/id_rsa.pub
The key fingerprint is:
SHA256:Rt2hjeqtA9jVxKGIECadG6pL/PRsACZynvtaZWoN8+Q alonso_uni@alonso-deb
The key's randomart image is:
+---[RSA 3072]----+
|..+o     ....    |
| o+. . . oo= .   |
| . o. . oo+ o    |
|+oo    ....      |
|*o..oo+.S        |
|.ooo.@oo .       |
|..o.B E.. .      |
|. .+ +  ..       |
|  .oo   ..       |
+----[SHA256]-----+
```

Se puede realizar la misma operación con un solo comando:

```sh
ssh-keygen -t rsa -f /home/alonso_uni/.ssh/id_rsa -N '1234'
```

La siguiente captura de pantalla ilustra el proceso:

![Generación de claves SSH](img/1.1-ssh-keygen.png)

### 1.2. Distribución de clave pública

Para que la clave generada nos sea útil, hay que añadir la clave pública a la
lista de claves en las que confía el destino o *host* SSH. En este ejercicio,
será nuestro usuario del laboratorio de Telemática.

Para lograr esto se debe añadir la clave pública al archivo
`~/.ssh/authorized_keys` como una única línea.

#### 1.2.a. Copia y pega

Hay varias maneras de hacerlo. La primera y más sencilla para un usuario común
es copiar la *fingerprint* indicada durante la generación de la clave al
portapapeles del sistema (si lo tiene) y pegarla en el archivo destino,
accediendo a la máquina por cualquier método. Por ejemplo, accendiendo con ssh, usando autenticación con usuario y contraseña.

```sh
ssh <usuario>@<host>
```

El servidor nos pedirá la contraseña de nuestro usuario. Al introducirla,
accederemos a una consola en el host desde el terminal de nuestra máquina. Con
el siguiente comando podemos añadir la clave privada al archivo requerido:

```sh
echo "<clave copiada>" >> ~/.ssh/authorized_keys
```

Donde `<clave copiada>` es el texto copiado previamente, pegado con el
portapapeles de nuestro sistema.

#### 1.2.b. `scp`

Otra alternativa es copiar la clave pública al servidor usando `scp`,
conectarse usando `ssh` u otro método de acceso, y por último añadir el archivo
copiado a `~/.ssh/authorized_keys`. La ventaja de este método es que no
requiere un portapapeles.

Para copiar el archivo al servidor:

```sh
scp ~/.ssh/id_rsa.pub <usuario>@<host>:~/.ssh/authorized.pub
```

Deberemos autenticarnos con la contraseña de nuestro usuario.

Después, debemos acceder al servidor por cualquier método, llegando a un
terminal. Usando `ssh` y autenticándonos usando la contraseña, igual que
antes:

```sh
ssh <usuario>@<host>
```

Una vez hemos accedido a una consola en el destino, podemos usar

```sh
cat ~/.ssh/authorized.pub >> ~/.ssh/authorized_keys
rm ~/.ssh/authorized.pub
```

#### 1.2.c. Redirección

Este método es el más directo, pero también es de un nivel de conocimiento más
alto. Con una sola línea se puede lograr el objetivo:

```sh
ssh <usuario>@<host> "cat >> ~/.ssh/authorized_keys" < ~/.ssh/id_rsa.pub
```

Deberemos autenticarnos con la contraseña del usuario.

La siguiente captura de pantalla ilustra el proceso:

![Distribución de clave pública](img/1.2-ssh-copy.png)

Al ser la primera vez que esta máquina virtual se conecta a la máquina remota,
se avisa de que no se puede verificar la autenticidad de la máquina. Esto es
normal, y se puede aceptar escribiendo `yes`. Para mayor seguridad, se puede
comprobar la *fingerprint* de la máquina remota con la que se está conectando
usando métodos que no se explicarán aquí.

En mi caso, no tuve que introducir la contraseña de mi usuario, ya que la
autenticación por clave pública ya estaba configurada.

#### 1.2.d. `ssh-copy-id`

El script `ssh-copy-id` puede realizar esta función. No se documentará aquí.
Vea `ssh-copy-id(1)` para más información.

### 1.3. Configuración de SSH

Con los nombres de archivo por defecto y la configuración por defecto, la
autenticación con claves debería funcionar. Sin embargo, hay muchas
personalizaciones que pueden ser útiles.

#### Agente SSH

El agente `ssh-agent` se encarga de gestionar las claves privadas para evitar
tener que introducir la *passphrase* cada vez que se quiere usar. Vea
`ssh-agent(1)`.

#### Archivo de configuración

##### `PubKeyAuthentication`

Esta opción está habilitada por defecto. Sirve para habilitar explícitamente o
deshabilitar la autenticación con estas claves:

```ssh-config
PubKeyAuthentication yes
```

##### `AddKeysToAgent`

Para utilizar el agente SSH de forma más cómoda, es recomendable configurar el
cliente SSH para añadir las claves al agente de forma automática cuando se
utilicen:

```ssh-config
AddKeysToAgent yes
```

##### `Host`s

Se pueden definir nombres asociados a configuraciones específicas. Por ejemplo,
para acceder a una variedad de máquinas del lab de telemática utilizando
nombres acortados, puede ser útil el siguiente ejemplo:

```ssh-config
Host vit* doc* jbit* it* monitor01 monitor02 monitor03 v v?
    # El código `%h` se sustituye por el nombre utilizado, por lo que si
    # ejecutas `ssh vit100` se traducirá a `vit100.lab.it.uc3m.es`
    HostName %h.lab.it.uc3m.es
    # Evita tener que especificar el usuario (en lugar de <usuario>@<host>),
    # con <host> bastaría.
    User <usuario>
    # Pide un terminal obligatoriamente
    RequestTTY force
    # Especifica una clave pública para la autenticación
    IdentityFile ~/.ssh/id_telematica
    # Si quieres utilizar X-forwarding
    #RemoteCommand export DISPLAY=localhost:10.0; cd ~; bash -l
    # Cambia el directorio a tu $HOME al entrar. Útil cuando la entrada
    # por defecto no es igual a $HOME.
    RemoteCommand cd ~; bash -l
```

##### `ForwardAgent`

Esta opción permite utilizar las claves almacenadas por el agente SSH local
desde la sesión remota sin necesidad de copiar los archivos de clave privada,
ni volver a introducir las *passphrases*, ni generar claves nuevas. Si decides
mantenerla desactivada (por seguridad u otros motivos), puedes activarla en
sesiones concretas usando `ssh -o ForwardAgent=yes`. También puedes activarla
solo al conectarte a servidores concretos si la especificas bajo una sección
`Host`.

##### Más

Para más información, consulte `ssh_config(5)`.

## Ejercicio 2. Gestión de usuarios

El primer paso antes de empezar con las operaciones es decidir un nombre de
usuario que incluya el nombre del alumno. En mi caso, eligiré `alonso_ej3`.

### 2.1. Crea al usuario

Se puede lograr con `adduser`, pero requiere instalar el paquete
correspondiente. La herramienta del sistema `useradd` también nos servirá. En
mi sistema, se encuentra en `/usr/sbin/`, por lo que requiere permisos de
administrador para ejecutarse.

```sh
sudo useradd alonso_ej3
```

Al crear el usuario mediante `useradd` sin especificar una contraseña con la
opción `--password`, la cuenta estará bloqueada y sin contraseña. Para asignar
una contraseña, usaremos el comando `passwd`:

```sh
sudo passwd alonso_ej3
```

Tras introducir una contraseña dos veces, se actualizará la contraseña del
usuario y de desbloqueará.

La siguiente captura de pantalla ilustra el proceso:

![Creación de usuario y contraseña](img/2.1-useradd-passwd.png)

> Nota
>
> Se puede establecer la contraseña directamente usando el comando `useradd`
> mediante la opción `--password`, pero esto no es recomendable. Si se desea
> establecer una contraseña vacía pero con el usuario habilitado, se puede
> pasar una cadena vacía como parámetro de `--password` a `useradd` (`sudo
> useradd alonso_ej3 --password ""`), o se puede establecer con `passwd` con la
> opción `-d` (`sudo passwd -d alonso_ej3`).

Para más información, vea `useradd(1)`, `passwd(1)`

### 2.2. Comprueba `/etc/passwd`, `/etc/shadow`

El primero se puede comprobar con cualquier herramienta deseada (como `cat`),
mientras que el segundo requiere permisos de `root`. Los contenidos se muestran
a continuación:

#### 2.2.1. `(/etc/passwd)`

Usando `cat`:

```txt
$ cat /etc/passwd
root:x:0:0:root:/root:/usr/bin/zsh
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
list:x:38:38:Mailing List Manager:/var/list:/usr/sbin/nologin
irc:x:39:39:ircd:/run/ircd:/usr/sbin/nologin
_apt:x:42:65534::/nonexistent:/usr/sbin/nologin
nobody:x:65534:65534:nobody:/nonexistent:/usr/sbin/nologin
systemd-network:x:998:998:systemd Network Management:/:/usr/sbin/nologin
systemd-timesync:x:997:997:systemd Time Synchronization:/:/usr/sbin/nologin
messagebus:x:100:107::/nonexistent:/usr/sbin/nologin
alonso_uni:x:1000:1000:Alonso Herreros (Uni),,,:/home/alonso_uni:/usr/bin/zsh
sshd:x:101:65534::/run/sshd:/usr/sbin/nologin
alonso_ej3:x:1001:1001::/home/alonso_ej3:/bin/sh
```

Como podemos ver, el nuevo usuario está al final del archivo.

Para más información sobre este archivo, vea `passwd(5)`.

#### 2.2.2. `(/etc/shadow)`

Para ver este archivo, necesitamos permisos elevados:

```txt
$ sudo cat /etc/shadow
root:$y$j9T$d0lRN81LUjTsmO3bYcrsq1$eOQVFLZ7mJVwpCq3rPGtJ83s4d6gO39WM3gbGazEQI5:20125:0:99999:7:::
daemon:*:20125:0:99999:7:::
bin:*:20125:0:99999:7:::
sys:*:20125:0:99999:7:::
sync:*:20125:0:99999:7:::
games:*:20125:0:99999:7:::
man:*:20125:0:99999:7:::
lp:*:20125:0:99999:7:::
mail:*:20125:0:99999:7:::
news:*:20125:0:99999:7:::
uucp:*:20125:0:99999:7:::
proxy:*:20125:0:99999:7:::
www-data:*:20125:0:99999:7:::
backup:*:20125:0:99999:7:::
list:*:20125:0:99999:7:::
irc:*:20125:0:99999:7:::
_apt:*:20125:0:99999:7:::
nobody:*:20125:0:99999:7:::
systemd-network:!*:20125::::::
systemd-timesync:!*:20125::::::
messagebus:!:20125::::::
alonso_uni:$y$j9T$Fu7J2NlkodmyKql0/.xds1$oK87nuOwjqGn5BluvOe6OapIL6NJnanUjisNP70gH..:20125:0:99999:7:::
sshd:!:20125::::::
alonso_ej3:$y$j9T$f1Ri6c0QEszQh/EjQz0VI1$GrEr9lQ5s0GPoCFWZcB6/lrr1YYPYifH2tytAoGt.T0:20155:0:99999:7:::
```

De nuevo, la entrada correspondiente al usuario que acabamos de crear está al
final del archivo. La secuencia `$y$` nos indica que la contraseña está
encriptada con `yescrypt`, con la sal `j9T`.

Para más información sobre este archivo, vea `shadow(5)`.

### 2.3. Crea los grupos `poesia`, `teatro`, `ensayo`

Para crear un grupo, podemos usar el comando `groupadd`. Se requieren permisos
elevados. Además, el comando solo permite crear un grupo a la vez, por lo que
tendremos que crearlos uno a uno.

```sh
sudo groupadd poesia
sudo groupadd teatro
sudo groupadd ensayo
```

La siguiente captura de pantalla ilustra el proceso:

![Creación de grupos](img/2.3-groupadd.png)

También se puede lograr lo mismo en un solo comando usando un bucle:

```sh
for g in "poesia" "teatro" "ensayo"; do sudo groupadd "$g"; done
```

Para más información, vea `groupadd(1)`.

### 2.4. Haz al usuario miembro de los tres grupos

Usando el comando `usermod`:

```sh
sudo usermod alonso_ej3 -aG poesia,teatro,ensayo
```

La siguiente captura de pantalla ilustra el proceso:

![Añadir usuario a grupos](img/2.4-usermod-aG.png)

Para más información, vea `usermod(1)`.

### 2.5. Haz de `ensayo` el grupo principal del usuario

Se puede usar el mismo comando `usermod`:

```sh
sudo usermod alonso_ej3 -g ensayo
```

La siguiente captura de pantalla ilustra el proceso:

![Cambio de grupo principal](img/2.5-usermod-g.png)

### 2.6. Cambia el nombre del grupo `teatro` a `drama`

Usando el comando `groupmod`:

```sh
sudo groupmod teatro -n drama
```

La siguiente captura de pantalla ilustra el proceso:

![Cambio de nombre de grupo](img/2.6-groupmod.png)

Para más información, vea `groupmod(1)`.

### 2.7. Cambia la contraseña del usuario. Comprueba `/etc/shadow`

Usando el comando `passwd`:

```sh
sudo passwd alonso_ej3
```

La siguiente captura de pantalla ilustra el proceso:

![Cambio de contraseña](img/2.7-passwd.png)

Tras introducir la nueva contraseña dos veces, podemos comprobar si la línea
correspondiente de `/etc/shadow` ha cambiado:

```shadow
alonso_ej3:$y$j9T$AbRs5V6SP7DE4iY0pcWmf.$1GW88B.rjyq848j8sRw4ORZdYFXZiJY9WMkuRghzbyB:20155:0:99999:7:::
```

Como podemos comprobar, la sal no ha cambiado, pero la cadena de la contraseña
cifrada sí.  El resto de campos han mantenido su valor, aunque si se hubiera
realizado esta operación en una fecha distinta, el tercer campo (fecha de
último cambio de contraseña, actualmente `20155`) se habría actualizado.

### 2.8. Obliga al usuario a cambiar su contraseña en 14 días

Usando el comando `chage`:

```sh
sudo chage alonso_ej3 -M 14
```

Dado este cambio, el usuario tendrá que cambiar su contraseña cada 14 días.

La siguiente captura de pantalla ilustra el proceso:

![Cambio de contraseña en 14 días](img/2.8-chage.png)

Para más información, vea `chage(1)`.

### 2.9. Bloquea la cuenta del usuario. Explica `/etc/shadow`

Usando el comando `usermod`:

```sh
sudo usermod alonso_ej3 -L
```

La siguiente captura de pantalla ilustra el proceso:

![Bloqueo de cuenta](img/2.9-usermod-L.png)

Si miramos en `/etc/shadow`, podemos ver que la línea correspondiente al
usuario ha cambiado:

```shadow
alonso_ej3:!$y$j9T$AbRs5V6SP7DE4iY0pcWmf.$1GW88B.rjyq848j8sRw4ORZdYFXZiJY9WMkuRghzbyB:20155:0:14:7:::
```

Antes de la cadena `$y$` se ha introducido una exclamación. Como se indica en
`shadow(5)`, la presencia de este carácter impedirá que el usuario inicie
sesión usando su contraseña. Sin embargo, el usuario podría iniciar sesión por
otros medios.

Además, se puede ver el efecto del comando anterior en el campo que contiene
`14`. Este es el campo de edad máxima, y determina cada cuántos días se debe
cambiar la contraseña.

El resto de campos han mantenido su valor.

### 2.10. Desbloquéala y deshabilita su contraseña. Explica `/etc/shadow`

Usando los comandos indicados en la guía:

```sh
sudo usermod alonso_ej3 -U
sudo passwd -d alonso_ej3
```

La siguiente captura de pantalla ilustra el proceso:

![Desbloqueo y deshabilitación de contraseña](img/2.10-usermod-U-passwd-d.png)

La línea correspondiente en `/etc/shadow` tras estos comandos es la siguiente:

```shadow
alonso_ej3::20155:0:14:7:::
```

Como podemos observar, el campo de la contraseña (entre el segundo y el tercer
carácter `:`) está vacío. Según `shadow(5)`, esto indica que no se requiere una
contraseña para iniciar sesión, aunque algunas aplicaciones pueden decidir
negar el acceso al usuario completamente al encontrar el campo vacío.

El resto de campos han mantenido su valor, aunque de igual manera que en el
apartado 2.7, si esta operación se hubiera realizado en una fecha distinta, el
campo correspondiente habría cambiado.

### 2.11. Borra todos los grupos y usuarios creados

Usando los comandos `userdel` y `groupdel`:

```sh
sudo userdel alonso_ej3
sudo groupdel alonso_ej3
sudo groupdel poesia
sudo groupdel ensayo
sudo groupdel drama
```

Deberemos eliminar el grupo `alonso_ej3` manualmente ya que no es el principal
del usuario. Además, debemos tener en cuenta el cambio de nombre del grupo
`teatro` (que ya no existe) a `drama`.

La siguiente captura de pantalla ilustra el proceso:

![Borrado de usuarios y grupos](img/2.11-userdel-groupdel.png)
