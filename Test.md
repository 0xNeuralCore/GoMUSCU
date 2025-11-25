👍 Ce que tu dois implémenter (minimum viable)

1️⃣ Mappage IPv4 Multicast → MAC

Pour chaque group IP 224.x.x.x à 239.x.x.x

MAC = 01:00:5E:(IP & 0x7F: bit 23-0)

Exemple :
IP Multicast = 239.1.2.3
MAC = 01:00:5E:01:02:03

➡️ Au join group, tu configures la carte réseau pour accepter cette MAC.


---

2️⃣ IGMP v2 très réduit

Tu gères uniquement :

Membership Report (quand tu rejoins un groupe)

Ignore les Queries du routeur s’il n’y en a pas → réseau purement local


➡️ Si tu veux être safe : répondre aux Queries si reçues

Paquet IGMP Report (type 0x16) à envoyer à : → IP multicast du groupe → MAC derivée comme vu plus haut


---

3️⃣ API très simple dans ta stack

int joinMulticastGroup(int sock, ip_addr_t group);
int leaveMulticastGroup(int sock, ip_addr_t group);

Au joinMulticastGroup() :

1. Ajouter l’entrée dans table groupes du socket


2. Config NIC : accepter MAC multicast


3. Envoyer IGMP Report



Au leaveMulticastGroup() :

1. Retirer de la table


2. Envoyer IGMP Leave Group


3. Si plus aucun socket n’écoute → retirer le filtre MAC




---

4️⃣ Réception multicast

Quand un paquet arrive :

1. Verif si adresse MAC multicast → ok


2. Verif IP dest ∈ groupes abonnés


3. Distribuer aux sockets concernés (plusieurs possibles sur le même port)




---

🧱 Architecture minimale

┌────────────┐
│   App      │
│ join/leave │
└─────┬──────┘
      │
┌─────▼───────────────────┐
│     Socket API          │
│ group table per socket  │
└─────┬───────────────────┘
      │
┌─────▼────────────────┐
│ IGMP tiny module     │
│ send simple reports  │
└─────┬────────────────┘
      │
┌─────▼───────────────────────┐
│ NIV 3: IPv4 + multicast map │
└─────┬───────────────────────┘
      │
┌─────▼─────────────┐
│ Driver Ethernet   │
│ MAC filters       │
└───────────────────┘


---

✨ Résultat obtenu

✔ Multicast sur LAN
✔ Un seul émetteur → multiples récepteurs
✔ Aucun routage requis
✔ Charge CPU faible
✔ Facile sur microcontrôleurs / RTOS


---

👉 Ce que tu ne supportes pas (et c’est OK)

❌ TTL avancé / WAN multicast
❌ IGMP v3 / Sourcelist
❌ PIM / MOSPF / routage multi-interfaces
❌ Sécurité / filtrages poussés
