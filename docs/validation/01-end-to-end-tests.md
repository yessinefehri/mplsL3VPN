```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  V-01 · TESTS DE CONNECTIVITÉ BOUT EN BOUT                                   │
│  Validation architecture complète · Ping intra/inter-client                  │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 ALL PASS` · `SCOPE: E2E` · `STACK: OSPF+LDP+MP-BGP+VRF+PE-CE`

─── [ 01 / OBJECTIF ] ──────────────────────────────────────────────────────────

Valider l'architecture complète (pas un protocole isolé) via des pings réels entre équipements clients.

─── [ 02 / MATRICE DES TESTS ] ─────────────────────────────────────────────────

| ID | Source | Destination | Attendu | Résultat |
|:---|:-------|:------------|:--------|:---------|
| T-01 | CE-A1 | CE-A2 (`192.168.20.1`) | Succès intra-client | `🟢 100%` |
| T-02 | CE-A1 | CE-B1 (`192.168.11.1`) | Échec inter-client | `🟢 0%` |
| T-03 | CE-B1 | CE-B2 (`192.168.21.2`) | Succès intra-client | `🟢 100%` |

─── [ 03 / T-01 — INTRA-CLIENT CUST_A ] ────────────────────────────────────────

```
CE_A1> ping 192.168.20.1
Sending 5, 100-byte ICMP Echos to 192.168.20.1, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

```text
  CE-A1 ──► PE1 ═══ MPLS cœur ═══► PE2 ──► CE-A2
  192.168.10.1    VRF CUST_A         VRF CUST_A   192.168.20.1
                  RD 65000:10
                  RT 65000:10
```

CE-A1 atteint CE-A2 à travers tout le cœur MPLS — `🟢 PASS`.

─── [ 04 / T-02 — ISOLATION INTER-CLIENT ] ─────────────────────────────────────

```
CE_A1> ping 192.168.11.1
Sending 5, 100-byte ICMP Echos to 192.168.11.1, timeout is 2 seconds:
.U.U.
Success rate is 0 percent (0/5)
```

| Observation | Interprétation |
|:------------|:---------------|
| `0% success` | Aucune route vers CUST_B dans VRF CUST_A |
| `U` (Unreachable) | PE1 renvoie Destination Unreachable |
| Plan de données | Isolation confirmée bout en bout |

─── [ 05 / T-03 — INTRA-CLIENT CUST_B ] ────────────────────────────────────────

```
CE_B1> ping 192.168.21.2
Sending 5, 100-byte ICMP Echos to 192.168.21.2, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 1/1/2 ms
```

CE-B1 atteint CE-B2 à travers tout le cœur MPLS — `🟢 PASS`.

─── [ 06 / SYNTHÈSE VALIDATION ] ────────────────────────────────────────────────

| Couche | Validé par |
|:-------|:-----------|
| OSPF | Loopbacks joignables PE↔PE |
| LDP | Labels commutés dans le cœur |
| MP-BGP VPNv4 | Routes clients échangées |
| VRF + RT | Isolation plan de contrôle |
| PE-CE | Default route CE fonctionnelle |
| E2E | T-01, T-02, T-03 |

```text
  ╭─────────────────────────────────────────────────────────╮
  │  RÉSULTAT GLOBAL                                        │
  │                                                         │
  │  Connectivité intra-client (CUST_A, CUST_B)  🟢 PASS   │
  │  Isolation inter-client (CUST_A ↔ CUST_B)    🟢 PASS   │
  │  Architecture MPLS L3VPN complète            🟢 VALID  │
  ╰─────────────────────────────────────────────────────────╯
```

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ T-02 (échec attendu) est aussi important que T-01/T-03 : il prouve │
> │ que l'isolation RT fonctionne au plan de données, pas seulement    │
> │ dans les tables de routage.                                         │
> └────────────────────────────────────────────────────────────────────┘

─── [ 07 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Retour |
|:----------|:-------|
| [`../implementation/05-pe-ce-routing.md`](../implementation/05-pe-ce-routing.md) | [`../../README.md`](../../README.md) |
