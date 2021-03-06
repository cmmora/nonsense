# Uso de múltiples usuarios en GIT

GIT definitivamente es una herramienta vital para todos los procesos relacionados con desarrollo a todos los niveles.  En nuestros primeros acercamientos a esta herramienta nos conformamos con manejar los comandos básicos en un repositorio  local, pero a medida que vamos avanzando en nuestro proceso de **aprendizaje permanente**

A medida que vamos avanzando en nuestro proceso, encontramos que pueden presentarse nuevas necesidades, como el caso de necesitar conectarnos a diferentes repositorios con diferentes usuarios.

Personalmente experimenté esta situación hace poco cuando en un curso de desarrollo de la universidad nos solicitaron gestionar un repositorio en GitHub dedicado exclusivamente a los proyectos del mismo. Para conectarme a dicho repositorio debia usar un usuario distinto al que ya tenia, pues el proceso requeria evidenciar el ciclo completo de desarrollo, partiendo desde la creación del usuario en GitHub. 

En la actualidad experimiento una necesidad similar, debido a que necesito separar mis proyectos personales de los laborales.

Para lograr esto, simplemente se requiere crear llaves rsa separadas para autenticar cada usuario segun el servidor git al que se desee conectar.  En este documento se realizará el procedimiento para conectarse a repositorios de GitHub y GitLab, con llaves rsa distintas para cada caso.

## Procedimientos a realizar

### 1. Generar las llaves SSH

En este paso simplemente mostraré el proceso de creación de las llaves.  La primera se crea de manera predeterminada con el comando **ssh-keygen -t rsa** con contraseña vacía.

```bash

    $ ssh-keygen -t rsa
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/personalUser/.ssh/id_rsa):
    Created directory '/home/personalUser/.ssh'.
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/personalUser/.ssh/id_rsa.
    Your public key has been saved in /home/personalUser/.ssh/id_rsa.pub.
    The key fingerprint is:
    SHA256:s6N0OwlTDKjDez98kZRwUGZbTYaQUArv+EYC6sigFwA personalUser@localhost
    The key's randomart image is:
    +---[RSA 2048]----+
    |E   ..o=*o.+o    |
    |.   .oo+oo...    |
    |....  o=..       |
    | o+. o  =        |
    |o .oo ooS.       |
    |* ...+o oo       |
    |oo.. o+o+o       |
    | .   o+o+o       |
    |      .o..       |
    +----[SHA256]-----+

```

La salida del comando indica que el directorio del usuario se crearon las llaves privada (id_rsa) y pública (id_rsa.pub), la cual es **la única** que se debe compartir (por eso es pública).

Teniendo la llave predeterminada creada, que es la que se usara para los proyectos personales, se procede a crear la segunda llave para los proyectos laborales (o para cualquier otro proyecto que requiera autenticación con otro usuario).

En el ejemplo que se presenta a continuación, se debe prestar mucha atención a la primera y tercera linea, ya que son las que definen la información de esta segunda llave.

```bash

    ssh-keygen -t rsa -C "user@companydomain.net"
    Generating public/private rsa key pair.
    Enter file in which to save the key (/home/personalUser/.ssh/id_rsa): /home/personalUser/.ssh/id_rsa-workAccount  
    Enter passphrase (empty for no passphrase):
    Enter same passphrase again:
    Your identification has been saved in /home/personalUser/.ssh/id_rsa-workAccount.
    Your public key has been saved in /home/personalUser/.ssh/id_rsa-workAccount.pub.
    The key fingerprint is:
    SHA256:WTAF6lV8a3FyMdMp3yW4h4NjsrKqXxy0i9JVp1/8gzk user@companydomain.net
    The key's randomart image is:
    +---[RSA 2048]----+
    |        ++o  .+o.|
    |       . +. =.++o|
    |      o o oo Oo.o|
    |     o +.=+.* ...|
    |      = S+ ooo   |
    |   . +.o.. . +   |
    |  . o +o  . E o  |
    |   . ..      . . |
    |  .oo.           |
    +----[SHA256]-----+

```

A diferencia del primer proceso para la generación de la llave, en este segundo proceso se agrega el parámetro "-C" (Mayúscula), con el cual se agrega un comentario para la llave generada; en este ejemplo se agregó una cuenta de correo.

