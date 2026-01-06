# network-automation-dhcp-dns-stork
Déploiement complet DHCP KEA + DNS BIND9 + Supervision Stork sur Debian 12. Script d'installation automatisée (all-in-one), configurations réseau et services, documentation.

--------------------------------

Infrastructure Réseau : DHCP KEA + DNS BIND9 + Supervision Stork
📋 Description du projet
Ce projet met en place une infrastructure réseau complète et modulaire sur Debian 12 en mode serveur, incluant :
•	DHCP KEA : serveur d'attribution d'adresses IP (remplaçant moderne du ISC DHCP classique)
•	DNS BIND9 : serveur de résolution de noms (zones directes et inverses)
•	Stork : interface de supervision centralisée pour KEA et BIND
•	Configuration statique : serveur réseau avec IP fixe
L'objectif est d'automatiser la distribution des paramètres réseau (IP, DNS, passerelle) à des clients DHCP et de permettre la résolution de noms de domaine au sein d'un réseau local d'entreprise fictive projet.local.

 Fonctionnalités principales
1. Installation minimale de Debian 12
•	ISO netinstall pour installation légère
•	Partitionnement guidé
•	Configuration locale et langue française
•	Création d'utilisateurs (root et admin1)
2. Configuration réseau statique
•	Adresse IP fixe : 192.168.1.10
•	Netplan pour la configuration réseau
•	DNS statique configuré
3. Serveur KEA DHCP
•	Installation depuis les dépôts ISC officiels
•	Configuration JSON (kea-dhcp4.conf)
•	Pool DHCP : 192.168.1.100 - 192.168.1.200
•	Options DHCP : DNS, passerelle, domaine
•	Supervision via l'agent Stork
4. Serveur DNS BIND9
•	Installation et configuration de BIND
•	Zone directe : projet.local
•	Zone inverse : 1.168.192.in-addr.arpa
•	Enregistrements A, PTR
•	Tests avec dig et nslookup
5. Supervision Stork
•	Installation de Stork Server et Agent
•	Base de données PostgreSQL
•	Interface web pour supervision centralisée
•	Suivi de l'état des services KEA et BIND
📦 Prérequis
Matériel / Logiciel
•	Hyperviseur : VMware Workstation, VirtualBox ou Proxmox
•	ISO Debian 12 netinstall (minimum 1 GO)
•	RAM : 2 GB minimum (4 GB recommandé)
•	Disque : 20 GB minimum
•	Réseau : Mode NAT ou Bridge selon besoin
Accès système
•	Accès root sur le serveur Debian
•	Connexion réseau stable
•	Ports disponibles : 67/68 (DHCP), 53 (DNS), 8080 (Stork)
⚙️ Installation et configuration
Étape 1 : Installation minimale de Debian 12
Télécharger l'ISO Debian 12 netinstall
Créer une VM avec les paramètres ci-dessus
Démarrer sur l'ISO et suivre l'installateur
Configuration réseau durant l'installation
Nom d'hôte : debian
Domaine : projet.local
Utilisateur root : [définir mot de passe]
Utilisateur normal : admin1
Étape 2 : Configuration réseau statique (Netplan)
Installer Netplan
sudo apt install netplan.io
Créer le fichier de configuration
sudo nano /etc/netplan/01-debian.yaml
Contenu du fichier :
network:
version: 2
renderer: networkd
ethernets:
enp0s3:
dhcp4: no
addresses:
- 192.168.1.10/24
gateway4: 192.168.1.1
nameservers:
addresses:
- 192.168.1.10
- 8.8.8.8
Appliquer la configuration
sudo netplan apply
Vérifier
ip a
Étape 3 : Installation de KEA DHCP
Ajouter le dépôt ISC officiel
curl -1sLf 'https://dl.cloudsmith.io/public/isc/kea-3-0/setup.deb.sh' | bash
Mettre à jour le cache
sudo apt update
Installer KEA
sudo apt install kea
Démarrer et activer
sudo systemctl start kea-dhcp4-server
sudo systemctl enable kea-dhcp4-server
Étape 4 : Configuration KEA DHCP
Éditer la configuration
sudo nano /etc/kea/kea-dhcp4.conf
Voir le fichier configs/kea-dhcp4.conf pour la configuration complète.
Redémarrer après modification
sudo systemctl restart kea-dhcp4-server
Vérifier l'état
sudo systemctl status kea-dhcp4-server
Étape 5 : Installation de BIND9 DNS
Installer BIND9
sudo apt install bind9 bind9-utils bind9-doc
Démarrer et activer
sudo systemctl start bind9
sudo systemctl enable bind9
Étape 6 : Configuration DNS BIND9
Éditer la configuration locale
sudo nano /etc/bind/named.conf.local
Voir les fichiers dans configs/bind/ pour les zones DNS.
Vérifier la syntaxe
sudo named-checkconf
sudo named-checkzone projet.local /etc/bind/db.projet.local
sudo named-checkzone 1.168.192.in-addr.arpa /etc/bind/db.192
Redémarrer après modification
sudo systemctl restart bind9
Étape 7 : Installation de Stork (Supervision)
Ajouter le dépôt Stork
curl -1sLf 'https://dl.cloudsmith.io/public/isc/stork/setup.deb.sh' | bash
Installer Stork Server et Agent
sudo apt install stork-server stork-agent
Initialiser la base de données PostgreSQL
sudo systemctl start postgresql
sudo systemctl enable postgresql
Initialiser Stork
sudo stork-tool db-init
Démarrer Stork
sudo systemctl start stork-server stork-agent
sudo systemctl enable stork-server stork-agent
Étape 8 : Accès à Stork
Ouvrir navigateur
http://localhost:8080
Identifiants par défaut
Username: admin
Password: admin
   CHANGER LE MOT DE PASSE À LA PREMIÈRE CONNEXION
   Tests et validation
