```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  P-01 · OSPF — IGP DU CŒUR ISP                                               │
│  Link-State · LSDB · SPF · ECMP Diamond                                      │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `PROCESS: ospf 1` · `AREA: 0` · `TYPE: Link-State`

─── [ 01 / RÔLE DANS L'ARCHITECTURE ] ──────────────────────────────────────────

Dans un réseau MPLS, chaque routeur du cœur doit connaître la loopback de tous les PE et P. Ces loopbacks servent de points d'ancrage pour LDP (établissement des LSPs).

| Fonction | Détail |
|:---------|:-------|
| Distribution loopbacks | `1.1.1.1`, `2.2.2.2`, `3.3.3.3`, `4.4.4.4` |
| Liens point-à-point | `/30` entre PE/P |
| Base LDP | Principe « LDP follows IGP » |
| ECMP | Deux chemins PE1→PE2 via P1 ou P2 |

─── [ 02 / ÉCHANGE OSPF — MACHINE À ÉTATS ] ────────────────────────────────────

```text
  Router 1                              Router 2
     │                                      │
     │─────── Hello (Multicast) ───────────►│
     │◄────── Hello (Multicast) ────────────│
     │                                      │
     │◄────── DBD (Unicast) ────────────────│   Database Description
     │─────── DBD (Unicast) ──────────────►│
     │                                      │
     │─────── LSR (Unicast) ──────────────►│   LinkState Request
     │◄────── LSU (Unicast) ────────────────│   LinkState Update
     │                                      │
     │◄────── LSR (Unicast) ────────────────│
     │─────── LSU (Unicast) ──────────────►│
     │                                      │
     │─────── LSAck (Multicast) ───────────►│   Acknowledgement
     │◄────── LSAck (Multicast) ────────────│
```

| Paquet | Rôle |
|:-------|:-----|
| `Hello` | Découverte et maintien des voisins |
| `DBD` | Résumé de la LSDB |
| `LSR` | Demande d'informations manquantes |
| `LSU` | Réponse détaillée (LSAs) |
| `LSAck` | Confirmation de réception |

─── [ 03 / CALCUL SPF & ECMP DIAMOND ] ──────────────────────────────────────────

OSPF installe le chemin à coût minimal. Dans la topologie diamond, PE1 dispose de deux chemins vers PE2 :

```text
  PE1 ──► P1 ──► PE2     (coût = coût Et0/0 PE1 + coût Et0/1 P1)
  PE1 ──► P2 ──► PE2     (coût = coût Et0/1 PE1 + coût Et0/1 P2)
```

Si les deux chemins ont le même coût (interfaces Ethernet identiques), OSPF active l'**ECMP**.

─── [ 04 / CONFIGURATION TYPE — P1 ] ───────────────────────────────────────────

```
router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0       ! Loopback P1
 network 10.0.12.0 0.0.0.3 area 0     ! Lien P1 -> PE1
 network 10.0.13.0 0.0.0.3 area 0     ! Lien P1 -> PE2
 passive-interface Loopback0
```

| Choix config | Justification |
|:-------------|:--------------|
| `router-id` manuel | Stabilité — basé sur loopback |
| `Area 0` unique | Topologie simple, pas d'ABR nécessaire |
| `passive-interface Loopback0` | Annonce sans Hello (pas de voisin sur loopback) |

─── [ 05 / TABLE DE ROUTAGE RÉSULTANTE — PE1 ] ─────────────────────────────────

```
PE1# show ip route ospf
O    2.2.2.2/32 [110/11] via 10.0.12.2, Ethernet0/0
O    3.3.3.3/32 [110/21] via 10.0.12.2, Ethernet0/0
                         via 10.0.14.2, Ethernet0/1
O    10.0.13.0/30 [110/20] via 10.0.12.2, Ethernet0/0
O    10.0.15.0/30 [110/20] via 10.0.14.2, Ethernet0/1
```

| Préfixe | Next-hop(s) | Observation |
|:--------|:------------|:--------------|
| `3.3.3.3/32` | 2 next-hops | `🟢 ECMP diamond` |
| `2.2.2.2/32` | via P1 | Loopback P1 |

─── [ 06 / COMPARATIF IGP ] ─────────────────────────────────────────────────────

| Protocole | Verdict | Raison d'écart |
|:----------|:--------|:---------------|
| `EIGRP` | `🔴 REJETÉ` | Ne supporte pas MPLS/LDP |
| `IS-IS` | `🟡 ÉCARTÉ` | Config plus lourde (NET, L1/L2) sans gain |
| `OSPF` | `🟢 RETENU` | Standard ISP, intégration LDP native, simplicité |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Sans OSPF : loopbacks non joignables → LDP sans LSP → MP-BGP sans   │
> │ session VPNv4. OSPF est le socle de toute l'architecture MPLS.      │
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`../architecture/02-protocol-stack.md`](../architecture/02-protocol-stack.md) | [`02-ldp.md`](02-ldp.md) |
