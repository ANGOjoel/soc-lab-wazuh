# Playbook — Alerte FIM (File Integrity Monitoring)

## Préparation
- Module Wazuh syscheck activé sur un répertoire sensible (ex. `/etc`).
- Outils nécessaires : accès au dashboard Wazuh, accès console à l'hôte concerné, système de contrôle de version ou sauvegarde de référence pour comparaison.

## Identification
- Alerte FIM visible dans Security events, avec le fichier modifié, le type de modification (ajout/modification/suppression), et l'horodatage.
- Vérifier qui ou quel processus a réalisé la modification si l'information est disponible (utilisateur, PID).

## Vérification (changement légitime ou compromission)
- Comparer la modification avec les actions planifiées connues (mise à jour système, changement de configuration volontaire par un admin).
- Si aucune action légitime ne correspond : traiter comme suspect et passer en investigation approfondie.
- Vérifier le contenu du fichier modifié pour détecter un ajout malveillant (ex. backdoor dans un fichier de configuration, nouvel utilisateur dans `/etc/passwd`).

## Remédiation
- Si changement illégitime confirmé : restaurer le fichier depuis une version saine connue (sauvegarde ou dépôt de configuration versionné).
- Si un compte ou une clé d'accès a été ajouté frauduleusement : le supprimer immédiatement et effectuer une rotation des identifiants concernés.
- Documenter la modification dans un rapport d'incident, même en cas de faux positif, pour affiner les règles FIM à l'avenir.

## Retour d'expérience
- Le FIM est complémentaire aux règles de corrélation de logs : il détecte les changements sur le système de fichiers que d'autres règles (réseau, authentification) peuvent manquer.
- Limite observée pendant le lab : une attaque s'exécutant uniquement en mémoire (comme Meterpreter au Jour 3) n'écrit rien sur le disque surveillé et échappe donc totalement à la détection FIM — à ne pas considérer comme une protection suffisante à elle seule.
