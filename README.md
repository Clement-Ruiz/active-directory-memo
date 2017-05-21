# Active Directory - de l'installation à l'exploitation
Le document présent est un résumé / mémo de la procédure d'installation d'un service Active Directory classique dans le cadre d'une entreprise comme d'un laboratoire. Il décrit l'intégralité des différentes manipulations à effectuer pour fournir tous les services dont une entreprise aurait besoin. Ces services sont décrits ci-dessous.

## Sommaire
1. Architecture
  * Machines
  * Réseau
2. Installation des OS et des premières fonctionnalités
  * Systèmes d'exploitation
    * Serveur
    * Client
  * DNS / DHCP / Active Directory
  * Serveur de fichier et DFS
3. Installation des services
  * IIS & Services d'impression
  * WDS - Windows Deployment services
4. Configuration et Features
  * Quelques Stratégies de Groupe (GPO)
  * Ajout d'imprimante et drivers
  * Configuration type de l'AD
  * Installation de BGInfo

---
## Architecture
### Machines
Notre architecture se présente sous la forme suivante :
* 2 Serveurs Windows 2012 R2 respectivement nommés S1AD1 et S2AD2
* 1 ou plusieurs postes client utilisant Windows 7 Professionnel

### Réseau
Nos machines sont toutes dans le même réseau ipv4.
Dans le **cadre d'un laboratoire**, ce réseau n'a **pas besoin d'une connection internet**, c'est pourquoi nous utilisons sur chaque machine une interface virtuelle connectée sur un réseau interne ou un réseau host-only.
Pour notre exemple, nous avons choisi un **réseau host-only** en 10.33.10.0/24 **sans DHCP**. Nous avons utilisé ici un Réseau Host-only afin de faciliter les tests sur les différentes machines via l'Host. Bien évidemment, en "conditions réelles", les réseaux sont généralement routés sur internet et le DHCP est très souvent géré par le routeur. Bien que celà ne pose pas de problème, nous choisirons ici de détailler une installation comprenant les fonctionnalités DHCP Windows.

## Installation des OS et des premières fonctionnalités
### Systèmes d'exploitation
#### Serveur
Dans le cas de notre laboratoire, nous utilisons pour les serveurs S1AD1 et S2AD2 le même système d'exploitation : **Windows Server 2012 R2**</br>
* Lors du choix du type d'installation, choisir **l'installation avec interface graphique**.
* On choisit le disque et la partition sur lequel installer le système. Si aucun n'est disponible, une boîte de dialogue nous invite à créer une nouvelle partition NTFS où installer le système.
* On renseigne un mot de passe pour l'utilisateur Administrateur Local.
* En entreprise, lorsque le processus d'installation de Windows Server nous demande, nous **renseignons les clés d'activation produit**.</br>
Dans le cadre du laboratoire, nous n'avons pas de clés d'activation, nous utilisons KMSpico, un utilitaire permettant l'activation de Windows dans le cadre d'activité non-commerciales (alternative : Microsoft Toolkit). L'utilisation de ces logiciels est interdite dans le cadre de l'entreprise (ces derniers sont d'ailleurs souvent considérés par l'antivirus comme des programmes malveillants)
* On termine par configurer manuellement l'interface réseau en lui **attribuant une IP statique sur le réseau**. La spécification d'une passerelle par défault et d'un serveur DNS est fortement conseillée, en fonction des besoins de l'entreprise et de l'architecture historique de l'entreprise.</br>
Dans le cadre de notre laboratoire, S1AD1 et S2AD2 ont respectivement les IP 10.33.10.1 et 10.33.10.2 et un masque 255.255.255.0. Nous ne renseignons pas de Gateway ni de DNS auxiliaire.

#### Client
Les postes clients doivent être des postes Windows activés (comme pour les serveurs) et ne nécessitent pas de configuration manuelle initiale. Comme pour le serveur, on choisit le disque et la partition où installer le système, puis on se laisse guider jusqu'au processus d'activation de Windows. On crée un utilisateur Administrateur Local</br>
Dans le cadre du laboratoire, nous avons choisi **Windows 7 édition Professionnelle** activée avec KMSpico.

