```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  I-04 · DÉPLOIEMENT VRF                                                      │
│  CUST_A / CUST_B · Interfaces CE · Étape 04/05                               │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 DEPLOYED` · `STEP: 04/05` · `VERIFY: show ip route vrf`

─── [ 01 / OBJECTIF ] ──────────────────────────────────────────────────────────

Créer les VRF `CUST_A` et `CUST_B` sur PE1 et PE2, affecter les interfaces CE-facing.

─── [ 02 / CONFIG VRF — PE1 & PE2 (identique) ] ─────────────────────────────────

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

─── [ 03 / AFFECTATION INTERFACES ] ────────────────────────────────────────────

**PE1**

```
interface Ethernet0/2
 ip vrf forwarding CUST_A
 ip address 192.168.10.2 255.255.255.252
!
interface Ethernet0/1
 ip vrf forwarding CUST_B
 ip address 192.168.11.2 255.255.255.252
```

**PE2**

```
interface Ethernet0/2
 ip vrf forwarding CUST_A
 ip address 192.168.20.2 255.255.255.252
!
interface Ethernet0/1
 ip vrf forwarding CUST_B
 ip address 192.168.21.2 255.255.255.252
```

─── [ 04 / MATRICE D'AFFECTATION ] ─────────────────────────────────────────────

| PE | Interface | VRF | IP PE | IP CE |
|:---|:----------|:----|:------|:--------|
| PE1 | `Et0/2` | `CUST_A` | `192.168.10.2` | `192.168.10.1` (CE-A1) |
| PE1 | `Et0/1` | `CUST_B` | `192.168.11.2` | `192.168.11.1` (CE-B1) |
| PE2 | `Et0/2` | `CUST_A` | `192.168.20.2` | `192.168.20.1` (CE-A2) |
| PE2 | `Et0/1` | `CUST_B` | `192.168.21.2` | `192.168.21.1` (CE-B2) |

─── [ 05 / VÉRIFICATION — PE1 VRF CUST_A ] ─────────────────────────────────────

| Préfixe | Type | Détail |
|:--------|:-----|:-------|
| `192.168.10.0/30` | `C` | CE-A1 local |
| `192.168.20.0/30` | `B` | CE-A2 via MP-BGP |

Isolation CUST_A / CUST_B confirmée — aucune route croisée — `🟢 PASS`.

─── [ 06 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`03-mp-bgp-deployment.md`](03-mp-bgp-deployment.md) | [`05-pe-ce-routing.md`](05-pe-ce-routing.md) |
