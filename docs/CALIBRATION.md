# Calibration RSSI

## Principes

Le RSSI (Received Signal Strength Indicator) mesure la puissance du signal BLE reçu.
- Plus **proche** = valeur **moins négative** (ex. -60 dBm près, -90 dBm loin)
- Seuil : si RSSI ≥ seuil → badge détecté
- Deux seuils indépendants : Porte (1m) et Maison (5m)

## Préparation

1. Un badge flashé et configuré dans HA
2. Le TdB Jardin ouvert (ou Développeur → États)
3. Marcher jusqu'aux limites souhaitées du capteur
4. Lire les valeurs RSSI affichées en temps réel

## Calibration Porte (ESP32 Porte)

### Objectif
Détecter les badges au portail (≈1m) sans faire gaffes au-delà.

### Procédure

1. **Position 0** : Badge à proximité immédiate du Shelly
   - Observer `sensor.badge_1_rssi_porte`
   - Noter la valeur (ex. -65 dBm)

2. **Position 1** : Badge à 1-2m du Shelly
   - Observer la valeur (ex. -75 dBm)

3. **Position 2** : Badge à la limite souhaitée (ex. 3m)
   - Observer la valeur (ex. -85 dBm)

4. **Régler le curseur** `input_number.badge_rssi_filtrage_porte`
   - Exemple : pour détecter jusqu'à 2m, mettre -80 dBm
   - Le badge s'affichera "À portée Porte" si RSSI ≥ -80 dBm

### Vérification
- Avancer : "À portée" passe à ON
- Reculer : "À portée" passe à OFF
- Testez plusieurs fois, plusieurs positions

## Calibration Maison (ESP32 Maison)

### Objectif
Détecter quand la personne entre dans la maison (≈5m).

### Procédure

Même que Porte, mais :
- Lire `sensor.badge_1_rssi_maison`
- Poser le badge dans le jardin, graduellement plus loin
- Trouver la limite à 5-10m

Exemple :
- À 3m → -75 dBm
- À 5m → -82 dBm
- À 10m → -90 dBm
→ Mettre seuil à -85 dBm pour détecter jusqu'à 6-7m

## Réglages recommandés

| Endroit | Distance | Seuil suggéré |
|---------|----------|---------------|
| Portail (Porte) | 1-2m | -70 à -75 dBm |
| Maison (Maison) | 5m | -80 à -85 dBm |

**Ajuster selon la propagation réelle** (murs, obstacles, TX badge).

## Optimisation avancée

### Hystérésis
À faire plus tard si déclenchements erratiques :
- Armement : RSSI ≥ -80 dBm
- Désarmement : RSSI < -86 dBm (hystérésis 6 dB)

Évite les basculements rapides près du seuil.

### Nombre de trames minimum
À faire plus tard si pics de propagation faux-positifs :
- Exiger 2 trames au-dessus du seuil en 20s

## Diagnostic

**"RSSI = unknown"**
→ Badge pas encore détecté. Approcher du Shelly.

**"À portée" clignote**
→ Seuil trop juste, hystérésis à ajouter.

**"À portée" jamais OFF**
→ Seuil trop bas, augmenter (moins négatif).

**"À portée" jamais ON**
→ Seuil trop haut, diminuer (plus négatif).

## Affinement après 1 semaine

Une fois le système en place :
- Observer les vrais passages (journal)
- Noter si des passages sont manqués ou faux
- Ajuster ±5 dBm si besoin

Exemple : "Badge toujours détecté à la cuisine (5m)" → augmenter seuil à -88.

## Impact de TX badge

Vous avez réglé la puissance TX à -10 dBm dans Feasycon.

Si RSSI trop faible (toujours unknown) :
- Augmenter TX power dans l'app (+5, +10 dBm)
- Recalibrer les seuils

Si RSSI trop fort (détection au loin) :
- Diminuer TX power (-15, -20 dBm)
- Recalibrer les seuils

**Règle** : Moins vous montez le TX, plus le système est stable.

## Checklist calibration

- [ ] Badge 1 détecté à Porte
- [ ] Badge 1 détecté à Maison
- [ ] Seuil Porte réglé (limite 2-3m)
- [ ] Seuil Maison réglé (limite 5-10m)
- [ ] Tester approche/éloignement × 5
- [ ] Lumière s'allume à durée du badge
- [ ] Lumière s'éteint à la durée
- [ ] Répéter pour Badge 2 et 3

Fait ! ✓