### DNS / DHCP / Active Directory
#### S1AD1
Nos machines sont prêtes à l'emploi. Installons les différents services de base sur S1AD1. Pour se faire, nous nous rendons dans le **Gestionnaire de Serveur**, par défaut lancé automatiquement lors du démarrage de Windows Server 2012 R2.
* Depuis le bouton **Gérér** en haut à droite, nous sélectionnons **Ajouter des rôles et Fonctionnalités**.
  * Une fenêtre s'ouvre, nous demandant de choisir un serveur sur lequel installer le service ainsi qu'un type d'installation, nous gardons les configurations par défaut. </br>--> **Suivant**
  * Lorsqu'apparaît la liste des rôles à installer, Sélectionner **Serveur DHCP**, **Serveur DNS** et **Serveur AD DS** puis Valider avec **Suivant**, laissez les fonctionnalités préselectionnées par défaut et finaliser l'installation. Des redémarrages sont nécessaires.

* Une fois l'installation terminée, nous devons effectuer les configurations post-déploiements des services DHCP et AD. les processus de configurations sont accessible depuis l'icône de **Notifications** (le drapeau) en haut à droite.
  * Pour se faire, nous allons commencer par **Promouvoir ce serveur en contrôleur de domaine** (Service AD).
    * Nous créons une nouvelle forêt avec le nom de domaine du client (ici comme exemple _domain.com_). </br>--> **Suivant**
    * Nous choisissons le **Niveau Fonctionnel de la forêt** correspondant à la version d'OS la plus ancienne que l'on puisse trouver dans la forêt. Comme ici nous n'utilisons que des machines sous _Windows Server 2012 R2_, nous sélectionnons cette valeur dans la liste défilante. On choisit également un **mot de passe DSRM**. </br>--> **Suivant**
    * Cliquer sur **Suivant** jusqu'à l'étape de confirmation de la configuration. On peut choisir une autre nom de domaine _NetBIOS_ que celui fournit par défaut, si souhaité.
    * Cliquer sur **Installer**. A la fin de l'opération, le serveur redémarrera automatiquement. Notez qu'après ce redémarrage, Windows vous demande de vous authentifier en tant qu'Administrateur du Domaine AD (ici DOMAIN\\Administrateur).
  * Ensuite, nous **Terminons la configuration de DHCP** depuis le même menu **Notifications**.
    * Sélectionner l'administrateur de l'AD que nous venons de créer (1ère option) </br>--> **Suivant**
    * **Terminer** la configuration.
    * Cependant, c'est un faux ami puisqu'en réalité on termine l'_installation_ du service. Nous devons toujours configurer une étendue d'IP à attribuer. Pour ce faire, se rendre dans la configuration **DHCP** depuis les **Outils** du **Gestionnaire de Serveur**. Depuis la fenêtre qui s'ouvre sélectionner le **serveur** DHCP (s1ad1.domain.com pour notre laboratoire), faire _Clic Droit_ sur **IPv4** et configurer une **Nouvelle étendue**. On renseigne alors les **IPs de début et de fin**, optionnellement quelques IPs à bannir des IPs statiques qui seraient comprises entre les bornes de l'étendue, comme par exemple des serveurs ou des équipement réseau), puis on finit par le **masque** et la **passerelle par défaut** (.254 usuellement).

#### S2AD2
* Nous ajoutons maintenant S2AD2 au domaine que nous venons de créer. Pour ce faire, on se rend dans **l'Explorateur de fichier** de S2AD2.
  * Clic droit sur **Ce PC** --> **Prorpiétés**
  * Dans la section _Paramètres de nom de l'ordinateur, de domaine et de groupe de travail_ on clique sur **Modifier les paramètres**. Une fenêtre s'ouvre, et en bas de l'onglet _Nom de l'ordinateur_, nous cliquons sur **Modifier**
  * Nous choisissons d'intégrer l'ordinateur au domaine créé précédement (ici domain.com). Une fenêtre d'authentification apparaît. Nous indiquons ici en utilisateur "Administrateur" et en mot de passe le mot de passe de l'administrateur de l'AD. Une fenêtre de confirmation nous souhaite la bienvenue dans le domaine "domain.com". Un redémarrage est nécessaire ici. Encore une fois,


