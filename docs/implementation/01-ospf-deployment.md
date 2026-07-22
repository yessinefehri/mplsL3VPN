```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  I-01 · DÉPLOIEMENT OSPF                                                     │
│  Configuration PE/P · Vérification · Étape 01/05                             │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 DEPLOYED` · `STEP: 01/05` · `VERIFY: show ip route ospf`

─── [ 01 / OBJECTIF ] ──────────────────────────────────────────────────────────

Activer OSPF sur les quatre routeurs du cœur (`PE1`, `P1`, `P2`, `PE2`) pour distribuer loopbacks et liens `/30`. Condition préalable à LDP.

─── [ 02 / CONFIG P1 ] ─────────────────────────────────────────────────────────

```
interface Loopback0
 ip address 2.2.2.2 255.255.255.255
!
interface Ethernet0/0
 ip address 10.0.12.2 255.255.255.252
 mpls ip
!
interface Ethernet0/1
 ip address 10.0.13.1 255.255.255.252
 mpls ip
!
router ospf 1
 router-id 2.2.2.2
 network 2.2.2.2 0.0.0.0 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.13.0 0.0.0.3 area 0
 passive-interface Loopback0
```

─── [ 03 / CONFIG P2 ] ─────────────────────────────────────────────────────────

```
interface Loopback0
 ip address 4.4.4.4 255.255.255.255
!
interface Ethernet0/0
 ip address 10.0.14.2 255.255.255.252
 mpls ip
!
interface Ethernet0/1
 ip address 10.0.15.1 255.255.255.252
 mpls ip
!
router ospf 1
 router-id 4.4.4.4
 network 4.4.4.4 0.0.0.0 area 0
 network 10.0.14.0 0.0.0.3 area 0
 network 10.0.15.0 0.0.0.3 area 0
 passive-interface Loopback0
```

─── [ 04 / CONFIG PE1 ] ────────────────────────────────────────────────────────

```
interface Loopback0
 ip address 1.1.1.1 255.255.255.255
!
interface Ethernet0/0
 ip address 10.0.12.1 255.255.255.252
 mpls ip
!
interface Ethernet0/3
 ip address 10.0.14.1 255.255.255.252
 mpls ip
!
router ospf 1
 router-id 1.1.1.1
 network 1.1.1.1 0.0.0.0 area 0
 network 10.0.12.0 0.0.0.3 area 0
 network 10.0.14.0 0.0.0.3 area 0
 passive-interface Loopback0
```

─── [ 05 / CONFIG PE2 ] ────────────────────────────────────────────────────────

```
interface Loopback0
 ip address 3.3.3.3 255.255.255.255
!
interface Ethernet0/0
 ip address 10.0.13.2 255.255.255.252
 mpls ip
!
interface Ethernet0/3
 ip address 10.0.15.2 255.255.255.252
 mpls ip
!
router ospf 1
 router-id 3.3.3.3
 network 3.3.3.3 0.0.0.0 area 0
 network 10.0.13.0 0.0.0.3 area 0
 network 10.0.15.0 0.0.0.3 area 0
 passive-interface Loopback0
```

─── [ 06 / VÉRIFICATION ] ──────────────────────────────────────────────────────

| Commande | Attendu | État |
|:---------|:--------|:-----|
| `show ip ospf neighbor` | FULL sur tous les liens cœur | `🟢 PASS` |
| `show ip route ospf` | Loopbacks PE/P visibles | `🟢 PASS` |
| `ping 3.3.3.3 source 1.1.1.1` | Réponse depuis PE1 | `🟢 PASS` |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ P1/P2 : routeurs P purs — aucune VRF, aucun MP-BGP. Rôle = OSPF+LDP. │
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`../protocols/04-vrf.md`](../protocols/04-vrf.md) | [`02-ldp-deployment.md`](02-ldp-deployment.md) |
