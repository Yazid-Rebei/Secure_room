# ConsentMesh

> **Un maillage auto-surveillant de nœuds ESP32 qui rend l'activité sans fil d'une salle visible — publiquement, en continu, et automatiquement.**

## Présentation

**ConsentMesh** est un système embarqué distribué de sensibilisation à la confidentialité. Il observe l'environnement sans fil d'une salle à l'aide d'un maillage de nœuds ESP32, établit un consensus sur l'activité détectée, apprend ce qui est normal, et rend l'invisible visible via un affichage public.

### Ce que le système fait

- 📡 Observe l'activité RF passive (Wi-Fi 802.11 + Bluetooth) sans lire le contenu des trames
- 🤝 Agrège les données par consensus Byzantine-résilient pour garantir la fiabilité
- 🧠 Détecte les anomalies comportementales par apprentissage automatique (IsolationForest)
- 🖥️ Affiche l'état en temps réel sur un écran HDMI public et un bandeau LED WS2812B

### Ce que le système n'est PAS

| ❌ Ce n'est pas... | ✅ Mais c'est... |
|---|---|
| Un brouilleur de signal | Un observateur passif non-intrusif |
| Un système de surveillance | Un outil de contre-surveillance |
| Un détecteur d'enregistrement certifié | Un détecteur d'anomalies comportementales |
| Un système de stockage | Entièrement éphémère (aucune donnée conservée) |
| Connecté au cloud | 100% local, traitement dans la salle |

---

## Architecture du système

### Vue d'ensemble des couches

```
┌────────────────────────────────────────────────────┐
│         COUCHE 4 — Sortie publique                 │
│     Affichage HDMI + Bandeau LED WS2812B           │
├────────────────────────────────────────────────────┤
│         COUCHE 3 — Détection d'anomalies ML        │
│     Modèle IsolationForest sur Raspberry Pi        │
├────────────────────────────────────────────────────┤
│         COUCHE 2 — Consensus Byzantine-Résilient   │
│     Agrégation médiane + détection de défaillances │
├────────────────────────────────────────────────────┤
│         COUCHE 1 — Observation RF Passive          │
│     4 nœuds ESP32 en mode moniteur 802.11 passif   │
└────────────────────────────────────────────────────┘
```

### Topologie du maillage

```
Nœud 1 (ESP32) ──┐
                  │  ESP-NOW
Nœud 2 (ESP32) ──┤           ┌──────────────┐     UART      ┌─────────────────┐
                  ├──────────►│ Nœud Pont    ├──────────────►│  Raspberry Pi   │
Nœud 3 (ESP32) ──┤           │  (ESP32)     │               │  (Agrégation +  │
                  │  ESP-NOW  └──────────────┘               │   ML + Display) │
Nœud 4 (ESP32) ──┘                                          └─────────────────┘
                                                                      │
                                                           ┌──────────┴──────────┐
                                                      HDMI │               GPIO  │
                                                    ┌──────┴───┐      ┌──────────┴──┐
                                                    │ Moniteur │      │ Bandeau LED │
                                                    └──────────┘      └─────────────┘
```

### Pipeline de données

```
[ESP32 — Mode Moniteur Passif]
        │
        ▼
[Calcul du Vecteur d'Activité] (toutes les 3 secondes)
        │
        ▼
[Diffusion ESP-NOW entre pairs]
        │
        ▼
[Nœud Pont → Raspberry Pi via UART 115200 baud]
        │
        ▼
[Consensus Byzantine-Résilient — médiane + scoring de déviation]
        │
        ▼
[Inférence ML — IsolationForest → Score d'Anomalie ∈ [0, 1]]
        │
        ▼
[Sorties : Affichage HDMI + Bandeau LED WS2812B]
```

---

## Structure des fichiers

