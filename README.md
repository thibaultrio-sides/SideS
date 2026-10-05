# 🛡️ SideS

Logo SideS

> **Signaler • Informer • Protéger**

Android  
Kotlin  
Licence  
Statut

**SideS** est une application Android native qui rend la route plus sûre pour les usagers vulnérables. Lorsqu'un cycliste, coureur, cavalier, utilisateur de trottinette ou de fauteuil roulant active le signalement, sa position est transmise toutes les 3 secondes : les conducteurs SideS proches (moins de 200 m) reçoivent une alerte sonore. L'application détecte aussi automatiquement les chutes et envoie un SOS par SMS aux contacts d'urgence. Une intégration Waze (programme Connected Citizens, candidature en cours d'approbation) est prévue pour alerter à terme tous les conducteurs Waze.

**Site officiel :** [https://applisides.netlify.app](https://applisides.netlify.app) · **Contact :** [thibault.rio@gmail.com](mailto:thibault.rio@gmail.com)  
**Télécharger :** [sides.apk — v1.0](https://github.com/thibaultrio-sides/SideS/releases/download/v1.0/sides.apk) (Android 8.0 minimum)

---

## 📱 Pourquoi SideS ?

Chaque année, les usagers vulnérables sont les premières victimes de la route. SideS répond à un constat simple : **un conducteur qui sait qu'un cycliste arrive le voit**. L'app connecte les usagers et les conducteurs, avec ou sans installation côté conducteur.


| Sans SideS                                | Avec SideS                                           |
| ----------------------------------------- | ---------------------------------------------------- |
| Le conducteur découvre le cycliste à 30 m | Il est alerté à 200 m                                |
| Une chute seule peut rester invisible     | SMS automatique aux proches après 30 s sans réaction |
| Aucun signalement en cas d'accident       | Lien GPS exact envoyé aux contacts d'urgence         |


## ✨ Fonctionnalités

### Côté usager vulnérable

- 🚴 **6 profils disponibles** : cycliste, trottinette, cavalier, coureur, randonneur, fauteuil roulant
- 🗺️ **Signalement GPS en temps réel** — position haute précision toutes les 3 s, même app en arrière-plan (`ForegroundService`)
- 📊 **Carte temps réel** — parcours, vitesse, altitude, distance (Google Maps)
- ⚠️ **Détection automatique de chute** — algorithme en 2 phases (chute libre puis impact) via l'accéléromètre
- ⏱️ **Compte à rebours de 30 s** annulable pour éviter les faux positifs
- 🆘 **Bouton SOS manuel** — SMS immédiat aux contacts d'urgence avec lien Google Maps
- 👥 **Jusqu'à 5 contacts d'urgence**, stockés uniquement sur l'appareil

### Côté conducteur

- 🚗 **Mode conducteur** intégré dans la même app
- 🔔 **Alerte sonore + vibration** dès qu'un usager vulnérable est à moins de 200 m, même en arrière-plan (Firebase Cloud Messaging)
- 📍 **Carte avec marqueurs** des usagers proches

### Bientôt, pour tous les conducteurs

- 🔵 **Signalements Waze automatiques** via le programme Waze Connected Citizens (candidature en cours d'approbation) — aucun besoin d'installer quoi que ce soit

## 🔄 Comment ça marche

```
┌─────────────────────┐         ┌──────────────────────────┐
│  App SideS           │         │   Backend Firebase        │
│  (Usager vulnérable) │──GPS──▶ │   Cloud Functions         │
│                      │  3 s    │                          │
│  sendPosition()      │         │  ├─ Firestore (TTL 30 s) │
└─────────────────────┘         │  ├─ Signalement Waze CCP  │
                                │  └─ Notification FCM      │
┌─────────────────────┐         │     conducteurs < 200 m   │
│  Waze               │◀──API───└──────────────────────────┘
│  (tous conducteurs) │                  │ FCM Push
│  Danger sur trajet 🚨│                  ▼
└─────────────────────┘         ┌──────────────────────────┐
                                │  App SideS (Conducteur)  │
                                │  Son + vibration + carte │
                                └──────────────────────────┘

> ℹ️ Le flux Waze (bloc de gauche) sera actif après approbation du programme Connected Citizens. Les alertes FCM vers les conducteurs SideS fonctionnent déjà.
```

## 🔒 Confidentialité — la transparence d'abord

SideS est conçu autour d'un principe : **collecter le strict minimum, ne rien garder**.


| Donnée                | Envoi                                        | Conservation                                           |
| --------------------- | -------------------------------------------- | ------------------------------------------------------ |
| Position GPS          | Uniquement pendant le signalement (3 s)      | **Effacée automatiquement après 30 s** (TTL Firestore) |
| Contacts d'urgence    | **Jamais en ligne** — stockés sur l'appareil | Supprimés à la désinstallation                         |
| Jeton de notification | Firebase Cloud Messaging                     | Lié à l'app                                            |
| SMS d'urgence         | Directement depuis le téléphone              | Aucune copie conservée                                 |


- ❌ Aucun compte requis, aucun e-mail, aucun profilage
- ❌ Aucune publicité, aucune revente de données
- ✅ Politique de confidentialité complète : [site officiel](https://applisides.netlify.app#politique)

## 📐 Architecture du projet

```
SideS/
├── app/src/main/java/com/sides/
│   ├── MainActivity.kt              # Écran principal
│   ├── services/
│   │   └── TrackingService.kt       # Service GPS + détection chute (ForegroundService)
│   ├── ui/
│   │   ├── EmergencyContactActivity.kt  # Gestion contacts d'urgence
│   │   ├── DriverActivity.kt        # Mode conducteur
│   │   └── MapActivity.kt           # Carte temps réel
│   ├── utils/
│   │   └── SecurePrefs.kt           # Préférences chiffrées (Android Keystore)
│   ├── adapters/
│   │   └── ContactAdapter.kt        # RecyclerView contacts
│   └── models/
│       └── EmergencyContact.kt      # Modèle de données
├── app/src/main/res/
│   ├── layout/
│   └── values/                      # colors.xml, themes.xml, strings.xml
└── app/src/main/AndroidManifest.xml
```

## 🚀 Installation et compilation

### Pré-requis

- Android Studio Hedgehog (2023.1+) ou plus récent
- SDK Android 26+ (Android 8.0 Oreo minimum)
- Un compte Google Cloud pour l'API Maps

### Étapes

1. **Cloner le dépôt**
  ```bash
   git clone https://github.com/thibaultrio-sides/SideS.git
  ```
2. **Clé API Google Maps** — sur [console.cloud.google.com](https://console.cloud.google.com), activer *Maps SDK for Android*, puis remplacer `VOTRE_CLE_API_GOOGLE_MAPS_ICI` dans `AndroidManifest.xml`
3. **Backend Firebase** — créer un projet sur [console.firebase.google.com](https://console.firebase.google.com), activer Firestore + Cloud Messaging + Cloud Functions, déposer `google-services.json` dans `app/`
4. **Compiler**
  ```bash
   ./gradlew assembleRelease
  ```

> Le backend (Cloud Functions, règles Firestore, configuration Waze CCP) est détaillé dans [`DEPLOYMENT_GUIDE.md`](DEPLOYMENT_GUIDE.md).

## 🔐 Permissions requises


| Permission                   | Usage                 |
| ---------------------------- | --------------------- |
| `ACCESS_FINE_LOCATION`       | GPS précis            |
| `ACCESS_BACKGROUND_LOCATION` | Suivi en arrière-plan |
| `FOREGROUND_SERVICE`         | Service persistant    |
| `SEND_SMS`                   | Alertes d'urgence     |
| `CALL_PHONE`                 | Appel de secours      |
| `VIBRATE`                    | Alerte locale chute   |
| `BODY_SENSORS`               | Accéléromètre         |


## 🗺️ Feuille de route

- [ ] **Signalements Waze (Waze for Cities)** — à l'étude, via un partenariat avec une collectivité
- [ ] Authentification Firebase (anonyme, pour la vie privée)
- [ ] Intégration Apple CarPlay / Android Auto
- [ ] Détection de chute depuis Wear OS
- [ ] Mode groupe — partager sa position avec ses compagnons de ride
- [ ] Intégration 112 (appel automatique aux secours européens)
- [ ] OpenStreetMap en alternative gratuite à Google Maps
- [ ] Version iOS

## 🤝 Contribuer

Les issues et pull requests sont bienvenues ! Pour une contribution importante, ouvrez d'abord une issue pour en discuter.

## 📄 Licence

Ce projet est distribué sous licence MIT. Voir [`LICENSE`](LICENSE).

## 👤 Auteur

**Thibault RIO** — [thibault.rio@gmail.com](mailto:thibault.rio@gmail.com)

---

*SideS — Signaler • Informer • Protéger* 🛡️
