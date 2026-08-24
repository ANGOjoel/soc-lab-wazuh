# Playbook — Brute force SSH

## Préparation
- Règle de détection : 5763 (native Wazuh), niveau 10, déclenchée après 8 échecs d'authentification SSH en 120 secondes depuis la même IP source.
- Outils nécessaires : accès au dashboard Wazuh, accès SSH/console à la machine ciblée, pare-feu (iptables ou équivalent) pour le confinement.

## Identification
- Alerte visible dans Security events : `rule.id: 5763`, description "sshd: brute force trying to get access to the system".
- Vérifier l'IP source de l'attaque (`srcip` dans les détails de l'alerte) et l'utilisateur ciblé (`dstuser`).
- Confirmer qu'il ne s'agit pas d'un faux positif (ex. un script légitime mal configuré qui échoue en boucle).

## Confinement
- Bloquer immédiatement l'IP source identifiée :
```bash
  sudo iptables -A INPUT -s <IP_source> -j DROP
```
- Alternative recommandée en production : utiliser `fail2ban` pour un blocage automatique et temporaire.

## Éradication / Reprise
- Vérifier si l'attaquant a réussi à se connecter (croiser avec les logs d'authentification réussie sur la même période).
- Si une connexion réussie est constatée : rotation immédiate du mot de passe compromis, et de tout mot de passe réutilisé ailleurs.
- Vérifier l'absence de compte ou de clé SSH ajoutée frauduleusement (`~/.ssh/authorized_keys`, `/etc/passwd`).

## Retour d'expérience
- Ce scénario a été simulé au Jour 3 (Hydra) sans détection corrélée, puis confirmé détecté au Jour 4 grâce à la règle native 5763.
- Amélioration possible : intégrer un blocage automatique (SOAR / script déclenché par l'alerte) plutôt qu'une intervention manuelle.