```
ConsentMesh/
│
├── README.md                          ← Ce fichier
├── .gitignore
│
├── docs/                              ← Documentation du projet
│   ├── ConsentMesh_v4_CDC.pdf         ← Cahier des charges technique
│   ├── architecture.md                ← Diagrammes d'architecture détaillés
│   ├── calibration.md                 ← Procédure de calibration de salle
│   └── demo_guide.md                  ← Guide de démonstration jury
│
├── hardware/                          ← Schémas et listes de matériel
│   ├── BOM.md                         ← Bill of Materials (liste complète)
│   ├── wiring_diagram.md              ← Schéma de câblage général
│   ├── node_corner.md                 ← Câblage nœud de détection (ESP32 + PIR + OLED)
│   └── node_bridge.md                 ← Câblage nœud pont
│
├── firmware/                          ← Code embarqué ESP32 (C/C++)
│   │
│   ├── node_detection/                ← [Membre #1] Nœuds de détection (×4)
│   │   ├── node_detection.ino         ← Point d'entrée principal
│   │   ├── wifi_monitor.cpp           ← Mode moniteur 802.11 passif
│   │   ├── wifi_monitor.h
│   │   ├── bt_scanner.cpp             ← Scan Bluetooth passif
│   │   ├── bt_scanner.h
│   │   ├── activity_vector.cpp        ← Calcul du Vecteur d'Activité
│   │   ├── activity_vector.h
│   │   ├── mac_fingerprint.cpp        ← Contournement randomisation MAC
│   │   ├── mac_fingerprint.h
│   │   ├── pir_sensor.cpp             ← Gestion capteur PIR HC-SR501
│   │   ├── pir_sensor.h
│   │   ├── oled_display.cpp           ← Affichage OLED SSD1306 (santé nœud)
│   │   ├── oled_display.h
│   │   └── config.h                   ← Paramètres (ID nœud, canaux, seuils)
│   │
│   └── node_bridge/                   ← [Membre #2] Nœud pont + maillage ESP-NOW
│       ├── node_bridge.ino            ← Point d'entrée principal
│       ├── espnow_mesh.cpp            ← Protocole maillage ESP-NOW pair-à-pair
│       ├── espnow_mesh.h
│       ├── serial_bridge.cpp          ← Transmission UART vers Raspberry Pi
│       ├── serial_bridge.h
│       └── config.h                   ← Paramètres (adresses MAC pairs, baud rate)
│
├── raspberry/                         ← Code Raspberry Pi (Python 3.10+)
│   │
│   ├── main.py                        ← Point d'entrée — orchestrateur principal
│   │
│   ├── consensus/                     ← [Membre #3] Consensus Byzantine-Résilient
│   │   ├── __init__.py
│   │   ├── aggregator.py              ← Agrégation médiane multi-nœuds
│   │   ├── deviation_scorer.py        ← Scoring de déviation par nœud
│   │   └── fault_detector.py          ← Quarantaine des nœuds défaillants
│   │
│   ├── ml/                            ← [Membre #3] Pipeline ML
│   │   ├── __init__.py
│   │   ├── inference.py               ← Inférence temps réel IsolationForest
│   │   ├── baseline_updater.py        ← Mise à jour incrémentale de la référence
│   │   └── models/
│   │       └── isolation_forest.pkl   ← Modèle entraîné (≤ 20 Mo)
│   │
│   ├── serial_reader/                 ← Lecture UART depuis nœud pont
│   │   ├── __init__.py
│   │   └── reader.py                  ← Désérialisation des Vecteurs d'Activité
│   │
│   ├── display/                       ← [Membre #4] Affichage public HDMI
│   │   ├── __init__.py
│   │   ├── public_screen.py           ← Interface Pygame/Tkinter
│   │   └── assets/
│   │       └── fonts/                 ← Polices d'affichage
│   │
│   ├── led/                           ← [Membre #4] Contrôle bandeau LED
│   │   ├── __init__.py
│   │   └── led_controller.py          ← GPIO WS2812B (vert / ambre / rouge)
│   │
│   ├── door_sensor/                   ← [Membre #4] Capteur de porte
│   │   ├── __init__.py
│   │   └── door_sensor.py             ← Détection ouverture/fermeture porte
│   │
│   └── config.py                      ← Configuration globale (seuils, ports, pins)
│
├── training/                          ← Entraînement ML (sur PC portable)
│   ├── data/
│   │   ├── baseline_raw.csv           ← Données brutes collectées en salle vide
│   │   └── sessions/                  ← Données de sessions annotées
│   ├── notebooks/
│   │   ├── 01_exploration.ipynb       ← Analyse exploratoire des données
│   │   ├── 02_training.ipynb          ← Entraînement IsolationForest
│   │   └── 03_evaluation.ipynb        ← Évaluation et choix des seuils
│   └── export_model.py                ← Export du modèle vers raspberry/ml/models/
│
├── scripts/                           ← Scripts utilitaires
│   ├── flash_nodes.sh                 ← Flash firmware sur tous les ESP32
│   ├── collect_baseline.py            ← Collecte de données de référence en salle
│   ├── calibrate_room.py              ← Calibration RSSI périmètre de salle
│   └── system_status.py               ← Vérification santé du système complet
│
└── tests/                             ← Tests et validation
    ├── test_consensus.py              ← Tests unitaires algorithme Byzantine
    ├── test_ml_inference.py           ← Tests pipeline ML
    ├── test_led_controller.py         ← Tests contrôleur LED
    └── simulate_node_failure.py       ← Simulation de panne de nœud
```

