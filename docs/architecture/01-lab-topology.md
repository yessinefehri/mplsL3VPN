```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  A-01 · TOPOLOGIE LAB MPLS L3VPN                                             │
│  Architecture Diamond · Adressage · Matrice des liens                        │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `TOPO: DIAMOND` · `ASN: 65000` · `LAB: EVE-NG`

─── [ 01 / VUE D'ENSEMBLE ] ────────────────────────────────────────────────────

Le cœur adopte une structure **diamond** : deux routeurs Provider (`P1`, `P2`) et deux Provider Edge (`PE1`, `PE2`), connectés à quatre Customer Edge (`CE-A1`, `CE-A2`, `CE-B1`, `CE-B2`).

```text
                    ╭───────────╮         ╭───────────╮
                    │  CE-A1    │         │  CE-B1    │
                    │192.168.10.1│         │192.168.11.1│
                    ╰─────┬─────╯         ╰─────┬─────╯
                          │                     │
                    ╭─────┴─────────────────────┴─────╮
                    │            PE1 · 1.1.1.1         │
                    │     Et0/2 (CUST_A)  Et0/1 (CUST_B)│
                    ╰─────┬─────────────────────┬─────╯
                          │                     │
              Et0/0       │                     │       Et0/3
           10.0.12.0/30   │                     │   10.0.14.0/30
                          │                     │
                    ╭─────┴─────╮         ╭─────┴─────╮
                    │ P1·2.2.2.2│         │ P2·4.4.4.4│
                    │           │         │           │
                    │ Et0/1     │         │ Et0/0     │
                    ╰─────┬─────╯         ╰─────┬─────╯
                          │                     │
              10.0.13.0/30│                     │10.0.15.0/30
                          │                     │
                    ╭─────┴─────────────────────┴─────╮
                    │            PE2 · 3.3.3.3         │
                    │     Et0/2 (CUST_A)  Et0/1 (CUST_B)│
                    ╰─────┬─────────────────────┬─────╯
                          │                     │
                    ╭─────┴─────╯         ╰─────┬─────╯
                    │  CE-A2    │         │  CE-B2    │
                    │192.168.20.1│         │192.168.21.1│
                    ╰───────────╯         ╰───────────╯
```

─── [ 02 / INVENTAIRE ÉQUIPEMENTS ] ────────────────────────────────────────────

| Routeur | Rôle | Loopback | État |
|:--------|:-----|:---------|:-----|
| `PE1` | Provider Edge | `1.1.1.1/32` | `🟢 UP` |
| `PE2` | Provider Edge | `3.3.3.3/32` | `🟢 UP` |
| `P1` | Provider Core | `2.2.2.2/32` | `🟢 UP` |
| `P2` | Provider Core | `4.4.4.4/32` | `🟢 UP` |
| `CE-A1` | Customer Edge (CUST_A) | — | `🟢 UP` |
| `CE-A2` | Customer Edge (CUST_A) | — | `🟢 UP` |
| `CE-B1` | Customer Edge (CUST_B) | — | `🟢 UP` |
| `CE-B2` | Customer Edge (CUST_B) | — | `🟢 UP` |

─── [ 03 / MATRICE DES LIENS CŒUR ] ─────────────────────────────────────────────

| Lien | Interface A | Interface B | Réseau | MPLS |
|:-----|:------------|:------------|:-------|:-----|
| PE1 ↔ P1 | `PE1 Et0/0` | `P1 Et0/0` | `10.0.12.0/30` | `🟢` |
| P1 ↔ PE2 | `P1 Et0/1` | `PE2 Et0/0` | `10.0.13.0/30` | `🟢` |
| PE1 ↔ P2 | `PE1 Et0/3` | `P2 Et0/0` | `10.0.14.0/30` | `🟢` |
| P2 ↔ PE2 | `P2 Et0/1` | `PE2 Et0/3` | `10.0.15.0/30` | `🟢` |

─── [ 04 / MATRICE DES LIENS CE ] ───────────────────────────────────────────────

| Client | CE | PE | Interface PE | Réseau | VRF |
|:-------|:---|:---|:-------------|:-------|:----|
| CUST_A | CE-A1 | PE1 | `Et0/2` | `192.168.10.0/30` | `CUST_A` |
| CUST_A | CE-A2 | PE2 | `Et0/2` | `192.168.20.0/30` | `CUST_A` |
| CUST_B | CE-B1 | PE1 | `Et0/1` | `192.168.11.0/30` | `CUST_B` |
| CUST_B | CE-B2 | PE2 | `Et0/1` | `192.168.21.0/30` | `CUST_B` |

─── [ 05 / VRF & IDENTIFIANTS VPN ] ─────────────────────────────────────────────

| VRF | RD | RT Export | RT Import | CE pairs |
|:----|:---|:----------|:----------|:---------|
| `CUST_A` | `65000:10` | `65000:10` | `65000:10` | CE-A1 ↔ CE-A2 |
| `CUST_B` | `65000:20` | `65000:20` | `65000:20` | CE-B1 ↔ CE-B2 |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Les interfaces CE-facing ne portent JAMAIS `mpls ip`. Le trafic      │
> │ client entre en IP pur dans la VRF, puis reçoit un label de transport│
> │ via MP-BGP avant d'entrer dans le cœur MPLS.                         │
> └────────────────────────────────────────────────────────────────────┘

─── [ 06 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`../theory/03-mpls-l3vpn-model.md`](../theory/03-mpls-l3vpn-model.md) | [`02-protocol-stack.md`](02-protocol-stack.md) |
