# Laravel + React + Vite + Tailwind Starter Kit

Starter kit pour démarrer rapidement un projet **Laravel (API)** + **React (SPA)** + **Vite** + **TailwindCSS** avec une structure propre et un environnement de dev confortable.

## 🚀 Stack

- **Backend** : Laravel
- **Frontend** : React + Vite
- **Style** : TailwindCSS
- **Auth** : Laravel Breeze / Sanctum (selon config)
- **Qualité de code** : Laravel Pint, ESLint, Prettier
- **Dev env** : VS Code

---

## ✅ Prérequis

- PHP 8.2+  
- Composer  
- Node.js (LTS 18 ou 20) + npm ou pnpm  
- MySQL / MariaDB / PostgreSQL (ou Docker avec Laravel Sail)
- Git

---

## 🔧 Installation

```bash
# 1. Cloner le repo
git clone https://github.com/Sud0ck3rs/laravel-starter-kit.git
cd laravel-starter-kit

# 2. Installer les dépendances PHP
composer install

# 3. Installer les dépendances JS
npm install

# 4. Copie du fichier d'env
cp .env.example .env

# 5. Générer la clé d'application
php artisan key:generate
```

Configure ta base de données dans .env :

```js
DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=ton_projet
DB_USERNAME=root
DB_PASSWORD=
```

Puis lance les migrations :

```bash
php artisan migrate
```

Si tu utilises **Laravel Sail** : remplace php par `./vendor/bin/sail` dans les commandes.

## ▶️ Lancer le projet en développement

Serveur Laravel

```bash
php artisan serve
# ou avec Sail
./vendor/bin/sail up
```

Vite (React + Tailwind)

```bash
npm un dev
```

