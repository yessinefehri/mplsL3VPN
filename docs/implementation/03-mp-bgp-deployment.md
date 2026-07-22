```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  I-03 · DÉPLOIEMENT MP-BGP VPNv4                                             │
│  Session iBGP PE1↔PE2 · Étape 03/05                                          │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 DEPLOYED` · `STEP: 03/5` · `VERIFY: show ip bgp vpnv4 all summary`

─── [ 01 / OBJECTIF ] ──────────────────────────────────────────────────────────

Établir la session iBGP VPNv4 entre PE1 (`1.1.1.1`) et PE2 (`3.3.3.3`) pour le transport futur des routes clients.

─── [ 02 / CONFIG PE1 ] ────────────────────────────────────────────────────────

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

─── [ 03 / CONFIG PE2 ] ────────────────────────────────────────────────────────

```
router bgp 65000
 bgp router-id 3.3.3.3
 neighbor 1.1.1.1 remote-as 65000
 neighbor 1.1.1.1 update-source Loopback0
 !
 address-family vpnv4
  neighbor 1.1.1.1 activate
  neighbor 1.1.1.1 send-community both
 exit-address-family
```

─── [ 04 / VÉRIFICATION ] ──────────────────────────────────────────────────────

| Routeur | Voisin | AS | État/PfxRcd |
|:--------|:-------|:---|:------------|
| PE1 (`1.1.1.1`) | `3.3.3.3` | `65000` | `1` |
| PE2 (`3.3.3.3`) | `1.1.1.1` | `65000` | `2` |

Session iBGP VPNv4 active des deux côtés — `🟢 PASS`.

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Les `address-family ipv4 vrf` (redistribute static) sont configurés│
> │ avec le routage PE-CE (étape 05) — dépendent du choix statique.     │
> └────────────────────────────────────────────────────────────────────┘

─── [ 05 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`02-ldp-deployment.md`](02-ldp-deployment.md) | [`04-vrf-deployment.md`](04-vrf-deployment.md) |