* Nous installons comme pour le serveur S1AD1 le rôle **Serveur AD DS** (sans les rôles DHCP et DNS) puis nous procédons à la configuration post-déploiement en cliquant sur **Promouvoir ce serveur en contrôleur de domaine** dans le menu des **Notifications**.
  * Nous **ajoutons le contrôleur de domaine à un domaine existant** : ici domain.com</br>
  Des informations d'authentification sont nécessaires : **on s'authentifie** en tant qu'administrateur du domaine (Administrateur/MotDePasseAdminDeActiveDirectory). </br>--> **Suivant**
  * Le nouveau contrôleur de domaine peut être utilisé comme DNS secondaire. Cocher ou non la case correspondante pour activer le rôle. On tape un **mot de passe DSRM** comme pour le 1er serveur AD. </br>--> **Suivant**
  * Après vérification de la configuration, cliquer sur **Installer**. A la fin de l'installation, comme pour le premier serveur, un redémarrage sera nécessaire, et nous pourrons nous connecter en tant qu'utilisateur ou administrateur du domaine.

### Serveur de fichier et DFS
Le rôle **Serveur de fichier** est un des rôles les plus couramment utilisés. Pour que ce rôle soit utilisable, nous avons besoin d'une **partition NTFS** de données. On peut facilement en créer une depuis le **Gestionnaire de Serveur** via le bouton **Outils** en haut à droite et l'option **Gestion de l'ordinateur**. Dans la fenêtre qui s'ouvre, sélectionner l'onglet **Stockage** --> **Gestion des Disques**.

Une fois notre partition créée, on ajoute les rôles **Serveur de fichiers** --> **Réplication DFS**, **Espace de nom DFS** et **Gestionnaire de ressources du serveur de fichiers**. </br>--> **Suivant** --> **Suivant** --> **Suivant**--> **Installer**.</br>
Une fois l'installation terminée, on ouvre l'interface de configuration DFS via le bouton **Outils** --> **Gestion du système de fichiers distribués DFS** du **Gestionnaire de Serveur**.

#### Espace de noms DFS
Pour commencer, nous allons créer un espace de nom en cliquant sur **Action** --> **Nouvel espace de nom** dans la barre de menu.
* Depuis cette nouvelle fenêtre, Sélectionner un serveur DFS qui hébergera l'espace de nom. (ici s1ad1)</br>--> **Suivant**
* Nommer l'espace de nom (ex: Ressources)</br>--> **Suivant**
* Choisir un type d'espace de nom (Local / Domaine)</br>--> **Suivant**
* Confirmer la configuration et **Créer** l'espace de nom.

Dès lors, nous pouvons accéder au contenu du dossier ainsi créé depuis n'importe quel ordinateur du domaine via l'explorateur de fichier ( \\\\domain.com\NomDeLEspaceDeNom ). Le chemin d'accès du dossier sur le serveur hébergeur est par défault sur la partition système (ex C:) dans C:\\_DFSRoots_\\_NomDeLEspaceDeNom_

#### Réplication DFS
Toujours depuis le **Gestionnaire du système de fichier distribués DFS**, on crée un **Nouveau groupe de réplication**, et une fenêtre s'ouvre.
* On commence par sélectionner le **type** (_Groupe de réplication multi-usage_), le **nommer** puis on **ajoute au moins 2 serveurs** (dans notre laboratoire, S1AD1 et S2AD2) au groupe.
* Il nous faut ensuite choisir la **topologie** de notre groupe de réplication parmi Hub et **Maille pleine**, que nous choisirons. Une dernière option permet de personnaliser la topologie.
* Nous pouvons accessoirement définir des restrictions de bande passante et horaires sur le processus de réplication automatique.
* On choisit un **Membre principal**, correspondant au serveur sur lequel le ou les dossiers que nous voulons répliquer sont hébergés.
* Nous renseignons les **chemins des dossiers à répliquer** sur le **Membre principal**.
* Puis nous renseignons le **chemin du dossier de destination** sur chacun des serveurs membre du groupe de réplication (avec la possibilité de le paramétrer en lecture seule).
* Finalement, on **Vérifie les paramètres** puis **Confirme** la création du groupe de réplication.

Nos dossiers sont maintenant correctement répliqués sur chacun des membres du groupe de réplication.

## Installation des services
### IIS & Services d'impression
Ces 2 fonctionnalités sont facilement installables via le bouton **Gérer** du **Gestionnaire de serveur**. Comme pour les autres services, nous allons les sélectionner dans la liste, ajoutons les fonctionnalités proposées par défaut et procédons à l'installation.

#### ISS - Serveur Web
Windows propose une interface de création, administration et configuration de sites web sur une flotte de serveur présent dans un même domaine, que l'on peut trouver grâce au bouton **Outils** --> **Gestionnaire des services internet (IIS)** du **Gestionnaire de serveur**. Ajouter des pages, restreindre des IP, gérer directement les domaines et sous domaines, Imposer des authentifications ou activer le chiffrement des échanges par les protocoles SSL/TLS; toutes ces options sont disponibles depuis cet interface.

