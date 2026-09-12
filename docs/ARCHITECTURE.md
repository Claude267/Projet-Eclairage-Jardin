# Architecture Éclairage Jardin

## Vue d'ensemble

Système de gestion de l'éclairage du jardin par badges BLE, avec temporisation intelligente basée sur la proximité.

```
Portail                           Maison
  │                                 │
  ├─ ESP32 Porte ◄────UART────► ESP32 Maison ──► Wi-Fi ──► Home Assistant
  │  (scan BLE)              (scan BLE local +              (logique + Shelly)
  │                          réception UART)
  │
  └─ Badges (BLE)
      ├─ Badge 1
      ├─ Badge 2
      └─ Badge 3

     Shelly 1 Gen4
     (relais + auto_off)
            ▲
            │ REST API
            │
       Home Assistant
```

## Composants

### Matériel
- **ESP32 Porte** : Scanner BLE continu, transmission UART des trames vers Maison
- **ESP32 Maison** : Récepteur UART + scanner BLE local, émission événements vers HA
- **Shelly 1 Gen4** (192.168.1.36) : Relais Wi-Fi avec extinction temporisée (auto_off_delay)
- **Badges BLE** (3) : Balises émettant du BLE avec puissance TX réduite (-10 dBm)

### Logique Home Assistant
1. **Détection** : ESP Maison envoie événement `esphome.presence_ble`
2. **Normalisation** : MAC mise en majuscules, espaces retirés
3. **RSSI affiché** : Capteur de RSSI mis à jour **avant filtrage** (voir calibration)
4. **Filtrage** : RSSI ≥ seuil (négatif, -60 > -80, plus proche)
5. **Durée badge** : Si franchissement seuil → durée personnalisée du badge
6. **Shelly** : Envoi `toggle_after` pour prolonger l'extinction
7. **Journal** : Ajout à la liste JSON des 10 derniers passages

## Temporisation (Option C)

### Durée de base
- Gravée dans le Shelly (`auto_off_delay`)
- Repoussée par HA à 3 moments :
  - Démarrage de HA
  - Modification du curseur dans le TdB
  - Au premier contact avec le Shelly
- Avantage : Marche aussi Wi-Fi coupé

### Durée badge
- Stockée dans `input_datetime.badge_N_duree` (format HH:MM:SS)
- À chaque détection d'un badge au-dessus du seuil :
  - HA envoie `Switch.Set` avec `toggle_after: <durée badge>`
  - Le Shelly reprogramme son auto_off sans couper la lumière
- Avantage : Aucune usure flash du Shelly en usage courant

### Extinction manuelle
- Détection de l'extinction (state: "off")
- Restauration immédiate de la durée de base au Shelly
- Empêche qu'un passant sans badge hériterait de la durée longue

## Flux données

```
Événement esphome.presence_ble
    ↓
Normalisation MAC (upper + trim)
    ↓
Recherche badge (input_text.badge_N_mac)
    ↓
Mise à jour RSSI (attribut du input_text)
    ↓
Horodatage dernier passage
    ↓
Filtrage RSSI ≥ seuil?
    ├─ Non → Fin
    │
    └─ Oui → Récupérer durée badge
                ↓
            Ajouter journal
                ↓
            REST API Shelly
            toggle_after: <durée badge>
```

## Stockage

- **input_text** : Noms et MAC des badges (+ attributs rssi_porte/rssi_maison)
- **input_number** : Durée de base, seuils RSSI
- **input_datetime** : Durée de chaque badge, horodatage dernier passage
- **sensor (template)** : RSSI par badge/source (lecture des attributs)
- **binary_sensor (template)** : "À portée" par badge/source (comparaison RSSI/seuil)
- **input_text.jardin_journal_passages** : JSON avec 10 derniers passages

Pas d'écriture Flash du Shelly en usage normal (aucune usure).

## Sécurité

- Clé Noise `api_encryption_key` en secrets.yaml (jamais committée)
- MAC privées/éphémères filtrées (test du bit localement administré)
- Dédoublonnage 5-10s : limite débits
- Validation MAC au format HH:HH:HH:HH:HH:HH
- Shelly sans mot de passe, accès local seulement (192.168.1.36)

## Calibration (voir CALIBRATION.md)

À faire une seule fois ou lors de changement de position :
1. Noter le RSSI à différentes distances
2. Ajuster `badge_rssi_filtrage_porte` et `_maison` pour délimiter la zone
3. Tester : avancer/reculer, vérifier déclenchements

Recommandations :
- Porte : 1m environ → seuil ~-70 dBm
- Maison : 5m environ → seuil ~-80 dBm
