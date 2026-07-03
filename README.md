# SNORTPFSENSE


## Architecture

## pfSense (VirtualBox, 2 interfaces réseau)
- WAN : carte en mode Bridge, IP 192.168.1.131/24 — simule l'accès depuis "l'extérieur"
- LAN : réseau interne 10.10.0.1/16
## Machine Attaquante
- VM Ubuntu en Bridge sur le même sous-résau que le WAN pfSense
- Outils utilisé : nmap, hydra

## Configuration

1. Installation de Snort

Installation du paquet via System > Package Manager > Available Packages.

2. Règles de firewall sur WAN

Ajout de règles autorisant explicitement le trafic entrant sur les ports 22 (SSH), 80 (HTTP) et 443 (HTTPS) dans Firewall > Rules > WAN, conformément au scénario (seuls ces ports doivent être exposés).

3. Configuration de l'interface Snort

Dans Services > Snort > Snort Interfaces :


Interface surveillée : WAN
Block Offenders activé 
Block Which Host : Both
Kill States activé → coupe les connexions déjà établies avec l'IP bloquée

![SCREEN1](/SCREENS/1ERSCREEN.png)

4. Règles activées (Categories / Rules)

Catégories de règles activées (ruleset Emerging Threats Open) :


emerging-scan.rules
emerging-policy.rules
emerging-attack_response.rules

Recherche et activation manuelle des règles spécifiques contenant les mots-clés ssh, scan, nmap, brute dans l'onglet Rules.

5. Démarrage de Snort

Activation de l'inspection sur l'interface WAN

![SCREEN2](/SCREENS/2EMESCREEN.png)

Test 1 — Scan de ports (nmap)

Depuis la VM Ubuntu attaquante :

```sudo nmap -sS -p- 192.168.1.131```

Test 2 — Brute force SSH (hydra)

```hydra -L /tmp/users.txt -P /tmp/pass.txt ssh://192.168.1.131```

![SCREEN3](/SCREENS/3EMESCREEN.png)

Résultats — Détection et blocage confirmés

Alertes générées (Services > Snort > Alerts)

Snort a détecté les deux types d'attaques et généré des alertes correspondantes :

| Règle déclenchée | Type d'attaque détectée |
|---|---|
| ET SCAN Suspicious inbound to mySQL port 3306 | Scan de ports (nmap) |
| ET SCAN Potential SSH Scan | Scan de ports ciblant SSH |
| ET SCAN LibSSH Based Frequent SSH Connections Likely BruteForce Attack | Brute force SSH (hydra) |

![SCREEN4](/SCREENS/SCREEN4.jpg.png)

Hôtes bloqués (Services > Snort > Blocked)

L'IP de la machine attaquante (192.168.1.2) a été automatiquement bloquée par Snort suite aux deux types d'attaques 

![SCREEN5](/SCREENS/SCREEN5.jpg.png)

