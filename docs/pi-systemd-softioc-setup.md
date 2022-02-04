# First setup of systemd-ioc
You will download and perform first setup of an IOC management tool called "systemd-ioc".

This tool enables easy management of multiple IOCs across multiple computers. It also allows the IOC to start automatically on system boot.

## Background
Systemd-iocs is developed by Hu Yong of the the National Synchrotron Light Source II. It was recommended to me by the developer in the EPICS user forums. Other alternatives for remote management of IOCs, also suggested in the forum included tools like `tmux` and `procServ`.

## References
1. [Systemd-SoftIOC GitHub Page](https://github.com/NSLS-II/systemd-softioc)
1. [Ask Ubuntu: How to Allow a Non-Default User Access TTYUSB0](https://askubuntu.com/questions/112568/how-do-i-allow-a-non-default-user-to-use-serial-device-ttyusb0)
1. [What does the 's' flag mean in chmod?](https://unix.stackexchange.com/questions/182212/chmod-gs-command)

## Pre-requisites
* Internet connection

## Materials
* Raspberry Pi with EPICS installed

## Steps
### Create a system folder for your IOCs
You will create a new folder in your root directory specifically for the IOCs you want systemd-iocs to manage.

```
sudo mkdir -p /epics/iocs
```

### Create a new user, `softioc`, with IOC-relevant permissions
Even though you are about to create a new user in your Linux instance, stay logged in as your main user, `pi`, for all of the steps in this guide.

Create a new user called `softioc`. This user will be utilized by systemd-softioc, to run your software-based IOCs.

```
sudo useradd softioc
sudo chown softioc:softioc /epics/iocs
sudo chmod g+ws /epics/iocs
```

Above, the `useradd` command creates a new group called `softioc`, of which the new user `softioc` is the only member. The `chown` command transfers ownership of `/epics/iocs` (the system folder you created for IOCs) to your new user, `softioc`. The `chmod g+ws` command ensures that future folders and files copied into `/epics/iocs` will be attributed to `softioc` by default, rather than being attributed to the user that created this folder.

### Give the `softioc` user permissions to access to the Arduino USB connection

Tip: You can see all current group permissions for a user with `sudo groups [username]`. You may want to check what results when you type `sudo groups pi`, for the pi user. If you are using other connections in the future besides Arduinos connected via USB, you may want to add other important groups you notice besides `dialout` to your `softioc` user. You can do this by repeating the steps above for the additional group.

##### (Optional) Plug in the Arduino, then confirm that the ttyUSB0 port is in the dialout group.
```bash
sudo ls -ld /dev/ttyUSB0
```
The above should respond with something like:
```bash
crw-rw---- 1 root dialout 188, 0 Feb  4 10:10 /dev/ttyUSB0
```
Note above that the group affiliation of the Arduino connection (/dev/ttyUSB0) is listed as `dialout`.

##### Once you've confirmed the Arduino's affiliation, add the `softioc` user to the `dialout` group.
```bash
sudo adduser softioc dialout
```


### Download and install software dependencies for systemd-softioc
procServ and telnet are two dependencies for systemd-softioc. Install them both via:

```bash
sudo apt install procserv telnet
```

### Download and install systemd-softioc
In any folder of your choice, download the systemd-softioc package.

```bash
git clone https://github.com/NSLS-II/systemd-softioc
```