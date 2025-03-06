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
autenticación con claves debería funcionar. Para más detalles, consulte
`ssh_config(5)`.

## Ejercicio 2. Gestión de usuarios

### 1. Crea al usuario

### 2. Comprueba `/etc/passwd`, `/etc/shadow`

### 3. Crea los grupos `poesia`, `teatro`, `ensayo`

### 4. Haz al usuario miembro de los tres grupos

### 5. Haz de `ensayo` el grupo principal del usuario

### 6. Cambia el nombre del grupo `teatro` a `drama`.

### 7. Cambia la contrasena del usuario. Comprueba `/etc/shadow`

### 8. Obliga al usuario a cambiar su contraseña en 14 días

### 9. Bloquea la cuenta del usuario. Explica `/etc/shadow`

### 10. Desbloquéala y deshabilita su contraseña. Explica `/etc/shadow`

### 11. Borra todos los grupos y usuarios creados