Test DHCP
Sur une autre machine cliente
Configurer l'interface en DHCP
sudo nano /etc/netplan/01-dhcp.yaml
network:
version: 2
ethernets:
enp0s3:
dhcp4: yes
Appliquer et tester
sudo netplan apply
ip a
Test DNS
Depuis le serveur ou un client
Test résolution directe
dig @192.168.1.10 ns1.projet.local
dig @192.168.1.10 pc1.projet.local
Test résolution inverse
dig -x 192.168.1.100 @192.168.1.10
Alternative avec nslookup
nslookup pc1.projet.local 192.168.1.10

🛠️Technologies utilisées
Service	Version	Rôle
Debian	12	Système d'exploitation
KEA	3.0+	Serveur DHCP moderne
BIND9	9.x	Serveur DNS autoritatif
Stork	2.x	Supervision centralisée
PostgreSQL	15+	Base de données Stork
Netplan	-	Configuration réseau

Problèmes rencontrés et solutions
KEA ne démarre pas
Problème : Service KEA inactif après installation.
Solution : Vérifier la syntaxe JSON de kea-dhcp4.conf avec kea-dhcp4 -t et consulter les logs journalctl -u kea-dhcp4-server.
Clients ne reçoivent pas d'IP
Problème : Pool DHCP vide ou configuration incorrecte.
Solution : Vérifier le fichier de configuration KEA, s'assurer que le pool contient des adresses libres et que les permissions réseau le permettent.
BIND ne résout pas les noms
Problème : Erreurs dans les zones DNS.
Solution : Valider les zones avec named-checkzone et vérifier les permissions des fichiers de zone.
Stork ne trouve pas KEA/BIND
Problème : Agents Stork non connectés au serveur.
Solution : Vérifier les configurations dans /etc/stork/agent.env et redémarrer les agents.
Interface Stork inaccessible
Problème : Port 8080 bloqué ou service non démarré.
Solution : Vérifier le firewall et les logs de Stork avec journalctl -u stork-server.
📚 Documentation complémentaire
•	Documentation KEA officielle
•	Documentation BIND9
•	Documentation Stork
•	Netplan - Gestion réseau Debian
🎯 Objectifs pédagogiques atteints
•	✅ Installation minimale Debian 12 en mode serveur
•	✅ Configuration réseau statique avec Netplan
•	✅ Déploiement d'un serveur DHCP moderne (KEA)
•	✅ Mise en place d'une infrastructure DNS (BIND9)
•	✅ Configuration de zones DNS directes et inverses
•	✅ Supervision centralisée avec Stork
•	✅ Tests et diagnostics réseau (dig, nslookup)
•	✅ Concepts DevOps : Infrastructure as Code, modularité
•	✅ Administration système avancée
 Sécurité
 Attention : Ce projet est à des fins pédagogiques. Pour une utilisation en production :
•	✅ Utiliser des mots de passe robustes (min 12 caractères, complexes)
•	✅ Configurer l'authentification par clés SSH
•	✅ Activer un pare-feu (UFW ou nftables)
•	✅ Limiter l'accès aux ports 53 (DNS), 67/68 (DHCP), 8080 (Stork)
•	✅ Mettre en place DHCP snooping sur les switches
•	✅ Surveiller les logs pour détecter des serveurs DHCP non autorisés
•	✅ Utiliser DNSSEC pour sécuriser les zones DNS
•	✅ Chiffrer les communications entre agents et serveur Stork
 Licence
Ce projet a été réalisé dans un cadre académique au Cégep Bois-de-Boulogne dans le programme Infrastructures TI et cybersécurité.
 Auteur
Projet réalisé dans le cadre du cours de Configuration réseau avancée (DHCP, DNS, supervision).
________________________________________
Note finale : Les fichiers de configuration fournis sont des exemples. Adaptez les adresses IP, domaines et mots de passe à votre environnement avant de les utiliser en production.
