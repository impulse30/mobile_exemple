# Tutoriel : Reconstruire le Template Mobile de Zéro

Ce guide vous montrera comment recréer l'architecture de ce template en partant d'un projet Flutter vide.

## Phase 1: Initialisation du Projet

### 1. Créer le projet Flutter
Ouvrez votre terminal et exécutez la commande suivante :
```bash
flutter create mobile_template_from_scratch
cd mobile_template_from_scratch
```

### 2. Ajouter les Dépendances
Ouvrez le fichier `pubspec.yaml` et ajoutez les dépendances suivantes :

```yaml
dependencies:
  flutter:
    sdk: flutter

  # Icons
  cupertino_icons: ^1.0.8

  # State Management
  flutter_riverpod: ^2.6.1

  # Dependency Injection
  get_it: ^8.0.3

  # Routing
  go_router: ^14.1.3

  # Network
  http: ^1.2.2

  # Environment Variables
  flutter_dotenv: ^5.2.1

  # Local Storage
  get_storage: ^2.1.1

  # Other utilities
  cached_network_image: ^3.4.1
  intl: ^0.19.0
  uuid: ^4.4.0
  path: ^1.9.1

dev_dependencies:
  flutter_test:
    sdk: flutter
  flutter_lints: ^5.0.0
```
*N.B. : Les versions peuvent être mises à jour. Utilisez les dernières versions compatibles.*

### 3. Installer les Dépendances
Dans votre terminal, exécutez :
```bash
flutter pub get
```

---

## Phase 2: Structure des Dossiers

Dans le dossier `lib/`, créez l'arborescence suivante pour organiser votre code selon les principes de la Clean Architecture :

```
lib/
├── business/
│   ├── models/
│   └── services/
├── framework/
│   ├── services/
│   └── utils/
├── pages/
├── utils/
└── main.dart (existant)
```
- **`business`**: Le cœur de votre application (logique métier pure, sans dépendances Flutter).
- **`framework`**: L'implémentation des services (appels API, base de données locale, etc.).
- **`pages`**: L'interface utilisateur (widgets, écrans).
- **`utils`**: Petits helpers et utilitaires transverses.

---

## Phase 3: Configuration de l'Environnement (.env)

### 1. Créer les fichiers .env
À la racine de votre projet, créez deux fichiers :
- `.env.example` (pour le versionnement)
- `.env` (pour vos variables locales, ce fichier doit être dans `.gitignore`)

Dans `.env.example`, mettez :
```
BASE_URL=http://your.api.url/api
```
Copiez ce contenu dans `.env` et ajustez l'URL.

### 2. Configurer `pubspec.yaml`
Pour que votre application puisse accéder au fichier `.env`, déclarez-le comme un "asset" dans `pubspec.yaml`:
```yaml
flutter:
  uses-material-design: true
  assets:
    - .env
```

### 3. Charger les variables dans `main.dart`
Modifiez votre `lib/main.dart` pour charger les variables au démarrage :
```dart
import 'package:flutter/material.dart';
import 'package:flutter_dotenv/flutter_dotenv.dart';

void main() async {
  // Assurer l'initialisation des bindings
  WidgetsFlutterBinding.ensureInitialized();

  // Charger les variables d'environnement
  await dotenv.load(fileName: ".env");

  runApp(const MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      home: Scaffold(
        body: Center(
          child: Text('Projet Initialisé!'),
        ),
      ),
    );
  }
}
```

---

## Phase 4: Création des Utilitaires (HttpUtils)

### 1. Créer l'Interface (Abstraction)
Créez le fichier `lib/utils/http_utils.dart` :
```dart
// lib/utils/http_utils.dart
abstract class HttpUtils {
  Future<dynamic> getData(String url);
  Future<dynamic> postData(String url, {Map<String, dynamic>? body});
}
```

### 2. Créer l'Implémentation
Créez le fichier `lib/framework/utils/remote_http_utils.dart` :
```dart
// lib/framework/utils/remote_http_utils.dart
import 'dart:convert';
import 'package:http/http.dart' as http;
import '../../utils/http_utils.dart';

class RemoteHttpUtils implements HttpUtils {
  @override
  Future<dynamic> getData(String url) async {
    try {
      final response = await http.get(Uri.parse(url));
      if (response.statusCode == 200) {
        return jsonDecode(response.body);
      } else {
        throw Exception('Failed to load data');
      }
    } catch (e) {
      throw Exception('Failed to connect to the server');
    }
  }

  @override
  Future<dynamic> postData(String url, {Map<String, dynamic>? body}) async {
    // Implémentation similaire pour POST
  }
}
```

