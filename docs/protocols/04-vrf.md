```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  P-04 · VRF — VIRTUAL ROUTING AND FORWARDING                                 │
│  Isolation multi-client · CUST_A / CUST_B · RFC 4364                         │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `INSTANCES: 2` · `MODE: FULL ISOLATION`

─── [ 01 / PROBLÈME RÉSOLU ] ─────────────────────────────────────────────────────

Sans VRF, tous les routeurs partagent une table de routage globale. Deux clients utilisant `192.168.0.0/24` entrent en conflit — l'une écrase l'autre.

La VRF crée des **tables de routage virtuelles indépendantes** sur un même routeur physique : table de routage, CEF, et interfaces dédiées par instance.

─── [ 02 / VRFs DU LAB ] ─────────────────────────────────────────────────────────

```text
  PE1                                    PE2
  ╭──────────────────────╮              ╭──────────────────────╮
  │ VRF CUST_A           │              │ VRF CUST_A           │
  │  RD 65000:10         │◄── MP-BGP ──►│  RD 65000:10         │
  │  Et0/2 → CE-A1       │              │  Et0/2 → CE-A2       │
  ╰──────────────────────╯              ╰──────────────────────╯
  ╭──────────────────────╮              ╭──────────────────────╮
  │ VRF CUST_B           │              │ VRF CUST_B           │
  │  RD 65000:20         │◄── MP-BGP ──►│  RD 65000:20         │
  │  Et0/1 → CE-B1       │              │  Et0/1 → CE-B2       │
  ╰──────────────────────╯              ╰──────────────────────╯
```

─── [ 03 / CONFIGURATION TYPE ] ──────────────────────────────────────────────────

```
ip vrf CUST_A
 rd 65000:10
 route-target export 65000:10
 route-target import 65000:10
!
ip vrf CUST_B
 rd 65000:20
 route-target export 65000:20
 route-target import 65000:20
```

Affectation interfaces CE-facing :

| PE | Interface | VRF | IP | CE connecté |
|:---|:----------|:----|:---|:------------|
| PE1 | `Et0/2` | `CUST_A` | `192.168.10.2/30` | CE-A1 |
| PE1 | `Et0/1` | `CUST_B` | `192.168.11.2/30` | CE-B1 |
| PE2 | `Et0/2` | `CUST_A` | `192.168.20.2/30` | CE-A2 |
| PE2 | `Et0/1` | `CUST_B` | `192.168.21.2/30` | CE-B2 |

─── [ 04 / TABLES DE ROUTAGE VRF — PE1 ] ────────────────────────────────────────

**VRF CUST_A**

| Préfixe | Type | Détail |
|:--------|:-----|:-------|
| `192.168.10.0/30` | `C` | Et0/2 — CE-A1 local |
| `192.168.20.0/30` | `B` | via `3.3.3.3` — CE-A2 via MP-BGP |

**VRF CUST_B**

| Préfixe | Type | Détail |
|:--------|:-----|:-------|
| `192.168.11.0/30` | `C` | Et0/1 — CE-B1 local |
| `192.168.21.0/30` | `B` | via `3.3.3.3` — CE-B2 via MP-BGP |

| Code | Signification |
|:-----|:--------------|
| `C` | Directement connecté |
| `B` | Appris par BGP (MP-BGP VPNv4) |

Aucune route CUST_B dans table CUST_A — isolation confirmée au plan de contrôle.

─── [ 05 / COMPARATIF ISOLATION ] ────────────────────────────────────────────────

| Approche | Verdict | Limitation |
|:---------|:--------|:-----------|
| VLANs seuls | `🔴 INSUFFISANT` | Isolation L2, conflits L3 |
| PE dédié/client | `🔴 NON SCALABLE` | Coût matériel, complexité |
| `VRF` | `🟢 RETENU` | Standard RFC 4364, centaines de clients/PE |

─── [ 06 / CHAÎNE D'ISOLATION ] ──────────────────────────────────────────────────

```text
  Interface CE ──► VRF ──► RD (unicité BGP) ──► RT (import/export)
       │              │              │                    │
       └──────────────┴──────────────┴────────────────────┘
                    isolation bout en bout
```

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Sans VRF : PE1/PE2 incapables d'héberger plusieurs clients avec    │
> │ des espaces d'adressage identiques. VRF = fondement du L3VPN.      │
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`03-mp-bgp-vpnv4.md`](03-mp-bgp-vpnv4.md) | [`../implementation/01-ospf-deployment.md`](../implementation/01-ospf-deployment.md) |
