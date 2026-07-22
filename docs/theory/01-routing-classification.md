```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  T-01 · CLASSIFICATION DES PROTOCOLES DE ROUTAGE                             │
│  IGP vs EGP · Systèmes Autonomes                                             │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `LAYER: CONTROL-PLANE` · `ASN: 65000`

─── [ 01 / SYSTÈME AUTONOME ] ──────────────────────────────────────────────────

Un **système autonome** (Autonomous System, AS) est un ensemble de réseaux IP placés sous une administration technique unique, partageant une politique de routage commune, et identifié par un numéro d'AS (ASN).

| Paramètre | Valeur lab |
|:----------|:-----------|
| ASN opérateur | `65000` |
| Périmètre | Cœur ISP + PE |
| Politique | iBGP interne + MP-BGP VPNv4 |

─── [ 02 / IGP — INTERIOR GATEWAY PROTOCOL ] ───────────────────────────────────

Un IGP opère **à l'intérieur** d'un même système autonome. Son objectif : assurer la joignabilité interne avec convergence rapide et configuration relativement simple.

| Protocole | Type | Usage lab |
|:----------|:-----|:----------|
| `RIP` | Vecteur de distance | Non retenu |
| `EIGRP` | Hybride (Cisco) | Non retenu — pas de MPLS/LDP |
| `OSPF` | État de lien | `🟢 RETENU` — IGP cœur |
| `IS-IS` | État de lien | Écarté — complexité config |

```text
╭─────────────────────────────────────────────────────────────╮
│                    AS 65000 — PÉRIMÈTRE IGP                 │
│                                                             │
│   ╭───────╮    OSPF Area 0    ╭───────╮                    │
│   │  PE1  │◄─────────────────►│  PE2  │                    │
│   │1.1.1.1│                   │3.3.3.3│                    │
│   ╰───┬───╯                   ╰───┬───╯                    │
│       │         ╭───────╮         │                        │
│       └────────►│ P1 P2 │◄────────┘                        │
│                 │ Cœur  │                                   │
│                 ╰───────╯                                   │
╰─────────────────────────────────────────────────────────────╯
```

Métrique IGP : basée sur des critères techniques (coût, bande passante, nombre de sauts).

─── [ 03 / EGP — EXTERIOR GATEWAY PROTOCOL ] ───────────────────────────────────

Un EGP permet la communication entre réseaux indépendants gérés par des organisations différentes. Il ne choisit pas le chemin le plus rapide techniquement, mais celui dicté par des **politiques commerciales**.

| Protocole | Statut | Rôle |
|:----------|:-------|:-----|
| EGP (historique) | `🔴 OBSOLÈTE` | Remplacé par BGP |
| `BGP-4` | `🟢 STANDARD` | Seul EGP sur Internet |

─── [ 04 / iBGP — USAGE INTERNE AU LAB ] ──────────────────────────────────────

BGP est nativement un EGP, mais peut fonctionner en mode **interne** (iBGP) entre routeurs d'un même AS.

```text
╭──────────╮                              ╭──────────╮
│   PE1    │  iBGP VPNv4 (AFI 1/SAFI 128) │   PE2    │
│ 1.1.1.1  │◄────────────────────────────►│ 3.3.3.3  │
│  AS 65000 │   update-source Loopback0    │  AS 65000│
╰──────────╯                              ╰──────────╯
```

Dans ce lab, PE1 et PE2 établissent une session iBGP non pas pour interconnecter deux AS, mais pour exploiter **MP-BGP** : transport de routes VPNv4, Route Distinguisher, Route Target.

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ La frontière IGP/EGP est une question de portée d'usage, pas une     │
> │ propriété figée du protocole. iBGP interne illustre ce principe.     │
> └────────────────────────────────────────────────────────────────────┘

─── [ 05 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`../INDEX.md`](../INDEX.md) | [`02-algorithms-dv-ls.md`](02-algorithms-dv-ls.md) |
