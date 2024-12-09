
# Installation de Snort 3 avec LuaJIT sur Ubuntu

Ce guide détaille les étapes pour installer Snort 3 avec ses dépendances, y compris LuaJIT, sur Ubuntu.

## Prérequis

Assurez-vous que votre système est à jour.

```bash
sudo apt update
sudo apt upgrade -y
```

## Étape 1 : Installation des dépendances

Snort 3 nécessite plusieurs bibliothèques et outils pour la compilation. Installez les dépendances de base :

```bash
sudo apt install -y build-essential cmake libpcap-dev libpcre3-dev zlib1g-dev \
libhwloc-dev libluajit-5.1-dev libssl-dev libdnet-dev bison flex \
liblzma-dev libunwind-dev uuid-dev asciidoc dblatex source-highlight w3m
```

## Étape 2 : Installation et configuration de DAQ (Data Acquisition Library)

DAQ est nécessaire pour la capture de paquets dans Snort.

1. **Téléchargez DAQ** :

   ```bash
   wget https://www.snort.org/downloads/snort/daq-2.0.7.tar.gz
   tar -xzvf daq-2.0.7.tar.gz
   cd daq-2.0.7
   ```

2. **Compilez et installez DAQ** :

   ```bash
   ./configure --prefix=/usr/local/lib/daq_s3
   make
   sudo make install
   ```

   DAQ sera installé dans le répertoire `/usr/local/lib/daq_s3`.

## Étape 3 : Installation de LuaJIT

Snort 3 utilise LuaJIT, qui doit être installé séparément.

1. **Téléchargez et compilez LuaJIT** :

   ```bash
   wget http://luajit.org/download/LuaJIT-2.0.5.tar.gz
   tar -xzf LuaJIT-2.0.5.tar.gz
   cd LuaJIT-2.0.5
   make
   sudo make install
   ```

2. **Mettez à jour le cache des bibliothèques** :

   ```bash
   sudo ldconfig
   ```

## Étape 4 : Téléchargement et configuration de Snort 3

1. **Téléchargez Snort 3** :

   ```bash
   wget https://www.snort.org/downloads/snort/snort3-3.1.35.0.tar.gz
   tar -xzvf snort3-3.1.35.0.tar.gz
   cd snort3-3.1.35.0
   ```

2. **Configurez Snort 3 avec les chemins de DAQ** :

   Définissez une variable pour le chemin d’installation de Snort, comme suit :

   ```bash
   export my_path=/usr/local/snort
   ```

3. **Lancez la configuration de CMake** :

   ```bash
   ./configure_cmake.sh --prefix=$my_path \
                        --with-daq-includes=/usr/local/lib/daq_s3/include/ \
                        --with-daq-libraries=/usr/local/lib/daq_s3/lib/
   ```

4. **Compilez et installez Snort 3** :

   ```bash
   cd build
   make -j $(nproc)
   sudo make install
   ```

5. **Vérifiez que Snort 3 est installé** :

   Pour confirmer que Snort est bien installé, exécutez :

   ```bash
   $my_path/bin/snort -V
   ```

## Étape 5 : Configuration supplémentaire

1. **Ajouter Snort au PATH** (optionnel) :

   Pour accéder à Snort sans spécifier le chemin complet, ajoutez-le au `PATH` :

   ```bash
   export PATH=$PATH:$my_path/bin
   ```

2. **Vérifiez les modules DAQ disponibles** :

   ```bash
   snort --daq-list
   ```

## Résumé des étapes de vérification

- Assurez-vous que Snort est installé et accessible avec `$my_path/bin/snort -V`.
- Confirmez que DAQ est correctement configuré avec `snort --daq-list`.

___

# Explication de `$(nproc)` et des Cœurs de Processeur

## `$(nproc)`

La commande `$(nproc)` retourne le nombre de cœurs de processeur disponibles sur une machine. Elle est particulièrement utile lors de la compilation de programmes, car elle permet d'utiliser plusieurs cœurs pour paralléliser les tâches, ce qui accélère considérablement le processus.

### Exemple d'utilisation

```bash
make -j $(nproc)
````
Dans cet exemple, la commande make utilisera autant de tâches parallèles qu'il y a de cœurs de processeur. Cela permet d'optimiser la compilation en tirant parti des ressources de la machine.

## À quoi servent les Cœurs de Processeur ?

Les cœurs de processeur permettent d'exécuter plusieurs tâches simultanément, améliorant ainsi la vitesse et l'efficacité du traitement. En réalité, plus il y a de cœurs, plus le processeur peut gérer simultanément de processus ou de threads, ce qui :
- Accélère le traitement de tâches complexes : De lourdes tâches complexes sont exécutées plus rapidement.
- Améliore les performances dans les environnements multitâches : Plusieurs applications peuvent fonctionner sans ralentissement.
- Réduit le temps de compilation de programmes ou de traitement de données massives : Les processus intensifs bénéficient d'un gain de temps significatif

___

> - **ld.so.conf.d** est un dossier sur Linux qui contient des fichiers de configuration listant les chemins des bibliothèques partagées. Le système utilise ces fichiers pour savoir où chercher les bibliothèques nécessaires à l’exécution des programmes.

- Snort + its Libraries (libpcap, DAQ, LuaJIT, etc...)

````bash
 /usr/local/snort/bin/snort -V --daq-list

   ,,_     -*> Snort++ <*-
  o"  )~   Version 3.5.0.0
   ''''    By Martin Roesch & The Snort Team
           http://snort.org/contact#team
           Copyright (C) 2014-2024 Cisco and/or its affiliates. All rights reserved.
           Copyright (C) 1998-2013 Sourcefire, Inc., et al.
           Using DAQ version 3.0.17
           Using libpcap version 1.10.5 (with TPACKET_V3)
           Using LuaJIT version 2.1.1727870382
           Using LZMA version 5.6.2
           Using OpenSSL 3.3.1 4 Jun 2024
           Using PCRE version 8.39 2016-06-14
           Using ZLIB version 1.3.1
````
___

## Creating an alias

- **Ouvrez votre fichier de configuration de shell** (par exemple, .bashrc ou .zshrc si vous utilisez Zsh) :

````bash
nano ~/.bashrc
````
- **Ajoutez l'alias en ajoutant cette ligne à la fin du fichier** :

````bash
alias sn='/usr/local/snort/bin/snort'
````
- **Rechargez le fichier de configuration pour appliquer l'alias** :

````bash
source ~/.bashrc
````
- **Utiliser sn pour exécuter Snort** :

````bash
sn -v
````
___

- la commande ethtool -K eth0 gro off lro off désactive les fonctionnalités GRO (Generic Receive Offload) et LRO (Large Receive Offload) sur l'interface eth0. Ces fonctionnalités sont souvent utilisées pour améliorer les performances en regroupant plusieurs paquets en un seul avant de les transmettre à la pile réseau.