---

## Phase 5: Injection de Dépendances (get_it)

### 1. Configurer `get_it`
Dans `lib/main.dart`, nous allons configurer un "service locator".

```dart
// ... autres imports
import 'package:get_it/get_it.dart';
import 'utils/http_utils.dart';
import 'framework/utils/remote_http_utils.dart';

// Créer une instance globale de GetIt
final getIt = GetIt.instance;

void setupLocator() {
  // Enregistrez vos services ici
  getIt.registerLazySingleton<HttpUtils>(() => RemoteHttpUtils());
  // ... enregistrez d'autres services de la même manière
}

void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await dotenv.load(fileName: ".env");

  // Appeler la configuration du locator
  setupLocator();

  runApp(const MyApp());
}
// ...
```

Maintenant, chaque fois que vous aurez besoin de `HttpUtils` dans votre code, vous pourrez l'obtenir via `getIt<HttpUtils>()`.

---

## Phase 6: Stockage Local (get_storage)

### 1. Initialiser GetStorage
Dans `lib/main.dart` :
```dart
// ... autres imports
import 'package:get_storage/get_storage.dart';

void main() async {
  // ...
  await GetStorage.init(); // Initialise le stockage
  setupLocator();
  runApp(const MyApp());
}
```

### 2. Utilisation
Vous pouvez maintenant utiliser `GetStorage` n'importe où pour lire/écrire des données simples.
```dart
final box = GetStorage();
box.write('user_token', 'votre_token_ici');
String token = box.read('user_token');
```

---

## Phase 7: Navigation (go_router)

### 1. Créer le fichier de routes
Créez `lib/router.dart` :
```dart
// lib/router.dart
import 'package:go_router/go_router.dart';
import 'package:flutter/material.dart';
import 'pages/home_page.dart'; // Créez cette page simple

final GoRouter router = GoRouter(
  routes: <RouteBase>[
    GoRoute(
      path: '/',
      builder: (BuildContext context, GoRouterState state) {
        return const HomePage();
      },
    ),
  ],
);
```
Créez une page `lib/pages/home_page.dart` basique pour que cela fonctionne.

### 2. Connecter le routeur à l'application
Modifiez `lib/main.dart` pour utiliser `MaterialApp.router` :
```dart
// ...
import 'router.dart';

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp.router(
      routerConfig: router,
      title: 'Mon App',
    );
  }
}
```

---

## Phase 8: Gestion d'État (Riverpod)

### 1. Ajouter le `ProviderScope`
Le `ProviderScope` est le widget qui stocke l'état de tous vos providers. Entourez votre `MyApp` avec lui dans `lib/main.dart` :
```dart
// ...
import 'package:flutter_riverpod/flutter_riverpod.dart';

void main() async {
  // ...
  runApp(
    ProviderScope(
      child: const MyApp(),
    ),
  );
}
```

### 2. Créer un Provider
Exemple simple : un provider qui fournit une valeur.
```dart
// Dans un fichier de providers, ex: lib/providers.dart
import 'package:flutter_riverpod/flutter_riverpod.dart';

final greetingProvider = Provider<String>((ref) {
  return 'Hello Riverpod!';
});
```

### 3. Utiliser le Provider dans l'UI
Modifiez votre `HomePage` pour qu'elle devienne un `ConsumerWidget` :
```dart
// lib/pages/home_page.dart
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import '../providers.dart'; // Importez votre fichier de providers

class HomePage extends ConsumerWidget {
  const HomePage({super.key});

  @override
  Widget build(BuildContext context, WidgetRef ref) {
    // "watch" le provider pour obtenir sa valeur
    final String greeting = ref.watch(greetingProvider);

    return Scaffold(
      appBar: AppBar(title: const Text('Accueil')),
      body: Center(
        child: Text(greeting),
      ),
    );
  }
}
```

---

## Conclusion

Vous avez maintenant une base de projet solide avec :
- Une architecture propre.
- L'injection de dépendances.
- La gestion d'état.
- La navigation.
- La gestion des variables d'environnement.
- Un utilitaire pour les requêtes HTTP.

À partir de là, vous pouvez suivre le `Guide d'implémentation` du `README.md` principal pour construire vos fonctionnalités.
