Mettre en place un LAN segmenté par VLAN avec :

- Un routeur central (R) faisant l’inter-VLAN routing et serveur DHCP.
- Trois switches d’accès (S1, S2, S3).
- Six hôtes (deux par switch).
- Trois VLANs :
  - VLAN 10 : Utilisateurs
  - VLAN 20 : Serveurs
  - VLAN 30 : Administration (inclut management des switches)
- Deux serveurs HTTP (m3, m4) dans le VLAN 20.

------

## 2) Topologie logique et câblage

Architecture “router-on-a-stick” :

- R connecté en trunk 802.1Q vers S1.
- Trunks 802.1Q entre S1 ↔ S2 et S1 ↔ S3.
- VLANs 10/20/30 transportés sur tous les trunks.

Répartition des hôtes (2 par switch) :

- S1: m1-user (VLAN 10), m3-serverhttp (VLAN 20)
- S2: m2-user (VLAN 10), m4-serverhttp (VLAN 20)
- S3: m5-admin (VLAN 30), m6-admin (VLAN 30)

------

## 3) VLANs et plan d’adressage

Passerelles (interfaces subif sur R) :

- VLAN 10 Utilisateurs : 192.168.10.1/24
- VLAN 20 Serveurs : 192.168.20.1/24
- VLAN 30 Administration : 192.168.30.1/24

Plages DHCP (par VLAN) :

- VLAN 10 : 192.168.10.10 – 192.168.10.250

- VLAN 20 : 192.168.20.10 – 192.168.20.250

- VLAN 30 : 192.168.30.20 – 192.168.30.250

Réservations :

- Réservation pour serveurs HTTP (adresses statiques): 
  - m3: 192.168.20.10/24 (GW 192.168.20.1)
  - m4: 192.168.20.11/24 (GW 192.168.20.1)

- Réservations pour les Switchs Management :
  - S1: 192.168.30.11/24 (passerelle: 192.168.30.1)
  
  - S2: 192.168.30.12/24 (passerelle: 192.168.30.1)
  - S3: 192.168.30.13/24 (passerelle: 192.168.30.1)
  
- Domaine: local.lan

- Durée de bail indicative: 7 jours

------

## 4) Affectation des hôtes et management

Hôtes:

- m1-user (S1): VLAN 10, IP par DHCP
- m2-user (S2): VLAN 10, IP par DHCP
- m3-serverhttp (S1): VLAN 20, 192.168.20.10/24, GW 192.168.20.1 (statique)
- m4-serverhttp (S2): VLAN 20, 192.168.20.11/24, GW 192.168.20.1 (statique)
- m5-admin (S3): VLAN 30, IP par DHCP
- m6-admin (S3): VLAN 30, IP par DHCP

------

## 6) Politique de sécurité (ACL) et placement

Objectifs de filtrage :

- Utilisateurs (VLAN 10) → Serveurs (VLAN 20) :
  - Autorisé : ICMP (ping) et HTTP (TCP/80) uniquement
  - Interdit : tout autre flux
  - Interdit : accès vers VLAN 30 (Admin)
- Serveurs (VLAN 20) → Utilisateurs (VLAN 10):
  - Interdiction d’initiation de flux
  - Autorisés uniquement: retours TCP d’une session établie depuis VLAN 10 (established) et réponses ICMP (echo-reply)
  - Autorisé: trafic vers VLAN 30 (Administration)
- Administration (VLAN 30) → tous:
  - Tout autorisé (gestion switches/serveurs incluse)

Placement des ACL (inbound sur le routeur R):

- Sur l’interface subif G0/0.10 (VLAN 10): ACL USERS_IN
- Sur l’interface subif G0/0.20 (VLAN 20): ACL SERVERS_IN
- Sur l’interface subif G0/0.30 (VLAN 30): ACL ADMIN_IN

Remarque DHCP :

- Le serveur DHCP étant sur R, prévoir l’autorisation du trafic DHCP client → serveur (UDP 68 → 67) dans USERS_IN et ADMIN_IN si l’ACL bloque les paquets destinés au routeur.