#### Serveur d'impression
On peut le configurer lui aussi depuis le bouton **Outils** --> **Gestion de l'impression** du **Gestionnaire de serveur**. Il permet de gérer les différentes imprimantes et serveurs d'impressions de l'AD, ainsi que les différents pilotes installés. Une fonctionnalité permet de créer des filtres d'imprimantes pour les très grosses architectures.

### WDS - Windows Deployment Services
#### Installation
D'après le MSDN "Les Services de déploiement Windows ("Windows Deployment Services" ou WDS) sont une **technologie** de Microsoft **permettant d'installer un système d'exploitation Windows via le réseau**". On installe la foncitonnalité depuis le **Gestionnaire de Serveur** en conservant les paramètres par défaut.</br>
Une fois installé on le configure : **Outils** --> **Services de déploiement Windows**. Une fenêtre apparaît :
* **Ajouter un serveur** à la liste des serveur DFS grâce à un _Clic Droit_ sur **Serveurs** et sélectionner le serveur local. (S1AD1)
* **Configurer le serveur** en faisant _Clic Droit_ sur le serveur fraîchement ajouté.
* Une fenêtre s'ouvre et nous renseigne les différents **éléments requis** pour le bon fonctionnement du service, qui sont **DHCP**, **DNS** (couplé à un AD ou non) et une **partition NTFS** pour stocker les images disques. On choisira généralement de créer une partition dédiée ou un espace dédié dans une partition de données existante.
* Si un AD est installé sur le serveur et si celui en est un des contrôleurs de domaine, on peut **ajouter le service de déploiement au domaine de l'AD**. Nous choisissons ici cette option.
* On **renseigne le chemin du dossier** contenant les images disques.
* On **configure DHCP** automatiquement en cochant les cases correspondantes dans _l'Assistant de configuration_. Cocher ces cases correspond à indiquer à notre serveur DHCP qu'il est à présent serveur PXE. On configure le service PXE en déterminant le comportement du serveur lorsqu'il reçoit des requêtes DHCP d'un poste sur le domaine.
* Selon le choix effectué à l'étape précédente, il sera peut-être nécessaire de **réserver une IP sur le réseau** pour chaque machine que l'on veut déployer automatiquement.

#### Configuration
Une fois l'ajout et la configuration du serveur terminé, une arborescence apparaît dans la section **Serveurs** du **Gestionnaire de service de déploiement Windows**.
* Depuis celle-ci, **Ajouter une Image d'installation**, encore une fois grâce à un _Clic Droit_ sur _Images d'installation_.
 * On commence par choisir le groupe d'images auquel il correspond (on en crée nouveau si il n'en existe pas encore)
 * On sélectionne le fichier **install.wim** pour une installation, situé dans le dossier _sources_ de l'image disque que nous voulons déployer et que nous aurons **montée sur le serveur** au préalable. (Depuis un hyperviseur, simplement insérer l'image disque dans le lecteur CD du serveur).
 * On sélectionne les installations que l'on désire déployer parmi celles disponibles dans le fichier. Windows en vérifie l'intégrité et l'ajoute au pool d'images.
* Puis, de la même façon, **Ajouter une Image de démarrage** en sélectionnant par le même procédé dans le même dossier le fichier **boot.wim**.
* Finalement, on crée une nouvelle **Transmission par multidiffusion** en sélectionnant le groupe d'images que l'on souhaite diffuser et **valider** (Une option permet de planifier les diffusions et installation afin de réduire les impacts sur les performance du réseau, en les programmant la nuit par exemple).


#### Test du déploiement sur une Machine virtuelle
Pour ce faire, créer une nouvelle machine virtuelle.
* Ouvrir l'interface de configuration du **DHCP** grâce au bouton **Outils** du **Gestionnaire de Serveur**, et sélectionner une étendue DHCP sur lequel réserver l'adresse.
* Configurer la carte réseau pour intégrer la machine dans le réseau de notre serveur Active Directory. Récupérer **l'adresse MAC** de la carte et réserver une IP dans le réseau au travers de la **configuration DHCP**. On crée une nouvelle **Réservation** dans la section **IPv4** --> **<Notre Etendue DHCP>**, on renseigne l'IP à attribuer et l'adresse MAC de la carte réseau de la machine à déployer. On garde les 2 types de requêtes activés (DHCP et BOOTP) ainsi que les options DHCP paramétrées par défaut.
* **Modifier le Boot Order** en ajoutant la possibilité de **boot sur le réseau** et en désignant cette option comme prioritaire.
* Lancer la VM.

