```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  P-02 · LDP — LABEL DISTRIBUTION PROTOCOL                                    │
│  LIB · LFIB · Push/Swap/Pop · PHP · RFC 5036                                 │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `RFC: 5036` · `MODE: LDP follows IGP`

─── [ 01 / RÔLE DANS L'ARCHITECTURE ] ──────────────────────────────────────────

OSPF assure la joignabilité IP. MPLS commute des **labels**, pas des paquets IP. LDP attribue ces labels et construit les LSPs.

| Fonction LDP | Description |
|:-------------|:------------|
| Découverte voisins | Automatique sur interfaces `mpls ip` |
| Attribution labels | Label local par préfixe OSPF |
| Construction LIB/LFIB | Base labels → table de commutation |

─── [ 02 / OPÉRATIONS SUR LES LABELS ] ─────────────────────────────────────────

```text
  CE ──► PE1 ══ Push ══► P1 ══ Swap ══► P2 ══ Pop(imp-null) ══► PE2 ──► CE
         ingress              LSR              PHP               egress
```

| Opération | Routeur | Action |
|:----------|:--------|:-------|
| `Push` | PE ingress | Ajoute label MPLS au paquet |
| `Swap` | P1, P2 | Remplace label entrant → sortant |
| `Pop` | PHP (P avant PE2) | Retire label (imp-null) |

Les routeurs P ne consultent **jamais** la table de routage IP — commutation label uniquement.

─── [ 03 / LIB — LABEL INFORMATION BASE ] ──────────────────────────────────────

```
PE1# show mpls ldp bindings
  lib entry: 1.1.1.1/32, rev 8
        local binding:  label: imp-null
        remote binding: lsr: 4.4.4.4:0, label: 21
        remote binding: lsr: 2.2.2.2:0, label: 22
  lib entry: 3.3.3.3/32, rev 12
        local binding:  label: 19
        remote binding: lsr: 4.4.4.4:0, label: 18
        remote binding: lsr: 2.2.2.2:0, label: 19
  lib entry: 4.4.4.4/32, rev 10
        local binding:  label: 18
        remote binding: lsr: 4.4.4.4:0, label: imp-null
        remote binding: lsr: 2.2.2.2:0, label: 18
```

| Terme | Signification |
|:------|:--------------|
| `local binding` | Label que PE1 attribue et annonce |
| `remote binding` | Label du voisin pour ce préfixe |
| `imp-null` | « Retire le label avant de m'envoyer le paquet » (PHP) |

─── [ 04 / LFIB — LABEL FORWARDING INFORMATION BASE ] ────────────────────────────

```
PE1# show mpls forwarding-table
Local   Outgoing   Prefix           Outgoing   Next Hop
Label   Label                         interface
18      Pop Label  4.4.4.4/32       Et0/3      10.0.14.2
19      19         3.3.3.3/32       Et0/0      10.0.12.2
        18         3.3.3.3/32       Et0/3      10.0.14.2
20      Pop Label  10.0.13.0/30     Et0/0      10.0.12.2
21      Pop Label  10.0.15.0/30     Et0/3      10.0.14.2
22      No Label   2.2.2.2/32       Et0/0      10.0.12.2
```

| Observation | Détail |
|:------------|:-------|
| Label 19 → `3.3.3.3/32` | 2 next-hops : `🟢 ECMP diamond` |
| Pop Label → `4.4.4.4/32` | PHP via P2 (imp-null annoncé) |
| No Label → `2.2.2.2/32` | P1 directement connecté |

─── [ 05 / CONFIGURATION TYPE ] ──────────────────────────────────────────────────

```
mpls label protocol ldp
mpls ldp router-id Loopback0 force
```

Le mot-clé `mpls ip` sur les interfaces cœur active LDP interface par interface.

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ INTERDIT : `mpls ip` sur interfaces CE-facing (VRF). Le trafic      │
> │ client entre en IP pur dans la VRF, label de transport imposé par   │
> │ MP-BGP uniquement à l'entrée du cœur.                               │
> └────────────────────────────────────────────────────────────────────┘

─── [ 06 / COMPARATIF DISTRIBUTION LABELS ] ─────────────────────────────────────

| Protocole | Verdict | Raison |
|:----------|:--------|:-------|
| `RSVP-TE` | `🔴 REJETÉ` | Trop complexe pour objectif L3VPN lab |
| `Segment Routing` | `🟡 ÉCARTÉ` | IOS-XE/XR requis — IOL L3 non compatible |
| `LDP` | `🟢 RETENU` | Suit OSPF auto, RFC 5036, adéquation lab |

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`01-ospf.md`](01-ospf.md) | [`03-mp-bgp-vpnv4.md`](03-mp-bgp-vpnv4.md) |
