# SOC Lab — Wazuh SIEM

Projet d'apprentissage : déploiement d'une stack SIEM Wazuh, détection d'incidents, et réponse à incident.

## Statut

🚧 En cours — Jour 4/5 (règles de détection custom)

- Jour 1 ✅ Stack Wazuh déployée (manager, indexer, dashboard)
- Jour 2 ✅ VM cible Metasploitable2 + agent Wazuh connecté
- Jour 3 ✅ 3 attaques simulées et documentées (Nmap, brute force SSH, backdoor vsftpd)
- Jour 4 🚧 Règles de détection custom (en cours)
- Jour 5 ⬜ Playbooks + finalisation du dépôt

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

| Rule ID | Déclencheur | Sévérité | Type |
|---|---|---|---|
| 100100 | Connexion FTP suivie d'un changement de port dans les 15s (backdoor vsftpd) | 12 | Custom |
| 5760 | Échec d'authentification SSH (mot de passe invalide) | 5 | Native |
| 5763 | 8 échecs SSH en 120s depuis la même IP (brute force) | 10 | Native |

Contrairement à la règle vsftpd (nécessitant une règle composite custom), le brute force SSH est déjà couvert nativement par la règle 5763 de Wazuh (frequency=8, timeframe=120s, same_source_ip). Après vérification via wazuh-logtest, cette règle native remplit exactement le besoin — écrire une règle custom redondante n'aurait apporté aucune valeur.# SOC Lab — Wazuh SIEM

Projet d'apprentissage : déploiement d'une stack SIEM Wazuh, détection d'incidents, et réponse à incident.

## Statut
🚧 En cours — Jour 1/5 (déploiement de la stack Wazuh)

## Stack technique
- Wazuh 4.9.0 (manager, indexer, dashboard) déployé via Docker Compose
- Metasploitable2 comme cible vulnérable (à venir Jour 2)

## Structure du dépôt
- `docs/` — schémas et documentation technique
- `attacks/` — logs et résultats des scénarios d'attaque simulés
- `rules/` — règles de détection Wazuh personnalisées
- `playbooks/` — playbooks de réponse à incident
- `screenshots/` — captures d'écran des étapes clés
