# Sensorgnome security and networking

{% hint style="info" %}
This document is intended for security/network admins to provide an overview of the security and communication methods employed by a Sensorgnome devic&#x65;**.**
{% endhint %}

The Sensorgnome (SG) software is open source and available for audit at:

* https://github.com/tve/sensorgnome-control (web interface and uploader)
* https://github.com/tve/sensorgnome-support (system services)
* https://github.com/tve/sensorgnome-build (system image build)

The overall strategy employed can be summarized as follows:

* leverage as standard an OS image as possible
* limit protocols to SSH and HTTPS
* enforce the selection of a reasonably strong password at configuration time
* use a single password for the system to avoid "password fatigue"

### Operating system image and updates

* The SG base image is a standard Raspberry Pi OS image, i.e. Debiean Bullseye as of 2022.
* The image uses the standard repositories, plus and additional repository for the Sensorgnome software.
* The Raspberry Pi foundation does not publish security updates like Ubuntu does, for example. Thus it is not possible to auto-install just security updates. However, security fixes are promptly made available in the general repo. It is thus possible to enable automatic "full" updates, but the system stability of doing so is unknown.

### Passwords

* Sensorgnomes use a non-standard 'gnome' linux user and users are forced to provide a password when first deploying a Sensorgnome (the software does not work unless a password is set).
* The password is checked against a "top 100k passwords" list.
* It is possible to use SSH keys and disable password login using standard linux procedures.
* The same password is used to log in as user `gnome` and to access the web interface (the web interface uses a PAM plugin to auth)

### Network interfaces

* The Sensorgnome supports or four network interfaces: ethernet, wifi client, wifi hot-spot (access point), and, if installed, cellular.
* The SG never forwards (routes) packets between the interfaces.
* The ethernet interface uses DHCP and is the preferred interface for internet access.
* The wifi client interface can be configured through the SG web interface and supports WPA2 PSK. It should be possible to configure WPA2 EAP or WPA3, but not through the web interface. The interface expects a DHCP server for configuration.
* The wifi hot-spot can be enabled/disabled through the web interface or by pressing a button on the SG. It runs a WPA2 secured network, the password is configured when first deploying a Sensorgnome.
* The cellular interface currently supports only the Sixfab LTE HAT.

### Data upload

* The SG uploads new data automatically approximately every hour via HTTPS to a Motus server (port 443). This assumes internet connectivity via one of the ethernet, wifi client, or cellular interfaces.
* Network connectivity (when a default route is provided via DHCP) is probed by connecting to http://connectivitycheck.gstatic.com/generate\_204, which is one of the standard Android connectivity check addresses.
* Users can manually trigger an upload using the web interface.
* Users can also manually download data to their laptop/phone via the web interface.
* It is planned for the Sensorgnome to connect to a Motus server to send status and monitoring data and to present a portion of the web interface through a Motus web site.

### Open ports

* port 22 (SSH): SSH access, root login disabled, only user 'gnome' access.
* port 80 (HTTP): redirect to port 443 (HTTPS), except on the hot-spot interface where the Web UI is accessible via HTTP (security being provided by WiFi WPA2).
* port 443 (HTTPS): web interface to monitor Sensorgnome status and perform configuration changes.

### HTTPS certificates

* Due to the fact that a Sensorgnome needs to operate without having a public DNS name it cannot use "normal" public HTTPS certificates (e.g. from Let's Encrypt). Instead a wild-card certificate `*.my.local-ip.co` is used and HTTPS links have the form `https://1-2-3-4.my.local-ip.co` where `1.2.3.4` is the SG's local IP address. See [http://local-ip.co](http://local-ip.co) for details and rationale.
