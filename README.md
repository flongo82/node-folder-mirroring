# node-folder-mirroring
A FUSE filesystem for local directory mirroring written in Node.js. 

Usage: 

```
$ sudo node node-folder-mirroring.js <source_dir> <target_dir> [-o <option> ...]
```

## Mirroring the GPIO folder on a Raspberry PI 2
This tool has been tested on a Raspberry PI 2 Model B V1.1 with the aim of mirroring the GPIO folder. The tests have been performed with both the Raspbian Jessie Lite distribution and the Ubuntu 16.04 LTS distribution.

Note that the fuse-bindings library on which we base our implementation does not support the poll() POSIX call. Thus, at the moment, it is not possible to wait for some event on a file descriptor. This means that, for example, the watch() call of the [onoff library](https://github.com/fivdi/onoff) does not currently work on the mirrored GPIO folder. 

### Raspbian Jessie Lite

#### Update and upgrade the system:
```
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot
```

#### Install Node.js, FUSE, and other dependencies of node-folder-mirroring.js
```
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ sudo apt install nodejs
$ sudo apt install fuse libfuse-dev
$ mkdir /home/pi/test_gpio_mirroring
$ cd /home/pi/test_gpio_mirroring
$ npm install fuse-bindings fs mknod path minimist
$ npm install https://github.com/PlayNetwork/node-statvfs/tarball/v3.0.0
```

#### Check GPIO
```
$ echo "4" > /sys/class/gpio/export
$ echo "out" > /sys/class/gpio/gpio4/direction
$ echo "1" > /sys/class/gpio/gpio4/value
$ cat /sys/class/gpio/gpio4/value
$ echo "4" > /sys/class/gpio/unexport
```

#### Mirror the GPIO folder
```
$ mkdir -p /home/pi/test_gpio_mirroring/sys/class/gpio
$ mkdir /home/pi/test_gpio_mirroring/sys/devices
$ wget https://raw.githubusercontent.com/flongo82/node-folder-mirroring/master/node-folder-mirroring.js
$ node node-folder-mirroring.js /sys/devices/ /home/pi/test_gpio_mirroring/sys/devices/ &> log_devices &
$ node node-folder-mirroring.js /sys/class/gpio/ /home/pi/test_gpio_mirroring/sys/class/gpio/ &> log_gpio &
```

#### Test the mirror folder
```
$ echo "4" > /home/pi/test_gpio_mirroring/sys/class/gpio/export
$ echo "out" > /home/pi/test_gpio_mirroring/sys/class/gpio/gpio4/direction
$ echo "1" > /home/pi/test_gpio_mirroring/sys/class/gpio/gpio4/value
$ cat /home/pi/test_gpio_mirroring/sys/class/gpio/gpio4/value
$ echo "4" > /home/pi/test_gpio_mirroring/sys/class/gpio/unexport
```

### Ubuntu 16.04 LTS

#### Update and upgrade the system:
```
$ sudo apt update
$ sudo apt upgrade
$ sudo reboot
```

#### Install Node.js, FUSE, and other dependencies of node-folder-mirroring.js
```
$ sudo apt install fuse libfuse-dev pkg-config python2.7
$ curl -sL https://deb.nodesource.com/setup_7.x | sudo -E bash -
$ sudo apt install nodejs
$ mkdir /home/ubuntu/test_gpio_mirroring
$ cd /home/ubuntu/test_gpio_mirroring
$ npm config set python `which python2.7`
$ npm install fuse-bindings fs mknod path minimist
$ npm install https://github.com/PlayNetwork/node-statvfs/tarball/v3.0.0
```

#### Allow ubuntu user to access the GPIO folder
```
$ sudo addgroup gpio
$ sudo usermod -a -G gpio ubuntu
$ cat<<EOF | sudo tee /etc/udev/rules.d/99-gpio.rules
SUBSYSTEM=="input", GROUP="input", MODE="0660"
SUBSYSTEM=="i2c-dev", GROUP="i2c", MODE="0660"
SUBSYSTEM=="spidev", GROUP="spi", MODE="0660"
SUBSYSTEM=="bcm2835-gpiomem", GROUP="gpio", MODE="0660"

SUBSYSTEM=="gpio*", PROGRAM="/bin/sh -c '\\
        chown -R root:gpio /sys/class/gpio && chmod -R 770 /sys/class/gpio;\\
        chown -R root:gpio /sys/devices/virtual/gpio && chmod -R 770 /sys/devices/virtual/gpio;\\
        chown -R root:gpio /sys\$devpath && chmod -R 770 /sys\$devpath\\
'"

KERNEL=="ttyAMA[01]", PROGRAM="/bin/sh -c '\\
        ALIASES=/proc/device-tree/aliases; \\
        if cmp -s \$ALIASES/uart0 \$ALIASES/serial0; then \\
                echo 0;\\
        elif cmp -s \$ALIASES/uart0 \$ALIASES/serial1; then \\
                echo 1; \\
        else \\
                exit 1; \\
        fi\\
'", SYMLINK+="serial%c"

KERNEL=="ttyS0", PROGRAM="/bin/sh -c '\\
        ALIASES=/proc/device-tree/aliases; \\
        if cmp -s \$ALIASES/uart1 \$ALIASES/serial0; then \\
                echo 0; \\
        elif cmp -s \$ALIASES/uart1 \$ALIASES/serial1; then \\
                echo 1; \\
        else \\
                exit 1; \\
        fi \\
'", SYMLINK+="serial%c"
EOF
$ sudo reboot
```

#### Check GPIO
```
$ echo "4" > /sys/class/gpio/export
$ echo "out" > /sys/class/gpio/gpio4/direction
$ echo "1" > /sys/class/gpio/gpio4/value
$ cat /sys/class/gpio/gpio4/value
$ echo "4" > /sys/class/gpio/unexport
```

#### Mirror the GPIO folder
```
$ mkdir -p /home/ubuntu/test_gpio_mirroring/sys/class/gpio
$ mkdir /home/ubuntu/test_gpio_mirroring/sys/devices
$ wget https://raw.githubusercontent.com/flongo82/node-folder-mirroring/master/node-folder-mirroring.js
$ node node-folder-mirroring.js /sys/devices/ /home/ubuntu/test_gpio_mirroring/sys/devices/ &> log_devices &
$ node node-folder-mirroring.js /sys/class/gpio/ /home/ubuntu/test_gpio_mirroring/sys/class/gpio/ &> log_gpio &
```

#### Test the mirror folder
```
$ echo "4" > /home/ubuntu/test_gpio_mirroring/sys/class/gpio/export
$ echo "out" > /home/ubuntu/test_gpio_mirroring/sys/class/gpio/gpio4/direction
$ echo "1" > /home/ubuntu/test_gpio_mirroring/sys/class/gpio/gpio4/value
$ cat /home/ubuntu/test_gpio_mirroring/sys/class/gpio/gpio4/value
$ echo "4" > /home/ubuntu/test_gpio_mirroring/sys/class/gpio/unexport
```
