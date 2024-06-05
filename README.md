# Steps to setup RP with Aravis and get it working oh yeah

## Pi setup and connection
This section explains how to setup a blank sd card and Pi5 as a headless Pi you can SSH into

- Flash OS using Raspbian Imager
- Unplug and replug into PC
- Create files in boot partition (NOT root/boot):
-- `touc ssh.txt`

-- `touch wpa_supplicant.conf` with the following:
```
ctrl_interface=DIR=/var/run/wpa_supplicant GROUP=netdev
update_config=1
country=GB

network={
	ssid="WiFi SSID"
	psk="WPA/WPA2 passphrase"
	key_mgmt=WPA-PSK
}
```
Insert into PI and power on, wait until pi turns up as connected on hotspot 

- Find ip address of Pi : `ping raspberrypi.local`
- SSH to pi : `ssh pi@IP`

## Python Setup

In root directory of the pi (or create project dir with all files in)
`git clone https://github.com/lionfish0/bee_track.git`

Setup virtual environment
`python -m venv bee-venv`

Activate - (if using SSH terminal to change python env)
Make sure all commands are run in the virtual environment from now on
`source bee-venv/bin/activate`

`pip install -e bee_track/.`
`sudo apt install libatlas3-base`



## Aravis setup

`git clone https://github.com/AravisProject/aravis.git`

`sudo apt install cmake`

and

`sudo apt install libgtk-3-dev libnotify-dev libgstreamer1.0-dev libgstreamer-plugins-base1.0-dev gstreamer1.0-plugins-bad`

and

`sudo apt-get install gnome-common intltool valac libglib2.0-dev gobject-introspection libgirepository1.0-dev libgtk-3-dev libgnome-desktop-3-dev libcanberra-dev libgdata-dev libdbus-glib-1-dev libgstreamer1.0-dev libupower-glib-dev libxml2-dev`

`meson setup aravis/`

`cd aravis`

`meson build`

`cd build`

`meson configure -Dviewer=enabled -Dintrospection=enabled -Dgst-plugin=enabled`

`sudo ninja install` might have to `pip install ninja`


## Beetrack setup

Important installs:

`pip install PyGObject rpimotorlib rpi-lgpio psutil spidev requests`

```
pip install scipy
git clone https://github.com/lionfish0/retrodetect.git
cd retrodetect
pip install -e .
git clone https://github.com/lionfish0/QueueBuffer.git
cd QueueBuffer
pip install -e .
pip install libsvm
pip install -U flask-cors
pip install mem_top
pip install flask_compress
```


## Running Beetrack

Make sure you are in venvironment where all the installs have occured

`cd bee_track`

`sudo ifconfig eth0 up 169.254.160.220`
Check cameras are working:
`arv-camera-test-0.10`

`./startupfast`
If something goes wrong use
`killall python3`

Connect to pi with:

http://raspberrypi.local:8000/

## Running Beetrack on Boot automatically
Edit rc.local
`sudo nano /etc/rc.local`
Add the following line:


`su - pi -c /home/pi/bee_track/startup &`


TODO: Change `bee_track/startup` to run on venv we have created.
```

export GI_TYPELIB_PATH=$GI_TYPELIB_PATH:/home/pi/aravis/build/src
export LD_LIBRARY_PATH=/home/pi/aravis/build/src
cd /home/pi/bee_track/webinterface
/home/pi/bee-venv/bin/python3 -m http.server &

sudo ifconfig eth0 up 169.254.160.220

sudo sysctl -w net.core.rmem_max=67108864 net.core.rmem_default=67108864
sudo sysctl -w net.ipv4.route.flush=1
sudo ifconfig eth0 mtu 9000
/home/pi/bee-venv/bin/python3 ../bee_track/core.py

```
The last line runs the application in the virtualenvironment


Add the following to `/etc/network/interfaces`

```
auto lo
iface lo inet loopback

auto eth0
iface eth0 inet static
address 169.254.160.220
```

Reboot and the project should run without any interaction
