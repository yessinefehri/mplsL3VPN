```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  P-03 · MP-BGP VPNv4 — TRANSPORT DES ROUTES CLIENTS                          │
│  RD · RT · VPN-Label · iBGP PE-PE · RFC 4760 / 4364                          │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `AFI/SAFI: 1/128` · `SESSION: iBGP PE1↔PE2`

─── [ 01 / RÔLE DANS L'ARCHITECTURE ] ──────────────────────────────────────────

OSPF + LDP suffisent pour le trafic interne ISP. Ils ne transportent **aucune route client**. MP-BGP VPNv4 (RFC 4364) transporte des NLRIs VPNv4 : RD (8 octets) + préfixe IPv4.

```text
  VRF CUST_A (PE1)                    VRF CUST_A (PE2)
  192.168.10.0/30                     192.168.20.0/30
       │                                    ▲
       │  RD 65000:10 + label VPN           │
       └────────── MP-BGP VPNv4 ──────────────┘
                    via cœur MPLS
              (transport label LDP)
```

─── [ 02 / ROUTE DISTINGUISHER (RD) ] ──────────────────────────────────────────

Le RD (8 octets) préfixe chaque route IPv4 pour former un préfixe VPNv4 unique de 12 octets.

| VRF | RD | Préfixe client | Préfixe VPNv4 |
|:----|:---|:---------------|:--------------|
| `CUST_A` | `65000:10` | `192.168.10.0/30` | `65000:10:192.168.10.0/30` |
| `CUST_B` | `65000:20` | `192.168.11.0/30` | `65000:20:192.168.11.0/30` |

Le RD assure l'unicité — deux clients peuvent utiliser `10.0.0.0/8` sans conflit.

─── [ 03 / ROUTE TARGET (RT) ] ─────────────────────────────────────────────────

Le RT (BGP Extended Community, RFC 4360) contrôle l'import/export entre VRFs.

| VRF | RT Export | RT Import | Politique |
|:----|:----------|:----------|:----------|
| `CUST_A` | `65000:10` | `65000:10` | CE-A1 ↔ CE-A2 |
| `CUST_B` | `65000:20` | `65000:20` | CE-B1 ↔ CE-B2 |

```text
  CUST_A ── RT 65000:10 ──► MP-BGP ◄── RT 65000:10 ── CUST_A
                                    ✗
  CUST_B ── RT 65000:20 ──► MP-BGP ◄── RT 65000:20 ── CUST_B
         (aucun échange inter-client)
```

─── [ 04 / CONFIGURATION TYPE — PE1 ] ────────────────────────────────────────────

```
router bgp 65000
 bgp router-id 1.1.1.1
 neighbor 3.3.3.3 remote-as 65000
 neighbor 3.3.3.3 update-source Loopback0
 !
 address-family vpnv4
  neighbor 3.3.3.3 activate
  neighbor 3.3.3.3 send-community both
 exit-address-family
```

| Paramètre | Valeur | Rôle |
|:----------|:-------|:-----|
| `remote-as 65000` | iBGP interne | Même AS opérateur |
| `update-source Loopback0` | `1.1.1.1` | Session stable |
| `send-community both` | Standard + Extended | Transport RT |

─── [ 05 / COMPARATIF PROTOCOLES VPN ] ─────────────────────────────────────────

| Alternative | Verdict | Limitation |
|:------------|:--------|:-----------|
| `RIP` | `🔴 REJETÉ` | Pas de VRF/RD/RT, métrique 15 sauts |
| OSPF inter-PE | `🔴 REJETÉ` | Pas scalable, conflits adressage (RFC 4364) |
| `EIGRP` | `🔴 REJETÉ` | Propriétaire, pas de transport VPN label |
| SR + BGP-LU | `🟡 ÉCARTÉ` | IOL L3 sans SR-MPLS |
| `MP-BGP VPNv4` | `🟢 RETENU` | Standard ISP, isolation native, scalable |

─── [ 06 / EXIGENCES L3VPN COUVERTES ] ─────────────────────────────────────────

| Exigence | Mécanisme MP-BGP |
|:---------|:-----------------|
| Transport routes client PE↔PE | NLRIs VPNv4 (AFI 1 / SAFI 128) |
| Isolation multi-client | Route Targets (import/export) |
| Scalabilité ISP | 1 session iBGP = tous les clients |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ En production : Route Reflector centralise les peerings PE.          │
> │ Nouveau client = nouvelle VRF + RT, pas nouvelle session BGP.        │
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`02-ldp.md`](02-ldp.md) | [`04-vrf.md`](04-vrf.md) |
