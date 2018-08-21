# APL Thermal

## Introduction
The generic thermal sysfs provides a set of interfaces for thermal zone devices (sensors) and thermal cooling devices (fan, processor...).
On APL, it has CBC Thermal utility, which provides interfaces to access temperature sensors and cooling devices on IO Controller.
This how-to focuses on enabling intel thermal daemon for APL thermal management.

### Thermal Zones
Thermal zone, by definition, not only gives the temperature reading of a thermal sensor, but also binding cooling devices associated with trip points.
### Thermal sensors
Thermal sensor is used to take temperature measurements for specific thermal zone.
### Cooling devices
Cooling device is used to dissipate heat for specific thermal zone.
### Trip points
Trip point describes key temperatures at which cooling is recommended.
### Sysfs I/F
```
/sys/class/thermal/thermal_zone[0-*]:
    |---type:			Type of the thermal zone
    |---temp:			Current temperature
    |---trip_point_[0-*]_temp:	Trip point temperature
    |---trip_point_[0-*]_type:	Trip point type
    |---cdev[0-*]:		[0-*]th cooling device in current thermal zone
    |---cdev[0-*]_trip_point:	Trip point that cdev[0-*] is associated with

/sys/class/thermal/cooling_device[0-*]:
    |---type:			Type of the cooling device(processor/fan/...)
    |---max_state:		Maximum cooling state of the cooling device
    |---cur_state:		Current cooling state of the cooling device
```
### Thermal daemon
Thermald is a Linux daemon used to prevent the overheating of platforms before HW takes aggressive correction action.
Thermal daemon looks for thermal sensors, thermal cooling devices and trip points in the linux thermal sysfs and builds a list of sensors, cooling devices and trip points. Each of the thermal sensors can optionally be binded to a cooling device. In this case, thermald can take actions based on the trip points for each sensor and associate cooling device.
Thermal daemon allows users to change this relationship or add new one via a thermal configuration file (thermal-conf.xml).

```
                       Thermal management architecture

 +-----------------------------------------------------------------------------+
 |                           Thermal Daemon                                    |
 +-----------------------------------------------------------------------------+
                                    ^
                                    |
USER SPACE                          |
 ~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~|~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~
                                    |
                                    |
KERNEL SPACE                        v
 +-----------------------------------------------------------------------------+
 |                               sysfs I/F                                     |
 +-----------------------------------------------------------------------------+
         ^                                       ^
         |                                       |
         |                                       |
         v                                       v
 +-----------------+     +-------------+   +-------------+
 | thermal drivers |<--->| cpu drivers |   | RAPL drivers|     ... ...
 +-----------------+     +-------------+   +-------------+
         ^                     ^                 ^
         |                     |                 |
         |                     |                 |
         v                     v                 v
 +-----------------------------------------------------------------------------+
 |                      Platform hardware, BIOS, Firmware                      |
 +-----------------------------------------------------------------------------+
```

## APL Thermals
### acpitz
ACPI thermal zone, which is provided by ACPI thermal table. ACPI always assumes that the CPU is the major thermal contributor and Processor/Fan are the major cooling devices. ACPI also defines serveral trip points. Thermal daemon will scan it from below sysfs I/F:

```
/sys/class/thermal/thermal_zone0/type
/sys/class/thermal/thermal_zone0/temp
/sys/class/thermal/thermal_zone0/cdev*
/sys/class/thermal/thermal_zone0/trip_point_*
...
```
### Fan cooling devices
The ACPI fan driver supports only two cooling states: state 0 means the fan is off, state 1 means the fan is on. Normally, it will be used by acpitz. Thermal daemon will scan it from below sysfs I/F:
```
/sys/class/thermal/cooling_device0/type
/sys/class/thermal/cooling_device0/max_state
/sys/class/thermal/cooling_device0/cur_state
...
```
### Processor cooling devices
ACPI processor cooling state is a combination of the processor P-state and T-state. The ACPI CPU frequency driver prefers to reduce the frequency first, and then to throttle. Normally, processor cooling devices will be used by acpitz. Thermal daemon will scan it from below sysfs I/F:
```
/sys/class/thermal/cooling_device[1-cpunum]/type
/sys/class/thermal/cooling_device[1-cpunum]/max_state
/sys/class/thermal/cooling_device[1-cpunum]/cur_state
...
```
### x86_pkg_temp
Each x86 package will register as a thermal zone under /sys/class/thermal. By default, it has no cooling device and trip point enabled. User can define trip point in thermald config file to use it.
### cbc_env_temp
This thermal sensor is provided by IOC with CBC protocol. The temperature value can be read from /run/cbc_thermal/cbc_env_temp.
### cbc_amplifier_temp
This thermal sensor is provided by IOC with CBC protocol. The temperature value can be read from /run/cbc_thermal/cbc_amplifier_temp
### cbc_ambient_temp
This thermal sensor is provided by IOC with CBC protocol. The temperature value can be read from /run/cbc_thermal/cbc_ambient_temp
### CBC cooling devices
### cbc_fan0
This cooling device is provided by IOC with CBC protocol. The fun duty cycle [0-100] can be read/written from/to /run/cbc_thermal/cbc_fan0
## Tuning User Guide
### user defined zone

