# Born2beroot

# Resumo
Apos instalacao do debian.

```su -``` --> login como root

apt install sudo -->
adduser <username> sudo --> Add usuario ao sudo
sudo apt update -y --> Atualizando para versao mais recente
sudo visudo --> Abrir Arquivos sudoers
Em # User privilege specification, acrescentamos apos o root, your_username  	ALL=(ALL) ALL
sudo addgroup user42 --> Criando grupo user42
sudo adduser <username> user42 --> Adicionando usuario ao grupo
apt-get install git -y -->

## Installing and Configuring SSH (Secure Shell Host)
sudo apt install openssh-server 
sudo nano /etc/ssh/sshd config --> Editar arquivo

Alterar #Port 22 para Port 4242

#PermitRootLogin prohibit-password --> PermitRootLogin no

sudo reboot
sudo service ssh status

## Installing and Configuring UFW
sudo apt install ufw --> UFW - Gestao de Firewall
sudo ufw enable --> Habilitando UFW
sudo ufw allow 4242 --> Criando porta(regra) para acesso remoto 
sudo ufw status

SSH >> TCP --> 127.0.0.1 --> 4242 --> 10.0.2.15 --> 4242 // Alterei esta parte para o VM utilizar o medoto de Network "Bridge Adapter" 
HTTP >> TCP --> 127.0.0.1 --> 80 -->10.0.2.15 --> 80 //2 BONUS
FTP >> TCP -->127.0.0.1 --> 21 --> 10.02.15 21 //3 BONUS

No terminal fora do VM, executamos ```ssh your_username@127.0.0.1 -p 4242``` --> No meu caso ficou ```ssh lumarque@10.0.248.66 -p 4242```
Coloque a senha e pronto, ja tem acesso a VM, atraves do seu terminal. xD

## Configuracoes sudo, login, password
sudo nano /etc/sudoers --> Alterar arquivo com as configuracoes exigidas no projeto para sudo

----------  Defaults --------- passwd tries=3
----------  Defaults --------- badpass message-"Password is wrong, please try again!”
----------  Defaults --------- logfile="/var/log/sudo.log"
----------  Defaults --------- log_input, log_output
----------  Defaults --------- iolog_dir="/var/log/sudo"
----------  Defaults --------- requiretty
----------  Defaults --------- secure_path="/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/snap/bin"

sudo nano /etc/login.defs --> Alterar o arquivo de config de login
-->PASS MAX DAYS — 99999 --> PASS MAX DAYS 30 
-->PASS MIN DAYS — 6: -->-PASS MIN DAYS/2
-->PASS WARN AGE — 7

sudo apt install libpam-pwquality
sudo nano /etc/pam.d/common-password --> Alterar o arquivo de config para as senhas.
—>password —> requisite —> pam puquality.so retry=3" —> password —> requisite

