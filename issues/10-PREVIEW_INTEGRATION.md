# Intégration Preview dans le Kernel VeltPHP

## Vue d'ensemble

Le module Preview est maintenant intégré au kernel VeltPHP via le `PreviewServiceProvider`. Cette intégration permet d'utiliser les fonctionnalités de preview (sessions, parser AST, endpoints API) directement depuis le kernel.

## Architecture

```
Kernel VeltPHP
├── PreviewServiceProvider
│   ├── Enregistre les services Preview dans le container
│   ├── Configure les paths (templates, storage)
│   └── Publie la configuration
├── PreviewService (contrat kernel)
│   ├── createSession()
│   ├── getPreviewData()
│   ├── getSession()
│   └── deleteSession()
└── Modules Preview externes
    ├── velt-ast (AST nodes)
    ├── velt-parser (Parser .velt)
    ├── velt-view (VeltView + Repository)
    ├── preview-session-store
    ├── preview-endpoints
    └── preview-qr-cli
```

## Installation

### 1. Ajouter les dépendances

Dans le `composer.json` de votre application :

```json
{
    "repositories": [
        {
            "type": "path",
            "url": "../velt-preview/preview-session-store"
        },
        {
            "type": "path",
            "url": "../velt-preview/preview-contracts"
        },
        {
            "type": "path",
            "url": "../velt-preview/preview-endpoints"
        },
        {
            "type": "path",
            "url": "../velt-preview/preview-qr-cli"
        },
        {
            "type": "path",
            "url": "../velt-preview/velt-ast"
        },
        {
            "type": "path",
            "url": "../velt-preview/velt-parser"
        },
        {
            "type": "path",
            "url": "../velt-preview/velt-view"
        }
    ],
    "require": {
        "velt/preview-session-store": "*",
        "velt/preview-contracts": "*",
        "velt/preview-endpoints": "*",
        "velt/preview-qr-cli": "*",
        "velt/velt-ast": "*",
        "velt/velt-parser": "*",
        "velt/velt-view": "*"
    }
}
```

### 2. Enregistrer le ServiceProvider

Dans le bootstrap de votre application :

```php
use Velt\Kernel\Application;
use Velt\Kernel\PreviewServiceProvider;

$app = new Application(__DIR__);
$app->registerProvider(new PreviewServiceProvider($app));
$app->boot();
```

### 3. Configurer les paths

Créer le fichier `config/preview.php` (généré automatiquement au premier boot) :

```php
<?php

use Velt\Kernel\Facades\App;

return [
    'templates_path' => App::basePath() . '/resources/views',
    'storage_path' => App::basePath() . '/storage/preview',
    'base_url' => env('PREVIEW_BASE_URL', 'http://127.0.0.1:8000'),
    'session_ttl' => env('PREVIEW_SESSION_TTL', 3600),
];
```

## Utilisation

### Via le contrat PreviewServiceInterface

```php
use Velt\Kernel\Contracts\PreviewServiceInterface;

class MyController
{
    public function __construct(
        private PreviewServiceInterface $preview
    ) {}

    public function createPreview()
    {
        // Créer une session preview
        $session = $this->preview->createSession('auth.login');
        
        return [
            'id' => $session['id'],
            'url' => $session['url'],
            'qr_payload' => $session['qrPayload']
        ];
    }

    public function getPreview(string $sessionId)
    {
        // Récupérer les données JSON de preview
        $data = $this->preview->getPreviewData($sessionId);
        
        return $data;
    }
}
```

### Via le container directement

```php
$generator = $app->container()->get('preview.url_generator');
$session = $generator->createForView('auth.login');

$controller = $app->container()->get('preview.controller');
$response = $controller->preview($session['id']);
```

## Services enregistrés

Le `PreviewServiceProvider` enregistre les services suivants dans le container :

| Service | Description |
|---------|-------------|
| `preview.templates_path` | Chemin vers les templates .velt |
| `preview.storage_path` | Chemin vers le stockage des sessions |
| `preview.base_url` | URL de base pour les endpoints preview |
| `preview.parser` | Instance de VeltParser |
| `preview.page_repository` | Instance de VeltPageRepository |
| `preview.session_store` | Instance de PreviewSessionStore |
| `preview.controller` | Instance de PreviewController |
| `preview.url_generator` | Instance de PreviewUrlGenerator |
| `PreviewServiceInterface` | Service Preview du kernel |

## Format des fichiers .velt

Les templates Velt utilisent une syntaxe simple basée sur l'indentation :

```velt
VStack class="flex-1 p-4"
  Text value="Se connecter" class="text-2xl font-bold mb-4"
  Input name="email" label="Email" type="email" class="mb-4"
  Input name="password" label="Mot de passe" type="password" class="mb-4"
  Button text="Connexion" class="bg-blue-500 text-white"
```

## Endpoints HTTP

Si vous utilisez le module HTTP du kernel, les endpoints suivants sont disponibles :

- `GET /api/preview/{id}` - Récupérer le JSON de preview
- `GET /api/session/{id}` - Récupérer les infos de session

## CLI

Le CLI preview est disponible via :

```bash
php bin/velt preview auth.login
```

## Tests

Pour tester l'intégration :

```php
use Velt\Kernel\Contracts\PreviewServiceInterface;

test('preview service creates session', function () {
    $preview = app(PreviewServiceInterface::class);
    $session = $preview->createSession('auth.login');
    
    expect($session)->toHaveKey('id');
    expect($session)->toHaveKey('url');
});
```
