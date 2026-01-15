# Vajra Cast - Contenu à Fournir

## Screenshots Requis

Capturer depuis l'interface Prism SRT en production.

| # | Nom fichier | Page/Vue | Ce qu'il faut montrer |
|---|-------------|----------|----------------------|
| 1 | `dashboard.png` | Page principale `/` | Liste des routes avec statuts (running/stopped), indicateurs RTT/bandwidth, boutons actions |
| 2 | `route-config.png` | Modal "Add Route" ou "Edit Route" | Formulaire avec nom, mode (passthrough/transcode), section inputs et outputs |
| 3 | `metrics.png` | Vue metrics d'une route | Graphique temps réel RTT/bandwidth/packet loss, sélecteur de période (1h/24h/7d) |
| 4 | `audit.png` | Page `/audit` | Timeline ou tableau des events avec filtres visibles |
| 5 | `audio-matrix.png` | Config transcode d'une route | Interface audio matrix avec canaux, gains, routing |
| 6 | `system.png` | Page `/system` | Dashboard système avec CPU, RAM, uptime, graphiques |

**Format recommandé:** PNG, 1920x1080 ou 2560x1440, thème sombre

**Astuce:** Avoir 2-3 routes actives avec du trafic réel pour montrer les metrics en action.

---

## Vidéos Tutoriels

Format: MP4, 1920x1080, 30fps, audio voix + screen recording

| # | Nom fichier | Durée | Contenu |
|---|-------------|-------|---------|
| 1 | `getting-started.mp4` | ~5 min | Installation serveur, premier lancement, accès interface web |
| 2 | `first-route.mp4` | ~8 min | Créer une route SRT listener → outputs RTMP (YouTube) + HLS |
| 3 | `srtla-setup.mp4` | ~10 min | Config SRTLA avec BELABOX, latence recommandée (1200ms+), test terrain |
| 4 | `audio-matrix.mp4` | ~7 min | Mode transcode, config audio 7.1→stereo, extraction canaux |
| 5 | `api-integration.mp4` | ~12 min | Exemples curl/fetch, auth JWT, WebSocket pour dashboard custom |

**Script suggéré pour chaque vidéo:**
1. Intro rapide (10s) - "Dans cette vidéo..."
2. Démo pas à pas avec narration
3. Résultat final visible
4. Récap des points clés (15s)

---

## Placement des fichiers

```
assets/
├── screenshots/
│   ├── dashboard.png
│   ├── route-config.png
│   ├── metrics.png
│   ├── audit.png
│   ├── audio-matrix.png
│   └── system.png
└── videos/
    ├── intro.mp4          (déjà existant)
    ├── getting-started.mp4
    ├── first-route.mp4
    ├── srtla-setup.mp4
    ├── audio-matrix.mp4
    └── api-integration.mp4
```

Une fois les fichiers placés, je mettrai à jour le code HTML pour les afficher.
