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

