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
This guide assumes you are root or are able to use sudo and you are using Naemon. If you are using op5 or Nagios you will have to use chown to correct owner
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

Now test the script. It's assumed you added your user key and application token to /etc/notify-by-pushover.conf
* sudo su naemon
* notify-via-pushover --sound 'alien' --title 'Aliens' --url 'http://www.imdb.com/title/tt0090605/' --url-title 'Aliens (1986)' --message 'Aliens, one of the best movies ever'
* Verify that you got the notification in your device
Example output:
```
sh-4.1$ notify-via-pushover --sound 'alien' --title 'Aliens' --url 'http://www.imdb.com/title/tt0090605/' --url-title 'Aliens (1986)' --message 'Aliens, one of the best movies ever'
{"status":1,"request":"fa76b13311fd1b5de9df3b9f4443acbc"}
sh-4.1$ 
```

# Configure Naemon
## Add commands
* vi /etc/naemon/conf.d/commands.cfg
Add the following:
```
# 'notify-via-pushover' command-notification
define command {
  command_name                   notify-via-pushover
  command_line                   /usr/local/bin/notify-via-pushover -m "Notification Type: $NOTIFICATIONTYPE$\n\nService: $SERVICEDESC$\nHost: $HOSTALIAS$\nAddress: $HOSTADDRESS$\nState: $SERVICESTATE$\n\nDate/Time: $LONGDATETIME$\n\nAdditional Info:\n\n$SERVICEOUTPUT$" -t "$NOTIFICATIONTYPE$ Service Alert: $HOSTALIAS$/$SERVICEDESC$ is $SERVICESTATE$"
}

# 'host-notify-via-pushover' command-notification
define command {
  command_name                   host-notify-via-pushover
  command_line                   /usr/local/bin/notify-via-pushover -m "Notification Type: $NOTIFICATIONTYPE$\nHost: $HOSTNAME$\nState: $HOSTSTATE$\nAddress: $HOSTADDRESS$\nInfo: $HOSTOUTPUT$\n\nDate/Time: $LONGDATETIME$" -t "$NOTIFICATIONTYPE$ Host Alert: $HOSTNAME$ is $HOSTSTATE$"
}
```
The last step is to add notify-via-pushover to your contact. I'm using the default generic-contact template bundled with Naemon for this example. 

* vi /etc/naemon/conf.d/templates/contacts.cfg
```
define contact {
  name                           generic-contact                     ; The name of this contact template
  host_notification_commands     host-notify-via-pushover
  host_notification_options      d,u,r,f,s                           ; send notifications for all host states, flapping events, and scheduled downtime events
  host_notification_period       24x7                                ; host notifications can be sent anytime
  register                       0                                   ; DONT REGISTER THIS DEFINITION - ITS NOT A REAL CONTACT, JUST A TEMPLATE!
  service_notification_commands  notify-via-pushover
  service_notification_options   w,u,c,r,f,s                         ; send notifications for all service states, flapping events, and scheduled downtime events
  service_notification_period    24x7                                ; service notifications can be sent anytime
}
```
The bundled naemonadmin contact user are using this template as default. You can test the setup by browsing your Naemon server and browse to a host or service and click on the "Send custom host notification" or "Send custom service notification"