## SCRIPT
sudo crontab -u root -e --> Editar as configuracoes do Crontab para execurar o comando a cada 10 minutos
—>*/10/* * * * sh /path/to/script

nano monitoring.sh
chmod 777 monitoring.sh

———————————SCRIPT——————————

#!/bin/bash

#Arch
arch =$(uname -a)

#Nucleos fisicos
cpuf= $(grep "physical id" /proc/cpuinfo | wc -l)

#Nucleos virtuais
cpuv = $(grep processor /proc/cpuinfo | wc -l)

#RAM

ram_use = $(free --mega | awk '$1 == "Mem.:" {print $3}')

ram_total = $(free --mega | awk '$1 == "Mem.:" {print $2}')

ram_percent = $(free --mega | awk '$1 == "Mem.:" {printf("(%.2f%%)\n", $3/$2*100)}')

#Disk
disk_use = $(df -m | grep "/dev/" | grep -v "/boot" | awk '{memory_use += $3} END {print memory_use}')

disk_total = $(df -m | grep "/dev/" | grep -v "/boot" | awk '{memory_result += $2} END {printf("%.0fGb\n"), memory_result/1024}')

disk_percent = $(df -m | grep "/dev/" | grep -v "/boot" | awk '{use += $3} {total += $2} END {printf("(%d%%)\n"), use/total*100}')

#CPU LOAD
cpul = $(vmstat 1 2 | tail -1 | awk '{printf $15}')

cpu_op = $(expr 100 - $cpul)

cpu_fin = $(printf "%.1f" $cpu_op)

#Last Boot
lb = $(who -b | awk '$1 == "arranque" {print $4 " " $5}')

#LVM USE
lvmu = $(if [ $(lsblk | grep "lvm" | wc -l) -gt 0 ]; then echo yes; else echo no; fi)

#Conexoes TCP
tcpc = $(ss -ta | grep ESTAB | wc -l)

#Numero de Utilizadores
ulog = $(users | wc -w)

#Endereco MAC
ip = $(hostname -I)
mac = $(ip link | grep "link/ether" | awk '{print $2}')

#SUDO
cmnd = $(journalctl _COMM=sudo | grep COMMAND | wc -l)

wall "	Architecture: $arch
	CPU physical: $cpuf
	vCPU: $cpuv
	Memory Usage: $ram_use/${ram_total}MB ($ram_percent%)
	Disk Usage: $disk_use/${disk_total} ($disk_percent%)
	CPU load: $cpu_fin%
	Last boot: $lb
	LVM use: $lvmu
	Connections TCP: $tcpc ESTABLISHED
	User log: $ulog
	Network: IP $ip ($mac)
	Sudo: $cmnd cmd"


## BONUS
Obs. A parte bonus de instalacao separacao dos LVM eu ja tinha feito. Caso va fazer o bonus aconselho ja fazer de primeira.

1. Install lighttpd
sudo apt install lighttpd		--> NGNIX e APACHE Servidor externo alternativo.
sudo ufw allow 80				--> Porta HTTP standart

2. Install Mariadb, MySql
sudo apt install mariadb-server	--> Mariadb é um banco de dados relacional. Derivado do código-fonte do MySQL Databse Server.
sudo mysql_secure_installation	--> Para definir configurações padroes de seguranca...
	@-)No primeiro passo vai pedir uma senha, diga enter e pule.
	@-)Switch to unix option = N
	@-)Change the root password = N
	@-)Remove anonymous users = Y
	@-)Disallow root login remotely = Y
	@-)Remove test database and access to it = Y
	@-)Reload privilege tables now = Y

3.	Criando database
sudo mariadb				--> Entre no console do mariadb.
CREATE DATABASE db_42porto;	--> Um novo banco de dados é criado.
GRANT ALL ON <db_name>.* TO '<db_user>'@'localhost' IDENTIFIED BY '<db_password>' WITH GRANT OPTION;
GRANT ALL ON db_42porto.* TO 'db_lumarque'@'localhost' IDENTIFIED BY 'psw' WITH GRANT OPTION;
	-Criamos um novo usuário do banco de dados e informamos que ele estará na rede local e definimos uma senha para este uso 
	-OBS: Não confunda o usuário aqui com os usuários do sistema!
	-Dizemos a esse usuário recém-criado para dar autorização total ao banco de dados recém-criado.

FLUSH PRIVILEGES; Lemos as mudanças que fizemos e saimos -->
exit --> Saímos do console do Mariadb.

mariadb -u <db_user> -p	--> Tentaremos nos conectar ao banco de dados do usuário que criamos. Depois de digitar isso, ele nos pedirá uma senha, a database_user_password que definimos.
SHOW DATABASES;	--> Mostra bancos de dados. Deve haver 2 bancos de dados, um que criamos e o outro information_schema
exit							--> Sair.

4. Install PHP, Wget
sudo apt install php-cgi php-mysql	--> COMMON GATEWAY INTERFACE php-cgi (common gateway interface) web serverinin apache, lighttpd etc. 
	É uma tecnologia web que permite que aplicativos externos se comuniquem com intérpretes das linguagens PHP, PYTHON, PERL.
	-Instalamos esses plugins porque o Wordpress é desenvolvido com PHP.

sudo apt install wget	--> Instalamos este plug-in para obter downloads por HTTP HTTPS E FTP. Na próxima etapa, vamos extrair os dados do próprio site do Wordpress.
sudo wget http://wordpress.org/latest.tar.gz -P /var/www/html	--> Baixe o painel wordpress mais recente para o diretório especificado (se nao, ele será criado).
sudo tar -xzvf /var/www/html/latest.tar.gz		--> Extraia os arquivos da pasta tar.gz que baixamos.
sudo rm /var/www/html/latest.tar.gz		--> Depois de extrair os arquivos, excluímos o arquivo compactado baixado.
sudo cp -r /var/www/html/wordpress/* /var/www/html		--> Nome da pasta extraída do arquivo wordpress Com este comando, copiamos todos os dados do arquivo wordpress para o caminho especificado.

sudo rm -rf /var/www/html/wordpress		--> Depois de copiar o conteúdo da pasta, também estamos excluindo a pasta, pois não estamos mais trabalhando com a pasta.
sudo cp /var/www/html/wp-config-sample.php /var/www/html/wp-config.php	--> Copie o arquivo wp-config-sample com o nome wp-config.
sudo nano /var/www/html/wp-config.php			--> abrimos o arquivo copiado com o nano e editamos as seguintes partes...
	-define( 'DB_NAME', '<db_name>' );^M
	-define( 'DB_USER', '<db_user>' );^M
	-define( 'DB_PASSWORD', '<db_password>' );^M
@@sudo lighty-enable-mod fastcgi ---------
@@sudo lighty-enable-mod fastcgi-php ------> habilite o módulo fastcgi e a configuração do php
@sudo service lighttpd force-reload ------> reiniciamos o servidor lighttpd

5. Servico adicional 
5.1 FTP
sudo apt install vsftpd		--> Pacote necessário para o servidor FTP.
sudo ufw allow 21			--> Damos permissão TCP UDP para a porta 21 do firewall.
sudo nano /etc/vsftpd.conf	-->
	@-)#write_enable=YES	// # remover para escrever comandos ftp
	@-)user_sub_token=$USER
	@-)local_root=/home/$USER/ftp
	@-)userlist_enable=YES
	@-)userlist_file=/etc/vsftpd.userlist
	@-)userlist_deny=NO
sudo mkdir /home/<username>/ftp 	--> Criamos a pasta ftp para o diretório do nosso usuário.
sudo mkdir /home/<username>/ftp/files	--> Criamos a pasta Arquivos
sudo chown nobody:nogroup /home/<username>/ftp	--> Definimos a propriedade e o grupo da pasta ftp para nobody.
sudo chmod a-w /home/<username>/ftp		--> Concedemos permissão de gravação a todos os usuários.
sudo nano /etc/vsftpd.userlist			--> Faça CTRL + O, depois CTRL + X, salve e saia. Seu objetivo é permitir que nos conectemos com usuários permitidos.
echo <username> | sudo tee -a /etc/vsftpd.userlist 	--> Adicionamos nosso usuário à lista de usuários.//

5.2 LiteSpeed zap
sudo apt update
sudo apt upgrade
wget -O - http://rpms.litespeedtech.com/debian/enable_lst_debian_repo.sh | sudo bash --> OpenLiteSpeed está disponível no repositório base do Debian 11
sudo apt update
sudo apt install openlitespeed
sudo /usr/local/lsws/admin/misc/admpass.sh --> config login e psw (idroot, 123456)
sudo ufw allow 8088/tcp
sudo ufw allow 7080/tcp
sudo ufw reload//


# 9- Guia de correcção ✅
- `ls /usr/bin/*session`
- `sudo ufw status`
- `sudo service ssh status`
- `getent group sudo`
- `getent group user42`
- `sudo adduser new username`
- `sudo groupadd groupname`
- `sudo usermod -aG groupname username`
- `sudo chage -l username` - check password expire rules
- `hostnamectl`
- `hostnamectl set-hostname new_hostname` - to change the current hostname
- Restart your Virtual Machine.
- `sudo nano /etc/hosts` - change current hostname to new hostname
- `lsblk` to display the partitions
- `sudo -V` to show version sudo is installed
- `sudo ufw status numbered`
- `sudo ufw allow port-id`
- `sudo ufw delete rule number`
- `ssh your_user_id@127.0.0.1 -p 4242` -  do this in terminal to show that SSH to port 4242 is working
  
  Fontes:
https://github.com/pasqualerossi/Born2BeRoot-Guide
https://github.com/gemartin99/Born2beroot-Tutorial
https://github.com/burak-yldrm
