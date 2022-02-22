# Installer-Dolibarr
Sur un server ubuntu 20.04

En suivant ce lien : 
https://www.howtoforge.com/how-to-install-dolibarr-erp-crm-on-ubuntu-1804/

Aller vers le répertoire :

        /temp
        cd /tmp 
    
Télécharger depuis sourceforge le fichier dolibarr-xx.x.x.zip

        wget https://sourceforge.net/projects/dolibarr/files/Dolibarr%20ERP-CRM/15.0.0/dolibarr-15.0.0.zip
Unzipper le fichier

        unzip dolibarr-15.0.0.zip

Next, copy the extracted directory to the Apache web root and give proper permissions:

        sudo mkdir /var/www/html/dolibarr
        sudo cp -r dolibarr-15.0.0/htdocs/* /var/www/html/dolibarr/
        sudo chown -R www-data:www-data /var/www/html/dolibarr/
        sudo chmod -R 755 /var/www/html/dolibarr/

Create a folder for Dolibarr to store uploaded documents:

        sudo mkdir /var/documents
        sudo chown www-data:www-data /var/documents
        sudo chmod 700 /var/documents

Next, create an Apache virtual host file with the following command:

        sudo nano /etc/apache2/sites-available/dolibarr.conf
Add the following lines:

     <VirtualHost *:80>
     ServerAdmin amaury@terre-alternative.fr
     DocumentRoot /var/www/html/dolibarr
     ServerName dolibarr.terre-alternative.fr

     <Directory /var/www/html/dolibarr>
          Options +FollowSymlinks
          AllowOverride All
          Require all granted
     </Directory>

     ErrorLog ${APACHE_LOG_DIR}/dolibarr_error.log
     CustomLog ${APACHE_LOG_DIR}/dolibarr_access.log combined
     
     </VirtualHost>

Then save the file, then enable apache virtual host file with the following command:

        sudo a2ensite dolibarr
Next, enable Apache rewrite module and reload apache service with the following command:

        sudo a2enmod rewrite
        sudo systemctl restart apache2

# Access Dolibarr

Now, open your web browser and type the URL of your Dolibarr website, 
in my case http://dolibarr.terre-alternative.fr You will be redirected to the following page: (image) sur https://www.howtoforge.com/how-to-install-dolibarr-erp-crm-on-ubuntu-1804/
==> ça ne marche pas... ni en tapant 192.168.0.18/dolibarr...  d'ailleurs, openproject ne fonctionne pas non plus, alors  que  hier soir si.

Je vais sur OVH pour ajouter un sous-domaine dolibarr.terre-alternative.fr associé en mode A à 192.168.0.18
J'en profite pour créer les sous-domaines openproject.terre-alternative.fr et nextcloud23.terre-alternative.fr
...
Finalement, en tapant bêtement 192.168.0.18 je tombe sur la page d'installation de dolibarr ! peut-être parce qu'il a été installé directement dans /var/www/html/dolibarr !? (cf ligne 31)

Je me  dis que c'est peut-être parce que les trois app (nextcloud-openproject-dolibarr) écoutent sur le même port :80
je suis donc un nouveau tuto : https://www.it-connect.fr/changer-le-port-decoute-dapache2/

J'accède au fichier de configuration des ports d'écoute d'Apache2 

      sudo nano /etc/apache2/ports.conf

Avertissement :
If you just change the port or add more ports here, you will likely also
have to change the VirtualHost statement in
/etc/apache2/sites-enabled/000-default.conf

je modifie et j'ajoute : 
        
        Listen 8021
        Listen 2326
        Listen 210

        <IfModule ssl_module>
                Listen 528
                Listen 9128
                Listen 4853
        </IfModule>

        <IfModule mod_gnutls.c>
                Listen 528
                Listen 9128
                Listen 4853
        </IfModule>

Modifier le fichier .conf par  défaut

        sudo nano /etc/apache/sites-enabled/000-default.conf
--> il n'existe pas encore ...


Je retourne ensuite sur https://www.it-connect.fr/apache-heberger-plusieurs-apps-sur-differents-ports/
Je décide de modifier le précédent fichier de configuration de dolibar r (ligne 29)

      sudo nano /etc/apache2/sites-available/dolibarr.conf

      <VirtualHost 192.168.0.18:528>
           ServerAdmin amaury@terre-alternative.fr
           DocumentRoot /var/www/html/dolibarr
           ServerName dolibarr.terre-alternative.fr

           <Directory /var/www/html/dolibarr>
                Options +FollowSymlinks
                AllowOverride All
                Require all granted
           </Directory>

           ErrorLog ${APACHE_LOG_DIR}/dolibarr_error.log
           CustomLog ${APACHE_LOG_DIR}/dolibarr_access.log combined
      </VirtualHost>

j'enregistre et  je relance apache 

      sudo systemctl restart apache2

je  rafraichi la page d'instalation de dolibarr (192.168.0.18) erreur : la cnnexion a échoué : normal, j'ai  du tombé  sur le firewall qui n'est pas configuré sur les noouveaux ports

# le firewall ufw

je retourne chercher la commande du firewall du serveur ufw je crois
je retrouve https://fr.joecomp.com/how-set-up-firewall-with-ufw-ubuntu-18
je vérifie l'état du firewall 

      sudo ufw status verbose

J'indique les nouveaux ports d'écoute autorisés

      sudo ufw allow 8021/tcp
      sudo ufw allow 2326/tcp
      sudo ufw allow 210/tcp
      sudo ufw allow 528/tcp
      sudo ufw allow 9128/tcp
      sudo ufw allow 4853/tcp
je  n'arrive pas à me reconnecter à aucun de  mes trois apps... j'essaie en vain d'autoriser apache sur les ports précédement modifiés, de remettre à zéro ufw
finalement, j'opte pour la  desactivation de ufw et un retour au port 80 sur le fichier .conf


Je retente d'accéder à dolibarr --> ça fonctionne

# reconfigurer les valeurs régionales de Nextcloud
Par contre j'ai une erreur UTF-8, https://help.nextcloud.com/t/solved-echec-de-la-specification-des-parametres-regionaux-error-setting-locale-to/85589/6
Il faut utiliser vim (un équivalent de nano si j'ai bien compris --> https://www.syloe.com/utiliser-vim-guide/
=> résolu

# récupérer le mot de pass du compte admin de Nextcloud
Mais il y a maintenant un problème de mot de passe et login : je ne me souviens plus du mot de passe que j'ai renseigné lors de la création du compte 
et la fonction "récupérer mon mot de passe" est inopérante surement parce que je n'ai pas configuré le smtp

j'utilise la commande : 
sudo -u www-data php /var/www/nextcloud/occ user:resetpassword VOTREUTILISATEUR --> remplacé par amaury22
et je renseigne un nouveau mot de passe de plus de 10 caractères

# Autres ressources pour la configuration d'un serveur next cloud : 

https://doc.ubuntu-fr.org/nextcloud-serveur

https://www.digitalocean.com/community/tutorials/how-to-install-the-apache-web-server-on-ubuntu-20-04-fr
