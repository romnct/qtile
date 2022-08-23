# Qtile!

Que sirva como referencia de cuando vuelva a instalar _**Qtile**_. 

Instalado en una máquina con ***Arch Linux***.

## Durante la instalación de Arch Linux

Durante la instalación de Arch Linux se deberán instalar los siguientes paquetes en `chroot`.

```
sudo pacman -S grub efibootmgr networkmanager wpa_supplicant base-devel git sudo nano neovim python-psutil pacman-contrib unzip
```

Iniciar el servicio de *networkmanager * para tener internet.

```
sudo systemctl enable NetworkManager.service
```

```
sudo systemctl start NetworkManager.service
```

## Problemas con los *locales*

A veces los *locale* pueden dar problemas y es por ello que debemos de configurarlos correctamente siguiendo lo dispuesto en la [wiki](https://wiki.archlinux.org/title/locale).

Tenemos que generar los locales que queramos en nuestro sistema. En mi caso inglés y español. Para ello descomentamos del archivo `/etc/locale.gen` las siguientes líneas:

```
en_US.UTF-8
```

```
es_ES.UTF-8
```

Y generamos los locales.


```
locale-gen
```

Mi sistema está en inglés, pero con números y fechas regionales, es decir, de España. Esto se refleja de la siguiente forma en el archivo `/etc/locale.conf`.

```
LANG=es_ES.UTF-8
LANGUAGE=es_ES
LC_ADDRESS=es_ES.UTF-8
LC_COLLATE=es_ES.UTF-8
LC_CTYPE=es_ES.UTF-8
LC_IDENTIFICATION=es_ES.UTF-8
LC_MEASUREMENT=es_ES.UTF-8
LC_MESSAGES=en_US.UTF-8
LC_MONETARY=es_ES.UTF-8
LC_NAME=es_ES.UTF-8
LC_NUMERIC=es_ES.UTF-8
LC_PAPER=es_ES.UTF-8
LC_TELEPHONE=es_ES.UTF-8
LC_TIME=es_ES.UTF-8
```

## Previo a _Qtile_

### AUR helper

Necesitaremos ciertos paquetes que no se encuentran disponible en los repositorios oficiales de *Arch*. Estos paquetes los podemos encontrar en [AUR](https://aur.archlinux.org/), el repositorio de usuarios de *Arch Linux*.

Con [yay](https://github.com/Jguer/yay) instalamos los paquetes de *AUR*.

```
git clone https://aur.archlinux.org/yay.git && cd yay
```

```
makepkg -si
```

### Algunos paquetes necesarios
Previo a instalar _Qtile_ debemos instalar los siguientes paquetes:

| Paquete | Descripción |
|---------|-------------|
| intel-ucode | Actualizaciones para las CPU de Intel |
| xf86-video-intel | Drivers para gráficas *Intel* |
| mesa | Implementación de código abierto de la especificación OpenGL. En otras palabras, son conjunto de software para el procesamiento de gráficos avanzados |
| xorg-server | Sistema de ventanas |
| lightdm | Gestor de acceso |
| lightdm-webkit2-greeter| Interfaz gráfica de lightdm |
| lightdm-webkit-theme-sequoia-git| Un tema para webkit2 |


```
sudo pacman -Syu
```

```
sudo pacman -S intel-ucode
```

```
sudo pacman -S xf86-video-intel
```

```
sudo pacman -S mesa
```

```
sudo pacman -S xorg-server
```

```
sudo pacman -S lightdm
```

```
pacman -S lightdm-webkit2-greeter
```

```
yay -S lightdm-webkit-theme-sequoia-git
```

Una vez instalados los paquetes debemos iniciar el servicio de lightdm.

```
sudo systemctl enable lightdm
```

E indicarle a *lightdm* que *greeter* debe utilizar. Para ello se debe de descomentar `greeter-session` del archivo de configuración de *lightdm* `/etc/lightdm/lightdm.conf` e indicarle el nombre del *greeter*.

```
greeter-session=lightdm-webkit2-greeter 
```

Y en la configuración de *webkit2* `/etc/lightdm/lightdm-webkit2-greter.conf`

```
webkit_theme=sequoia 
```

## Instalación de Qtile

Instalamos *Qtile* desde el repositorio oficial de *Arch Linux*. También lo podremos hacer desde su repositorio en *Github*. Cuestión de gustos.

```
sudo pacman -S qtile
```

Ahora ya tenemos todo listo para poder iniciar Qtile. Tenemos el servidor de ventanas, el gestor de acceso y el gestor de ventanas. **Reiniciamos** o escribimos el siguiente comando:


```
sudo systemctl start lightdm
```

---

**_Nota:_** Se debería de hacer uno de los siguientes pasos antes de reiniciar o ejecutar el comando anterior.

1. Instalar la ***xterm***.

```
sudo pacman -S xterm
```

2. Cambiar en el archivo *config* `/.config/qtile/config.py`  de *Qtile* la terminal que usará por defecto. En mi caso la [kitty](https://sw.kovidgoyal.net/kitty/).

```
sudo pacman -S kitty
```

```
Key([mod], "Return", lazy.spawn("kitty"), desc='Launches My Terminal' )
```

Debido a que en caso contrario no podremos iniciar la terminal desde *Qtile* porque, este, tiene asignada como por defecto la *xterm*.

---

## Configuración de Qtile

### Primeros pasos en Qtile

#### Definir el archivo `autostart`

A veces necesitaremos que ciertos paquetes se inicicen en el mismo momento que Qtile, es por ello que debemos definir un archivo donde estén listado estos paquetes. El archivo en cuestión podrá estar situado en cualquier lugar, pero lo más lógico es que se encuentre en el directorio de configuración de *Qtile* `$HOME.config/qtile`.

Este archivo será llamado a través de un *hook* de Qtile que definiremos en su archivo de configuración `$HOME.config/qtile`.

### Establecer una distribución del teclado

Para mantener fija la distribución del teclado debemos crear el siguiente archivo `$HOME/.Xkbmap` especificando en él la distribución deseada.

```
echo "es" > $HOME/.Xkbmap
```

Este arhivo es leído por el *script* `/etc/lightdm/Xsession` el cual ejecutará el siguiente comando cada vez que iniciemos sesión en *lightdm*.

```
setxkbmap `cat /home/.Xkbmap`
```

### Touchpad¨- (No probado)

De esto se encarga el *driver* [libinput](https://wiki.archlinux.org/title/libinput), el cual, si tienes instalado [xorg](https://wiki.archlinux.org/title/Xorg) ya lo tienes incluido.

El *touchpad* se configura a través del archivo de configuración `/etc/X11/xorg.conf.d/30-touchpad.conf`. Se debe de crear en el caso de no existir.

```
Section "InputClass"
    Identifier "SynPS/2 Synaptics TouchPad"
    Driver "libinput"
    Option "Tapping" "on"
    Option "NaturalScrolling" "true"
EndSection
```

El `Identifier` se puede conocer a gracias al comando `libinput list-devices`.

###  Creación de los directorios de usuario

Podemos crearlos gracias al paquete `xdg-user-dirs`.

```
sudo pacman -S xdg-user-dirs
```

Y para crearlos.

```
xdg-user-dirs-update
```

### Necesitamos audio

Recuerda que Qtile es un gestor de ventanas y nada más. No hay audio por lo tanto. Instalamos **pulseaudio* y **pamixer** para controlar el sonido o **pavucontrol** que tiene interfaz gráfica.

```
sudo pacman -S pulseaudio
```

```
sudo pacman -S pulseaudio-alsa
```

```
sudo pacman -S alsa-utils
```

```
sudo pacman -S pamixer
```

```
sudo pacman -S pavucontrol
```

***Alsa*** ya está incluido por defecto en el kernel de Arch por lo que no es necesaria su instalación explícitamente.

### Las notificaciones

Para tener notificaciones en el escritorio instalamos ***libnotify*** y ***notification-daemon***.

```
sudo pacman -S libnotify notification-daemon
```

Y seguimos la *wiki* que nos indica que debemos crear el archivo `/usr/share/dbus-1/services/org.freedesktop.Notifications.service` y añadir lo siguiente.

```
[D-BUS Service]
Name=org.freedesktop.Notifications
Exec=/usr/lib/notification-daemon-1.0/notification-daemon
```

Y para probarlo escribimos el siguiente comando en consola:

```
notify-send "Hello!"
```

### Algun lanzador chulo

Me he decantado por [ulauncher](https://ulauncher.io/).

```
git clone https://aur.archlinux.org/ulauncher.git && cd ulauncher && makepkg -si
```

Debemos añadirlo al archivo `.config/qtile/autostart.sh` para que inicie cuando lo haga *Qtile*.

```
ulauncher --hide-window &
```

### Unas fuentes bonitas

Primero, debemos instalar algunas fuentes básicas.

```
sudo pacman -S ttf-dejavu ttf-liberation noto-fonts
```

Ahora vamos a instalar las ***Ubuntu Nerd Font***, pero si quisieramos otras el método es el mismo. Siempre y cuando estén en *AUR* claro.

```
yay -S nerd-fonts-ubuntu-mono
```

```
yay -S nerd-fonts-hack 
```

```
yay -S nerd-fonts-hack 
```

```
yay -S ttf-roboto
```

```
yay -S ttf-iosevka-nerd
```

### Compositor para Xorg

Para darle algo de vidilla al gestor de ventanas, descargamos un compositor **[Picom](https://wiki.archlinux.org/index.php/Picom)** para, por ejemplo, crear transparencias.

```
sudo pacman -S picom
```

### Captura de pantalla

**[scrot](https://wiki.archlinux.org/index.php/Screen_capture)**


### Brillo para portátiles

**[brightnessctl](https://www.archlinux.org/packages/community/x86_64/brightnessctl/)**


### Fondo de pantalla

### La *zsh*

Seguir al pie de la letra [zsh](https://github.com/romnct/zsh).

### Mi *kitty*

### *nvim*

Instalación y configuración de [neovim](https://github.com/romnct/nvim). 

Para permitir el uso del *clipboard* debemos instalar el paquete [clip](https://archlinux.org/packages/extra/x86_64/xclip/)

```
sudo pacman -S xclip
```

### *bat*

### Tamaño y tema del cursor

El archivo que modifica las opciones del tamaño o del tema del cursor instalados son `Xcursor.theme` y `Xcursor.size`. Para modificarlos debemos hacerlo en el archivo `.Xresources` que se creará si no existe.

```
Xcursor.theme: Adwaita
Xcursor.size: 20
```

Se pueden obtener los temas instalados haciendo uso del comando ` find /usr/share/icons ~/.local/share/icons ~/.icons -type d -name "cursors"`.

### Mi configuración

#### El *script* autostart

## Líneas generales a seguir después de la instalación

Tomando como referencia las recomendaciones generales de la *wiki* de [*Arch Linux*](https://wiki.archlinux.org/title/General_recommendations).

### Mantenimiento del sistema
#### Actualización
#### Paquetes huérfanos
### Configuración del cortafuegos


