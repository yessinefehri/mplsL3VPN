```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  CONFIGS — FICHIERS IOS ROUTEURS                                             │
│  Configurations complètes · Import EVE-NG                                    │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 READY` · `FORMAT: Cisco IOS` · `LAB: EVE-NG IOL L3`

─── [ 01 / INVENTAIRE ] ────────────────────────────────────────────────────────

| Fichier | Routeur | Loopback | Rôle |
|:--------|:--------|:---------|:-----|
| [`PE1.cfg`](PE1.cfg) | PE1 | `1.1.1.1` | Provider Edge + VRF |
| [`PE2.cfg`](PE2.cfg) | PE2 | `3.3.3.3` | Provider Edge + VRF |
| [`P1.cfg`](P1.cfg) | P1 | `2.2.2.2` | Provider Core |
| [`P2.cfg`](P2.cfg) | P2 | `4.4.4.4` | Provider Core |

─── [ 02 / ORDRE D'IMPORT EVE-NG ] ─────────────────────────────────────────────

```text
  1. P1.cfg + P2.cfg     (cœur — OSPF + LDP)
  2. PE1.cfg + PE2.cfg   (bordure — stack complète)
  3. Config CE manuelle  (voir docs/implementation/05-pe-ce-routing.md)
```

─── [ 03 / CONTENU PAR FICHIER ] ───────────────────────────────────────────────

| Fichier | OSPF | LDP | MP-BGP | VRF | Static |
|:--------|:-----|:----|:-------|:----|:-------|
| `P1.cfg` | `🟢` | `🟢` | — | — | — |
| `P2.cfg` | `🟢` | `🟢` | — | — | — |
| `PE1.cfg` | `🟢` | `🟢` | `🟢` | `🟢` | `🟢` |
| `PE2.cfg` | `🟢` | `🟢` | `🟢` | `🟢` | `🟢` |

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ Documentation détaillée : [`../docs/implementation/`](../docs/implementation/) │
> └────────────────────────────────────────────────────────────────────┘
