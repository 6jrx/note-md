## 查看GPIO控制器

```bash
$ ls -al /sys/bus/gpio/devices
total 0
drwxr-xr-x 2 root root 0 Jan  1 00:47 .
drwxr-xr-x 4 root root 0 Jan  1 00:47 ..
lrwxrwxrwx 1 root root 0 Jan  1 00:47 gpiochip0 -> ../../../devices/soc0/soc/2000000.aips-bus/209c000.gpio/gpiochi0
lrwxrwxrwx 1 root root 0 Jan  1 00:47 gpiochip1 -> ../../../devices/soc0/soc/2000000.aips-bus/20a0000.gpio/gpiochi1
lrwxrwxrwx 1 root root 0 Jan  1 00:47 gpiochip2 -> ../../../devices/soc0/soc/2000000.aips-bus/20a4000.gpio/gpiochi2
lrwxrwxrwx 1 root root 0 Jan  1 00:47 gpiochip3 -> ../../../devices/soc0/soc/2000000.aips-bus/20a8000.gpio/gpiochi3
lrwxrwxrwx 1 root root 0 Jan  1 00:47 gpiochip4 -> ../../../devices/soc0/soc/2000000.aips-bus/20ac000.gpio/gpiochi4
lrwxrwxrwx 1 root root 0 Jan  1 00:47 gpiochip5 -> ../../../devices/soc0/spi4/spi_master/spi32766/spi32766.0/gpioc5
```

这里可以查看到有6个控制器

## 查看控制器详细

```bash
$ ls -al /sys/class/gpio/gpiochip*
lrwxrwxrwx 1 root root 0 Jan  1 00:31 gpiochip0 -> ../../devices/soc0/soc/2000000.aips-bus/209c000.gpio/gpio/gpioch0
lrwxrwxrwx 1 root root 0 Jan  1 00:31 gpiochip128 -> ../../devices/soc0/soc/2000000.aips-bus/20ac000.gpio/gpio/gpio8
lrwxrwxrwx 1 root root 0 Jan  1 00:31 gpiochip32 -> ../../devices/soc0/soc/2000000.aips-bus/20a0000.gpio/gpio/gpioc2
lrwxrwxrwx 1 root root 0 Jan  1 00:31 gpiochip504 -> ../../devices/soc0/spi4/spi_master/spi32766/spi32766.0/gpio/gp4
lrwxrwxrwx 1 root root 0 Jan  1 00:31 gpiochip64 -> ../../devices/soc0/soc/2000000.aips-bus/20a4000.gpio/gpio/gpioc4
lrwxrwxrwx 1 root root 0 Jan  1 00:31 gpiochip96 -> ../../devices/soc0/soc/2000000.aips-bus/20a8000.gpio/gpio/gpioc6
```

## 控制器结构

```bash
$ ls -al
total 0
drwxr-xr-x 3 root root    0 Jan  1 00:00 .
drwxr-xr-x 3 root root    0 Jan  1 00:00 ..
-r--r--r-- 1 root root 4096 Jan  1 00:55 base    # 基值
lrwxrwxrwx 1 root root    0 Jan  1 00:55 device -> ../../../209c000.gpio
-r--r--r-- 1 root root 4096 Jan  1 00:55 label   # 名字
-r--r--r-- 1 root root 4096 Jan  1 00:55 ngpio   # 引脚个数
drwxr-xr-x 2 root root    0 Jan  1 00:55 power
lrwxrwxrwx 1 root root    0 Jan  1 00:55 subsystem -> ../../../../../../../class/gpio
-rw-r--r-- 1 root root 4096 Jan  1 00:00 uevent
```

## 查看使用情况

```bash
$ cat /sys/kernel/debug/gpio
```
