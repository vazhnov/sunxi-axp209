# Get information from AXP209

This script was originally created for CHIP single-board computer.

Shell script for reading voltage & current from the AXP209 Power Management IC for the CHIP $9 computer
* Battery charge %
* Battery voltage & current
* USB voltage & current
* "ACIN" (Pin 2 of U13) voltage & current
* and more...

## WARNING

Some of the voltage outputs are programmable via registers, so it appears to be possible to damage your CHIP computer by writing the wrong data to certain registers on this chip.

This script only uses one write command to enable ADC registers, everything else is only reading registers.

## Examples

```
root@chip:~# axp209.sh
Battery: 92%
ACIN:	0V
VBUS:	4.8892V   899.625mA
VBAT:	4.2075 V  Charging at 633.0mA
Vout:	4.3876 V
```

with `-a` switch:

```
root@chip:~# axp209.sh -a
              ACIN: 0	Avail: 0
              VBUS: 1	Avail: 1
             VHOLD: 1 (Whether VBUS is above 4.4V before being used)
  Shutdown voltage: 2.9V
VBUS current limit: 900mA
Boot source isn't ACIN/VBUS
Battery charging
Battery connected: 1
Temperature:	56.8°C (I've seen as high as 65°C)
Battery: 92%
ACIN:	0V
VBUS:	4.8858V   899.625mA
VBAT:	4.2119 V  Charging at 633.0mA
Vout:	4.5962 V
```

**100mA USB current limit** (causing the AXP209 to power the CHIP from both USB and the battery):

```
root@chip:~# axp209.sh -a
              ACIN: 0	Avail: 0
              VBUS: 1	Avail: 1
             VHOLD: 1 (Whether VBUS is above 4.4V before being used)
  Shutdown voltage: 2.9V
VBUS current limit: 100mA
Boot source isn't ACIN/VBUS
Battery charging
Battery connected: 1
Temperature:	45.0°C (I've seen as high as 65°C)
Battery: 99%
ACIN:	0V
VBUS:	5.1442V   100.500mA
VBAT:	4.1371 V  Discharging at 68.5mA
Vout:	4.1006 V
```

## Compatibility

The script also works with [Olimex A20-OLinuXino-LIME2](https://www.olimex.com/Products/OLinuXino/A20/A20-OLinuXino-LIME2/) (Allwinner A20 + AXP209) with [official Olimex Debian image](https://images.olimex.com/release/a20/).

## Another scripts

* https://github.com/Photonicsguy/CHIP — original script, MIT license
* https://github.com/mzhboy/sunxi-axp209 — first changed fork, MIT license
* https://gist.github.com/Jooshboy/68793914490b90b1cdd8 — very simple script, unknown license
* https://gist.github.com/andreibosco/9709ea89492e5bdd931d841999a7910c — more complicated script, unknown license
* https://forum.armbian.com/topic/45-battery/?do=findComment&comment=350, no license/proprietary

## Another methods

### Armbian

Armbian 21.02.3, Linux kernel `Linux lime2 5.10.21-sunxi #21.02.3 SMP Mon Mar 8 00:28:04 UTC 2021 armv7l GNU/Linux`:

Script `/etc/update-motd.d/30-armbian-sysinfo` gives such output:

```
lime2:~:% bash /etc/update-motd.d/30-armbian-sysinfo
System load:   5%           	Up time:       12 min
Memory usage:  6% of 998M   	IP:            192.168.1.182
CPU temp:      39°C           	Usage of /:    12% of 9.8G   	Battery:       91% discharging
```

In this script, directory `/sys/power/axp_pmu` named as `mainline_dir`, while `/sys/class/power_supply` is `legacy_dir`.

```
lime2:~:% ls -lAF /sys/class/power_supply/axp20x-battery/
total 0
-r--r--r-- 1 root root 4096 Apr 20 22:32 capacity
-rw-r--r-- 1 root root 4096 Apr 20 22:32 constant_charge_current
-rw-r--r-- 1 root root 4096 Apr 20 22:32 constant_charge_current_max
-r--r--r-- 1 root root 4096 Apr 20 22:32 current_now
lrwxrwxrwx 1 root root    0 Apr 20 22:32 device -> ../../../axp20x-battery-power-supply/
-r--r--r-- 1 root root 4096 Apr 20 22:32 health
drwxr-xr-x 3 root root    0 Apr 20 22:23 hwmon0/
-r--r--r-- 1 root root 4096 Apr 20 22:32 online
drwxr-xr-x 2 root root    0 Apr 20 22:23 power/
-r--r--r-- 1 root root 4096 Apr 20 22:32 present
-r--r--r-- 1 root root 4096 Apr 20 22:32 status
lrwxrwxrwx 1 root root    0 Apr 20 22:23 subsystem -> ../../../../../../../../../class/power_supply/
-r--r--r-- 1 root root 4096 Apr 20 22:32 type
-rw-r--r-- 1 root root 4096 Apr 20 22:23 uevent
-rw-r--r-- 1 root root 4096 Apr 20 22:32 voltage_max_design
-rw-r--r-- 1 root root 4096 Apr 20 22:32 voltage_min_design
-r--r--r-- 1 root root 4096 Apr 20 22:25 voltage_now
-r--r--r-- 1 root root 4096 Apr 20 22:32 voltage_ocv
drwxr-xr-x 2 root root    0 Apr 20 22:23 wakeup4/

lime2:~:% ls -lAF /sys/power/axp_pmu/battery/
total 0
-r--r--r-- 1 root root 4096 Apr 20 22:31 amperage
-r--r--r-- 1 root root 4096 Apr 20 22:24 capacity
-r--r--r-- 1 root root 4096 Apr 20 22:31 charge
-r--r--r-- 1 root root 4096 Apr 20 22:31 charging
-r--r--r-- 1 root root 4096 Apr 20 22:24 connected
-r--r--r-- 1 root root 4096 Apr 20 22:31 power
-r--r--r-- 1 root root 4096 Apr 20 22:31 ts_voltage
-r--r--r-- 1 root root 4096 Apr 20 22:31 voltage
```

The values are slightly different:

```
lime2:~:% cat /sys/power/axp_pmu/battery/voltage /sys/class/power_supply/axp20x-battery/voltage_now
3977600
3977000

lime2:~:% cat /sys/class/power_supply/axp20x-battery/current_now /sys/power/axp_pmu/battery/amperage
485000
489500
```

Loaded kernel modules:

```
lime2:~:% lsmod| grep axp
axp20x_battery         16384  0
axp20x_adc             16384  0
industrialio           53248  3 axp20x_battery,sun4i_gpadc_iio,axp20x_adc
pinctrl_axp209         16384  0
```

## TODO

* Try `/proc/acpi/battery/BAT0`


## References

* https://docs.getchip.cc/ — CHIP Documentation
* https://linux-sunxi.org/AXP209 — information on chip registers
