# Schéma réseau du lab

## Vue d'ensemble

Machine hôte Windows, avec WSL2/Docker qui fait tourner le manager (ports 1514/1515), l'indexer (port 9200) et le dashboard (port 443).

Cette machine hôte possède une carte réseau Host-only à l'adresse 192.168.56.1, connectée au même réseau que Metasploitable2 (192.168.56.101) et, plus tard, une VM Kali attaquante.

## Adressage

| Machine | Interface | Adresse IP | Rôle |
|---|---|---|---|
| Hôte Windows | Host-only | 192.168.56.1 | Fait tourner Docker/Wazuh manager |
| Metasploitable2 | eth0 (Host-only) | 192.168.56.101 | Cible vulnérable, agent Wazuh |
| Metasploitable2 | eth1 (NAT) | 10.0.3.x | Accès Internet (téléchargements) |
| Kali (à venir) | Host-only | 192.168.56.x | Machine attaquante |

## Ports utilisés

- 1514/TCP : communication agent vers manager (envoi des logs)
- 1515/TCP : enregistrement initial de l'agent auprès du manager
- 9200/TCP : API interne de l'indexer (OpenSearch)
- 443/TCP : dashboard web (HTTPS)

## Notes techniques

- Le réseau Host-only isole le lab d'Internet tout en permettant la communication entre VMs et la machine hôte.
- Metasploitable2 nécessite une seconde carte réseau (NAT) pour l'accès Internet, le Host-only seul ne le permettant pas.
- L'agent Wazuh installé est en version 3.13.6 (plutôt que 4.9.0) en raison d'une incompatibilité de bibliothèques système avec l'architecture 32-bit de Metasploitable2.
- Protocole de communication agent/manager : TCP (ajusté manuellement, la version 3.13.6 utilisant UDP par défaut, incompatible avec le manager qui écoute en TCP).
