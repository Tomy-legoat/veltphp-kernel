# Issue 10 - Intégration Preview dans le Kernel VeltPHP

## Objectif

Intégrer le module Preview dans le kernel VeltPHP pour permettre l'utilisation des fonctionnalités de preview (sessions, parser AST, endpoints API) directement depuis le kernel via des contrats et services.

## Contexte

Le module Preview est déjà développé et fonctionnel dans `velt-preview/`. Il contient :
- Génération de sessions preview
- Parser de fichiers `.velt` vers AST
- API REST endpoints
- Génération QR code
- Intégration complète des 7 sous-modules

## Tâches à accomplir

### 1. Créer le contrat PreviewServiceInterface

**Fichier** : `packages/kernel/src/Contracts/PreviewServiceInterface.php`

```php
<?php

namespace Velt\Kernel\Contracts;

interface PreviewServiceInterface
{
    public function createSession(string $view): array;
    public function getPreviewData(string $sessionId): ?array;
    public function getSession(string $sessionId): ?array;
    public function deleteSession(string $sessionId): bool;
}
```

### 2. Créer le PreviewServiceProvider

**Fichier** : `packages/kernel/src/PreviewServiceProvider.php`

Le service provider doit :
- Étendre la classe `ServiceProvider` du kernel
- Enregistrer les services Preview dans le container
- Configurer les paths (templates, storage, base_url)
- Créer les répertoires nécessaires au boot
- Publier la configuration par défaut

**Services à enregistrer** :
- `preview.templates_path`
- `preview.storage_path`
- `preview.base_url`
- `preview.parser` (VeltParser)
- `preview.page_repository` (VeltPageRepository)
- `preview.session_store` (PreviewSessionStore)
- `preview.controller` (PreviewController)
- `preview.url_generator` (PreviewUrlGenerator)
- `preview.qr_generator` (QRGenerator)
- `PreviewServiceInterface`

### 3. Créer le PreviewService

**Fichier** : `packages/kernel/src/PreviewService.php`

Implémentation de `PreviewServiceInterface` qui utilise les services Preview enregistrés dans le container.

### 4. Ajouter les dépendances Preview au composer.json du kernel

Ajouter les modules Preview comme dépendances dans le `composer.json` du kernel :
- velt/preview-session-store
- velt/preview-contracts
- velt/preview-endpoints
- velt/preview-qr-cli
- velt/velt-ast
- velt/velt-parser
- velt/velt-view

### 5. Mettre à jour la documentation

Créer ou mettre à jour la documentation d'intégration dans `docs/`.

## Architecture cible

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

## Dépendances Preview

Les modules Preview sont situés dans `../velt-preview/` avec les dépendances centralisées :

- **composer.json centralisé** dans `velt-preview/`
- **Autoloading PSR-4** configuré pour tous les modules
- **Un seul vendor/** partagé

## Exemple d'utilisation

Une fois l'intégration terminée, les développeurs pourront utiliser :

```php
use Velt\Kernel\Contracts\PreviewServiceInterface;

class MyController
{
    public function __construct(
        private PreviewServiceInterface $preview
    ) {}

    public function createPreview()
    {
        $session = $this->preview->createSession('auth.login');
        return [
            'id' => $session['id'],
            'url' => $session['url'],
            'qr_payload' => $session['qrPayload']
        ];
    }
}
```

## Documentation de référence

Pour plus de détails sur le module Preview lui-même, voir :
- `../velt-preview/PREVIEW_DOCUMENTATION.md` - Documentation complète du module Preview
- `../velt-preview/README.md` - Vue d'ensemble du module Preview

## Critères d'acceptation

- [ ] PreviewServiceInterface créé avec les 4 méthodes requises
- [ ] PreviewServiceProvider créé et étend ServiceProvider du kernel
- [ ] Tous les services Preview enregistrés dans le container
- [ ] PreviewService implémente correctement l'interface
- [ ] Les dépendances Preview ajoutées au composer.json du kernel
- [ ] La documentation d'intégration créée/mise à jour
- [ ] Tests unitaires pour PreviewService
- [ ] Tests d'intégration avec le container
