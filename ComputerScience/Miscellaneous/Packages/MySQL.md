## Install

My enviroment:

- Architecture: x86_64
- OS: Ubuntu 22.04 LTS

Update packages then install mysql:

```shell
sudo apt update
sudo apt upgrade
sudo apt install mysql-server
```

Test if it works and check version:

```shell
sudo systemctl enable mysql
sudo systemctl start mysql
sudo systemctl status mysql
mysql -V
```

## Security

````ad-warning
title: **\[Warning\]** 🪓 Before runing *mysql_secure_installation* script
An [error](https://www.digitalocean.com/community/tutorials/how-to-install-mysql-on-ubuntu-22-04#step-2-configuring-mysql) occurs as of July 2022, when you run_mysql_secure_installation_ script without some further configuration.
Login to mysql as root without providing password:

```shell
sudo mysql -uroot
```

Enter the following command to set root's password with the `<password>` replaced by your password:

```SQL
ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '<password>';
```

Exit:

```SQL
exit
```
````

Enhance your database security by running *mysql_secure_installation* script:

```shell
sudo mysql_secure_installation
```

choose the options:

```
- Set root password? [Y/n] y
- Remove anonymous users? [Y/n] y
- Disallow root login remotely? [Y/n] y
- Remove test database and access to it? [Y/n] y
- Reload privilege tables now? [Y/n] y
```

Then you can login to mysql as root with password:

```shell
mysql -u root -p
```