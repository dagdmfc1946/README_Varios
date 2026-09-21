# **Telsy Monitor**
Vital signs monitor firmware of Telsy Hogar

Copyright (C) 2022  Elmer Eduardo Rocha Jaime, Dirección de Innovación y Desarrollo Tecnológico - Fundación Cardiovascular de Colombia FCV

## **📑 Menu**
- [📋 Requirements](#-requirements)
- [💾 Raspberry Pi OS Installation](#-raspberry-pi-os-installation-on-microsd)
- [🔧 Raspberry Pi OS Setup](#-raspberry-pi-os-setup)
- [⚙️ Change boot image](#%EF%B8%8F-change-boot-image)
- [🚀 Starting](#-starting)
- [💿 OS Cloning (optional)](#-os-cloning-optional)
- [🛠️ Built with](#%EF%B8%8F-built-with)
- [✒️ Authors](#%EF%B8%8F-authors)
- [📄 License](#-license)

## **📋 Requirements**
This is what is necessary to run the project:
- [Raspberry Pi 4 Model B 4GB](https://www.raspberrypi.com/products/raspberry-pi-4-model-b/)
- [Raspberry Pi OS with desktop - Debian 11 Bullseye (Sep 22nd, 2022)](https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2022-09-26/)
- [Python 3.9.2](https://www.python.org/downloads/release/python-392/)
- [Django 4.1.3](https://docs.djangoproject.com/en/4.1/releases/4.1.3/)

Python libraries needed:
- [smbus](https://pypi.org/project/smbus/)
- [pyserial](https://pypi.org/project/pyserial/)
- [RPi.GPIO](https://pypi.org/project/RPi.GPIO/)
- [typing-extensions](https://pypi.org/project/typing-extensions/)
- [channels](https://pypi.org/project/channels/)
- [daphne](https://pypi.org/project/daphne/)

## **💾 Raspberry Pi OS installation on MicroSD**
These installation instructions are written for the Windows 10 operating system.

It's necessary a MicroSD with more than 8 GB and an SD Adapter or a MicroSD USB adapter.


### Step 1. Download the [Raspberry Pi Imager](https://www.raspberrypi.com/software/) program and install it.

![Download Raspberry Pi Imager](/telsy/monitor/static/images/installation/download_raspberry_pi_imager.png)

### Step 2. Select the operating system image
Scroll down in the [page](https://www.raspberrypi.com/software/) until "Manually install an operating system" image and click on [See all download options](https://www.raspberrypi.com/software/operating-systems/).

![OS Image](/telsy/monitor/static/images/installation/os_image.png)

### Step 3. Download the OS image
Scroll down in the [page](https://www.raspberrypi.com/software/operating-systems/) and go to [Raspberry Pi OS with desktop](https://www.raspberrypi.com/software/operating-systems/#raspberry-pi-os-32-bit) and click on [Archive](https://downloads.raspberrypi.org/raspios_armhf/images/).

![Archive Raspberry Pi OS with desktop](/telsy/monitor/static/images/installation/raspberry_pi_os_with_desktop.png)

In [archive page](https://downloads.raspberrypi.org/raspios_armhf/images/) select [`raspios_armhf-2022-09-26/ 2022-09-26 09:37`](https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2022-09-26/).

![Index of Raspi Os](/telsy/monitor/static/images/installation/index_raspios.png)

Once there download the [`2022-09-22-raspios-bullseye-armhf.img.xz`](https://downloads.raspberrypi.org/raspios_armhf/images/raspios_armhf-2022-09-26/2022-09-22-raspios-bullseye-armhf.img.xz) file.

![Raspi OS ARMHF](/telsy/monitor/static/images/installation/raspios_armhf.png)

### Step 4. Setup Raspberry Pi Imager
At the Raspberry Pi Imager program click on `CHOOSE OS` button.

![Raspberry Pi Imager - CHOOSE OS](/telsy/monitor/static/images/installation/raspberry_pi_imager_1.png)

Scroll down to `Use custom`.

![Use custom OS image](/telsy/monitor/static/images/installation/raspberry_pi_imager_2.png)

Once there, select the OS image file downloaded (2022-09-22-raspios-bullseye-armhf.img.xz).

![Custom OS Image](/telsy/monitor/static/images/installation/raspberry_pi_imager_3.png)

Select the MicroSD device by clicking on `CHOOSE STORAGE`.

![Select storage](/telsy/monitor/static/images/installation/raspberry_pi_imager_4.png)

Select your MicroSD device.

![Select MicroSD device](/telsy/monitor/static/images/installation/raspberry_pi_imager_5.png)

After that, press the gear (⚙️) button.

![Gear button](/telsy/monitor/static/images/installation/raspberry_pi_imager_6.png)

In the _Set hostname:_ write **monitor** and select `Enable SSH`.

![Setup Hostname](/telsy/monitor/static/images/installation/raspberry_pi_imager_7.png)

Now select `Set username and password`, in _Username_ write **`telsy`** and in _Password_ write **`T3lsyh0gar`**, select `Configure wireless LAN`, in _SSID_ write **`127.0.0.1`** and in _Password_ write **`11223344`**.

![Setup username](/telsy/monitor/static/images/installation/raspberry_pi_imager_8.png)

Select **`CO`** in `Wireless LAN country`, select **`America/Bogota`** in `Time zone` and **`latam`** in `Keyboard layout`. Then press `SAVE` button.

![Setup locale and time zone](/telsy/monitor/static/images/installation/raspberry_pi_imager_9.png)

After all, press `WRITE` button to start the OS writing in the MicroSD, it will ask if you want to continue, press `YES`.

![Start writing](/telsy/monitor/static/images/installation/raspberry_pi_imager_10.png)

### Step 5. Configure the wireless hotspot

Using a laptop connected to the internet via Ethernet with Windows 10, press the computer (🖥) icon on the right end of the taskbar.

![Hotspot 1](/telsy/monitor/static/images/installation/hotspot_1.png)

Then right-click on `Mobile hotspot` and press `Go to Settings`.

![Hotspot 2](/telsy/monitor/static/images/installation/hotspot_2.png)

![Hotspot 3](/telsy/monitor/static/images/installation/hotspot_3.png)

Select `WiFi` and press the `Edit` button.

![Hotspot 4](/telsy/monitor/static/images/installation/hotspot_4.png)

In the _Network name_ write **`127.0.0.1`** and in _Network password_ write **`11223344`**.

![Hotspot 5](/telsy/monitor/static/images/installation/hotspot_5.png)

Then activate the hotspot by pressing the button.

![Hotspot 6](/telsy/monitor/static/images/installation/hotspot_6.png)

### Step 6. Start Raspberry Pi OS

Insert the MicroSD in the Raspberry Pi 4, connect the touchscreen, and power the board.

## **🔧 Raspberry Pi OS Setup**

### Step 1. Update Raspberry Pi OS
When you power the Raspberry Pi board for the first time you will have to update the system by pressing the icon on the right top.

![Update 1](/telsy/monitor/static/images/installation/update_1.png)

It will start to download the packages:

![Update 2](/telsy/monitor/static/images/installation/update_2.png)

When it finishes press `Reboot`.

![Update 3](/telsy/monitor/static/images/installation/update_3.png)

### Step 2. Login to the Raspberry over SSH
In the hotspot opened you should see the IP assigned to your Raspberry Pi once it is connected.

![Hotspot 7](/telsy/monitor/static/images/installation/hotspot_7.png)

For SSH connection you could use any method, for this time we will use [Git Bash](https://git-scm.com/download/win), so skipping the installation process and assuming you already have Git Bash installed: Opens a new terminal and put the next. The pi user is called **`telsy`** (configured [here](#step-4-setup-raspberry-pi-imager)), so the way to log in over ssh is:
```
ssh telsy@<raspberry.ip>
```
In this case, the `raspberry.ip` is **192.168.137.67** as seen in `Devices connected` in the hotspot, so connect the Raspberry over SSH:

![SSH 1](/telsy/monitor/static/images/installation/ssh_1.png)

The program will ask you for an ED25519 key fingerprint, so you have to write **yes**.

![SSH 2](/telsy/monitor/static/images/installation/ssh_2.png)

Then continue writing the user password: **T3lsyh0gar** and press <kbd>Enter</kbd>.

![SSH 3](/telsy/monitor/static/images/installation/ssh_3.png)

<p>If everything was successful you should see <span style="color:#2BBF5D; font-weight: bold">telsy@monitor</span>:<span style="color:#257DCD; font-weight: bold">~ $</span></p>

![SSH 4](/telsy/monitor/static/images/installation/ssh_4.png)

### Step 3. Configure interfaces settings
Once you remotely connect to the pi over SSH, run the following command:
```
sudo raspi-config
```
![SSH 5](/telsy/monitor/static/images/installation/ssh_5.png)

![SSH 6](/telsy/monitor/static/images/installation/ssh_6.png)

From the menu select:
- **3 Interface Options**
    - **I5 I2C**
        - **`Yes`**

![SSH 7](/telsy/monitor/static/images/installation/ssh_7.png)

![SSH 8](/telsy/monitor/static/images/installation/ssh_8.png)

Then go to:
- **3 Interface Options**
    - **I6 Serial Port**
        - **Login Shell**
            - **`No`**
        - **Hardware**
            - **`Yes`**

![SSH 9](/telsy/monitor/static/images/installation/ssh_9.png)

![SSH 10](/telsy/monitor/static/images/installation/ssh_10.png)

Press the <kbd>Tab</kbd> key twice to get to the `Finish` option, then press the <kbd>Enter</kbd> key.

![SSH 11](/telsy/monitor/static/images/installation/ssh_11.png)

When asked to reboot, select `Yes`.

![SSH 12](/telsy/monitor/static/images/installation/ssh_12.png)

Reconnect after reboot via SSH.

### Step 4. Install some components and Python libraries
While remotely logged in to the Raspberry over SSH, run the following at the command line:

```
sudo su
```

You should see **root@monitor:/home/telsy#**

![SSH 13](/telsy/monitor/static/images/installation/ssh_13.png)


Now you are in the `root` so proceed with caution, run the following commands at the command line:

```
apt-get update
```
```
apt-get install -y xscreensaver python3-dev libzbar-dev libzbar0 libffi-dev
```
```
apt-get install -y clang gcc libssl-dev build-essential plymouth plymouth-themes pix-plym-splash
```

Rust will now be installed to compile the installation of some dependencies:

```
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

After it downloads the installer, it will ask to choose an installation way. To proceed with installation press <kbd>Enter</kbd>.

![SSH 14](/telsy/monitor/static/images/installation/ssh_14.png)

After the installation is finished put the following command:

```
source "$HOME/.cargo/env"
```

![SSH 15](/telsy/monitor/static/images/installation/ssh_15.png)

Then put:

```
apt-get remove -y lxplug-ptbatt lxplug-updater
```

```
pip install Django==4.1.3 typing-extensions==4.4.0 smbus pyserial RPi.GPIO
```

The next installation will take a little longer:

```
pip install channels["daphne"]
```

### Step 5. Edit the boot config.txt file

After all libraries and dependencies are installed edit the config boot file:

```
nano /boot/config.txt
```

Find `dtoverlay=vc4-kms-v3d` and change it to **`dtoverlay=vc4-fkms-v3d`**

![SSH 16](/telsy/monitor/static/images/installation/ssh_16.png)

Then, check that `enable_uart=1` is on the last line, otherwise add it.

And add the following in the last line:
```
disable_splash=1
dtoverlay=uart2
avoid_warnings=1
dtoverlay=disable-bt
lcd_rotate=2
```
The last lines should look like this:

![SSH 17](/telsy/monitor/static/images/installation/ssh_17.png)

To save the file press <kbd>Ctrl</kbd>+<kbd>S</kbd> and to close press <kbd>Ctrl</kbd>+<kbd>X</kbd>.

### Step 6. Edit the cmdline.txt file
Now edit the cmdline boot:
```
nano /boot/cmdline.txt
```
Remove this (if exists):
```
console=serial0,115200
```
Add this to the end:
```
logo.nologo vt.global_cursor_default=0
```

After that, the file should have `console=tty1 root=PARTUUID=<xxxxxxxx-xx> rootfstype=ext4 fsck.repair=yes rootwait quiet splash plymouth.ignore-serial-consoles logo.nologo vt.global_cursor_default=0` avoiding the PARTUUID.

## **⚙️ Change boot image**

### Step 1. Edit the Plymouth pix.script file
```
nano /usr/share/plymouth/themes/pix/pix.script
```
Remove the following (stay at the line to remove and press <kbd>Ctrl</kbd>+<kbd>K</kbd>):
```
message_sprite = Sprite();
message_sprite.SetPosition(screen_width * 0.1, screen_height * 0.9, 10000);

my_image = Image.Text(text, 1, 1, 1);
message_sprite.SetImage(my_image);
```

The last lines should look like this:

![SSH 18](/telsy/monitor/static/images/installation/ssh_18.png)

Press <kbd>Ctrl</kbd>+<kbd>S</kbd> and press <kbd>Ctrl</kbd>+<kbd>X</kbd>.

### Step 2. Restart
To finish this configuration reboot the Raspberry Pi, so put the next in the terminal:
```
reboot
```

## **🚀 Starting**
### Step 1. Clone the repository
Once connected to the Raspberry Pi over SSH check that the current path is `/home/telsy/` (with **`pwd`** command), then run the next command line:
```
git clone -b dev https://telsy:glpat-jYUEozh72_VAZBeE2dEa@gitlab.com/businesslab/telsy-monitor.git
```

### Step 2. Put boot splash image to the specific path
Run the command:
```
sudo cp /home/telsy/telsy-monitor/telsy/monitor/static/images/splash.png /usr/share/plymouth/themes/pix
```

### Step 3. Change wallpaper
If you have a mouse connected to the Raspberry Pi press right-click on the desktop and go to `Desktop preferences`, otherwise go to the main menu (Raspberry Pi icon), Preferences, then **Appearance Settings**:

![Wallpaper 1](/telsy/monitor/static/images/installation/wallpaper_1.png)

Then deselect _Wastebasket_, and press `clouds.jpg` in front of _Picture_ and select **`splash.png`** in the path _/home/telsy/telsy-monitor/telsy/monitor/static/images_

![Wallpaper 2](/telsy/monitor/static/images/installation/wallpaper_2.png)

![Wallpaper 3](/telsy/monitor/static/images/installation/wallpaper_3.png)

This is the result:

![Wallpaper 4](/telsy/monitor/static/images/installation/wallpaper_4.png)

### Step 4. Disable sleep mode
Go to the main menu (Raspberry Pi icon), then Preferences, then **Screensaver**, now change _Mode_ to **Disable Screen Saver** and close the window:

![Wallpaper 5](/telsy/monitor/static/images/installation/wallpaper_5.png)

![Wallpaper 6](/telsy/monitor/static/images/installation/wallpaper_6.png)

### Step 5. Hide pointer
Put the command line:
```
sudo nano /etc/lightdm/lightdm.conf
```
Go to line 95 pressing <kbd>Alt</kbd>+<kbd>C</kbd> to see the current line, uncomment it, and add -nocursor:
```
xserver-command=X -nocursor
```
It should look like this:

![SSH 19](/telsy/monitor/static/images/installation/ssh_19.png)

Press <kbd>Ctrl</kbd>+<kbd>S</kbd> and press <kbd>Ctrl</kbd>+<kbd>X</kbd>.

### Step 6. Configure kiosk startup
Put the command line:
```
sudo nano /etc/xdg/lxsession/LXDE-pi/autostart
```
Comment (add #) to _@lxpanel --profile LXDE-pi_ and add this to the last lines:
```
@sh /home/telsy/telsy-monitor/startserver.sh
@sh /home/telsy/telsy-monitor/startweb.sh
```
It should look like this:

![SSH 20](/telsy/monitor/static/images/installation/ssh_20.png)

Press <kbd>Ctrl</kbd>+<kbd>S</kbd> and press <kbd>Ctrl</kbd>+<kbd>X</kbd>.

### Step 7. Create device serial number file
Put the command line:
```
nano /home/telsy/telsy_serial_number
```
Write the current serial number device:

![SSH 21](/telsy/monitor/static/images/installation/ssh_21.png)

Press <kbd>Ctrl</kbd>+<kbd>S</kbd> and press <kbd>Ctrl</kbd>+<kbd>X</kbd>.

### Step 8. Restart
Put the command line:
```
sudo reboot
```
If you followed the steps correctly you should see the Telsy Home interface.

## **💿 OS Cloning (optional)**
If you already have a Raspberry Pi OS with all the Telsy Hogar project installation with all the configurations installed on a MicroSD, you can choose to clone the entire system, for this process you need to have a USB MicroSD adapter and a USB keyboard:

### Step 1. Start cloning process
Insert the MicroSD USB Adapter with an empty MicroSD and a USB keyboard to the Raspberry Pi, then with the keyboard press <kbd>Alt</kbd>+<kbd>F4</kbd> to close the current Telsy Hogar interface, then, press <kbd>Ctr</kbd>+<kbd>Alt</kbd>+<kbd>T</kbd> to open a new terminal, then, put the next to clear the Chromium navigator cache:

```
rm -rf ~/.cache/chromium
```
![Clone 0](/telsy/monitor/static/images/installation/clone_0.png)

### Step 2. Cloning process
Now, put the next to open the taskbar:

```
lxpanel --profile LXDE-pi
```

![Clone 1](/telsy/monitor/static/images/installation/clone_1.png)

Go to the main menu (Raspberry Pi icon), Accessories, then **SD Card Copier**, in the window select **`USD (/dev/mmcblk0)`** in the `Copy From Device` option, then, select the MicroSD adapter **`(/dev/sda)`** and **check** the `New Partition UUIDs` option.

![Clone 2](/telsy/monitor/static/images/installation/clone_2.png)

![Clone 3](/telsy/monitor/static/images/installation/clone_3.png)

It will ask if you are sure to erase all content of the MicroSD, to continue press `Yes`:

![Clone 4](/telsy/monitor/static/images/installation/clone_4.png)

The process will take a long time to finish.

## **🛠️ Built with**
* [Django v4.1.3](https://docs.djangoproject.com/en/4.1/) - Web Framework
* [Django Channels v4.4.0](https://channels.readthedocs.io/) - Django WebSocket
* [Bootstrap v5.2.3](https://getbootstrap.com/docs/5.2/getting-started/introduction/) - CSS Framework
* [Fontawesome Free v6.2.1](https://fontawesome.com/icons) - Icons
* [jQuery v3.6.2](https://api.jquery.com/) - JavaScript AJAX Library
* [Splide v4.1.3](https://splidejs.com/category/users-guide/) - JavaScript Slider
* [Sweetalert2 v11.7.3](https://sweetalert2.github.io/) - JavaScript popup messages
* [Roboto](https://fonts.google.com/specimen/Roboto) - Google Typographic Fonts

## **✒️ Authors**
* **Juan Sebastián Barrios** - *Graphic interface* - [s3ba5t1an](https://github.com/s3ba5t1an)
* **David Vásquez** - *Telecentre platform* - [davidvasquezr](https://github.com/davidvasquezr)
* **Elmer Rocha** - *Monitor firmware* - [elmerrocha](https://github.com/elmerrocha)

## **📄 License**
This project is under the GNU AGPLv3 License - look at the file [LICENSE.md](LICENSE.md) for more details