# stack-servarr-deploiement

Déploiement d'une infrastructure multimédia 100% automatisée sous Docker (Jellyfin, Servarr, qBittorrent). Gestion de l'acquisition P2P sécurisée, optimisation du stockage via hardlinks et accès distant par Tailscale.

## Configuration Détaillée par Application

La configuration doit suivre un ordre précis pour garantir la bonne communication des API.

## 1. qBittorrent (Acquisition & Sécurité P2P)

Le client doit être sécurisé pour l'utilisation de trackers privés et configuré pour l'automatisation.

Dans l'onglet qBittorrent : 

* **Mode de chiffrement :** Forcé sur exiger le chiffrement.

* **Réseaux décentralisés :** DHT, PeX et LSD décochés pour éviter les fuites d'adresses IP hors des trackers privés.

* **Catégorisation :** Création des catégories internes radarr et tv-sonarr pour aiguiller les téléchargements.

* **Ratio & Rétention :** Limite de partage fixée à un Ratio de 1.0 (ou 7 jours) pour automatiser la mise en pause des torrents complétés.

## 2. Prowlarr & FlareSolverr (Indexation Centralisée)

Prowlarr remplace Jackett et agit comme l'annuaire central des trackers : 

* **Liaison FlareSolverr :** Dans Settings > Indexers > Proxies, ajout de FlareSolverr avec l'URL interne http://flaresolverr:8191.

* **Ajout des Trackers (ex: C411, Torrent9) :** Saisie des identifiants ou de l'API Key spécifique au tracker.

    - Tags : Le champ Tags (ex: flare) doit impérativement rester vide pour ne pas bloquer la synchronisation vers Sonarr/Radarr.

    Synchronisation Automatique (Settings > Apps) :

        Ajout de Sonarr (http://sonarr:8989) et Radarr (http://radarr:7878) en utilisant leurs API Keys respectives.

        Sync Level réglé sur Full Sync. Prowlarr pousse désormais tous les trackers configurés directement dans les applications cibles.

3. Sonarr & Radarr (Orchestration PVR)

Ces applications automatisent la recherche, la sélection (ex: ciblage de la qualité Remux 1080p/4K) et le tri des médias.

    Liaison qBittorrent (Settings > Download Clients) :

        Host : Utilisation du nom du conteneur qbittorrent au lieu de l'adresse IP locale (évite les pannes en cas de changement d'IP dynamique).

        Catégories : Association stricte aux catégories tv-sonarr et radarr créées précédemment.

        Activation de la fonction Remove Completed pour supprimer automatiquement le .torrent de qBittorrent une fois le ratio de seed atteint.

    Gestion des Fichiers (Settings > Media Management) :

        Activation de l'option Hardlinks. Sonarr/Radarr crée un lien physique au lieu de copier le fichier, économisant 100% de l'espace disque pendant la période de seed.

4. Jellyseerr (Portail Utilisateur)

    Connecté aux API de Sonarr, Radarr et Jellyfin pour permettre aux utilisateurs de formuler des requêtes d'ajout de contenu d'un simple clic. Les requêtes approuvées déclenchent toute la chaîne automatisée.

🛡️ Procédures de Maintenance et Bonnes Pratiques

Gestion des suppressions (Libération d'espace disque)
La suppression d'un média (ex: une saison déjà visionnée) ne doit jamais être effectuée depuis l'interface de Jellyfin. Si Jellyfin supprime le fichier, Sonarr/Radarr détectera l'absence et relancera le téléchargement.

    Procédure : Aller dans Sonarr/Radarr, cliquer sur la corbeille rouge pour supprimer les fichiers physiques, puis passer le média en statut Unmonitored (icône marque-page) pour empêcher toute nouvelle recherche automatique.

Mise à jour des conteneurs
Afin de maintenir la stabilité et la sécurité des API (notamment TMDB/TVDB pour Jellyseerr), les conteneurs sont mis à jour individuellement de manière transparente :
