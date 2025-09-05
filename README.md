# ODC Mobile Template

Ce projet est un template pour démarrer des applications mobiles Flutter en suivant une architecture claire et maintenable. Il inclut une configuration de base pour la gestion des dépendances, la gestion d'état, la navigation, et plus encore.

## Table des matières
- [Démarrage rapide](#démarrage-rapide)
  - [Prérequis](#prérequis)
  - [Clonage du dépôt](#clonage-du-dépôt)
  - [Configuration du projet](#configuration-du-projet)
- [Lancement du projet](#lancement-du-projet)
- [Dépendances principales](#dépendances-principales)
- [Structure du projet](#structure-du-projet)
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

## Structure du projet

L'architecture du projet est conçue pour séparer les préoccupations et faciliter la maintenance.

```
lib/
├── business/
│   ├── models/    # Modèles de données (objets métier)
│   └── services/  # Services abstraits (interfaces/contrats)
├── framework/     # Implémentations concrètes des services
├── pages/         # Couche de présentation (UI)
├── utils/         # Fonctions et classes utilitaires
├── main.dart      # Point d'entrée de l'application
└── routers.dart   # Configuration de la navigation et des routes
```

---

## Guide d'implémentation

Suivez ces étapes pour ajouter une nouvelle fonctionnalité.

#### 1. Créer un modèle
Dans `lib/business/models`, créez un fichier pour votre modèle de données.
```dart
// lib/business/models/article/article_model.dart
class Article {
  final String title;
  // ... autres champs

  Article({required this.title, /* ... */});

  factory Article.fromJson(Map<String, dynamic> json) {
    return Article(title: json['title'], /* ... */);
  }

  Map<String, dynamic> toJson() {
    return {'title': title, /* ... */};
  }
}
```

#### 2. Créer un service abstrait
Dans `lib/business/services`, définissez l'interface pour votre service.
```dart
// lib/business/services/article/article_service.dart
import '../../models/article/article_model.dart';

abstract class ArticleService {
  Future<List<Article>> getAllArticles();
  // ... autres méthodes
}
```

#### 3. Implémenter le service
Dans `lib/framework`, écrivez l'implémentation concrète du service.
```dart
// lib/framework/article/article_service_impl.dart
import 'package:http/http.dart' as http;
// ... autres imports

class ArticleServiceImpl implements ArticleService {
  final String baseUrl = Env.baseUrl;

  @override
  Future<List<Article>> getAllArticles() async {
    // Logique de récupération des données (ex: appel HTTP)
  }
}
```

#### 4. Écrire des tests
Créez des tests unitaires pour vos implémentations de service.
```dart
// test/article_service_test.dart
import 'package:flutter_test/flutter_test.dart';
// ...

void main() {
  // Vos tests ici
}
```

#### 5. Créer les pages de l'interface utilisateur
Pour chaque nouvelle page, il est recommandé de créer un dossier dans `lib/pages` contenant :
*   Un fichier pour l'**état** (State) géré par Riverpod.
*   Un fichier pour le **contrôleur** (Controller/StateNotifier).
*   Un fichier pour la **vue** (la page Flutter elle-même).

#### 6. Configurer les routes
Ajoutez votre nouvelle page dans le fichier `lib/routers.dart`.
```dart
// ...
GoRoute(
  path: "/app/articles",
  name: 'articles_page',
  builder: (ctx, state) {
    return ArticlesPage();
  },
),
// ...
```

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
