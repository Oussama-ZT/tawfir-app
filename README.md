# EcoBudget

Application mobile de gestion financière personnelle (Android et iOS, Flutter).

## Fonctionnalités

| Exigence | Où dans l'app |
|---|---|
| Introduction | Au tout premier lancement : 3 étapes (noter une dépense, la jauge du budget, les conseils et objectifs), avec « Passer » |
| Visite guidée | À la première arrivée sur l'accueil : 6 étapes qui éclairent les vrais éléments (carte du budget, portefeuilles, bouton +, onglets Dépenses et Budgets, profil). « Passer la visite » ou toucher n'importe où pour avancer ; à revoir depuis **Profil → Revoir la visite guidée** (`lib/widgets/tour.dart`) |
| Compte utilisateur | **Continuer avec Google**, **Continuer avec Apple** (iPhone) ou e-mail ; session mémorisée ; déconnexion et suppression du compte dans **Profil** (avatar en haut de l'accueil) |
| Formules Starter / Pro / Premium | **Profil → Formule** : tableau comparatif ; limites définies dans `lib/models/plan.dart` |
| Portefeuilles | **Profil → Portefeuilles** : espèces, compte bancaire, carte, épargne… ; chacun peut avoir sa propre devise (ex. Paysera en euros, avec votre taux de change) ; chaque dépense est liée à un portefeuille ; filtre dans **Dépenses**, total du mois sur l'**Accueil** |
| Solde des portefeuilles | Boutons **Ajouter / Retirer / Transférer** sur l'Accueil et sur chaque portefeuille ; transferts entre devises ; **Corriger le solde** (menu ⋮) ; puces en haut de l'écran pour passer d'un portefeuille à l'autre ; historique des mouvements |
| Catégories personnalisées | **Profil → Catégories** ou tuile « Nouvelle » dans le formulaire de dépense (Pro et Premium) |
| Trois langues | Français, anglais et arabe (de droite à gauche), dans **Profil → Langue** ou sur l'écran de connexion ; « Automatique » suit la langue du téléphone |
| Devise universelle | 26 devises au choix à l'inscription (devinée d'après la région du téléphone), modifiable dans **Profil** |
| Enregistrement et catégorisation des dépenses | Bouton **+** de la barre du bas (10 catégories, date, note) ; onglet **Dépenses** : filtrer, modifier en touchant, supprimer en glissant |
| Budgets mensuels | Onglet **Budgets** : budget global en **enveloppe** (argent réservé dans les portefeuilles, voir plus bas) + budgets par catégorie |
| Graphiques | **Accueil** : jauge du mois et répartition par catégorie ; **Budgets** : six derniers mois avec ligne de budget |
| Suivi des objectifs d'épargne | Onglet **Budgets → Objectifs** : montant visé, échéance, versements / retraits, épargne mensuelle nécessaire |
| Innovation : recommandations et dépenses inhabituelles | **Conseils** sur l'accueil, liste complète via « Tout voir » ; alerte immédiate à l'ajout d'une dépense inhabituelle |

