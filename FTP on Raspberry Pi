#!/bin/bash
RED="\e[0;31m"
GREEN="\e[0;32m"
YELLOW="\e[1;33m"
END="\e[0m"
DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )/conf"

echo -e "${GREEN}"
read -p "Mise a jour du nom de l'ordinateur : " compName
echo -e "${END}"
rm -f /etc/hostname
echo $compName >> /etc/hostname

echo -e "${GREEN}Mise a jour du mot de passe de l'utilisateur ROOT...${END}"
passwd

echo -e "${GREEN}"
read -p "Nom de l'utilisateur commun : " userName
echo -e "${END}"

echo -e "\n${GREEN}Creation de l'utilisateur ${YELLOW}$userName${GREEN}... ${END}"
useradd -d /home/$userName -g users -m $userName
passwd $userName

echo -e "\n${GREEN}Mises a jour... ${END}"
pacman -Ssy --noconfirm

echo -e "\n${GREEN}Installation des serveurs Web et FTP... ${END}"
pacman -Syu --noconfirm yaourt vsftpd apache php php-apache mariadb phpmyadmin php-mcrypt vim git clamav php-pear binutils

echo -e "\n${GREEN}Configuration de Very Secure FTP..."
if [ -f $DIR/vsftpd.conf ]; then
cp -f $DIR/vsftpd.conf /etc/vsftpd.conf
else
echo -e "${RED}vsftpd.conf ne se trouve pas a la racine du script, configuration incomplete...${END}\n"
exit 1
fi

echo -e "\n${GREEN}Configuration de Apache..."
if [ -f $DIR/httpd.conf ]; then
cp -f $DIR/httpd.conf /etc/httpd/conf/httpd.conf
else
echo -e "${RED}httpd.conf ne se trouve pas a la racine du script, configuration incomplete...${END}\n"
exit 1
fi

echo -e "\n${GREEN}Configuration de PHP..."
cp /etc/webapps/phpmyadmin/apache.example.conf /etc/httpd/conf/extra/httpd-phpmyadmin.conf
if [ -f $DIR/php.ini ]; then
cp -f $DIR/php.ini /etc/php/php.ini
else
echo -e "${RED}php.ini ne se trouve pas a la racine du script, configuration incomplete...${END}\n"
exit 1
fi

echo -e "\n${GREEN}Clamav update..."
freshclam

echo -e "\n${GREEN}Enabling and restarting VSFTP... "
systemctl enable vsftpd
systemctl restart vsftpd
echo -e "Enabling and restarting MariaDB... "
systemctl enable mysqld
systemctl restart mysqld
echo -e "Enabling and restarting Apache... ${END}"
systemctl enable httpd
systemctl restart httpd

echo -e "\n${GREEN}Configuration de MariaDB..."
read -s -p "    Nouveau mot de passe root SQL : " sqlPass
echo -e "${END}"
mysql -uroot -e "USE mysql; SET PASSWORD FOR 'root'@'localhost' = PASSWORD('$sqlPass')"
mysql -uroot -p$sqlPass -e "USE mysql; SET PASSWORD FOR 'root'@'127.0.0.1' = PASSWORD('$sqlPass')"
mysql -uroot -p$sqlPass -e "USE mysql; UPDATE user SET Host = '$compName' WHERE Host = 'alarmpi'"
mysql -uroot -p$sqlPass -e "USE mysql; SET PASSWORD FOR 'root'@'$compName' = PASSWORD('$sqlPass')"
mysql -uroot -p$sqlPass -e "USE mysql; SET PASSWORD FOR 'root'@'::1' = PASSWORD('$sqlPass')"

echo -e "${GREEN}"
read -p "Telechargement du repo Arnaud-Assurance ? (y / n)" response
echo -e "${END}"
if [ $reponse = 'y' ]; then
cd /srv/http
git clone http://bitbucket.org/enoxime/arnaud-assurance.git
fi

echo -e "${GREEN}Mise a jour du mot de passe de l'utilisateur HTTP${END}"
passwd http
echo -e "${GREEN}Octroyage des droits HTTP\n${END}"
chown -R http:http /srv/http
chmod -R 755 /srv/http

tput sgr0
exit 0