Accède ensuite à l’URL Laravel (ex: http://127.0.0.1:8000).

## 🔧 Créer un projet

```bash
composer create-project laravel/laravel mon-projet
cd mon-projet
cp .env.example .env
php artisan key:generate
```

Configure ta DB dans `.env`, puis :

```bash
php artisan migrate
```

---

## 1) Auth de base : Laravel Breeze (simple et moderne)

Breeze = starter auth officiel, minimal, parfait pour builder dessus.

```bash
composer require laravel/breeze --dev
php artisan breeze:install
# Choisis l’option blade ou inertial+react selon ton goût
npm install
npm run dev
php artisan migrate
```

Tu récupères :

- inscription / login / reset password
- gestion du profil de base
- pages déjà stylées avec Tailwind
- routes et contrôleurs auth propres

Si tu veux vraiment 0 front Blade (parce que tu gères tout en React à part), tu peux ignorer les vues Blade de Breeze, mais tu gardes la logique auth.


## 2) Organisation `starter` de ton code Laravel

Voici une structure propre côté back que tu peux adopter d’office :

```bash
app/
├── Http/
│   ├── Controllers/
│   │   ├── Web/
│   │   │   └── HomeController.php
│   │   └── Api/
│   │       └── V1/
│   │           ├── Auth/
│   │           │   └── MeController.php
│   │           └── Users/
│   │               └── UserController.php
│   ├── Middleware/
│   └── Requests/
│       └── User/
│           ├── StoreUserRequest.php
│           └── UpdateUserRequest.php
├── Models/
│   └── User.php
├── Services/
│   └── UserService.php
└── ...
```

L’idée :

- `App\Http\Controllers\Web\...` → tout ce qui rend des vues Blade (ou la SPA React).
- `App\Http\Controllers\Api\V1\...` → tes endpoints d’API versionnés.
- `App\Http\Requests\...` → Form Requests pour validation propre.
- `App\Services\...` → logique métier un peu avancée, réutilisable.

## 3) Routes de base (Web + API)

### 3.1. Routes web “starter”

`routes/web.php`
```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Web\HomeController;

Route::get('/', [HomeController::class, 'index'])->name('home');

Route::middleware(['auth', 'verified'])->group(function () {
    Route::get('/dashboard', [HomeController::class, 'dashboard'])->name('dashboard');
});
```

`app/Http/Controllers/Web/HomeController.php`

```php
namespace App\Http\Controllers\Web;

use App\Http\Controllers\Controller;

class HomeController extends Controller
{
    public function index()
    {
        return view('welcome'); // ou ta vue SPA
    }

    public function dashboard()
    {
        return view('dashboard');
    }
}
```

### 3.2. Routes API versionnées

`routes/api.php`

```php
use Illuminate\Support\Facades\Route;
use App\Http\Controllers\Api\V1\Users\UserController;
use App\Http\Controllers\Api\V1\Auth\MeController;

Route::prefix('v1')->group(function () {

    Route::middleware('auth:sanctum')->group(function () {
        Route::get('/me', MeController::class)->name('api.v1.me');
        Route::apiResource('users', UserController::class)->only(['index', 'show', 'update', 'destroy']);
    });

    // endpoints publics
    Route::get('/status', fn () => ['status' => 'ok']);
});
```

## 4. Auth pour API : Sanctum

Pour un starter kit moderne, Sanctum est parfait pour API + SPA.

```bash
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

Dans `app/Http/Kernel.php`, vérifie que le middleware `EnsureFrontendRequestsAreStateful` est bien là pour les SPA, et ajoute `Sanctum` dans api guard si nécessaire.

Dans `config/sanctum.php`, tu peux définir ton domaine front / SPA plus tard.

Tu peux ensuite faire un petit contrôleur `MeController` :

```php
namespace App\Http\Controllers\Api\V1\Auth;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class MeController extends Controller
{
    public function __invoke(Request $request)
    {
        return $request->user();
    }
}
```

## 5. Validation propre avec Form Requests

Exemple de Form Request pour créer un utilisateur :

`app/Http/Requests/User/StoreUserRequest.php`

```php
namespace App\Http\Requests\User;

use Illuminate\Foundation\Http\FormRequest;

class StoreUserRequest extends FormRequest
{
    public function authorize(): bool
    {
        // à adapter selon ta politique d'auth
        return true;
    }

    public function rules(): array
    {
        return [
            'name' => ['required', 'string', 'max:255'],
            'email' => ['required', 'email', 'max:255', 'unique:users,email'],
            'password' => ['required', 'string', 'min:8'],
        ];
    }
}
```

Dans ton contrôleur API :

```php
use App\Http\Requests\User\StoreUserRequest;

public function store(StoreUserRequest $request)
{
    $data = $request->validated();

    $user = User::create([
        'name' => $data['name'],
        'email' => $data['email'],
        'password' => bcrypt($data['password']),
    ]);

    return response()->json($user, 201);
}
```

## 6. Qualité de code : Laravel Pint + tests

### 6.1. Laravel Pint

```bash
composer require laravel/pint --dev
```

Tu peux lancer :

```bash
./vendor/bin/pint
```

Et ajouter dans `composer.json` :

```json
"scripts": {
  "pint": "pint"
}
```

## 6.2. Tests de base

Créons un test d’API simple :

```bash
php artisan make:test ApiStatusTest
```

`tests/Feature/ApiStatusTest.php`

```php
namespace Tests\Feature;

use Tests\TestCase;

class ApiStatusTest extends TestCase
{
    /** @test */
    public function it_returns_ok_status()
    {
        $this->getJson('/api/v1/status')
            ->assertOk()
            ->assertJson(['status' => 'ok']);
    }
}
```

Lance :
```bash
php artisan test
```

## 7) Config front minimal côté Laravel

Même si tu fais React à côté, je te conseille :

- une vue `resources/views/app.blade.php` qui sert ta SPA
- une route catch-all qui renvoie cette vue

`resources/views/app.blade.php` (exemple minimal) :

```html
<!DOCTYPE html>
<html lang="{{ str_replace('_', '-', app()->getLocale()) }}">
<head>
    <meta charset="utf-8">
    <title>{{ config('app.name') }}</title>
    @vite(['resources/css/app.css', 'resources/js/app.jsx'])
</head>
<body class="antialiased">
    <div id="app"></div>
</body>
</html>
```

`routes/web.php` (ajout SPA) :

```php
Route::view('/app/{any?}', 'app')->where('any', '.*')->name('spa');
```

## Cheat Sheet CLI Laravel

### Gestion du projet

```bash
# Créer un nouveau projet Laravel
composer create-project laravel/laravel mon-projet

# Installer les dépendances PHP
composer install

# Installer les dépendances JS
npm install

# Copier le fichier d'environnement
cp .env.example .env

# Générer la clé d'application
php artisan key:generate
```

### Serveur & environnement

```bash
# Lancer le serveur de dev (par défaut : http://127.0.0.1:8000)
php artisan serve

# Afficher la version de Laravel
php artisan --version

# Lancer Tinker (console interactive PHP/Laravel)
php artisan tinker
```

### Base de données & migrations

```bash
# Créer une migration
php artisan make:migration create_users_table

# Lancer toutes les migrations
php artisan migrate

# Annuler la dernière migration
php artisan migrate:rollback

# Annuler toutes les migrations
php artisan migrate:reset

# Drop + recréer la base (reset complet)
php artisan migrate:fresh

# Drop + recréer + seed
php artisan migrate:fresh --seed

# Lancer les seeders
php artisan db:seed

# Lancer un seeder spécifique
php artisan db:seed --class=UserSeeder
```

### Models, Controllers, Requests…

Models

```bash
# Créer un modèle simple
php artisan make:model Post

# Modèle + migration
php artisan make:model Post -m

# Modèle + migration + factory + seeder
php artisan make:model Post -mfs
```

Controllers

```bash
# Controller simple
php artisan make:controller PostController

# Resource controller (index, show, create, store, edit, update, destroy)
php artisan make:controller PostController --resource

# API resource controller (sans create/edit)
php artisan make:controller Api/PostController --api
```

Form Requests

```bash
# Créer une Form Request
php artisan make:request StorePostRequest
```

Autres classes utiles

```bash
# Middleware
php artisan make:middleware EnsureUserIsAdmin

# Policy
php artisan make:policy PostPolicy --model=Post

# Service Provider
php artisan make:provider AppServiceProvider

# Event
php artisan make:event UserRegistered

# Listener
php artisan make:listener SendWelcomeEmail --event=UserRegistered

# Job
php artisan make:job SendEmailJob
```

### Vue / Blade / Front

```bash
# Créer un component Blade
php artisan make:component Alert
# => resources/views/components/alert.blade.php
# => app/View/Components/Alert.php
```

Si tu utilises Livewire, Inertia, etc., ils ont aussi leurs propres commandes (php artisan make:livewire, etc.).

### Auth & sécurité

```bash
# Installer Laravel Breeze (starter auth)
composer require laravel/breeze --dev
php artisan breeze:install

# Installer Laravel UI (sur anciens projets)
composer require laravel/ui
php artisan ui bootstrap --auth

# Installer Laravel Sanctum (auth pour SPA / API)
composer require laravel/sanctum
php artisan vendor:publish --provider="Laravel\Sanctum\SanctumServiceProvider"
php artisan migrate
```

### Tests

```bash
# Lancer tous les tests
php artisan test

# Créer un test Feature
php artisan make:test UserTest

# Créer un test Unit
php artisan make:test UserTest --unit
```

### Cache / config / optimisations

```bash
# Vider le cache applicatif
php artisan cache:clear

# Vider le cache de config
php artisan config:clear

# Vider le cache des routes
php artisan route:clear

# Vider le cache des vues compilées
php artisan view:clear

# Vider tout (pratique en dev)
php artisan optimize:clear

# Générer le cache de config (prod)
php artisan config:cache

# Générer le cache des routes (prod)
php artisan route:cache

# Optimiser l'autoload (prod)
composer dump-autoload -o
```

### Storage & fichiers
```bash
# Créer le lien symbolique storage -> public/storage
php artisan storage:link
```

### Jobs & queues

```bash
# Lancer le worker de queue
php artisan queue:work

# Lancer le worker avec une queue spécifique
php artisan queue:work --queue=emails

# Voir les jobs en échec
php artisan queue:failed

# Retenter un job en échec
php artisan queue:retry {id}

# Effacer les jobs en échec
php artisan queue:flush
```

```bash
# Lister toutes les routes
php artisan route:list

### Routes

# Lister les routes en filtrant par nom / middleware
php artisan route:list --name=api
php artisan route:list --middleware=auth
```

### Logs & debug

```bash
# Checker les logs (fichier)
tail -f storage/logs/laravel.log

# Créer un command artisan perso
php artisan make:command CleanOldLogs
```

### Divers utiles

```bash
# Générer un lien de reset password (user...)
php artisan tinker

# Dans tinker :
>>> $user = App\Models\User::first();
>>> Password::createToken($user);

# Envoyer une notification de test
php artisan tinker
>>> $user->notify(new \App\Notifications\TestNotification);
```