## Configuration et Features
### Configuration type de l'AD en entreprise
Depuis le bouton **Outils** du **Gestionnaire de Serveur**, sélectionner **Utilisateurs et ordinateurs Active Directory** pour ouvrir la fenêtre de gestion des différents objets de notre AD. L'architecture organisationnelle qui suit est à titre d'exemple.
* On crée 3 Unités d'Organisation distinctes : _PAYE_, _COMPTA_, et _Etudes_.
* Dans chacune des UO, on crée les utilisateurs dont nous avons besoin (ici, 3 par UO).
* Dans chacune des UO, on crée un groupe de diffusion portant le nom de l'UO et contenant tous les utilisateurs.

Ces UO peuvent permettre d'attribuer plus facilement des GPO à seulement certains utilisateurs ou certains ordinateurs.

### Quelques Stratégies de Groupe (GPO)
Toutes les stratégies de groupes décrites ci-dessous peuvent être ajoutées depuis le menu **Outils** du **Gestionnaire de Serveur** en cliquant sur **Gestion des stratégies de groupe**. Dans la fenêtre qui s'ouvre, utiliser l'arborescence de fichier à gauche en sélectionnant notre _forêt_, puis notre _domaine_ et enfin **Objets de stratégie de groupe**. _Clic droit_ ou _Action_ --> **Nouveau**. La nommer, valider, puis par le même procédé, la **Modifier**.</br>
_NB : Notons que les stratégies relatives à un ordinateur nécessitent que l'ordinateur soit ajouté à l'UO à laquelle nous appliquons la GPO._

#### Règlement intérieur au démarrage
Cette GPO permet d'afficher une fenêtre de validation de CGU au démarrage d'un ordinateur enregistré dans la configuration de l'AD.</br>
Nous sélectionnons 2 règles situées dans **Règles pour
ordinateurs** --> **Stratégie** --> **Paramètres Windows** --> **Paramètres de sécurité** --> **Stratégies locales**.
* _Ouverture de session interactive : contenu du message pour les utilisateurs essayant de se connecter_
* _Ouverture de session interactive : titre du message pour les utilisateurs essayant de se connecter_

Remplir respectivement dans les fenêtres qui s'ouvrent les champs textes avec les GCU de l'entreprise  et le titre du message et cocher les boîtes **Définir ce paramètre de stratégie dans le modèle**. Valider.

#### Déployer automatiquement Mozzilla Firefox
Pour cette stratégie, nous avons besoin du **fichier .msi de Firefox.** </br>
* Une fois à notre disposition,le **placer dans le DFS** que nous avons créé précédemment pour le rendre accessible publiquement à l'AD.
* Chercher la stratégie de groupe dans **Configuration utilisateurs** --> **Stratégies** --> **Paramètres du logiciel** --> **Installation du logiciel**.
* Depuis le menu contextuel (Clic Droit) créer un **Nouveau** (-->) **Package**.
On sélectionne le fichier .msi dans l'emplacement DFS.
* **Publier** et **Valider** en prenant soin de cocher la case d'application de la Stratégie.

#### Créer un .msi de la Calculatrice
Plusieurs solutions existent pour se faire. Une des plus connues est l'utilisation du logiciel **MSI Wrapper**, permettant de générer un fichier .msi à partir d'un .exe de façon simple et intuitive.</br>
_NB : MSI Wrapper exporte le fichier .msi dans un dossier System. L'outil recherche de Windows peut être un outil appréciable pour récupérer le dit fichier._

#### Supprimer la fonction "Rechercher"
La règle se situe dans **Configuration utilisateurs** --> **Stratégies** --> **Modèles d’administrations** --> **Tous les paramètres**. Il suffit d'activer les 2 règles :
* _Supprimer le bouton Rechercher de l’Explorateur de fichiers_
* _Supprimer le lien Rechercher du menu Démarrer_

**Valider** en appliquant la GPO.