La tercera linea es la que define en donde se van a almacenar localmentes las llaves pública y privada generadas.  Por defecto se intenta usar la ruta predeterminada, y es por eso que en este paso es **necesario declarar con que nombre se van a guardar estas segundas llaves**.

### 2. Crear un archivo de configuración para SSH

Ya teniendo las llaves creadas es necesario indicarle a SSH como las va a administrar para cada plataforma de desarrollo a la que se va a comectar.  En este ejemplo se conectará la cuenta predeterminada a GitHub y la segunda cuenta a GitLab.  Esta información se administra desde el archivo ```~/.ssh/config``` (El directorio ssh del usuario).

```bash
Host github.com
    Hostname github.com
    User git
    IdentityFile /home/personalUser/.ssh/id_rsa

Host gitlab.com-workAccount
    Hostname gitlab.com
    IdentityFile /home/personalUser/.ssh/id_rsa-workAccount

```

En este archivo la linea **Host** indica el con el que se va a identificar el servidor al que se va a conectar.  Esto es mas evidente en el segundo bloque, en donde se modificó el nombre del host para indicar explícitamente que será usado con el identificador "workAccount".

La linea **Hostname** indica el nombre real del servidor.  Aqui se declara la conexión hacia GitLab o GitHub.

La linea **IdentityFile** indica cual es la **llave privada**  con la cual se a realizar la autenticación del usuario ante el repositorio.  Esta es la que básicamente permite usar múltiples llaves a un mismo usuario.

Hay muchos más parámetros interesantes para este archivo, por lo cual recomiendo la lectura de [su manual](https://linux.die.net/man/5/ssh_config).

> Nota:
> Normalmente no es necesario declarar la conexión cuando se usa la llave predeterminada del usuario.  En este caso se declaró la llave predeterminada para usarla en GitHub simplemente con fines didácticos.  Lo normal y recomendado es solo declarar en ese archivo config las llaves distintas a la predeterminada.

### 3. Agregar las llaves públicas a cada servicio GIT al que se desea conectar

Ya teniendo las diferentes lllaves públicas que se desean usar en servicio GIT.  En este punto simplemente se necesita acceder a la cuenta de GitHub y agregar la primera llave por la opción __Settings / SSH and GPG keys__.  Luego se ingresa a GitLab y se agrega la segunda llave pública creada (***id_rsa-workAccount.pub***) usando la opción __Settings / SSH Keys__ del perfil de usuario.

> En este documento se asume que el usuario ya tiene las cuentas creadas en las plataformas requeridas.

### 4. Clonar el repositorio para que se autentique según el usuario requerido

Es en el proceso de clonación del repositorio donde se manifiesta la *magia* de usar múltiples llaves.  El repositorio de la primera cuenta se clona de la manera tradicional:

```bash

    git clone  git@github.com:gitHubUser/awsomeRepo.git

```

Al momento de clonar el segundo repositorio, es necesario editar la ruta usada para declarar que se va a usar la segunda llave.

Suponiendo que la ruta de descarga del segundo proyecto sea:

```bash

git@gitlab.com:company/latestWorkingProject.git

```

Se  edita la dirección del servidor (gitlab.com) por la ruta declarada en el archivo _~/.ssh/config_  para el uso de la segunda llave (***gitlab.com-workAccount***).

```bash

git clone git@gitlab.com-workAccount:company/latestWorkingProject.git

```

### 5. Información de usuario para el segundo repositorio

Con el objetivo de no usar la información global del usuario en el segundo repositorio, se sobreescriben los datos de usuario para que sean consecuentes con la información personal con la que desea registrar sus interacciones con el repositorio.

Lograr esto simplemente se ingresa al proyecto local y se declaran los datos básicos que desea usar para ese repositorio.

```bash

cd latestWorkingProject
git config user.name "First Name Last Name"
git config user.email "user@companydomain.net"

```

Con estos cinco sencillos pasos ya puedes trabajar con diferentes orígenes GIT con diferentes usuarios.

Una salida rápida pudo haber sido crear localmente un segundo usuario local para manejar los desarrollos separados, pero ya es cuestión de gustos.  Personalmente prefiero tener todo gestionado desde mi único usuario local. Además, comenté que en la universidad me solicitaron usuario exclusivo para los proyectos, asi que ya tres cuentas locales se comienza a hacer algo tedioso.