```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  REPO INDEX — MPLS L3VPN STAGE                                               │
│  Navigation centrale · Toutes sections                                     │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `REVISION: v1.0` · `DOCS: 14 fichiers`

─── [ 00 / THÉORIE ] ───────────────────────────────────────────────────────────

| ID | Fichier | Sujet |
|:---|:--------|:------|
| T-01 | [`theory/01-routing-classification.md`](theory/01-routing-classification.md) | IGP vs EGP · Systèmes autonomes · iBGP interne |
| T-02 | [`theory/02-algorithms-dv-ls.md`](theory/02-algorithms-dv-ls.md) | Vecteur de distance vs état de lien · SPF |
| T-03 | [`theory/03-mpls-l3vpn-model.md`](theory/03-mpls-l3vpn-model.md) | FEC · LSP · Modèle CE/PE/P · RFC 4364 |

─── [ 01 / ARCHITECTURE ] ──────────────────────────────────────────────────────

| ID | Fichier | Sujet |
|:---|:--------|:------|
| A-01 | [`architecture/01-lab-topology.md`](architecture/01-lab-topology.md) | Topologie diamond · Table des liens · Adressage |
| A-02 | [`architecture/02-protocol-stack.md`](architecture/02-protocol-stack.md) | Stack complète · Flux de contrôle / données |

─── [ 02 / PROTOCOLES ] ────────────────────────────────────────────────────────

| ID | Fichier | Sujet | État |
|:---|:--------|:------|:-----|
| P-01 | [`protocols/01-ospf.md`](protocols/01-ospf.md) | IGP cœur · LSDB · ECMP diamond | `🟢 UP` |
| P-02 | [`protocols/02-ldp.md`](protocols/02-ldp.md) | LIB · LFIB · Push/Swap/Pop · PHP | `🟢 UP` |
| P-03 | [`protocols/03-mp-bgp-vpnv4.md`](protocols/03-mp-bgp-vpnv4.md) | RD · RT · VPNv4 · iBGP PE-PE | `🟢 UP` |
| P-04 | [`protocols/04-vrf.md`](protocols/04-vrf.md) | Isolation multi-client · CUST_A/B | `🟢 UP` |

─── [ 03 / IMPLÉMENTATION ] ────────────────────────────────────────────────────

| ID | Fichier | Sujet | Ordre déploiement |
|:---|:--------|:------|:------------------|
| I-01 | [`implementation/01-ospf-deployment.md`](implementation/01-ospf-deployment.md) | Config OSPF PE/P | `01` |
| I-02 | [`implementation/02-ldp-deployment.md`](implementation/02-ldp-deployment.md) | Config LDP + vérification | `02` |
| I-03 | [`implementation/03-mp-bgp-deployment.md`](implementation/03-mp-bgp-deployment.md) | Session VPNv4 PE1↔PE2 | `03` |
| I-04 | [`implementation/04-vrf-deployment.md`](implementation/04-vrf-deployment.md) | VRF + interfaces CE | `04` |
| I-05 | [`implementation/05-pe-ce-routing.md`](implementation/05-pe-ce-routing.md) | Routes statiques + redistribute | `05` |

─── [ 04 / VALIDATION ] ────────────────────────────────────────────────────────

| ID | Fichier | Sujet | Résultat |
|:---|:--------|:------|:---------|
| V-01 | [`validation/01-end-to-end-tests.md`](validation/01-end-to-end-tests.md) | Ping intra/inter-client | `🟢 PASS` |

─── [ 05 / CONFIGS BRUTES ] ────────────────────────────────────────────────────

| Routeur | Fichier | Loopback | Rôle |
|:--------|:--------|:---------|:-----|
| PE1 | [`../configs/PE1.cfg`](../configs/PE1.cfg) | `1.1.1.1/32` | Provider Edge |
| PE2 | [`../configs/PE2.cfg`](../configs/PE2.cfg) | `3.3.3.3/32` | Provider Edge |
| P1 | [`../configs/P1.cfg`](../configs/P1.cfg) | `2.2.2.2/32` | Provider Core |
| P2 | [`../configs/P2.cfg`](../configs/P2.cfg) | `4.4.4.4/32` | Provider Core |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Retour racine : [`../README.md`](../README.md)                       │
> └────────────────────────────────────────────────────────────────────┘
