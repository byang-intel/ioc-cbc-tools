# ioc-cbc-tools

IOC stands for IO controller which is used for automotive system.
CBC stands for carrier board communiction protocol.

This package will provide the userspace systemd services to:
1. attach CBC Ldisc to a certain tty port
2. bind to /dev/cbc-lifecycle to provide lifecycle support for OS.

The package could work on Native Linux, Guest Linux and also the
Service OS Linux environment supported by ACRN.
