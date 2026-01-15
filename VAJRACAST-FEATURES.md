# Vajra Cast — Feature Reference

**SRT Gateway & Distribution Platform for Professional Broadcast**

Vajra Cast est une plateforme de distribution vidéo temps réel conçue pour les environnements broadcast professionnels. Elle reçoit des flux SRT/SRTLA/UDP, les route et les distribue vers de multiples destinations simultanément — sans compromis sur la latence ni la stabilité.

---

## Protocoles Supportés

### Inputs

| Protocole | Mode | Description |
|-----------|------|-------------|
| **SRT** | Listener / Caller | Listener (attend les connexions) ou Caller (initie la connexion). Encryption AES-128/256 optionnelle, latency configurable (défaut 200ms). Compatible avec tous les encodeurs SRT. |
| **SRTLA** | Listener | SRT Link Aggregation — bonding multi-connexions pour IFB mobile. Compatible BELABOX et encodeurs terrain. Agrège automatiquement les liens cellulaires. |
| **HTTP/HTTPS** | Pull | Pull depuis sources HTTP/TS. Idéal pour ingester des streams web existants ou des feeds CDN. |
| **UDP** | Unicast | Réception UDP classique depuis encodeurs hardware ou sources réseau local. |

### Outputs

| Protocole | Mode | Description |
|-----------|------|-------------|
| **SRT** | Listener / Caller | Distribution SRT vers N destinations simultanées. Listener pour clients qui se connectent, Caller pour push vers serveurs distants. |
| **SRTLA** | Caller | Bonding output vers récepteurs SRTLA distants. Répartition automatique sur liens multiples. |
| **RTMP** | Push | Push natif vers YouTube Live, Twitch, Facebook Live, ou tout serveur RTMP custom. Gestion des stream keys et URLs personnalisées. |
| **HTTP/TS** | Server | Serveur HTTP/TS intégré. Accessible depuis VLC, ffplay, ou tout player compatible MPEG-TS over HTTP. |
| **UDP** | Unicast | Output UDP vers destinations réseau spécifiques. |
| **HLS** | Server | Serveur HLS intégré avec segmentation configurable. Adaptive streaming pour distribution web large échelle. |

---

## Modes Opératoires

### Passthrough Mode

**Use case** : Distribution pure, maximum d'outputs, latence minimale.

| Métrique | Valeur |
|----------|--------|
| Latence ajoutée | < 50ms |
| CPU par output | 0% (multicast interne) |
| Traitement | Aucun — le flux traverse sans décodage |

Le mode Passthrough utilise un système de multicast interne : le flux entrant est dupliqué au niveau mémoire vers tous les outputs sans aucun traitement CPU. Chaque output additionnel a un coût quasi-nul.

**Idéal pour** : Distribution multi-plateforme, backup feeds, confidence monitoring, réplication vers équipes distantes.

---

### Transcode Mode

**Use case** : Conversion format, remixage audio, adaptation bitrate.

| Métrique | Valeur |
|----------|--------|
| Latence ajoutée | 200-500ms |
| CPU | Variable selon résolution et preset |
| Backend | FFmpeg |

Le mode Transcode décode le flux entrant et le ré-encode selon les paramètres souhaités. Permet notamment :

- **Audio Matrix** : Routing de canaux complexe (7.1 → stereo, extraction de canaux spécifiques, application de gains individuels)
- Conversion de codecs
- Adaptation de résolution/bitrate

**Idéal pour** : Feeds commentateurs multilingues, adaptation pour plateformes aux specs différentes, monitoring audio séparé.

---

## Features

### Hot Management

Gestion des outputs à chaud sans interruption du flux :

- **Add/Remove** : Ajout ou suppression d'outputs pendant la diffusion
- **Enable/Disable** : Toggle instantané sans supprimer la configuration
- **Move** : Déplacement d'un output d'une route à une autre en live

Aucun redémarrage, aucune interruption du flux source ou des autres outputs.

---

### Hot Swap

Échange d'inputs entre routes sans coupure visible :

- Bascule entre sources (principal/backup) sans interruption
- Transition frame-accurate quand les flux sont synchronisés
- Gestion des failover automatisables

---

### Zero-Copy Distribution

Architecture multicast interne pour distribution massive :

- Le flux source n'est lu qu'une fois en mémoire
- Duplication vers N outputs sans copie mémoire additionnelle
- Scaling linéaire : 10 ou 100 outputs, même charge CPU

---

### Metrics Temps Réel

Monitoring complet via WebSocket pour dashboards et alerting :

| Métrique | Description |
|----------|-------------|
| **RTT** | Round-trip time SRT |
| **Bandwidth** | Bande passante utilisée |
| **Packet Loss** | Perte de paquets (%) |
| **Flight Size** | Packets en transit |
| **Buffer** | État des buffers SRT |

Données poussées en temps réel, exploitables pour graphes ou systèmes d'alerte.

---

### Audio Matrix

Routing audio avancé disponible en mode Transcode :

- Remapping de canaux (ex: 7.1 → stereo personnalisé)
- Extraction de canaux individuels (commentaires, ambiance, mix)
- Gains par canal
- Mix custom multi-source

---

### Audit Log

Traçabilité complète de toutes les opérations :

- Rétention : **100 000 events**
- Tracking : création/modification/suppression de routes, inputs, outputs
- Horodatage précis
- Filtrage et recherche

---

### System Monitoring

Surveillance de la plateforme elle-même :

- CPU, RAM, utilisation disque
- Détection de memory leaks
- Historique des restart events
- Alerting sur seuils configurables

---

## API & Intégration

### REST API

API complète pour automatisation et intégration :

```
GET/POST/PUT/DELETE /api/routes
GET/POST/PUT/DELETE /api/routes/:id/inputs
GET/POST/PUT/DELETE /api/routes/:id/outputs
```

- Authentification JWT
- CRUD complet sur routes, inputs, outputs
- Documentation OpenAPI disponible

### WebSocket

Canal temps réel pour applications tierces :

- Updates de stats en push
- Changements de status instantanés
- Events système
- Intégration dashboards custom

---

## Specs Techniques

| Paramètre | Valeur |
|-----------|--------|
| Routes simultanées | **50+** |
| Latence passthrough | **< 50ms** |
| CPU par output (passthrough) | **0%** |
| Rétention audit log | **100 000 events** |
| Rétention metrics | **7 jours** |

---

## Architecture

### Design Pattern

Vajra Cast utilise un **Reconcile Pattern** inspiré de Kubernetes :

```
┌─────────────┐      ┌─────────────┐      ┌─────────────┐
│   Database  │ ───▸ │  Reconciler │ ───▸ │   Runtime   │
│  (desired)  │      │   (diff)    │      │  (actual)   │
└─────────────┘      └─────────────┘      └─────────────┘
```

- **Database** = source de vérité (état désiré)
- **Runtime** = projection de l'état actuel
- **Reconciler** = converge le runtime vers l'état désiré

Ce pattern garantit :
- Reprise automatique après crash
- État cohérent même après redémarrage
- Pas de drift entre config et réalité

### Stack Technique

| Composant | Technologie |
|-----------|-------------|
| Core | Node.js + Express |
| Database | SQLite |
| Transcode | FFmpeg |
| SRT Engine | srt-live-transmit |
| SRTLA | srtla_rec |

---

## Notes de Migration

> Vajra Cast était précédemment connu sous le nom **Prism SRT**. Toutes les fonctionnalités sont conservées, l'API reste compatible.

---

*Documentation technique Vajra Cast — v1.0*
