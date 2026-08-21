# SOC Lab — Wazuh SIEM

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
