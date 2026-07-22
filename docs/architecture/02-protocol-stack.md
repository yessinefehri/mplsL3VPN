```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  A-02 · STACK PROTOCOLAIRE COMPLÈTE                                          │
│  Plan de contrôle · Plan de données · Ordre de déploiement                   │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `STACK: 5 COUCHES` · `MODE: SEQUENTIAL DEPLOY`

─── [ 01 / STACK COMPLÈTE ] ────────────────────────────────────────────────────

| # | Couche | Protocole | Fonction | Dépend de |
|:--|:-------|:----------|:---------|:----------|
| 1 | IGP | `OSPF Area 0` | Loopbacks + liens cœur | — |
| 2 | MPLS | `LDP` | Labels + LSP | OSPF |
| 3 | VPN Control | `MP-BGP VPNv4` | Routes clients PE↔PE | OSPF (loopbacks) |
| 4 | Isolation | `VRF CUST_A/B` | Tables séparées | — |
| 5 | Edge | `Static + redistribute` | Injection routes CE | VRF + MP-BGP |

```text
╭─────────────────────────────────────────────────────────────────────╮
│                     PLAN DE CONTRÔLE                                │
│                                                                     │
│  OSPF ──► LDP ──► MP-BGP VPNv4                                     │
│    │       │            │                                           │
│    │       │            └── RD / RT / VPN-Label                    │
│    │       └── LIB / LFIB                                          │
│    └── LSDB / SPF / ECMP                                           │
│                                                                     │
│  VRF (CUST_A / CUST_B) ◄── import/export RT ──► MP-BGP             │
╰─────────────────────────────────────────────────────────────────────╯

╭─────────────────────────────────────────────────────────────────────╮
│                     PLAN DE DONNÉES                                 │
│                                                                     │
│  CE ──IP──► PE ──Push(label transport)──► P ──Swap──► PE ──Pop──► CE│
│              │                              │                       │
│              └── VRF lookup                  └── label only (no IP)  │
╰─────────────────────────────────────────────────────────────────────╯
```

─── [ 02 / FLUX DE DÉPLOIEMENT ] ────────────────────────────────────────────────

```text
  [01 OSPF] ──► [02 LDP] ──► [03 MP-BGP] ──► [04 VRF] ──► [05 PE-CE]
       │             │              │              │              │
       ▼             ▼              ▼              ▼              ▼
  show ip route  show mpls    show ip bgp    show ip route   ping CE
     ospf         ldp neighbor  vpnv4 summary   vrf           E2E
```

Chaque brique est vérifiée par commande `show` avant passage à la suivante.

─── [ 03 / MATRICE DES DÉPENDANCES ] ─────────────────────────────────────────────

| Protocole | Prérequis | Bloque si absent |
|:----------|:----------|:-----------------|
| `OSPF` | Interfaces IP up | LDP, MP-BGP |
| `LDP` | OSPF convergé | Commutation MPLS cœur |
| `MP-BGP VPNv4` | Loopbacks joignables (OSPF) | Échange routes clients |
| `VRF` | — | Isolation (configuré avec PE-CE) |
| `PE-CE static` | VRF créées | Routes clients dans MP-BGP |

─── [ 04 / ÉTAT OPÉRATIONNEL ] ─────────────────────────────────────────────────

| Composant | PE1 | PE2 | P1 | P2 |
|:----------|:----|:----|:---|:---|
| OSPF neighbors | `🟢 FULL` | `🟢 FULL` | `🟢 FULL` | `🟢 FULL` |
| LDP sessions | `🟢 2 peers` | `🟢 2 peers` | `🟢 2 peers` | `🟢 2 peers` |
| BGP VPNv4 | `🟢 Established` | `🟢 Established` | N/A | N/A |
| VRF CUST_A | `🟢 Active` | `🟢 Active` | N/A | N/A |
| VRF CUST_B | `🟢 Active` | `🟢 Active` | N/A | N/A |

─── [ 05 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`01-lab-topology.md`](01-lab-topology.md) | [`../protocols/01-ospf.md`](../protocols/01-ospf.md) |
