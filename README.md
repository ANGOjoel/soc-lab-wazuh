# SOC Lab — Wazuh SIEM

Projet d'apprentissage : déploiement d'une stack SIEM Wazuh, détection d'incidents, et réponse à incident.

## En résumé (pour les non-initiés)

Ce projet reproduit, à petite échelle, ce que fait une équipe de cybersécurité en entreprise (un "SOC", Security Operations Center) : surveiller des ordinateurs en temps réel pour détecter des attaques.

Concrètement :
1. J'ai installé un logiciel de surveillance (Wazuh) qui lit en continu les journaux d'activité d'une machine.
2. J'ai volontairement attaqué une machine test (scan, tentative de piratage par mot de passe, exploitation d'une faille connue) pour voir ce que le logiciel détectait ou ratait.
3. J'ai écrit des règles personnalisées pour améliorer la détection des attaques les plus dangereuses.
4. J'ai rédigé des "playbooks" — des procédures à suivre étape par étape en cas d'attaque réelle, comme un pompier a un protocole d'intervention.

Le reste de ce document est plus technique et détaille précisément comment tout ça a été construit.

## Liens rapides

- [Schéma réseau du lab](docs/network-diagram.md)
- [Scénarios d'attaque (Jour 3)](attacks/scenario-log.md)
- [Règles de détection personnalisées](rules/local_rules.xml)
- [Playbooks de réponse à incident](playbooks/)
- [Résultat du scan Nmap](scans/nmap-full.txt)
- [Captures d'écran](screenshots/)

## Chronologie du projet

| Jour | Objectif | Résultat |
|---|---|---|
| 1 | Déployer la stack Wazuh en Docker | ✅ Dashboard opérationnel, mot de passe changé |
| 2 | VM vulnérable + agent Wazuh connecté | ✅ Metasploitable2 + agent actif, logs remontés |
| 3 | Simuler 3 attaques, observer la détection par défaut | ✅ [Voir le détail](attacks/scenario-log.md) — Nmap invisible, SSH détecté sans corrélation, backdoor partiellement détecté |
| 4 | Écrire des règles de détection custom | ✅ Règle vsftpd ([100100](rules/local_rules.xml)) + validation brute force SSH (règle native 5763) |
| 5 | Playbooks + finalisation | ✅ [3 playbooks PICERL](playbooks/), dépôt taggé v1.0 |

## Stack technique

- Wazuh 4.9.0 (manager, indexer, dashboard) déployé via Docker Compose
- Metasploitable2 comme cible vulnérable, agent Wazuh actif et connecté

## Structure du dépôt

- `docs/` — schémas et documentation technique
- `attacks/` — logs et résultats des scénarios d'attaque simulés
- `rules/` — règles de détection Wazuh personnalisées
- `playbooks/` — playbooks de réponse à incident
- `screenshots/` — captures d'écran des étapes clés

## Règles de détection

| Rule ID | Déclencheur | Sévérité | Type | Constat Jour 3 comblé |
|---|---|---|---|---|
| 100100 | Connexion FTP suivie d'un changement de port dans les 15s (backdoor vsftpd) | 12 | Custom | Scénario 3 : signaux faibles non corrélés (FTP + port) |
| 5760 | Échec d'authentification SSH (mot de passe invalide) | 5 | Native | — |
| 5763 | 8 échecs SSH en 120s depuis la même IP (brute force) | 10 | Native | Scénario 2 : échecs individuels non corrélés en alerte unique |

Contrairement à la règle vsftpd (nécessitant une règle composite custom), le brute force SSH est déjà couvert nativement par la règle 5763 de Wazuh (frequency=8, timeframe=120s, same_source_ip). Après vérification via wazuh-logtest, cette règle native remplit exactement le besoin — écrire une règle custom redondante n'aurait apporté aucune valeur.# SOC Lab — Wazuh SIEM