#### Forcer un fond d'écran du Bureau et en empêcher la modification
Chercher la règle **Papier peint du Bureau** dans **Configuration utilisateurs** --> **Stratégies** --> **Modèles d’administration** --> **Bureau**.</br>
Sélectionnez un fond d'écran et **Valider** en appliquant la GPO.

#### Forcer une page de démarrage par défault sur Internet Explorer
Chercher la règle **Désactiver la modification de la page d’accueil** dans **Configuration utilisateurs** --> **Stratégies** --> **Composant Windows** --> **Internet explorer**.</br>
Activer la règle, renseigner une page d'accueil par défaut et **Valider** en appliquant la GPO.

#### Ajout d'imprimantes et drivers
Techniquement, ceci n'est pas une GPO, mais il est tout de même pratique d'ajouter une imprimante et la rendre disponibles aux machines connectées sur l'AD, ainsi qu'en installer automatiquement les drivers. </br>
Pour ce faire :
* Se rendre dans le **Panneau de Configuration** du contrôleur de domaine de l'AD, dans la section **Périphériques et Imprimantes** et **Ajouter une Imprimante**.
* Sélectionner l'imprimante à ajouter dans la liste qui apparaît. Si l'imprimante à ajouter  ne se trouve pas dans la liste, cliquer sur **l'imprimante que je veux n'est pas répertoriée** et se laisser guider par l'installeur.
* Dans le cadre du laboratoire, nous renseignons une imprimante avec des paramètres manuels, et gardons les paramètres par défaut.
* On **choisit le modèle** dans la liste proposée pour en installer les **drivers** ou on indique un emplacement du driver via un chemin UNC.
* On **termine la configuration** en nommant l'imprimante, choisissant de **l'exposer sur le réseau** et lui attribuant un point d'accès sur l'AD.

#### Monter un Répertoire Réseau pour un utilisateur
Pour se faire, créer d'ores-et-déjà un répertoire dans lequel seront stockés les fichiers utilisateurs (Par exemple C:\\UserSpace) et on l'expose sur le réseau (via DFS pour un chemin UNC en fonction du domaine AD, via l'option de **Partage** dans les propriétés du dossier pour un UNC en fonction du hostname du serveur).</br>
A partir de là, dans l'interface de gestion des GPO se rendre dans **Configurations utilisateur** --> **Préférences** --> **Paramètres Windows** --> Clic droit sur **Mappages de lecteurs** --> **Nouveau** --> **Lecteur Mappé**. S'ouvre une fenêtre de création de nouveau lecteur. Dans celle-ci, on renseigne le nom, le chemin du dossier que l'on vient de créer suivi de "\\%USERNAME%" (ex : \\\\domain.com\\UserSpace\\%USERNAME%), nous permettant d'affecter cette règle dynamiquement en fonction de l'utilisateur.  Il est conseilllé de créer 2 règles avec les mêmes propriétés mais pour 2 actions différentes : **Créer** et **Mettre à Jour**.

#### Paramétrer une limitation de stockage par utilisateur
Pour terminer, nous allons nous familiariser un peu avec le **Gestionnaire de ressources du serveur de fichier**. Dans l'arborescence de gauche, nous trouvons les fonctionnalités **Gestion de quota** --> **Modèles de Quotas**. Nous pouvons choisir de créer un Modèle de Quotas, que l'on pourra ensuite utiliser lors de la création de quotas. Pour créer un quota, _Clic droit_ sur **Quotas** --> **Créer un quota** nous ouvre une fenêtre de configuration. Dans celle-ci, on renseigne l'emplacement du dossier auquel appliquer la limitation et on sélectionne dans la liste des modèles la restriction voulue ou on en édite une à la volée grâce aux _Paramètres Personnalisés_. Si on sélectionne le dossier hébergeant tous les dossiers personnels des utilisateurs de l'AD, on peut alors cocher la case _Appliquer automatiquement au modèle et créer des quotas sur les sous-dossiers existants et nouveaux_ pour restreindre la taille de chaque dossier personnel au montant définit par le modèle de quota choisit.


### BGInfo
BGInfo est un petit logiciel permettant d'afficher sur le bureau des informations systèmes choisies par l'Administrateur. Ce logiciel est très pratique en entreprise pour avoir à éviter d'ouvrir plein de fenêtres de configurations pour obtenir 1 petit renseignement (comme par exemple la passerelle par défaut) multiplié par le nombre de petits renseignements dont on a besoin. En bref, il permet d'économiser du temps et de réduire les potentielles erreurs lors de la manipulations des postes.
