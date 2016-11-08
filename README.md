# node-folder-mirroring
A FUSE filesystem for local directory mirroring written in Node.js. 

Usage: 

```
$ sudo node node-folder-mirroring.js <source_dir> <target_dir> 
```

## Mirroring the GPIO folder on a Raspberry PI 2
This tool has been tested on a Raspberry PI 2 Model B V1.1 with the aim of mirroring the GPIO folder. The distribution was Raspbian Jessie Lite (version 2016-09-23). 

### Update and upgrade the system: 
```
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot 
```

### Install Node.js, FUSE, and other dependencies of mirror.js
```
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ sudo apt install nodejs
$ sudo apt install fuse libfuse-dev
$ mkdir /home/pi/test_gpio_mirroring
$ cd /home/pi/test_gpio_mirroring
$ npm install fuse-bindings fs mknod path
$ npm install https://github.com/PlayNetwork/node-statvfs/tarball/v3.0.0
```

### Check GPIO
```
$ echo "4" > /sys/class/gpio/export
$ echo "out" > /sys/class/gpio/gpio4/direction
$ echo "1" > /sys/class/gpio/gpio4/value
$ echo "4" > /sys/class/gpio/unexport
```

### Mirror the GPIO folder
```
$ mkdir -p /home/pi/test_gpio_mirroring/sys/class/gpio
$ mkdir /home/pi/test_gpio_mirroring/sys/devices
$ wget https://raw.githubusercontent.com/flongo82/node-folder-mirroring/master/node-folder-mirroring.js
$ node node-folder-mirroring.js /sys/devices/ /home/pi/test_gpio_mirroring/sys/devices/ &> log_devices &
$ node node-folder-mirroring.js /sys/class/gpio/ /home/pi/test_gpio_mirroring/sys/class/gpio/ &> log_gpio &
```

### Test the mirror folder
```
$ echo "4" > /home/pi/test_gpio_mirroring/sys/class/gpio/export
$ echo "out" > /home/pi/test_gpio_mirroring/sys/class/gpio/gpio4/direction
$ echo "1" > /home/pi/test_gpio_mirroring/sys/class/gpio/gpio4/value
```
