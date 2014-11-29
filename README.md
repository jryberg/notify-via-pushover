# About notify-via-pushover
I wanted to be able to send push notifications to a different set if hand held devices and I found Pushover.net, a great service that support both Android, iPhone, iPad and desktop clients. I also found a lot of different tools already created for Naemon, OP5, Nagios, Icinga and other monitoring solutions but all had one or more flaws. The two biggest flaws was security and lack of newline support. Credentials should not be used as arguments since they often are stored in files with poor access control, potentially visible by many. Newline support are also needed to be able to read the notification and this was missing on all scripts I had tested. 

notify-via-pushover are highly customizable and support almost the entire Pushover API (https://pushover.net/api). Configuration files can be used for different default values. Configuration files can also be skipped entirely by using arguments.

This tool was initialy written to be used with Naemon and OP5 to send notifications but it not hard to adopt this script to fit other areas as well.

# Supported OS
This bash script has been tested under OSX 10.10, Ubuntu 14.04 and CentOS 6.6. It should be very generic and work in most environments.

# Prerequirements
* curl or wget
* Pushover account - https://pushover.net/

# Installation
## CentOS
This guide assumes you are root or are able to use sudo and you are using naemon. If you are using op5 or Nagios you will have to use chown to correct owner
* cd
* mkdir notify-via-pushover
* cd notify-via-pushover
* wget https://raw.githubusercontent.com/jryberg/notify-via-pushover/master/notify-via-pushover
* wget https://raw.githubusercontent.com/jryberg/notify-via-pushover/master/notify-via-pushover.conf
* mv notify-via-pushover /usr/local/bin
* mv notify-via-pushover.conf /etc
* chown naemon:naemon /usr/local/bin/notify-via-pushover
* chown naemon:naemon /etc/notify-via-pushover.conf
* chmod 600 /etc/notify-via-pushover.conf
* chmod 755 /usr/local/bin/notify-via-pushover
* restorecon /usr/local/bin/notify-via-pushover
* cd ..
* rm -rf notify-via-pushover

notify-by-pushover.conf are not required, it's possible to ignore the configuration file and use arguments instead but for  security reasons you probably want to store credentials in the configuration and not use them as arguments.

Edit /etc/notify-via-pushover.conf and customize it as you need.
