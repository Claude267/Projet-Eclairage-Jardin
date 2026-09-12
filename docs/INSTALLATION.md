# Installation Tranche 2

## Prérequis

- Home Assistant avec intégration ESPHome installée
- Deux ESP32-C3 flashés et en ligne (Porte + Maison)
- Shelly 1 Gen4 sur le réseau local (192.168.1.36)
- Badges BLE avec TX power -10 dBm

## Étapes

### 1. Vérifier que les ESP sont en ligne

Aller à **Paramètres → Appareils et services → ESPHome**.

Tu dois voir :
- `esp32-porte` : En ligne
- `esp32-maison` : En ligne

Si "Impossible de se connecter", vérifier :
- Que les ESP sont allumés et connectés au Wi-Fi
- Que Home Assistant peut les atteindre (même réseau local)

### 2. Intégrer dans Home Assistant

#### Option A : Package unique (recommandé)

Ajouter à `configuration.yaml` :
```yaml
homeassistant:
  packages:
    eclairage_jardin: !include packages/eclairage_jardin.yaml
```

Redémarrer HA.

#### Option B : Copier-coller dans l'éditeur GUI

(Non recommandé, harder à maintenir)

### 3. Vérifier les entités créées

Dashboard → Développeur → États, chercher :
- `input_text.badge_1_mac` → `input_text.badge_3_mac` ✓
- `input_text.badge_1_nom` → `input_text.badge_3_nom` ✓
- `input_datetime.badge_1_duree` → `input_datetime.badge_3_duree` ✓
- `input_number.duree_active_d_eclairage` ✓
- `input_number.badge_rssi_filtrage_porte` / `_maison` ✓
- `sensor.badge_*_rssi_*` ✓
- `binary_sensor.badge_*_a_portee_*` ✓
- `switch.shelly1g4_ccba97c455a0` (découvert automatiquement via intégration Shelly) ✓

### 4. Configurer les badges

Tableau de bord → Jardin, remplir pour chaque badge :

**Badge 1**
- Nom : ex. "Alice"
- MAC : ex. "A4:C1:38:19:80:0B" (en majuscules)
- Durée : 00:05:30 (5 min 30 s)

**Badge 2, 3** : idem

**Réglage de base**
- Durée sans badge : 60 secondes (ajuster selon besoin, 15-300 s)

**Seuils de détection** (voir CALIBRATION.md)
- Porte : -70 dBm
- Maison : -80 dBm

### 5. Vérifier que le Shelly reçoit la durée de base

Une fois HA redémarré et le package chargé, tester :

```bash
curl -X GET "http://192.168.1.36/rpc/Switch.GetConfig?id=0"
```

Vous devez voir :
```json
{
  "auto_off": true,
  "auto_off_delay": 60.0
}
```

Si `auto_off_delay` est différent, HA n'a pas pu communiquer avec le Shelly.
Vérifier :
- IP 192.168.1.36 accessible
- Pas de firewall bloquant HTTP
- Pas de mot de passe sur le Shelly

### 6. Tester la détection d'un badge

- Approcher un badge de l'ESP Porte
- Vérifier dans les logs ESPHome : trame `PORTE;MAC;RSSI`
- Vérifier dans l'automatisation Jardin (Outils → Automatisations)
- Vérifier le RSSI du badge dans le TdB → doit se mettre à jour
- Approcher le badge du Shelly
- La lumière doit s'allumer avec la durée du badge
- La lumière doit s'éteindre automatiquement après cette durée
- Vérifier le journal : dernière entrée avec le nom du badge

### 7. Tableau de bord

Importer le dashboard :

**Home Assistant ≥ 2024.12**
- Créer un dashboard vide
- Cliquer sur "⋯" → "Éditer le YAML"
- Copier le contenu de `homeassistant/dashboards/jardin.yaml`

**Home Assistant < 2024.12**
- Créer les cartes manuellement (ui-lovelace.yaml ou GUI)

### 8. Supprimer les anciennes entités (si migration)

Si vous aviez d'autres configurations antérieures :
```
Développeur → États
Filtrer par "badge"
Supprimer les doublets (ex. badge_1_mac_2)
```

Vérifier que le TdB pointe toujours sur les bonnes entités.

### 9. Checkliste finale

- [ ] Les deux ESP32 connectés à HA
- [ ] Événement `esphome.presence_ble` publié (Outils → Événements)
- [ ] RSSI mis à jour dans le TdB
- [ ] Durée de base syncée au Shelly
- [ ] Un badge détecté → lumière s'allume la durée du badge
- [ ] Extinction manuelle restaure la durée de base
- [ ] Journal enregistre les passages

## Dépannage

**"Entity switch.shelly1g4_ccba97c455a0 not found"**
→ Intégration Shelly non détectée. Ajouter manuellement dans Paramètres → Appareils et services.

**"RSSI toujours 'unknown'"**
→ Badge detecté mais pas dans la liste. Ajouter son MAC à un input_text.badge_N_mac.

**"Lumière ne s'éteint pas"**
→ Vérifier que le Shelly a bien reçu la durée (curl ci-dessus).
→ Vérifier que le toggle_after est envoyé (Outils → Modèles).

**Déconnexions fréquentes des ESP**
→ Vérifier Wi-Fi (signaux faibles ?).
→ Augmenter le délai `reboot_timeout` dans les secrets.

## Sauvegarde

Committer régulièrement dans Git :
```bash
git add homeassistant/ esphome/*.yaml docs/
git commit -m "Configuration v1.0 : badges calibrés"
git push
```

Ne **jamais** committer `secrets.yaml`.
