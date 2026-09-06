# Friend'sGame

Application Android de mini-jeux compétitifs (projet PROMOB 2021–2022). Les joueurs enchaînent **3 épreuves tirées au hasard** et accumulent des points. On peut jouer **seul** ou **à plusieurs** en local via **Wi-Fi Direct** (pas de serveur, pas d’Internet obligatoire).

Package : `com.example.friendsgame`  
SDK : `minSdk 18` · `targetSdk 31` · Java 8 · Gradle 7.2 / Android Gradle Plugin 7.0.3

---

## Fonctionnalités

- Compte local (inscription / connexion) stocké avec **Room**
- Hub de partie : Wi-Fi, recherche de pairs, liste d’appareils, classement
- Séquence de **3 jeux** générée au lancement (identifiants 1 à 5)
- Mode **entraînement** pour chaque mini-jeu
- Écrans de chargement, victoire, défaite et fin de partie
- Communication P2P : sockets TCP (port 8888) et messages JSON (`start`, scores, états)

---

## Mini-jeux

| ID | Activité | Principe |
|----|----------|----------|
| 1 | `DiceGame` | Secouer le téléphone (accéléromètre) pour lancer un dé |
| 2 | `LuminoGame` | Obtenir la plus forte luminosité mesurée par le capteur (environ 10 s) |
| 3 | `TapGame` | Taper le buzzer un maximum de fois (timer) |
| 4 | `MathGame` | Résoudre un calcul sous contrainte de temps |
| 5 | `GestureGame` | Réaliser les gestes demandés (swipe, tap, double tap) |

Chaque partie enchaîne trois IDs aléatoires (`MainActivity.generateGames()`). Un écran de chargement (`LoadingScreen`) affiche une pause d’environ 5 s entre les épreuves.

> En code, le `switch` de `LoadingScreen` mappe encore l’ID 5 vers `TapGame` au lieu de `GestureGame`. Le jeu de gestes existe (mode compétition + pratique) mais n’est pas forcément tiré dans la séquence.

---

## Comment jouer

1. Lancer l’app → **Login** (ou **Create Account**).
2. Lire l’écran d’accueil (`WelcomeActivity`, texte dans `assets/text/explication.txt`) ou arriver directement sur le hub si déjà connecté.
3. **Solo** : **PLAY** lance la séquence de 3 jeux.
4. **Multijoueur** :
   - Activer le Wi-Fi (**WIFI**)
   - **SEARCH** pour découvrir les appareils proches
   - Sélectionner un appareil dans **Devices**
   - **PLAY** : l’hôte envoie l’ordre des jeux aux autres
5. **Practice** : s’entraîner à un jeu sans enchaînement ni score partagé.

Les scores sont comparés en fin de séquence (`determineWinner` / `determineRanking`). Un historique Room (`HistoryActivity`) est présent dans le code mais **n’est pas déclaré** dans le `AndroidManifest` ni branché depuis le hub.

---

## Stack

| Couche | Choix |
|--------|--------|
| UI | Activities, Material / AppCompat, ConstraintLayout |
| Persistance | Room (`User`, `Attempt`), SharedPreferences (`SharedPref`) |
| Réseau | Wi-Fi Direct (`WifiP2pManager`) + `ServerSocket` / `Socket` |
| JSON | Gson (JAR local `app/libs/gson-2.8.7.jar`) |
| Capteurs | Accéléromètre, luminosité, vibration |

Permissions : localisation (fine/coarse, exigée par Wi-Fi Direct), Wi-Fi / réseau, vibration.

---

## Structure du projet

```
app/src/main/
├── AndroidManifest.xml
├── assets/text/          # textes d’accueil, chargement, victoire, défaite
├── java/com/example/friendsgame/
│   ├── LoginActivity.java, RegisterActivity.java, WelcomeActivity.java
│   ├── MainActivity.java              # hub + Wi-Fi Direct + sockets
│   ├── WiFiDirectBroadcastReceiver.java
│   ├── DiceGame.java, LuminoGame.java, TapGame.java, MathGame.java, GestureGame.java
│   ├── PracticeActivity.java + practice/
│   ├── HistoryActivity.java
│   ├── data/                          # Room : User, Attempt, DAO, Database
│   ├── other/                         # SharedPref, adapters, utilitaires
│   └── temporary/                     # Loading / Victory / Defeat / Finished
└── res/layout/                        # écrans et jeux
```

---

## Prérequis et lancement

- Android Studio (Arctic Fox ou plus récent compatible AGP 7.0)
- JDK 8+
- Un ou plusieurs appareils physiques recommandés pour le Wi-Fi Direct (l’émulateur est limité)

Ouvrir le dossier racine, laisser Gradle sync, puis **Run** sur le module `app`.

En ligne de commande :

```bash
./gradlew assembleDebug
```

Sous Windows : `gradlew.bat assembleDebug`.

---

## État connu (notes de développement)

Issues déjà identifiées dans l’ancien README / le code :

- Les **transitions** entre jeux passent par un loading de 5 s ; le ressenti peut rester trop brusque ou trop long.
- L’échange de messages P2P a pu être **instable** (JSON, sockets sur le thread principal via `StrictMode`).
- **GestureGame** n’est pas correctement branché dans la séquence aléatoire (`LoadingScreen`).
- Timers de debug encore courts dans certains jeux (ex. Tap / Math à 10 s au lieu des durées visées 30 s / 45 s, d’après les commentaires).
- `HistoryActivity` n’est pas exposée dans le manifeste.
- Extension prévue à **6 jeux** (commentaires dans `table_games` / `GAME_COUNT`) non finalisée.

---

## Licence / contexte

Projet académique PROMOB (2021–2022). Pas de licence indiquée dans le dépôt.
