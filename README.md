# Let's take control

## Job 1
Installation su service SSH sur les machines primaires et secondaires:
```bash
apt install -y ssh
```
Message de bienvenue: édition du fichier /etc/update-motd.d/99-welcome-admin

```bash
#!/bin/bash
if [ $USER -eq "admin" ]; then
    echo "Hello $USER @ $HOSTNAME"
fi
exit 0
```
Puis le rendre exécutable
```bash
chmod +x /etc/update-motd.d/99-welcome-admin
```

## Job 2

Dans /etc/ssh/sshd_config on change le port de connexion

```bash
# /etc/ssh/sshd_conf
...
PORT=2022
...
```
Puis on recharge la configuration
```bash
systemctl restart ssh
```

Génération du clef SSH et copie de la clef public sur les différents serveurs.
En préfère utiliser des clefs utilisant l'algorithme ed25519 plutôt que RSA (confère les recommandations NIST).
```bash
ssh-keygen -t ed25519 -C "Admin Key" -N "MotDePasse" -f ~./ssh/keyServer
ssh-copy-id -i ./ssh/KeyServer.pub admin@primary:2022
ssh-copy-id -i ./ssh/KeyServer.pub admin@secondary:2022
```

## Job 3
- Création du groupe *sshusers*
- Désactivation de l'utilisateur *root*
- Modification de sshd_config  pour désactiver tous les utilisateurs hormis ceux du groupe *sshusers*

```bash
groupadd sshusers
usermod -aG sshusers admin
usermod -aG sshusers assistant

# Edition de /etc/ssh/sshd_config
...
PermitRootLogin no
PawwordAuthentification no
allowGroups sshusers
...
```
On recharge la configuration du serveur
```bash
systemctl restart ssh
```

## Job 4
```bash
# Sur les serveurs primary & secondary
mkdir -p /home/utilisateur/config
touch /home/utilisateur/config/conf{1..2}.conf

scp -P 2022 /home/utilisateur/config/conf1.conf admin@secondary:/home/utilisateur/config/.
scp -P 2022 admin@secondary:/home/utilisateur/config/conf2.conf /home/utilisateur/config/.
```

## Job 5
```bash
su - assistant
echo 'X5O!P%@AP[4\PZX54(P^)7CC)7}$ERICA-STANDARD-A script linux -ILE!$H+H* :://' >> /home/utilisateur/config/conf1.conf

scp -P 2022 /home/utilisateur/config/conf1.conf assistant@primary:/home/utilisateur/config/.
```

## Job 6
```bash
# Afficher les modifications entre 2 fichiers
vimdiff conf1.conf conf2.conf

# Enregistrement du rendu dans un fichier
diff conf1.conf conf2.conf > modifications

# Destruction du fichier malsain
rm conf1.conf

# Remplacement du fichier
cp conf2.conf conf1.conf
```

## Job 7
```bash
# On retire les droits d'écriture à assistant
setfacl -R -m u:assistant:r-X /home/utilisateur
```

## Job 8
```bash
#!/bin/bash
#
# filename: save_config
# Description: script de sauvegarde des fichiers de configuration
#
CONFIG=/home/utilisateur/config
DATE=$(date +"%Y%m%d")
SAVE=/home/utilisateur/sauvegarde_deploiements/config-$DATE"

cp -R $CONFIG $SAVE/.
exit 0
```
On configure le crontab de root pour faire une copie par
jour à 12h01 et on regroupe tout cela en une archive une fois
par semaine protégée par mot de passe.
```bash
crontab -e
1 12 * * * /root/bin/save_config
0 0 */7 * * zip -P "MotDePass" /home/utilisateur/sauvegarde_deploiements/weekly/conf-$(date +"%Y%m%d").zip /home/utilisateur/sauvegarde_deploiements
0 1 */7 * * rm -fR /home/utilisateur/sauvegarde_deploiements/conf-*
```

## Job 9
Il n'ont qu'à prendre l'excellente charte de La Plateforme pour avoir une petite idée de ce qu'ils doivent
ajouter...