Pour une présentation : **Profil → Charger les données de démo** (5 mois d'exemples).

## Le budget en enveloppe

Le budget du mois est de l'argent **réservé dans un ou plusieurs portefeuilles** (onglet Budgets → toucher la carte du budget).

- L'argent reste dans le portefeuille, mais il est mis de côté. La carte du portefeuille affiche ce qui est **libre**, et le détail « Libre / Réservé au budget / Total ».
- Chaque dépense est prise sur la part du portefeuille qui la paie. Au-delà de cette part, ou depuis un portefeuille qui ne finance pas le budget, la dépense est **hors budget** : elle est marquée en rose dans les listes, et le formulaire de dépense le signale avant l'enregistrement.
- La carte du budget affiche ce qui reste dans l'enveloppe. Elle devient rouge quand l'enveloppe est vide et que des dépenses continuent hors budget.
- Le même budget est réservé automatiquement chaque mois. Au début du mois suivant, l'accueil demande où va le reste : **le rendre aux portefeuilles** (il redevient libre) ou **le mettre dans un objectif** (il sort des portefeuilles et s'ajoute à l'objectif).
- Exemple : 75 000 DA en espèces, 35 000 DA réservés, un achat de 2 000 DA → il reste 73 000 DA dans le portefeuille, dont 40 000 DA libres et 33 000 DA dans l'enveloppe.

Les réglages de l'enveloppe sont stockés avec les budgets (clés `fund:<portefeuille>`, `closed:<mois>`, `envelope:since`). Un budget créé avant les enveloppes fonctionne comme avant (toutes les dépenses comptent) jusqu'à ce que l'utilisateur choisisse ses portefeuilles. Calculs dans `lib/services/envelope_math.dart`, testés dans `test/envelope_test.dart`.

## Supprimer un portefeuille

On choisit toujours le portefeuille qui reçoit son argent (rien n'est présélectionné). Ce qui restait arrive dans ce portefeuille, converti au taux des portefeuilles si les devises diffèrent ; ses dépenses y sont déplacées (totaux et historique conservés) sans être retirées une deuxième fois, et le solde des autres portefeuilles ne change pas. Sa part du budget passe au même portefeuille. Le seul portefeuille restant ne peut pas être supprimé.

## Langues

Le français est la langue source : chaque texte affiché passe par `tr('Texte en français', [valeurs])`, et ses traductions sont dans `lib/l10n/en.dart` et `lib/l10n/ar.dart`, indexées par le texte français. Les valeurs remplacent `{0}`, `{1}`… pour que chaque langue les place où sa grammaire le veut. Un texte sans traduction s'affiche en français.

- Arabe : interface de droite à gauche (marges et alignements en `start` / `end`, jauge du mois remplie depuis la droite), mois du Maghreb (جانفي، فيفري…) et chiffres latins ; les montants restent lisibles (« 35 000 DA »).
- Nombres : 35 000,50 en français et en arabe, 35,000.50 en anglais.
- Les noms saisis (portefeuilles, catégories personnelles, notes) restent tels qu'écrits ; les catégories de base, types de portefeuille et devises sont traduits.
- **Ajouter un texte** : l'écrire en français dans `tr(...)`, puis ajouter sa traduction dans `en.dart` et `ar.dart`. Le test `test/l10n_test.dart` échoue si un texte n'est pas traduit.

## L'innovation : le moteur de conseils

Tout est dans [lib/services/insights_service.dart](lib/services/insights_service.dart), calculé sur le téléphone (aucune donnée envoyée).

1. **Dépense inhabituelle** : comparaison avec l'historique de la catégorie par la médiane et l'écart absolu médian (MAD). Signalée au-delà de médiane + 3 écarts robustes et d'au moins 2× la médiane. Ces statistiques robustes ne sont pas faussées par d'anciennes dépenses exceptionnelles.
2. **Rythme du budget** : projection de fin de mois (coûts fixes comptés une fois, dépenses variables extrapolées) ; alerte *avant* le dépassement avec un montant maximum par jour.
3. **Tendance par catégorie** : hausse de 30 % ou plus par rapport à la moyenne des mois précédents (ou félicitations en cas de baisse).
4. **Objectifs** : épargne mensuelle nécessaire pour tenir l'échéance, détection du retard.
5. **Concentration** : un poste variable qui pèse lourd, avec l'économie possible en le réduisant de 10 %.
6. **Suggestion de budget** à partir des mois passés si aucun budget n'est défini.

## Comptes, données et abonnement

L'application a deux modes, choisis automatiquement au démarrage :

- **Mode en ligne** (dès que Firebase est configuré, voir plus bas) : comptes Google, Apple ou e-mail avec **Firebase Authentication**, données dans **Cloud Firestore**. Firestore garde une copie sur le téléphone : l'app marche hors connexion et synchronise ensuite. Les règles de [firestore.rules](firestore.rules) font que chacun ne lit que ses propres données.
- **Mode local** (tant que Firebase n'est pas configuré) : comptes et données sur le téléphone (SQLite), mot de passe haché (SHA-256 salé et itéré). Les boutons Google et Apple affichent qu'ils ne sont pas encore activés.

### Formules

| | Starter (gratuit) | Pro | Premium |
|---|---|---|---|
| Portefeuilles | 1 (espèces ou banque) | 2 (espèces et banque) | Illimités, tous types, noms libres |
| Portefeuilles en devises | Non | Non | Oui |
| Catégories personnalisées | Non | Oui | Oui |
| Objectifs d'épargne | 2 | Illimités | Illimités |
| Budgets par catégorie | 3 | Illimités | Illimités |
| Conseils et alertes | Oui | Oui | Oui |

**Solde des portefeuilles** : le solde est suivi à partir du premier ajout d'argent (ou du premier « Indiquer le solde ») : solde = ajouts + transferts reçus − transferts envoyés − retraits − dépenses depuis cette date. Un **retrait** sort l'argent du portefeuille sans compter comme dépense (mis de côté, prêté, déposé ailleurs). Les dépenses plus anciennes ne le faussent pas. Les transferts ne comptent ni comme revenu ni comme dépense. Calcul dans [lib/services/wallet_math.dart](lib/services/wallet_math.dart).

**Devises des portefeuilles** : chaque portefeuille en devise a un taux saisi par l'utilisateur (le taux auquel il change réellement son argent, souvent différent du taux officiel). Une dépense de 20 € est enregistrée avec son montant d'origine **et** sa valeur convertie au taux du jour : totaux, budgets, graphiques et conseils restent dans la devise principale, et un changement de taux ne modifie pas les dépenses passées.

Tout se règle dans [lib/models/plan.dart](lib/models/plan.dart).

**Fin d'abonnement** : une résiliation reste active jusqu'à la fin de la période payée (règle Google Play / App Store). Exemple : payé le 15/10/2026 pour un mois puis résilié, la formule reste active jusqu'au 15/11/2026. Le profil affiche alors « Pro jusqu'au 15/11/2026 ». Ensuite, ce qui dépasse la nouvelle formule est **gelé**, jamais supprimé :

| Élément | Ce qui reste actif | Gelé = |
|---|---|---|
| Portefeuilles | le portefeuille principal (le premier créé, marqué « Principal »), toujours ; en Pro, aussi le plus ancien espèces/banque dans la devise principale | solde et historique visibles ; plus de dépenses, ajouts, retraits ni transferts ; suppression possible |
| Catégories personnalisées | aucune en Starter | plus proposées pour les nouvelles dépenses ; les anciennes dépenses les gardent |
| Objectifs | les plus anciens, jusqu'à la limite | progression visible ; plus d'ajout ni de retrait |
| Budgets par catégorie | les premiers dans l'ordre des catégories, jusqu'à la limite | affichés avec un cadenas, ignorés par les alertes |

Se réabonner dégèle tout immédiatement. Règles dans `AppState` (section « freeze »), testées dans `test/freeze_test.dart`.

L'abonnement passe par **RevenueCat** (Google Play et App Store). Sans clés RevenueCat, l'écran des formules indique que les abonnements ne sont pas encore disponibles ; en version de développement, un bouton « Essayer Pro / Premium (mode test) » permet de tester les fonctions payantes.

## Mise en ligne

### 1. Firebase (comptes et base de données)

1. Sur [console.firebase.google.com](https://console.firebase.google.com) : créer un projet **EcoBudget** (Google Analytics facultatif).
2. **Authentication → Commencer → Mode de connexion** : activer **Google**, **E-mail/Mot de passe** et **Apple**.
3. **Firestore Database → Créer une base de données** (mode production, région `europe-west`), puis onglet **Règles** : coller le contenu de [firestore.rules](firestore.rules) et **Publier**.
4. **Paramètres du projet → Ajouter une application → Android** : nom du package `com.ecobudget.eco_budget`, et l'empreinte **SHA-1** de chaque ordinateur qui compile l'app :
   ```
   keytool -list -v -keystore %USERPROFILE%\.android\debug.keystore -alias androiddebugkey -storepass android
   ```
   (sans SHA-1, la connexion Google échoue sur Android). Ajouter plus tard le SHA-1 de la clé de publication Google Play.
5. Dans un terminal, à la racine du projet :
   ```
   firebase login
   flutterfire configure --project=<id-du-projet> --platforms=android,ios
   ```
   Cela remplace `lib/firebase_options.dart` et ajoute `android/app/google-services.json`. L'app passe alors en mode en ligne.

### 2. Apple (iPhone)

Compte Apple Developer (99 $/an) et un Mac avec Xcode : ouvrir `ios/Runner.xcworkspace`, onglet **Signing & Capabilities**, ajouter **Sign in with Apple**. Relancer `flutterfire configure` avec `ios` si l'app iOS n'avait pas été ajoutée.

### 3. Abonnements (RevenueCat)

1. Créer les abonnements dans **Google Play Console** (compte 25 $, application publiée au moins en test interne) et dans **App Store Connect**.
2. Sur [revenuecat.com](https://www.revenuecat.com) : créer un projet et relier les deux boutiques, puis :
   - deux **entitlements** : `pro` et `premium` ; les produits Premium débloquent **les deux** ;
   - deux **offerings** : `pro` (produits Pro mensuel / annuel) et `premium` (produits Premium mensuel / annuel).
3. Copier les clés publiques (`goog_…`, `appl_…`) dans [lib/config.dart](lib/config.dart).

## Design

Système défini dans [lib/theme/app_theme.dart](lib/theme/app_theme.dart) :

- **Couleurs** : bleu `#1B3FA0` (marque), jaune `#FFC727` (argent dépensé, bouton + et boutons de l'introduction), fond `#EEF1F8`, corail `#D9483A` (dépassement). Mode sombre inclus.
- **Typographies** (incluses dans l'app, licence OFL) : *Bricolage Grotesque* pour les montants et titres, *Instrument Sans* pour le texte.
- **Élément signature** : la jauge du mois sur l'accueil, avec un trait par jour. Les traits se colorent à mesure que le budget est utilisé : si la couleur dépasse le point « aujourd'hui », on dépense trop vite.
- Listes groupées, pas d'ombres, barre du bas avec bouton + central.

## Structure

```
lib/
  main.dart                       démarrage, thème, connexion → application
  theme/app_theme.dart            couleurs, typographies, composants
  models/                         Expense, Deposit, Wallet, ExpenseCategory, SavingsGoal, Plan, Insight, AppUser, Currency
  config.dart                     clés RevenueCat, limites de la version gratuite
  firebase_options.dart           généré par flutterfire configure
  data/repository.dart            interface de stockage + version SQLite
  data/firestore_repository.dart  version en ligne (Firestore)
  data/database.dart              SQLite local (v5 : portefeuilles, devises, catégories, ajouts d'argent), données séparées par compte
  data/demo_data.dart             données de démonstration
  services/insights_service.dart  moteur de conseils
  services/auth/                  comptes : interface, Firebase (Google, Apple, e-mail), local
  services/subscription_service.dart  abonnement via RevenueCat
  state/session_state.dart        utilisateur connecté
  state/app_state.dart            dépenses, budgets, objectifs de l'utilisateur
  state/plan_state.dart           gratuit ou Premium, limites
  screens/                        introduction, connexion, formules, portefeuilles, catégories, accueil, dépenses, formulaire, budgets, objectifs, conseils, profil
  widgets/                        jauge du mois, graphiques, listes groupées, sélecteur de devise
test/                             tests du moteur de conseils, des devises, des portefeuilles en devises, des mots de passe et des formules
tool/render_icon_test.dart        génère l'icône de l'application
```

## Publier (version de test)

La version publiée reste en **mode test** : tant que RevenueCat n'est pas configuré, l'écran des formules propose « Essayer Pro / Premium (mode test) », gratuitement et sans paiement. Pour la vraie version publique, compiler avec `--dart-define=EB_TEST_MODE=false` (voir `lib/config.dart`).

### Android

Fichiers prêts dans `release/` : `EcoBudget-1.0.0-test.apk` (installation directe) et `EcoBudget-1.0.0-test.aab` (Google Play). Pour les refaire : `flutter build apk --release` et `flutter build appbundle --release`.

**Clé d'envoi** : `android/app/upload-keystore.jks` et son mot de passe dans `android/key.properties`. Les sauvegarder ailleurs (clé USB, coffre de mots de passe) : sans elles, impossible de publier une mise à jour sous la même application. Ces fichiers ne doivent jamais être envoyés sur GitHub (déjà dans `.gitignore`).

Deux façons de donner un lien de téléchargement :

1. **Google Play, test interne** (conseillé) : compte Google Play Console (25 $, une fois) → Créer une application → *Tester → Test interne* → importer le `.aab` → ajouter les e-mails des testeurs → copier le **lien de participation**. Les testeurs installent depuis Play Store et reçoivent les mises à jour. Jusqu'à 100 testeurs, sans examen long.
2. **Lien direct vers l'APK** : déposer `EcoBudget-1.0.0-test.apk` sur Google Drive, GitHub Releases ou un site, et partager le lien. Le testeur doit autoriser « Installer des applications inconnues ». Les mises à jour se font à la main.

### iOS (TestFlight)

Il faut un Mac pour compiler l'app iOS. Sans Mac, `codemagic.yaml` la fait compiler par Codemagic (Mac dans le cloud) et l'envoie directement sur TestFlight.

1. **Apple Developer Program** (99 $/an) sur developer.apple.com.
2. **Identifiant** : *Certificates, Identifiers & Profiles → Identifiers* → App ID `com.ecobudget.ecoBudget`.
3. **App Store Connect** → *Apps → +* → nouvelle app iOS avec cet identifiant (nom : EcoBudget, ou un autre s'il est pris).
4. **Clé API** : App Store Connect → *Utilisateurs et accès → Intégrations → App Store Connect API* → nouvelle clé (rôle *App Manager*) ; noter l'Issuer ID, le Key ID et télécharger le fichier `.p8`.
5. **Codemagic** (codemagic.io, offre gratuite) : connecter le dépôt Git du projet, puis *Team → Integrations → App Store Connect* avec la clé ci-dessus, nommée exactement `EcoBudget App Store Connect`.
6. Lancer le workflow **iOS → TestFlight**. Le build arrive dans App Store Connect → *TestFlight* : ajouter des testeurs internes (immédiat), ou un groupe externe avec un **lien public** (après une courte vérification d'Apple).

Avec un Mac : ouvrir `ios/Runner.xcworkspace` dans Xcode, choisir l'équipe dans *Signing & Capabilities*, puis *Product → Archive → Distribute App → App Store Connect*.

Tant que Firebase n'est pas configuré, les comptes sont enregistrés sur le téléphone (mode local) et la connexion Google n'est pas active : c'est normal pour la version de test. Une fois Firebase créé, ajouter dans Firebase l'empreinte SHA-1 de la clé d'envoi Android (`44:3F:7D:5F:DD:74:13:61:25:93:61:A7:4B:A3:12:6C:8F:E6:2B:AB`) et celle de la clé de signature Google Play (*Play Console → Intégrité de l'application*).

## Lancer le projet

```
flutter pub get
flutter emulators --launch Pixel_Phone   # ou brancher un téléphone Android
flutter run
flutter test                             # tests unitaires
flutter build apk                        # APK Android
```

Icône de l'application (après modification du logo) :

```
flutter test tool/render_icon_test.dart
dart run flutter_launcher_icons
```

**iOS** : le même code fonctionne sur iPhone, mais la compilation iOS nécessite un Mac avec Xcode (ou un service cloud comme Codemagic) : `flutter build ios`.
