# Projet Éclairage Jardin

Gestion intelligente de l'éclairage du jardin par badges BLE, avec temporisation personnalisée basée sur la proximité.

**Note** : Ce repo contient la **configuration Home Assistant uniquement**. Les ESP32 sont déjà flashés et fonctionnels.

## Vue rapide

- **Matériel** : 2× ESP32-C3 flashés (Porte + Maison), 1× Shelly 1 Gen4, 3× Badges BLE
- **Logique** : ESP → événement `esphome.presence_ble` → Home Assistant → REST API Shelly
- **Automatisation** : Détection badge → durée d'éclairage personnalisée
- **Durée de base** : Configurable, persistée dans le Shelly (fonctionne Wi-Fi coupé)

## Déploiement rapide

### 1. Intégrer dans Home Assistant
Ajouter à `configuration.yaml` :
```yaml
homeassistant:
  packages:
    eclairage_jardin: !include packages/eclairage_jardin.yaml
```

Redémarrer Home Assistant.

### 2. Vérifier que les ESP sont détectés
Aller à Paramètres → Appareils et services → ESPHome.
Tu dois voir :
- `esp32-porte`
- `esp32-maison`

Les deux doivent être "En ligne". Si non, voir `docs/INSTALLATION.md`.

### 3. Configurer les badges
Tableau de bord Jardin → remplir pour chaque badge :
- **Nom** : ex. "Alice"
- **MAC** : ex. "A4:C1:38:19:80:0B" (en majuscules)
- **Durée** : ex. 00:05:30 (5 min 30 s)

### 4. Régler la durée de base
Réglages → 60 secondes (ajuster selon besoin, 15-300 s).

### 5. Calibrer les seuils RSSI
Voir `docs/CALIBRATION.md` pour régler `-70` dBm (Porte) et `-80` dBm (Maison).

## Structure

```
.
├── homeassistant/
│   ├── packages/
│   │   └── eclairage_jardin.yaml          # Package HA (helpers + automations)
│   └── dashboards/
│       └── jardin.yaml                    # Tableau de bord
├── docs/
│   ├── ARCHITECTURE.md                    # Schéma global
│   ├── INSTALLATION.md                    # Dépannage + détails
│   └── CALIBRATION.md                     # Réglage RSSI
├── .gitignore
└── README.md
```

## Documentation

- **[ARCHITECTURE.md](docs/ARCHITECTURE.md)** : Vue d'ensemble, flux données, option C
- **[INSTALLATION.md](docs/INSTALLATION.md)** : Étapes complètes, checkliste, dépannage
- **[CALIBRATION.md](docs/CALIBRATION.md)** : Réglage des seuils RSSI, optimisation

## Fonctionnalités

✅ **3 badges** : nom, MAC, durée d'éclairage  
✅ **Durée de base** : configurable, persistée dans Shelly  
✅ **Détection smart** : RSSI ≥ seuil → déclenche durée badge  
✅ **Extinction manuelle** : restaure durée de base  
✅ **RSSI en temps réel** : affichage par badge/source  
✅ **Seuils indépendants** : Porte (1m) vs Maison (5m)  
✅ **Journal** : 10 derniers passages (JSON)  
✅ **Wi-Fi coupé** : durée de base fonctionne (gravée Shelly)  

⏳ **À venir** : Graphique RSSI, hystérésis, détection multi-source.

## Archi matériel

```
Portail                          Maison
  │                                │
  ├─ ESP32 Porte ◄──UART 4800──► ESP32 Maison ◄─ Wi-Fi ─ Home Assistant
  │  (scan BLE)                 (scan local + UART)        (HA)
  │                                 │                        │
  │                                 └──────► événement ◄─────┤
  │                                          esphome.       │
  │                                          presence_ble   │
  │                                                          ↓
  │                                            Shelly 1 Gen4 (REST)
  │
  └─ Badges (BLE TX -10dBm) ─────────────► Détection proxiité
```

## Réglages par défaut

| Entité | Valeur | Min | Max |
|--------|--------|-----|-----|
| Durée base | 60 s | 15 s | 300 s |
| Badge durée | 5:30 | 00:00 | 23:59 |
| Seuil Porte | -80 dBm | -95 | -40 |
| Seuil Maison | -80 dBm | -95 | -40 |

## Sécurité

- Clé API Noise en `secrets.yaml` (jamais commitée)
- MAC privées filtrées (bit administré)
- Shelly local, sans authentification (réseau privé)

## Contribution

Pour améliorer :
1. Fork le repo
2. Tester sur vos ESP + HA
3. Pull request avec détails

## Licence

MIT (usage libre, modif bienvenue)

## Auteur

Codé par Claude Sonnet, pour Claude267.

---

**Besoin d'aide ?**
- Lire `docs/INSTALLATION.md` section "Dépannage"
- Vérifier les logs : `esphome logs esphome/esp32-maison.yaml`
- Home Assistant Logs → Automation history
