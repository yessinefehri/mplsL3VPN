```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  I-02 · DÉPLOIEMENT LDP                                                      │
│  Activation globale · Vérification voisins · Étape 02/05                     │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 DEPLOYED` · `STEP: 02/05` · `VERIFY: show mpls ldp neighbor`

─── [ 01 / OBJECTIF ] ──────────────────────────────────────────────────────────

Activer LDP globalement avec router-id forcé sur loopback. Le mot-clé `mpls ip` (déjà sur interfaces cœur, section OSPF) active LDP par interface.

─── [ 02 / CONFIGURATION GLOBALE ] ─────────────────────────────────────────────

Appliquer sur `PE1`, `PE2`, `P1`, `P2` :

```
mpls label protocol ldp
mpls ldp router-id Loopback0 force
```

─── [ 03 / RÈGLE CRITIQUE — INTERFACES CE ] ─────────────────────────────────────

| Interface | `mpls ip` | Raison |
|:----------|:----------|:-------|
| Liens cœur PE↔P | `🟢 OUI` | Commutation MPLS |
| Liens CE-facing (VRF) | `🔴 NON` | Trafic IP pur dans VRF |

─── [ 04 / VÉRIFICATION — PE1 ] ────────────────────────────────────────────────

```
PE1# show mpls ldp neighbor
Peer LDP Ident    Interface    État    Uptime
4.4.4.4:0 (P2)    Ethernet0/3  Oper    00:00:02
2.2.2.2:0 (P1)    Ethernet0/0  Oper    00:00:01
```

─── [ 05 / VÉRIFICATION — PE2 ] ────────────────────────────────────────────────

```
PE2# show mpls ldp neighbor
Peer LDP Ident    Interface    État    Uptime
4.4.4.4:0 (P2)    Ethernet0/3  Oper    00:02:26
2.2.2.2:0 (P1)    Ethernet0/0  Oper    00:02:25
```

| Routeur | Sessions LDP | État |
|:--------|:-------------|:-----|
| `PE1` | 2 (P1, P2) | `🟢 Oper` |
| `PE2` | 2 (P1, P2) | `🟢 Oper` |

─── [ 06 / COMMANDES COMPLÉMENTAIRES ] ─────────────────────────────────────────

| Commande | Usage |
|:---------|:------|
| `show mpls ldp bindings` | LIB — labels locaux/distants |
| `show mpls forwarding-table` | LFIB — commutation opérationnelle |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Deux sessions LDP opérationnelles confirment la connectivité diamond.│
> │ ECMP MPLS vers PE2 visible dans la LFIB (labels 18 et 19).          │
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`01-ospf-deployment.md`](01-ospf-deployment.md) | [`03-mp-bgp-deployment.md`](03-mp-bgp-deployment.md) |
