Bonjour,

Pour réaliser ce projet j'ai choisi de travailler sur des instances ec2 sur AWS.

J'ai utilisé des instances t3.medium avec ubuntu 24.04 pour le master et pour les serveurs clients (db et web).
Pour chacune de ces instances j'ai utilisé le user-data suivant :

############################################################################################################################
#!/bin/bash
sudo sed 's/PasswordAuthentication no/PasswordAuthentication yes/' -i /etc/ssh/sshd_config.d/60-cloudimg-settings.conf
sudo systemctl restart ssh
sudo service ssh restart

username=datascientest
password=Datascientest2022

sudo adduser --gecos "" --disabled-password $username
sudo chpasswd <<<"$username:$password"
sudo usermod -aG sudo datascientest

#############################################################################################################################

Sur le serveur master j'ai intallé ansible via apt (j'ai rencontré des difficulté avec pip, je n'ai pas pu installer pip si ce n'est en passant par .venv 
et cela me posait des problèmes):

sudo apt update
sudo apt install python3-pip
sudo apt-install python3-boto3
sudo apt-get update
sudo apt-get install software-properties-common
sudo apt-add-repository --yes --update ppa:ansible/ansible
sudo apt-get install ansible

j'ai également installé sur le serveur Master Molecule et Docker

sudo apt-get update
sudo apt install -y libssl-dev
python3 -m pip install molecule ansible-core molecule-docker   #(ici j'ai toujours le soucis avec pip comme mentionné plus haut, j'ai du passer par .venv, mais cela m'a
#généré des problèmes)
export PATH="$HOME/.local/bin:$PATH"
export PATH="$HOME/.venv/bin:$PATH"

intallation de DOCKER :

sudo apt install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu focal stable"
apt-cache policy docker-ce
sudo apt install docker-ce
sudo usermod -aG docker ${USER}
su - ${USER}

Il m'a fallu me créer un compte magento sur : https://commercemarketplace.adobe.com/ puis j'ai créé une access key (pub_key , pvt_key) 
J'ai rentré mes clés dans le fichier all.yml (je n'ai pas laissé les clés dans le fichier que je vous envoie, je vous laisse mettre vos clés)

J'ai également créé un nom de domaine via cloudns

j'ai saisi l'adresse dans le fichier all.yml en tant que base_url, j'ai également mis cette adresse dans le template nginx magento.conf.j2

Et enfin j'ai bien saisi l'adresse IP privée de mon serveur db dans le hosts.ini ainsi que dans le fichier all.yml en tant que db_host.
Et j'ai saisi l'adresse IP de mon serveur web dans le hosts.ini

J'ai choisi de creer plus que 2 roles comme initialement demandé (il était demandé de faire un role magento et un role mysql) car je trouve que cela est plus clair,
en effet magento nécessite l'installation préalable de plusieurs services. Si j'avais voulu vraiment ne faire que 2 roles, j'aurais fait plusieurs fichiers yaml dans mysql/tasks
un pour chaque service (nginx, php, elasticsearch ...).

J'ai créé chacun de mes roles à l'aide de la commande :
ansible-galaxy init

Puis j'ai configuré molecule pour chacun des roles avec la commande :
molecule init scenario --driver-name docker

je ne suis pas parvenu à lancer la commande complète suivante :
molecule init scenario --driver-name docker --verifier-name=testinfra
j'ai une erreur me disant que l'option verifier-name n'existe pas.

J'ai écrit tous les logs du déroulement de mon playbook que j'ai lancé sur des instances ec2 aws avec la commande :

ansible-playbook -i hosts.ini playbook-magento.yml -vvv > log_ansible.txt

Vous trouverez le fichier log_ansible.txt dans ce dossier.

A la fin du fichier log vous constaterez que le playbook ansible a bien opéré :

PLAY RECAP *********************************************************************
db                         : ok=10   changed=6    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   
web                        : ok=35   changed=31   unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   



Vous trouverez également l'arborescence du projet dans le fichier tree.txt


