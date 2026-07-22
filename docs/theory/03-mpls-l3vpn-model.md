```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  T-03 · FONDEMENTS MPLS & MODÈLE L3VPN                                       │
│  FEC · LSP · RFC 4364 · Rôles CE/PE/P                                        │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `RFC: 3031 / 4364` · `MODE: L3VPN`

─── [ 01 / LIMITE DU ROUTAGE IP ] ──────────────────────────────────────────────

Dans un réseau IP classique, chaque routeur effectue pour chaque paquet une recherche **longest prefix match** dans sa table de routage. Cette opération, répétée à chaque saut, devient coûteuse sur des cœurs à très haut débit et ne permet ni ingénierie de trafic ni isolation multi-client native.

─── [ 02 / COMMUTATION PAR LABEL ] ───────────────────────────────────────────────

MPLS (Multiprotocol Label Switching, RFC 3031) insère une étiquette de 20 bits entre L2 et L3. Les routeurs du cœur ne consultent plus l'IP de destination — ils commutent le label.

| Opération | Acteur | Action |
|:----------|:-------|:-------|
| `Push` | Ingress LER (PE) | Impose un label au paquet IP |
| `Swap` | LSR (P1, P2) | Remplace label entrant → sortant |
| `Pop` | PHP (avant-dernier saut) | Retire le label avant egress PE |

```text
  CE ──► PE1 ═══ Push ═══► P1 ═══ Swap ═══► P2 ═══ Pop ═══► PE2 ──► CE
              │                    │                    │
              └──── label MPLS ────┴──── label MPLS ────┘
                        (pas de lookup IP dans le cœur)
```

─── [ 03 / FEC — FORWARDING EQUIVALENCE CLASS ] ────────────────────────────────

Une FEC est un ensemble de paquets traités de manière identique : même chemin, même traitement, même label. Le plus souvent, une FEC correspond à un préfixe de destination (`3.3.3.3/32`).

| Élément | Rôle |
|:--------|:-----|
| Ingress LER | Classifie le paquet → impose le label |
| LSR intermédiaires | Commutent le label sans reclassifier |
| Egress LER | Retire le label → lookup VRF/IP |

─── [ 04 / LSP — LABEL SWITCHED PATH ] ─────────────────────────────────────────

Un LSP est le chemin unidirectionnel prédéterminé qu'un paquet labellisé suit du ingress LER à l'egress LER.

```text
  PE1 (ingress) ──LSP──► P1 ──LSP──► P2 ──LSP──► PE2 (egress)
       │                                              │
       └────────── deux LSPs ECMP (via P1 ou P2) ─────┘
                    (topologie diamond)
```

Dans ce lab, les LSPs PE1↔PE2 sont construits par **LDP** en suivant la topologie OSPF (« LDP follows IGP »).

─── [ 05 / MODÈLE VPN L3 — RFC 4364 ] ──────────────────────────────────────────

| Rôle | Routeur | Fonction | Connaissance client |
|:-----|:--------|:---------|:-------------------|
| `CE` | Customer Edge | Frontière réseau client | Son propre réseau |
| `PE` | Provider Edge | VRFs + MP-BGP + connexion CE | Routes de ses clients |
| `P` | Provider | Commutation MPLS pure | `🔴 Aucune route client` |

```text
╭─────────╮     ╭─────────────────────────────────────╮     ╭─────────╮
│  CE-A1  │────►│ PE1          CŒUR MPLS          PE2 │◄────│  CE-A2  │
│ client  │     │  VRF_A  ◄── MP-BGP VPNv4 ──►  VRF_A │     │ client  │
╰─────────╯     │         P1 ◄── LDP ──► P2           │     ╰─────────╯
                ╰─────────────────────────────────────╯
```

Ce modèle permet à un ISP d'héberger plusieurs clients sur une infrastructure partagée avec **isolation totale** — y compris lorsque deux clients utilisent le même espace d'adressage privé.

─── [ 06 / DE LDP AU VPN L3 ] ───────────────────────────────────────────────────

| Couche | Transporte | Limite |
|:-------|:-----------|:-------|
| LDP + LSP | Routes internes ISP (loopbacks PE/P) | Pas de routes clients |
| MP-BGP VPNv4 + VRF | Routes clients isolées | Complète le modèle L3VPN |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Mécanismes d'isolation (VRF, RD, RT) détaillés dans docs/protocols/.│
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`02-algorithms-dv-ls.md`](02-algorithms-dv-ls.md) | [`../architecture/01-lab-topology.md`](../architecture/01-lab-topology.md) |
