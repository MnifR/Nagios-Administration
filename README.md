<p align="center">
  <a href="https://www.nagios.org/projects/nagios-core/">
    <img src="https://www.nagios.org/wp-content/uploads/2015/06/Nagios-Logo.jpg" alt="Logo" width=250 height=150>
  </a>

  <p align="center">
    Nagios Administration
    <br>
    
  </p>
</p>


## Table of contents

- [What is Nagios](#What-is-Nagios)
- [Install](#install)
- [What's included](#whats-included)
- [Bugs and feature requests](#bugs-and-feature-requests)
- [Contributing](#contributing)
- [Creators](#creators)
- [Thanks](#thanks)
- [Copyright and license](#copyright-and-license)


## What is Nagios

Nagios is an open source continuous monitoring tool which monitors network, applications and servers. It can find and repair problems detected in the infrastructure, and stop future issues before they affect the end users. It gives the complete status of your IT infrastructure and its performance.

## Install

Installation on ubnutu 20.04

Update the Ubuntu repository and install some packages dependencies for the Nagios installation.

```shell
sudo apt update
```

Run the following command to install pre-required packages

```shell
sudo apt install -y autoconf bc gawk dc build-essential gcc libc6 make wget unzip apache2 php libapache2-mod-php libgd-dev libmcrypt-dev make libssl-dev snmp libnet-snmp-perl gettext
```
Download the latest Nagios package

```shell
cd ~/
wget https://github.com/NagiosEnterprises/nagioscore/archive/nagios-4.4.6.tar.gz
```

Extract the tarball file

```shell
tar -xf nagios-4.4.6.tar.gz
cd nagioscore-*/
```

Compile Nagios source code and define the Apache virtual host configuration for Nagios

```shell
sudo ./configure --with-httpd-conf=/etc/apache2/sites-enabled
sudo make all
```
Create user and group for Nagios and add them to Apache www-data user

```shell
sudo make install-groups-users
sudo usermod -a -G nagios www-data
```
Install Nagios binaries, service daemon script, the command and the sample script configuratio.

```shell
sudo make install
sudo make install-daemoninit
sudo make install-commandmode
sudo make install-config
```
Install the Apache configuration for Nagios, activate the mod_rewrite, mode_cgi modules and configure a user nagiosadmin..

```shell
sudo make install-webconf
sudo a2enmod rewrite cgi
sudo systemctl restart apache2
sudo htpasswd -c /usr/local/nagios/etc/htpasswd.users nagiosadmin
```
Install the Nagios Plugins and NRPE Plugins

```shell
sudo apt install monitoring-plugins nagios-nrpe-plugin
```
Create a new directory for for storing all server hosts configuration

```shell
cd /usr/local/nagios/etc
mkdir -p /usr/local/nagios/etc/servers
```
Edit the Nagios configuration 'nagios.cfg' using vim editor.

```shell
vim nagios.cfg
```
Uncomment the 'cfg_dir' option that will be used for sotring all server hots configurations.

```shell
cfg_dir=/usr/local/nagios/etc/servers
```

Edit the configuration file "resource.cfg" and define the path binary files of Nagios Monitoring Plugins

```shell
vim resource.cfg

//Define the Nagios Monitoring Plugins path by changing the default configuration as below

$USER1$=/usr/lib/nagios/plugins
```

Edit contacts configuration file

```shell
vim objects/contacts.cfg

//Change the email address with your own.

define contact{
        ......
        email             email@host.com
}
```
Now define the nrpe check command by editing the configuration file "objects/commands.cfg"


```shell
vim objects/commands.cfg

//Add the following configuration to the end of the line

define command{
        command_name check_nrpe
        command_line $USER1$/check_nrpe -H $HOSTADDRESS$ -c $ARG1$
}
```

Start the Nagios service and add it to the system boot

```shell
sudo systemctl start nagios
sudo systemctl enable nagios
sudo systemctl status nagios
sudo systemctl restart apache2
```

Add Linux Host to Monitor

Log in to the "Hostname" server using your ssh

```shell
ssh root@<ip-adress-client>
sudo apt update
sudo apt install nagios-nrpe-server monitoring-plugins
```

Next, go to the NRPE installation directory "/etc/nagios" and edit the configuration file "nrpe.cfg"

```shell
cd /etc/nagios/
vim nrpe.cfg
```

Uncomment the "server_address" line and change the value with the "<ip-adress-client>"

```shell
server_address=<ip-adress-client>

//One the "allowed_hosts" line, add the Nagios Server IP address "<ip-adress-server>"

allowed_hosts=127.0.0.1,::1,<ip-adress-server>

```

Edit the "nrpe_local.cfg" configuration

```shell
vim nrpe_local.cfg

Change the IP address with the "<ip-adress-client>", and paste the configuration into it

command[check_root]=/usr/lib/nagios/plugins/check_disk -w 20% -c 10% -p /
command[check_ping]=/usr/lib/nagios/plugins/check_ping -H <ip-adress-client> -w 100.0,20% -c 500.0,60% -p 5
command[check_ssh]=/usr/lib/nagios/plugins/check_ssh -4 <ip-adress-client>
command[check_http]=/usr/lib/nagios/plugins/check_http -I <ip-adress-client>
command[check_apt]=/usr/lib/nagios/plugins/check_apt

```

Restart the NRPE service and add it to the system boot

```shell

systemctl restart nagios-nrpe-server
systemctl enable nagios-nrpe-server
systemctl status nagios-nrpe-server
```


Next, back to the Nagios Server and check the "client01" NRPE server.
```shell
/usr/lib/nagios/plugins/check_nrpe -H <ip-adress-client>
/usr/lib/nagios/plugins/check_nrpe -H <ip-adress-client> -c check_ping

```

Add Hosts Configuration to the Nagios Server

Back to the Nagios server terminal, go to the "/usr/local/nagios/etc" directory and create a new configuration "server/client01.cfg"

```shell
cd /usr/local/nagios/etc
vim servers/client01.cfg

```

Change the IP address and the hostname with your own and paste the configuration into it.


# Ubuntu Host configuration file1
```shell
define host {
        use                          linux-server
        host_name                    client01
        alias                        Ubuntu Host
        address                      <ip-adress-client>
        register                     1
}

define service {
      host_name                       client01
      service_description             PING
      check_command                   check_nrpe!check_ping
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       client01
      service_description             Check Users
      check_command                   check_nrpe!check_users
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       client01
      service_description             Check SSH
      check_command                   check_nrpe!check_ssh
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       client01
      service_description             Check Root / Disk
      check_command                   check_nrpe!check_root
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       client01
      service_description             Check APT Update
      check_command                   check_nrpe!check_apt
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}

define service {
      host_name                       client01
      service_description             Check HTTP
      check_command                   check_nrpe!check_http
      max_check_attempts              2
      check_interval                  2
      retry_interval                  2
      check_period                    24x7
      check_freshness                 1
      contact_groups                  admins
      notification_interval           2
      notification_period             24x7
      notifications_enabled           1
      register                        1
}
```
Now restart the Nagios Server
```shell
systemctl restart nagios
```
## What's included

Some text

```text
folder1/
└── folder2/
    ├── folder3/
    │   ├── file1
    │   └── file2
    └── folder4/
        ├── file3
        └── file4
```

## Bugs and feature requests

Have a bug or a feature request? Please first read the [issue guidelines](https://reponame/blob/master/CONTRIBUTING.md) and search for existing and closed issues. If your problem or idea is not addressed yet, [please open a new issue](https://reponame/issues/new).

## Contributing

Please read through our [contributing guidelines](https://reponame/blob/master/CONTRIBUTING.md). Included are directions for opening issues, coding standards, and notes on development.

Moreover, all HTML and CSS should conform to the [Code Guide](https://github.com/mdo/code-guide), maintained by [Main author](https://github.com/usernamemainauthor).

Editor preferences are available in the [editor config](https://reponame/blob/master/.editorconfig) for easy use in common text editors. Read more and download plugins at <https://editorconfig.org/>.

## Creators

