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
SHA256:HXQ1YiXwW0tO3m9z7M1EAn3rvt4HI/r/MPmzpgBftgY alonso_uni@alonso-deb
The key's randomart image is:
+---[RSA 3072]----+
|          o.=o+  |
|         . + + . |
|          . o = .|
|         . . O +.|
|        S..E.o=.o|
|          o = ==.|
|           + ++=*|
|          . o .@*|
|           ..o=*%|
+----[SHA256]-----+
```

Se puede realizar la misma operación con un solo comando:

```sh
ssh-keygen -t rsa -f /home/alonso_uni/.ssh/id_rsa -N '1234'
```

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

> Nota

> Se puede establecer la contraseña directamente usando el comando `useradd`
> mediante la opción `--password`, pero esto no es recomendable. Si se desea
> establecer una contraseña vacía pero con el usuario habilitado, se puede
> pasar una cadena vacía como parámetro de `--password` a `useradd` (`sudo
> useradd alonso_ej3 --password ""`), o se puede establecer con `passwd` con la
> opción `-d` (`sudo passwd -d alonso_ej3`).

### 2.2. Comprueba `/etc/passwd`, `/etc/shadow`

### 2.3. Crea los grupos `poesia`, `teatro`, `ensayo`

### 2.4. Haz al usuario miembro de los tres grupos

### 2.5. Haz de `ensayo` el grupo principal del usuario

### 2.6. Cambia el nombre del grupo `teatro` a `drama`.

### 2.7. Cambia la contrasena del usuario. Comprueba `/etc/shadow`

### 2.8. Obliga al usuario a cambiar su contraseña en 14 días

### 2.9. Bloquea la cuenta del usuario. Explica `/etc/shadow`

### 2.10. Desbloquéala y deshabilita su contraseña. Explica `/etc/shadow`

### 2.11. Borra todos los grupos y usuarios creados
