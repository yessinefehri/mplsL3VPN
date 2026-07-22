```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  I-05 · ROUTAGE PE-CE                                                        │
│  Routes statiques · Redistribute · CE default-route · Étape 05/05            │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 DEPLOYED` · `STEP: 05/05` · `VERIFY: show ip route (CE)`

─── [ 01 / OBJECTIF ] ──────────────────────────────────────────────────────────

Injecter les préfixes clients dans MP-BGP via routes statiques explicites redistribuées (`redistribute static`). Approche courante ISP — contrôle précis des préfixes annoncés.

─── [ 02 / CONFIG PE1 ] ────────────────────────────────────────────────────────

```
ip route vrf CUST_A 192.168.10.0 255.255.255.252 Ethernet0/2
ip route vrf CUST_B 192.168.11.0 255.255.255.252 Ethernet0/1
!
router bgp 65000
 address-family ipv4 vrf CUST_A
  redistribute static
 exit-address-family
 !
 address-family ipv4 vrf CUST_B
  redistribute static
 exit-address-family
```

─── [ 03 / CONFIG PE2 ] ────────────────────────────────────────────────────────

```
ip route vrf CUST_A 192.168.20.0 255.255.255.252 Ethernet0/2
ip route vrf CUST_B 192.168.21.0 255.255.255.252 Ethernet0/1
!
router bgp 65000
 address-family ipv4 vrf CUST_A
  redistribute static
 exit-address-family
 !
 address-family ipv4 vrf CUST_B
  redistribute static
 exit-address-family
```

─── [ 04 / CONFIG CE ] ─────────────────────────────────────────────────────────

| CE | Interface IP | Default route |
|:---|:-------------|:--------------|
| CE-A1 | `192.168.10.1/30` | `0.0.0.0/0 → 192.168.10.2` |
| CE-A2 | `192.168.20.1/30` | `0.0.0.0/0 → 192.168.20.2` |
| CE-B1 | `192.168.11.1/30` | `0.0.0.0/0 → 192.168.11.2` |
| CE-B2 | `192.168.21.1/30` | `0.0.0.0/0 → 192.168.21.2` |

Exemple CE-A1 :

```
interface Ethernet0/0
 ip address 192.168.10.1 255.255.255.252
ip route 0.0.0.0 0.0.0.0 192.168.10.2
```

─── [ 05 / VÉRIFICATION CE-A1 ] ──────────────────────────────────────────────────

```
CE-A1# show ip route
Gateway of last resort is 192.168.10.2 to network 0.0.0.0
S*   0.0.0.0/0 [1/0] via 192.168.10.2
C    192.168.10.0/30 is directly connected, Ethernet0/0
```

Chaque CE ne connaît que son PE local + default route — `🟢 PASS`.

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Le PE, via sa VRF, porte la connaissance complète du réseau client. │
> │ Le CE reste minimal — pattern hub-and-spoke opérateur standard.      │
> └────────────────────────────────────────────────────────────────────┘

─── [ 06 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`04-vrf-deployment.md`](04-vrf-deployment.md) | [`../validation/01-end-to-end-tests.md`](../validation/01-end-to-end-tests.md) |
