```text
┌──────────────────────────────────────────────────────────────────────────────┐
│  T-02 · ALGORITHMES DE ROUTAGE DYNAMIQUE                                     │
│  Vecteur de Distance vs État de Lien                                         │
└──────────────────────────────────────────────────────────────────────────────┘
```

`STATUS: 🟢 OPERATIONAL` · `LAYER: CONTROL-PLANE` · `ALGO: Bellman-Ford / Dijkstra`

─── [ 01 / VECTEUR DE DISTANCE ] ───────────────────────────────────────────────

Principe du **« routage par la rumeur »** : chaque routeur ne connaît que ce que ses voisins directs annoncent — destination + distance — sans vision topologique complète.

| Paramètre | Détail |
|:----------|:-------|
| Algorithme | Bellman-Ford |
| Convergence | Lente |
| Risque | Count-to-infinity |
| Correctifs | Split horizon, poison reverse, holddown |
| Exemples | `RIP`, `EIGRP` (DUAL) |

```text
  Router A                    Router B
  ┌─────────┐                ┌─────────┐
  │ Dest: X │─── metric ───►│ Dest: X │
  │ via B   │   annoncée    │ via A   │
  └─────────┘                └─────────┘
       ▲                          │
       └──────── boucle ──────────┘   ← count-to-infinity
```

─── [ 02 / ÉTAT DE LIEN ] ──────────────────────────────────────────────────────

Chaque routeur **floode** l'état de ses liens à tous les autres routeurs de la zone via des LSA (OSPF). Chaque routeur construit une **LSDB** identique et complète, puis exécute **Dijkstra (SPF)**.

| Paramètre | Détail |
|:----------|:-------|
| Algorithme | Dijkstra (SPF) |
| Base | LSDB complète |
| Convergence | Rapide |
| Boucles | Absentes structurellement |
| Coût | Mémoire/CPU plus élevés |
| Exemples | `OSPF`, `IS-IS` |

```text
  Tous les routeurs partagent la même LSDB
  ╭───────────────────────────────────────╮
  │  PE1 ──10── P1 ──10── PE2            │
  │   │              │                    │
  │   └──10── P2 ────┘                    │
  │         (topologie diamond)           │
  ╰───────────────────────────────────────╯
           │
           ▼  SPF depuis chaque routeur
  Arbre des chemins les plus courts
```

─── [ 03 / MATRICE COMPARATIVE ] ───────────────────────────────────────────────

| Critère | Vecteur de distance | État de lien |
|:--------|:--------------------|:-------------|
| Vision réseau | Partielle (voisins) | Complète (LSDB) |
| Algorithme | Bellman-Ford | Dijkstra (SPF) |
| Convergence | Lente | Rapide |
| Boucles | Possibles | Non |
| ECMP natif | Limité | `🟢 Oui` (OSPF) |
| Usage MPLS | `🔴 EIGRP incompatible` | `🟢 OSPF retenu` |

─── [ 04 / IMPACT SUR MPLS ] ───────────────────────────────────────────────────

LDP applique le principe **« LDP follows IGP »** : il attribue un label à chaque route exactement telle qu'installée par l'IGP.

```text
  OSPF (LSDB cohérente)
       │
       ▼
  Table de routage IP
       │
       ▼
  LDP (labels par préfixe)
       │
       ▼
  LSP stables + ECMP diamond
```

Un protocole à état de lien comme OSPF permet à LDP de construire des LSPs stables et prévisibles, incluant le support natif de l'**ECMP** (Equal-Cost Multi-Path) observé dans la topologie diamond du lab.

> ┌─ 📡 NOC TELEMETRY NOTICE ──────────────────────────────────────────┐
> │ C'est l'une des raisons théoriques, indépendamment des arguments   │
> │ pratiques, qui motive le choix d'OSPF plutôt qu'EIGRP pour le cœur.│
> └────────────────────────────────────────────────────────────────────┘

─── [ 05 / NAVIGATION ] ────────────────────────────────────────────────────────

| Précédent | Suivant |
|:----------|:--------|
| [`01-routing-classification.md`](01-routing-classification.md) | [`03-mpls-l3vpn-model.md`](03-mpls-l3vpn-model.md) |