---

## Matériel requis

### Liste complète (BOM)
```
| Composant | Qté | Rôle | Coût estimé (USD) |
|---|---|---|---|
| ESP32-WROOM-32 Dev Board | 5 | 4 nœuds détection + 1 pont | 5–8 / u |
| Raspberry Pi Zero 2W | 1 | Agrégation, ML, affichage | ~15 |
| MicroSD 16 Go+ | 1 | OS Raspberry Pi + code | ~5 |
| PIR HC-SR501 | 4 | Détection d'occupation | 1–2 / u |
| Capteur magnétique de porte | 1 | Détection d'entrée | 2–4 |
| Bandeau LED WS2812B 1m (60 LED) | 1 | Indicateur d'activité principal | 8–12 |
| OLED SSD1306 0,96" I2C | 4 | Santé par nœud | 2–3 / u |
| Moniteur HDMI | 1 | Affichage public | prêté |
| Adaptateur Mini HDMI → HDMI | 1 | Pi Zero vers moniteur | 3–5 |
| Banque USB 10 000 mAh+ | 4 | Nœuds coins sans fil | 10–15 / u |
| Adaptateur secteur USB 5V 2A | 1 | Nœud d'agrégation | ~5 |
| Alimentation 5V 2A (bandeau LED) | 1 | Bandeau LED | 5–8 |
| Câbles Micro-USB / USB-C | 5 | Alimentation générale | 2–3 / u |
| Pack fils de liaison | 1 | Connexions GPIO | ~5 |
| Breadboards demi-taille | 4 | Prototypage capteurs | 2–3 / u |
| Résistance 470Ω | 1 | Protection données LED | <1 |
| Condensateur 1000 µF | 1 | Lissage alimentation LED | <1 |
| Fil 22AWG (1 rouleau) | 1 | Câblage capteur de porte | 3–5 |
| Boîtiers imprimés 3D | 4 | Protection nœuds coins | 3–8 / u |
| Bandes velcro / adhésif | 1 pack | Fixation aux murs | 3–5 |
| Colliers de câbles | 1 pack | Gestion des câbles | ~2 |

```

---


### 5. Entraînement du modèle ML

```bash
cd training/
# Lancer Jupyter pour exécuter les notebooks dans l'ordre
jupyter notebook

# Ou directement depuis le terminal :
python3 export_model.py --input data/baseline_raw.csv \
                        --output ../raspberry/ml/models/isolation_forest.pkl
```

---


## Utilisation

### Indicateurs de l'affichage public (HDMI)
```
| Indicateur | Description |
|---|---|
| **Sources actives** | Nombre de sources de transmission détectées par consensus |
| **Niveau d'activité** | Inactif / Actif / Élevé (piloté par le Score d'Anomalie ML) |
| **Indicateurs comportementaux** | Signaux lors de transmission continue anormale |
| **Santé du maillage** | Nœuds actifs / Nœuds signalés défaillants |
| **Durée de session** | Temps écoulé depuis le début de l'activité détectée |

### Signification du bandeau LED

| Couleur | État | Signification |
|---|---|---|
| 🟢 **Vert** | Inactif | Appareils présents, activité minimale |
| 🟡 **Ambre** | Actif | Comportement normal d'utilisation |
| 🔴 **Rouge** | Élevé | Un ou plusieurs appareils anormaux détectés |

> Le bandeau et l'affichage se **réinitialisent automatiquement** lorsque la salle est détectée vide. Aucun historique n'est conservé.

### Vérification de l'état du système

```bash
python3 scripts/system_status.py
```

---

## Membre Responsabilités
```
#1 Firmware ESP32 nœuds de détection — mode moniteur,
extraction de Vecteur d’Activité, randomisation MAC
#2 Firmware ESP32 maillage ESP-NOW — diffusion
pair-à-pair, nœud pont, protocole série vers Pi
#3 Raspberry Pi — algorithme de consensus
Byzantine-résilient, pipeline ML, inférence en temps réel
#4 Affichage public, contrôle bandeau LED, intégration
système, calibration de salle, démonstration
```


## Équipe
Miled Trabelsi
Houssem Khouja
Mohamed Yazid Rebai
Selim Hamdi

> *ConsentMesh v4 — Maillage ESP32 — Capteurs PIR et porte — Consensus Byzantine-Résilient — Détection d'anomalies ML embarquée — Éphémère, Transparent, Auto-surveillant*
