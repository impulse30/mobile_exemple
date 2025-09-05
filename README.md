# ODC Mobile Template

Ce projet est un template pour démarrer des applications mobiles Flutter en suivant une architecture claire et maintenable. Il inclut une configuration de base pour la gestion des dépendances, la gestion d'état, la navigation, et plus encore.

## Table des matières
- [Démarrage rapide](#démarrage-rapide)
  - [Prérequis](#prérequis)
  - [Clonage du dépôt](#clonage-du-dépôt)
  - [Configuration du projet](#configuration-du-projet)
- [Lancement du projet](#lancement-du-projet)
- [Dépendances principales](#dépendances-principales)
- [Architecture du projet](#architecture-du-projet)
- [Utilitaires clés](#utilitaires-clés)
- [Guide d'implémentation](#guide-dimplémentation)
  - [1. Créer un modèle](#1-créer-un-modèle)
  - [2. Créer un service abstrait](#2-créer-un-service-abstrait)
  - [3. Implémenter le service](#3-implémenter-le-service)
  - [4. Écrire des tests](#4-écrire-des-tests)
  - [5. Créer les pages de l'interface utilisateur](#5-créer-les-pages-de-linterface-utilisateur)
  - [6. Configurer les routes](#6-configurer-les-routes)
- [Générer une version de production](#générer-une-version-de-production)
- [Lignes directrices pour le développement](#lignes-directrices-pour-le-développement)

---

## Démarrage rapide

### Prérequis
Assurez-vous d'avoir installé le [SDK Flutter](https://flutter.dev/docs/get-started/install) sur votre machine.

### Clonage du dépôt
```bash
# Clonez le dépôt en utilisant l'URL HTTPS ou SSH
git clone <URL_DU_DEPOT> mon_projet_mobile

# Accédez au répertoire du projet
cd mon_projet_mobile
```
⚠️ **Important** : La branche de travail principale est `dev`. Veuillez créer vos branches de fonctionnalités à partir de celle-ci.

### Configuration du projet

#### 1. Installer les dépendances
```bash
flutter pub get
```

#### 2. Configurer les variables d'environnement
Le projet utilise un fichier `.env` pour gérer les variables d'environnement.

1.  Dupliquez le fichier `.env.example` et renommez-le en `.env`.
    ```bash
    cp .env.example .env
    ```
2.  Modifiez le fichier `.env` pour y ajouter vos configurations, comme l'URL de base de votre API.
    ```
    BASE_URL=http://VOTRE_IP_LOCALE:8000/api
    ```

---

## Lancement du projet

Pour lancer l'application en mode debug :
```bash
flutter run
```

Pour lister les appareils disponibles et lancer sur un appareil spécifique :
```bash
flutter devices
flutter run -d <device_id>
```

---

## Dépendances principales

Ce projet utilise plusieurs bibliothèques clés pour fonctionner. Voici un aperçu des plus importantes :

| Dépendance | Utilisation |
| --- | --- |
| **`flutter_riverpod`** | Une solution de gestion d'état réactive et robuste. |
| **`go_router`** | Un routeur déclaratif pour gérer la navigation de manière simple et prévisible. |
| **`get_it`** | Un localisateur de services pour l'injection de dépendances, facilitant l'accès aux services. |
| **`http`** | Le package standard pour effectuer des requêtes HTTP vers une API. |
| **`get_storage`** | Une solution de stockage clé-valeur légère et rapide pour la persistance locale. |
| **`flutter_dotenv`** | Pour charger les variables d'environnement à partir d'un fichier `.env`. |
| **`cached_network_image`**| Pour afficher et mettre en cache des images provenant d'Internet. |
| **`intl`** | Utilisé pour l'internationalisation et la localisation (i18n). |
| **`uuid`** | Pour générer des identifiants uniques universels (UUID). |

---

## Architecture du projet

L'architecture du projet s'inspire de la Clean Architecture pour séparer les responsabilités et garantir un code découplé et testable.

```
lib/
├── business/      # Couche métier (indépendante du framework)
│   ├── models/    # Modèles de données (objets purs)
│   └── services/  # Interfaces (contrats) des services
├── framework/     # Couche d'infrastructure (implémentations)
│   ├── .../       # Implémentations des services (API, BDD locale)
│   └── utils/     # Utilitaires liés au framework
├── pages/         # Couche de présentation (UI)
│   ├── .../       # Widgets, états et contrôleurs (Riverpod)
├── utils/         # Utilitaires transverses
├── main.dart      # Point d'entrée et injection des dépendances
└── routers.dart   # Configuration de la navigation
```

Le flux de dépendances est le suivant : **Pages -> Services (abstraits) <- Implémentations**.

1.  **`business` (Couche métier)** : Cœur de l'application. Elle contient les modèles de données (ex: `Article`) et les contrats de service (ex: `ArticleService`). Cette couche ne dépend d'aucune autre.
2.  **`framework` (Couche d'infrastructure)** : Contient les implémentations concrètes des services définis dans la couche `business`. Par exemple, `ArticleServiceImpl` implémente `ArticleService` en utilisant le package `http` pour communiquer avec une API. C'est ici que se trouvent les dépendances externes (BDD, réseau, etc.).
3.  **`pages` (Couche de présentation)** : Contient l'interface utilisateur. Les widgets et contrôleurs de cette couche dépendent des **interfaces** de service de la couche `business`, mais ignorent tout des implémentations de la couche `framework`.
4.  **Injection de dépendances (`main.dart`)** : Le fichier `main.dart` utilise `get_it` pour lier les abstractions (`ArticleService`) à leurs implémentations concrètes (`ArticleServiceImpl`). Cela permet de remplacer facilement une implémentation par une autre (par exemple, pour les tests).

---

## Utilitaires clés

Le répertoire `utils` (à la racine et dans `framework`) contient des classes d'aide réutilisables.

### `lib/utils/http/HttpUtils.dart`
C'est une **interface** (classe abstraite) qui définit les méthodes nécessaires pour effectuer des requêtes HTTP (`getData`, `postData`, etc.). L'application dépend de cette abstraction, pas d'une implémentation concrète.

### `lib/framework/utils/http/remoteHttpUtils.dart`
C'est l'**implémentation** concrète de `HttpUtils`. Elle utilise le package `http` pour effectuer les appels réseau.
- **Gestion des erreurs** : Elle encapsule les réponses HTTP et lève une `HttpRequestException` personnalisée en cas d'erreur.
- **Mocking local** : Si une URL commence par `local://`, elle charge un fichier JSON depuis `assets/fake/` au lieu d'effectuer un appel réseau. C'est très utile pour le développement et les tests en mode déconnecté.

---

## Guide d'implémentation

Suivez ces étapes pour ajouter une nouvelle fonctionnalité.

#### 1. Créer un modèle
Dans `lib/business/models`, créez un fichier pour votre modèle de données.

#### 2. Créer un service abstrait
Dans `lib/business/services`, définissez l'interface pour votre service.

#### 3. Implémenter le service
Dans `lib/framework`, écrivez l'implémentation concrète du service.

#### 4. Écrire des tests
Créez des tests unitaires pour vos implémentations de service.

#### 5. Créer les pages de l'interface utilisateur
Pour chaque nouvelle page, créez un dossier dans `lib/pages` contenant l'état, le contrôleur et la vue.

#### 6. Configurer les routes
Ajoutez votre nouvelle page dans le fichier `lib/routers.dart`.

---

## Générer une version de production

### Android (APK)
1.  Mettez à jour le code de version dans `pubspec.yaml` (`version: 1.0.0+1`).
2.  Générez l'APK :
    ```bash
    flutter build apk --release
    ```
L'APK sera disponible dans `build/app/outputs/flutter-apk/app-release.apk`.

Pour une configuration de signature avancée, suivez le [guide officiel de Flutter](https://docs.flutter.dev/deployment/android).

---

## Lignes directrices pour le développement

- **Branche `dev`** : Utilisez cette branche pour le développement principal.
- **Branches de fonctionnalités** : Créez toujours une nouvelle branche à partir de `dev` pour chaque nouvelle fonctionnalité.
- **Tests** : Assurez-vous que votre code est testé avant de le merger.
- **Respect de l'architecture** : Suivez la structure du projet pour maintenir la cohérence.
