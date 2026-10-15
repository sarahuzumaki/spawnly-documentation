# Documentation technique — InfoLeader

Analyse du code présent dans le dossier `C:\wamp64\www\infoleader` (dépôt Git, commit `d065bba` du 13/10/2026).

**Lecture des sources.** Chaque affirmation est suivie de sa source entre parenthèses ou dans une colonne « Source ».

- Un **chemin** (ex. `app/Services/AuthService.php`) désigne un fichier du dépôt.
- **Environnement** désigne une information lue sur le poste WAMP (version, réglage de `php.ini`). Elle n'est pas dans le dépôt.
- **Historique Git** désigne une information tirée de la commande `git log`.
- **Absent** signale qu'une information ou une fonctionnalité n'existe pas dans le code.

## Table des matières

1. [Présentation générale du site](#1-présentation-générale-du-site)
2. [Stack technique](#2-stack-technique)
3. [Arborescence du projet](#3-arborescence-du-projet)
4. [Points d'entrée et routage](#4-points-dentrée-et-routage)
5. [Stockage des données](#5-stockage-des-données)
6. [Authentification, sessions et sécurité](#6-authentification-sessions-et-sécurité)
7. [Front-end](#7-front-end)
8. [Intégrations externes](#8-intégrations-externes)
9. [Configuration](#9-configuration)
10. [Limites connues et risques](#10-limites-connues-et-risques)

---

## 1. Présentation générale du site

### 1.1 À quoi sert le site

InfoLeader est le site vitrine d'une entreprise fictive qui vend des périphériques informatiques
(`README.md`, `resources/views/pages/home.php`).
Il comprend un site public et un panneau d'administration réservé à un administrateur unique
(`routes/web.php`, `app/Models/AdminAccount.php`).

### 1.2 Fonctionnalités du site public

| Page | Adresse | Fonctionnalités | Source |
|---|---|---|---|
| Accueil | `/` | Bannière, slogan, bouton vers les produits, présentation, vignettes de catégories, 3 engagements, bandeau vers le contact | `resources/views/pages/home.php` |
| Nos Produits | `/produits` | 6 produits en grille (image, catégorie, nom, description, prix), recherche en temps réel, compteur de résultats | `resources/views/pages/products.php`, `resources/data/products.json`, `public/js/script.js` |
| Contact | `/contact` | Formulaire nom / email / message ; validation en JavaScript puis en PHP ; aucun e-mail envoyé ; message enregistré | `public/js/script.js` (`validateForm`), `app/Http/Controllers/ContactController.php`, `helpers.php` (`journal_contact`) |

### 1.3 Fonctionnalités du panneau d'administration

| Fonction | Adresse | Source |
|---|---|---|
| Création du compte au premier lancement | `/admin/installation` | `app/Http/Controllers/Admin/AuthController.php` (`setup`) |
| Connexion avec anti-brute force | `/admin/connexion` | `AuthController.php` (`login`), `app/Services/RateLimiter.php` |
| Mot de passe oublié (code à 8 chiffres) | `/admin/mot-de-passe-oublie`, `/admin/reinitialisation` | `app/Http/Controllers/Admin/PasswordResetController.php` |
| Tableau de bord : 4 outils et informations du compte | `/admin` | `app/Http/Controllers/Admin/DashboardController.php`, `resources/views/admin/dashboard.php` |
| Configuration des e-mails : PHP `mail()` ou Brevo, test d'envoi | `/admin/emails` | `app/Http/Controllers/Admin/EmailController.php` |
| Mot de passe | `/admin/mot-de-passe` | `app/Http/Controllers/Admin/PasswordController.php` |
| Utilisateur : adresse de connexion | `/admin/utilisateur` | `app/Http/Controllers/Admin/UserController.php` |
| Maintenance : Système, Journaux, Sauvegarde, Purger `storage/tmp` | `/admin/maintenance` | `app/Http/Controllers/Admin/MaintenanceController.php` |

### 1.4 Historique du projet

| Étape | Contenu | Source |
|---|---|---|
| TP 1 et TP 2 | Site vitrine, puis panneau d'administration | Historique Git (commit `bd33603`), `README.md` |
| Sécurité du dépôt | Blocage de l'accès web au dossier `.git` | Historique Git (commit `fb1f466`) |
| TP 3, atelier 1 | Réorganisation en MVC avec un framework « maison » | Historique Git (commits `63ebf0a`, `d065bba`), `README.md` |

### 1.5 Points forts

- **Architecture lisible** : un seul point d'entrée, contrôleurs courts, logique métier dans des services
  (`public/index.php`, `app/Http/Controllers/`, `app/Services/`).
- **Panel protégé** : empreintes de mots de passe, clé SMTP chiffrée, jetons CSRF, anti-brute force,
  sessions liées au compte (`AuthService.php`, `Encrypter.php`, `VerifyCsrfToken.php`, `RateLimiter.php`).
- **Retours clairs** : messages de succès et d'erreur, fenêtre de confirmation avant chaque action irréversible
  (`resources/views/admin/partials/alert.php`, `resources/views/admin/partials/confirm-dialog.php`).
- **Non-régression** : les anciennes adresses redirigent vers les nouvelles (`app/Http/Controllers/LegacyController.php`).
- **Une seule bibliothèque**, sans gestionnaire de paquets : PHPMailer (`vendor/phpmailer/`, aucun `composer.json`).
- **Code commenté en français** et réglages regroupés dans `config/`.

## 2. Stack technique

| Élément | Valeur | Source |
|---|---|---|
| Environnement | WAMP sous Windows | `README.md` |
| Serveur web | Apache. Le module `mod_rewrite` est nécessaire | `.htaccess`, `public/.htaccess` (`<IfModule mod_rewrite.c>`) |
| Version d'Apache | 2.4.65 | Environnement (section Système de la Maintenance, `$_SERVER['SERVER_SOFTWARE']`) |
| Langage serveur | PHP | `public/index.php` et tous les fichiers `.php` |
| Version de PHP | 8.3.28 | Environnement |
| Version minimale | Le code utilise des nouveautés de PHP 8.0 : `match`, types `mixed` et union, propriétés de constructeur promues, `str_starts_with`. La Maintenance exige PHP 8.1 ou plus | `core/*.php`, `app/Services/MaintenanceService.php` (`version_compare`) |
| Extensions PHP | `openssl`, `mbstring`, `json`, `session` | `app/Services/Encrypter.php` (`openssl_encrypt`), `app/Services/AuthService.php` (`mb_strtolower`), `app/Http/Middleware/StartSession.php` |
| Framework | Framework « maison » inspiré de Laravel, écrit dans le projet | `core/` |
| Front-end | HTML5, CSS3, JavaScript sans bibliothèque | `resources/views/`, `public/css/`, `public/js/` |
| Bibliothèque | PHPMailer 7.1.1, copiée dans `vendor/` sans Composer | `vendor/phpmailer/phpmailer/VERSION` ; absent : `composer.json` |
| Stockage | Fichiers JSON et texte dans `storage/` | `app/Services/JsonStore.php` |
| Base de données | **Absent.** Aucune connexion PDO ni requête SQL dans le code | Recherche de `PDO`, `SELECT`, `INSERT INTO` dans `app/` et `core/` : aucun résultat |
| Versionnage | Git | `.gitignore`, historique Git |

## 3. Arborescence du projet

Source : liste des fichiers suivis par Git (`git ls-files`).

```
infoleader/
├── .htaccess              Toutes les requêtes vers public/ ; HTTPS forcé hors local
├── .gitignore             Exclut storage/ sauf storage/.htaccess
├── README.md              Présentation, architecture, installation
├── helpers.php            Fonctions globales : app(), env(), config(), url(), asset(), route(), view(),
│                          redirect(), e(), client_ip(), format_date(), csrf_token(), csrf_field(),
│                          flash(), take_flashes(), journal_contact()
├── bootstrap/app.php      Chargement automatique des classes, création de l'Application, providers
├── core/                  Noyau du framework
│   ├── Application.php    Chemins, chargement de config/, providers, run()
│   ├── Container.php      Conteneur de services : fabrique les objets et leurs dépendances
│   ├── Facade.php         Accès statique à un service (Route::get)
│   ├── Request.php        Requête HTTP : chemin, champs, fichiers, IP, chemin de base
│   ├── Response.php       Réponses HTML, JSON, redirection, téléchargement
│   ├── Route.php          Une route : méthodes, adresse, action, middlewares, nom
│   ├── Router.php         Groupes, middlewares, recherche de la route, génération d'URL
│   ├── ServiceProvider.php Classe de base des providers
│   └── View.php           Moteur de templates : @extends, @section, @yield, @include, @if, @foreach, {{ }}
├── app/
│   ├── Facades/Route.php  Façade du routeur
│   ├── Http/Kernel.php    Middlewares globaux, alias, page d'erreur 500
│   ├── Http/Controllers/  Controller (base), Home, Product, Contact, Legacy
│   │   └── Admin/         Auth, PasswordReset, Dashboard, Email, Password, User, Maintenance
│   ├── Http/Middleware/   SecurityHeaders, AdminHeaders, StartSession, VerifyCsrfToken,
│   │                      Authenticate, RedirectIfAuthenticated, ThrottleLogin
│   ├── Models/            AdminAccount, Message, Product
│   ├── Providers/         AppServiceProvider, RouteServiceProvider, ViewServiceProvider
│   └── Services/          AuthService, RateLimiter, PasswordResetService, MailConfig, Mailer,
│                          Encrypter, EventLog, JsonStore, MaintenanceService
├── config/                app.php, admin.php, mail.php, rate.php, maintenance.php
├── routes/web.php         Toutes les routes
├── resources/
│   ├── data/products.json Catalogue des produits
│   └── views/
│       ├── layouts/       site.php, admin.php, auth.php
│       ├── partials/      site-header.php, site-footer.php
│       ├── pages/         home.php, products.php, contact.php
│       ├── admin/         Pages du panel ; partials/ : alertes, fenêtres, pagination, règles...
│       └── errors/        404.php, 405.php, 500.php
├── public/                Seul dossier servi au navigateur
│   ├── index.php          Point d'entrée unique
│   ├── .htaccess          Sert les fichiers réels, envoie le reste à index.php
│   ├── css/               style.css (site), admin.css (panel)
│   ├── js/                script.js (site), admin.js (panel)
│   └── img/               banner.jpg, products/*.jpg
├── storage/.htaccess      Seul fichier versionné de storage/ (données exclues par .gitignore)
└── vendor/phpmailer/      PHPMailer : src/, language/phpmailer.lang-fr.php, LICENSE, VERSION
```

Les dossiers `app`, `bootstrap`, `config`, `core`, `resources`, `routes`, `storage` et `vendor`
contiennent chacun un `.htaccess` « `Require all denied` » (fichiers `<dossier>/.htaccess`).

## 4. Points d'entrée et routage

### 4.1 Parcours d'une requête

1. Le `.htaccess` racine réécrit toute adresse vers `public/` (`.htaccess` : `RewriteRule ^(?!public/)(.*)$ public/$1`).
2. Les fichiers existants (CSS, JS, images) sont servis tels quels. Le reste va à `index.php` (`public/.htaccess`).
3. `public/index.php` charge `bootstrap/app.php` puis appelle `$app->run()` (`public/index.php`, `core/Application.php`).
4. Le middleware global `SecurityHeaders` s'applique (`app/Http/Kernel.php`).
5. Le routeur cherche la route, exécute ses middlewares puis l'action du contrôleur (`core/Router.php` : `dispatch`, `runRoute`, `pipeline`).
6. Le contrôleur appelle modèles et services, puis renvoie une vue (`helpers.php` : `view()` ; `core/View.php`).

Le chemin de base est déduit de `SCRIPT_NAME` (`core/Request.php` : `basePath()`).
Le site fonctionne donc aussi bien à l'adresse `http://localhost/infoleader` qu'à `http://infoleader.local` (`README.md`).

### 4.2 Routes

Source : `routes/web.php`.

| Méthode | Adresse | Contrôleur et méthode | Middlewares |
|---|---|---|---|
| GET | `/` | `HomeController@index` | — |
| GET | `/produits` | `ProductController@index` | — |
| GET, POST | `/contact` | `ContactController@index` | — |
| GET, POST | `/admin/installation` | `Admin\AuthController@setup` | admin, session, csrf |
| GET | `/admin/connexion` | `Admin\AuthController@showLogin` | admin, session, csrf, guest |
| POST | `/admin/connexion` | `Admin\AuthController@login` | admin, session, csrf, throttle |
| POST | `/admin/mot-de-passe-oublie` | `Admin\PasswordResetController@send` | admin, session, csrf, throttle |
| GET, POST | `/admin/reinitialisation` | `Admin\PasswordResetController@reset` | admin, session, csrf, throttle |
| POST | `/admin/deconnexion` | `Admin\AuthController@logout` | admin, session, csrf |
| GET | `/admin` | `Admin\DashboardController@index` | admin, session, csrf, auth |
| GET, POST | `/admin/emails` | `Admin\EmailController@index` | admin, session, csrf, auth |
| GET, POST | `/admin/mot-de-passe` | `Admin\PasswordController@index` | admin, session, csrf, auth |
| GET, POST | `/admin/utilisateur` | `Admin\UserController@index` | admin, session, csrf, auth |
| GET | `/admin/maintenance` | `Admin\MaintenanceController@index` | admin, session, csrf, auth |
| POST | `/admin/maintenance` | `Admin\MaintenanceController@action` | admin, session, csrf, auth |

### 4.3 Middlewares

Source : `app/Http/Kernel.php` et `app/Http/Middleware/`.

| Alias | Classe | Rôle |
|---|---|---|
| (global) | `SecurityHeaders` | Ajoute les en-têtes de sécurité à toutes les réponses |
| `admin` | `AdminHeaders` | Interdit la mise en cache et l'affichage du panel dans un cadre |
| `session` | `StartSession` | Démarre la session sécurisée |
| `csrf` | `VerifyCsrfToken` | Refuse un formulaire POST sans jeton valide (statut 400) |
| `auth` | `Authenticate` | Sans connexion valide : redirection vers `/admin/connexion` |
| `guest` | `RedirectIfAuthenticated` | Administrateur déjà connecté : redirection vers `/admin` |
| `throttle` | `ThrottleLogin` | Adresse IP bloquée : formulaire refusé, retour à la connexion |

### 4.4 Cas particuliers

- **Anciennes adresses** (`index.php`, `products.php`, `contact.php`, `admin/index.php`, `admin/setup.php`, `admin/reset.php`,
  `admin/panel.php`, `admin/email.php`, `admin/password.php`, `admin/user.php`, `admin/maintenance.php`) :
  redirection permanente (301), paramètres conservés (`routes/web.php`, `app/Http/Controllers/LegacyController.php`).
- **Adresse inconnue** : vue `errors/404`, statut 404 (`core/Router.php`).
- **Méthode non prévue** : vue `errors/405`, statut 405 (`core/Router.php`).
- **Erreur imprévue** : enregistrée dans `storage/logs/app.log`, vue `errors/500` ;
  le détail n'est affiché que si `debug` vaut `true` (`app/Http/Kernel.php` : `renderException`).

## 5. Stockage des données

### 5.1 Base de données

**Absent.** Le site n'utilise aucune base de données : pas de tables, de relations ni de requêtes SQL
(aucune occurrence de `PDO`, `SELECT` ou `INSERT INTO` dans `app/` et `core/`).
Les appels `$request->query(...)` des contrôleurs lisent des paramètres d'URL, pas une base (`core/Request.php` : `query`).

### 5.2 Fichiers de données

Les données sont enregistrées dans des fichiers du dossier `storage/`, exclu du dépôt (`.gitignore`) et protégé (`storage/.htaccess`).

| Fichier | Format | Contenu | Source |
|---|---|---|---|
| `storage/config/admin.json` | JSON | Email, empreinte du mot de passe, dates de création, de connexion (et IP), de connexion précédente (et IP), de changement du mot de passe, de changement de l'email | `app/Models/AdminAccount.php`, `app/Services/AuthService.php` (`login`), `app/Http/Controllers/Admin/UserController.php` |
| `storage/config/mail.json` | JSON | Mode actif (`php` ou `brevo`) ; pour chaque mode : expéditeur, destinataire ; pour Brevo : serveur, port, sécurité, identifiant, clé SMTP **chiffrée**, débogage ; dates d'enregistrement | `app/Services/MailConfig.php` (`validateBrevo`, `save`) |
| `storage/config/app.key` | Texte (base64) | Clé de chiffrement de 32 octets, créée au premier besoin | `app/Services/Encrypter.php` |
| `storage/security/ratelimit.json` | JSON | Par adresse IP : nombre d'échecs, date du dernier échec, prochain essai autorisé, fin de blocage | `app/Services/RateLimiter.php` |
| `storage/security/reset.json` | JSON | Empreinte du code, dates de création et d'expiration, nombre d'essais, adresse IP | `app/Services/PasswordResetService.php` (`create`) |
| `storage/logs/auth.log` | Une ligne JSON par événement | Date, type, adresse IP, détail | `app/Services/EventLog.php` (`log`) |
| `storage/logs/contact.log` | Une ligne JSON par message | Date, IP, nom, email, message | `app/Models/Message.php` (`create`) |
| `storage/logs/mail.log` | Texte | Mails non envoyés ou en échec ; copie et trace SMTP si le débogage est actif | `app/Services/Mailer.php` (`log`, `send`) |
| `storage/logs/app.log` | Texte | Exceptions non prévues | `app/Http/Kernel.php` (`renderException`) |
| `storage/framework/views/` | Fichiers PHP | Vues compilées (cache recréé automatiquement) | `core/View.php` (`compiled`), `app/Providers/ViewServiceProvider.php` |
| `storage/tmp/` | — | Fichiers temporaires, vidés par « Purger storage/tmp » | `config/maintenance.php`, `app/Services/MaintenanceService.php` (`purgeTmp`) |
| `resources/data/products.json` | JSON, versionné | 6 produits : `name`, `category`, `price` en texte (ex. `"149,99 €"`), `description`, `image` | `resources/data/products.json`, `app/Models/Product.php` (lecture seule) |

### 5.3 Écriture et accès

- **Verrou** : `JsonStore::update()` pose un verrou exclusif (`flock`), lit, modifie puis réécrit (`app/Services/JsonStore.php`).
- **Journaux** : ajout de lignes avec `FILE_APPEND | LOCK_EX` (`JsonStore::appendLine()`).
- **Accès aux données** : les contrôleurs ne lisent ni n'écrivent de fichier eux-mêmes.
  Ils passent par les modèles et les services (`app/Http/Controllers/` : aucun appel à `file_put_contents`, `fopen` ou `$_POST`).
- **Sessions PHP** : leur emplacement n'est pas défini dans le code (absent de `app/Http/Middleware/StartSession.php`).
  Le réglage de WAMP s'applique : `session.save_path = "c:/wamp64/tmp"` (Environnement : `phpForApache.ini`).

## 6. Authentification, sessions et sécurité

### 6.1 Compte administrateur

- **Création** : via `/admin/installation`, depuis `127.0.0.1` ou `::1` uniquement, tant qu'aucun compte n'existe
  (`AuthController.php` : `setup`).
- **Mot de passe** : stocké sous forme d'empreinte avec `password_hash()`, vérifié avec `password_verify()` (`AuthService.php`).
- **Règles** : 20 caractères minimum, une minuscule, une majuscule, un chiffre, un caractère spécial
  (`AuthService.php` : `passwordRules`, `passwordErrors` ; `config/admin.php`).
  Elles sont cochées en direct pendant la saisie (`public/js/admin.js`).

### 6.2 Connexion et anti-brute force

- **Message unique** en cas d'erreur : « Identifiant ou mot de passe incorrect » (`AuthController.php` : `login`).
- **Pas d'indice** : le mot de passe est toujours vérifié, même si l'email est faux (`AuthService.php` : `checkCredentials`).
- **Attente progressive** : 3, 3, 3, 10, 30, 60 puis 300 secondes après chaque échec (`config/rate.php` : `delays`).
- **Blocage** : au 8e échec, l'adresse IP est bloquée 14 400 s, soit 4 h (`config/rate.php`, `RateLimiter.php` : `fail`).
- **Oubli** : compteur effacé après 86 400 s sans échec (`config/rate.php`), et après une connexion réussie (`RateLimiter.php` : `clear`).
- **Périmètre** : connexion (`AuthController.php`), code de réinitialisation (`PasswordResetController.php`),
  mot de passe actuel des modules (`AuthService.php` : `verifyCurrentPassword`).
- **IP bloquée** : tout formulaire d'authentification est refusé (`app/Http/Middleware/ThrottleLogin.php`).

### 6.3 Mot de passe oublié

- Code de 8 chiffres tiré avec `random_int` (`PasswordResetService.php` : `create`).
- Valable 300 s, 5 essais au maximum, un envoi toutes les 60 s (`config/admin.php` : `reset`).
- Seule l'empreinte du code est stockée (`PasswordResetService.php`).
- Envoyé au destinataire défini dans « Configuration des e-mails », sinon à l'email du compte (`Mailer.php` : `recipient`).

### 6.4 Sessions

- **Cookie** `IL_ADMIN`, `HttpOnly`, `SameSite=Strict`, `Secure` en HTTPS (`config/admin.php`, `StartSession.php`).
- **Mode strict** et cookies seuls : `session.use_strict_mode`, `session.use_only_cookies` (`StartSession.php`).
- **Nouvel identifiant** à la connexion : `session_regenerate_id(true)` (`AuthService.php` : `login`).
- **Expiration** : 1 800 s d'inactivité ou 28 800 s au total (`config/admin.php`, `AuthService.php` : `check`).
- **Lien avec le compte** : la session garde une empreinte du compte et devient invalide s'il change (`AuthService.php` : `fingerprint`, `check`).
- **Autres sessions** déconnectées après un changement de mot de passe ou d'email (`AuthService.php` : `refreshSession`).

### 6.5 Autres protections

| Protection | Détail | Source |
|---|---|---|
| CSRF | Jeton secret dans chaque formulaire du panel, vérifié sur chaque POST | `helpers.php` (`csrf_token`, `csrf_field`), `app/Http/Middleware/VerifyCsrfToken.php` |
| Échappement | `{{ }}` dans les vues appelle `e()`, soit `htmlspecialchars` | `core/View.php` (`compile`), `helpers.php` (`e`) |
| Injection d'en-têtes mail | Retours à la ligne refusés dans « Configuration des e-mails » | `app/Services/MailConfig.php` (`hasLineBreak`) |
| Clé SMTP | Chiffrée en AES-256-GCM, jamais réaffichée | `app/Services/Encrypter.php`, `resources/views/admin/email.php` |
| Trace SMTP | Lignes d'authentification masquées | `app/Services/Mailer.php` (`redactSmtpLine`) |
| Export JSON | Mot de passe, clé SMTP et jeton d'API retirés | `app/Services/MaintenanceService.php` (`backupData`) |
| Purge | Seuls les fichiers situés dans `storage/tmp` sont supprimés | `MaintenanceService.php` (`purgeTmp`) |
| En-têtes (partout) | `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin`, `X-Frame-Options: SAMEORIGIN` | `app/Http/Middleware/SecurityHeaders.php` |
| En-têtes (panel) | `X-Frame-Options: DENY`, `Cache-Control: no-store`, `X-Robots-Tag: noindex, nofollow` | `app/Http/Middleware/AdminHeaders.php` |
| Fichiers sensibles | Inaccessibles depuis le navigateur | `.htaccess`, `public/.htaccess`, `<dossier>/.htaccess` |
| HTTPS | Forcé, sauf `localhost`, `127.0.0.1`, `[::1]` et les noms en `.local`, `.test`, `.localhost` | `.htaccess` |

### 6.6 Journal des événements

- **17 types** : connexion réussie, échec, tentative refusée, IP bloquée, code refusé, mot de passe actuel incorrect,
  compte créé, déconnexion, code demandé, mot de passe réinitialisé, mot de passe modifié, email modifié,
  configuration des e-mails, test d'envoi, fichiers temporaires purgés, journal vidé, sauvegarde exportée
  (`app/Services/EventLog.php` : `types`).
- **Affichage** : trois cartes dans la Maintenance — connexions réussies, échecs / blocages, total des événements
  (`app/Http/Controllers/Admin/MaintenanceController.php` : `FILTERS`, `index`).

## 7. Front-end

### 7.1 HTML

- **Balises sémantiques** : `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`, `dialog` ; langue `fr`
  (`resources/views/layouts/`, `resources/views/pages/contact.php`, `resources/views/admin/login.php`).
- **Gabarits partagés** : `layouts/site.php`, `layouts/admin.php`, `layouts/auth.php` (`resources/views/layouts/`).
- **Accessibilité** : 40 attributs `aria-*` (`aria-label`, `aria-current`, `aria-expanded`...), libellés reliés aux champs,
  textes alternatifs (`resources/views/`).

### 7.2 CSS

- **Deux feuilles** : `public/css/style.css` (site) et `public/css/admin.css` (panel).
- **Charte** : variables `--bleu: #0056b3`, `--gris: #2d2d2d`, `--blanc: #ffffff`, `--vert: #4caf50` (`public/css/style.css` : `:root`).
- **Polices** : « Segoe UI », Roboto, Arial, sans-serif (`public/css/style.css` : `body`).
- **Responsive « mobile-first »** : styles pour mobile, puis `@media (min-width: …)` (`public/css/style.css`, `public/css/admin.css`).
- **Version des fichiers** : `?v=` suivi de la date de modification, pour éviter l'ancien cache (`helpers.php` : `asset`).
- **Ressources externes** : **absent** (aucune police, feuille ou image chargée depuis un autre site dans `resources/views/`, `public/css/`, `public/js/`).

### 7.3 JavaScript

Écrit sans bibliothèque, avec `var` et `function` (`public/js/`). `admin.js` utilise aussi `Array.from`, les expressions
régulières Unicode `\p{…}` et l'élément `<dialog>`, qui demandent un navigateur à jour.

| Fichier | Rôle |
|---|---|
| `public/js/script.js` | Menu mobile, défilement fluide, fondu des sections, couleur du bouton au survol, filtre des produits, validation du contact |
| `public/js/admin.js` | Messages qui disparaissent après 6 s, bouton bloqué pendant l'attente, fenêtres `<dialog>`, afficher / masquer le mot de passe, règles du mot de passe, contrôles avant envoi, sécurité SMTP liée au port, compte à rebours du code |

### 7.4 Thème clair / sombre

**Absent** : aucune règle `prefers-color-scheme` dans `public/css/style.css` ni `public/css/admin.css`.

## 8. Intégrations externes

| Service | Usage | Source |
|---|---|---|
| **Brevo** : serveur SMTP `smtp-relay.brevo.com`, ports 587 et 2525 (STARTTLS) ou 465 (SSL) | Envoi du code de réinitialisation et des mails de test | `config/mail.php`, `app/Services/Mailer.php` |
| **PHPMailer 7.1.1** | Client SMTP, messages d'erreur en français | `vendor/phpmailer/phpmailer/`, `Mailer.php` (`setLanguage('fr', …)`) |
| Fonction PHP `mail()` | Mode « PHP mail » : fonctionne chez un hébergeur, échoue sous WAMP | `Mailer.php`, `resources/views/admin/email.php` (message d'information) |

**API externe et paiement : absents** (aucun appel `curl_*`, `fsockopen` ni `file_get_contents('http…')` dans `app/` et `core/` ;
la seule connexion sortante est la connexion SMTP ouverte par PHPMailer).

## 9. Configuration

### 9.1 Fichiers de configuration

| Fichier | Réglages |
|---|---|
| `config/app.php` | Nom du site (`APP_NAME`, défaut « InfoLeader »), fuseau `Europe/Paris`, `debug` (`APP_DEBUG`, défaut `false`), liste des providers |
| `config/admin.php` | Session : nom `IL_ADMIN`, 1 800 s d'inactivité, 28 800 s au maximum. Mot de passe : 20 caractères minimum. Code : 300 s, 5 essais, 60 s entre deux envois. Chemins des fichiers du compte |
| `config/rate.php` | Délais d'attente, blocage après 8 échecs pendant 14 400 s, oubli du compteur après 86 400 s, fichier du compteur |
| `config/mail.php` | Fichiers `mail.json` et `mail.log`, serveur Brevo par défaut, ports et sécurité associée, chemin de PHPMailer |
| `config/maintenance.php` | Dossier temporaire, fichier des messages, pagination (20 événements, 10 messages), dossiers contrôlés |

La configuration est chargée au démarrage et lue avec `config('fichier.cle')` (`core/Application.php` : `loadConfiguration`, `config`).

### 9.2 Variables d'environnement

`env()` lit un fichier `.env` à la racine, puis les variables du système (`helpers.php`).
Le fichier `.env` est **absent** : les valeurs par défaut de `config/app.php` s'appliquent.

### 9.3 Configuration hors du dépôt

- **Virtual host** `infoleader.local` : `httpd-vhosts.conf` d'Apache et fichier `hosts` de Windows (procédure dans `README.md`).
- **Réglages saisis dans le panel** (compte, e-mails) : dans `storage/`, non versionné (`.gitignore`).

## 10. Limites connues et risques

Classées de la plus importante à la moins importante.

### 10.1 Sécurité

1. **Formulaire de contact sans protection anti-abus.** La route `/contact` n'a ni jeton CSRF ni limitation (`routes/web.php`).
   Un robot peut envoyer des messages en masse et remplir `storage/logs/contact.log` (`app/Models/Message.php`).
2. **Codes de réinitialisation en clair dans `mail.log`.** Sans service d'envoi, en cas d'échec ou si le débogage est actif,
   le mail complet est copié dans `storage/logs/mail.log` (`app/Services/Mailer.php` : `send`).
   Ce fichier n'est ni purgé ni consultable dans le panel (`app/Http/Controllers/Admin/MaintenanceController.php` : seuls `auth` et `contact` sont purgeables).
3. **Erreurs PHP affichables.** `display_errors = On` sous WAMP (Environnement : `phpForApache.ini`).
   Les exceptions sont interceptées (`app/Http/Kernel.php`), mais un simple avertissement PHP s'afficherait dans la page.
4. **Pas d'en-tête `Content-Security-Policy`** : absent de `app/Http/Middleware/SecurityHeaders.php`.
5. **Anti-brute force fondé sur l'adresse IP seule** (`RateLimiter.php` : clé `$ip`). Sur un réseau partagé, un blocage touche
   tous les utilisateurs. En local, `127.0.0.1` et `::1` sont deux clés différentes.
6. **Clé de chiffrement à côté des données chiffrées** : `storage/config/app.key` et `storage/config/mail.json`
   (`config/admin.php`, `config/mail.php`). Le chiffrement protège une copie partielle, pas un accès complet à `storage/`.
7. **Recréation du compte.** Si `storage/config/admin.json` disparaît, une personne présente sur la machine peut créer
   un nouveau compte via `/admin/installation` (`AuthController.php` : `setup`).

### 10.2 Fiabilité et exploitation

8. **Aucun test automatisé** : aucun fichier de test, aucune intégration continue (absents du dépôt, pas de dossier `.github/`).
9. **Journaux sans limite de taille.** `auth.log`, `contact.log` et `mail.log` ne sont jamais réduits.
   Ils sont relus en entier à chaque affichage (`EventLog.php` : `entries` ; `Message.php` : `all`).
10. **Cache des vues non surveillé.** `storage/framework/views` doit être accessible en écriture (`core/View.php` : `compiled`),
    mais il manque dans la liste `'writable'` (`config/maintenance.php`). S'il devenait non accessible en écriture,
    toutes les pages tomberaient en erreur 500 sans alerte dans la section Système.
11. **`storage/logs/app.log` invisible dans le panel** : écrit par `app/Http/Kernel.php`, lu par aucun contrôleur.
12. **Sauvegarde partielle** : l'export contient la configuration, pas les messages ni les journaux (`MaintenanceService.php` : `backupData`).

### 10.3 Données et fonctionnalités

13. **Catalogue figé** : produits dans `resources/data/products.json`, sans gestion depuis le panel (`app/Models/Product.php` : lecture seule).
14. **Prix stockés en texte** (`"149,99 €"` dans `resources/data/products.json`) : impossible de trier, filtrer ou calculer.
15. **Lien « Un conseil ? »** : ouvre le contact sans indiquer le produit (`resources/views/pages/products.php` : `route('contact')` sans paramètre).

### 10.4 Accessibilité et ergonomie

Les ratios de contraste sont calculés selon la formule des WCAG 2.1 à partir des couleurs des feuilles de style.
Le minimum (niveau AA) est de 4,5:1 pour le texte et 3:1 pour les bordures de champ.

16. **Contrastes insuffisants** :
    - bouton principal au survol, texte blanc (`--blanc`) sur vert `#4caf50` : **2,78:1** (`public/css/style.css` : `.btn-cta.hovered`) ;
    - pastille « À surveiller », `#b26a00` sur `#fff6e5` : **3,95:1** (`public/css/admin.css` : `.pill-warn`) ;
    - bordures des champs, `#e2e7ee` sur `#ffffff` ou `#f5f7fa` : **1,24:1** et **1,16:1** (`--gris-bord` dans `style.css` et `admin.css`).
17. **Zones cliquables petites** (44 × 44 px recommandés). Hauteurs mesurées dans le navigateur :
    - « Afficher » le mot de passe : 30 px (`admin.css` : `.toggle-password`, `padding: 6px 10px`) ;
    - « Mot de passe oublié ? » : 18 px (`admin.css` : `.link-button`, sans marge intérieure) ;
    - liens du pied de page : 26 px (`style.css` : `.footer-nav a`) ;
    - bouton du menu mobile : 40 × 42 px (`style.css` : `.menu-toggle`, `padding: 8px`).
18. **Messages d'erreur temporaires** : ils disparaissent après 6 s (`resources/views/admin/partials/alert.php` : `data-autohide="6000"` ;
    `public/js/admin.js`), sans rappel permanent sous les champs.
19. **Animations non désactivables** : aucune règle `prefers-reduced-motion` dans `public/css/`.
20. **Lien d'évitement « Aller au contenu »** : absent de `resources/views/layouts/`.
21. **Fausses affordances** : les vignettes de catégories ressemblent à des cartes cliquables mais ne sont pas des liens
    (`resources/views/pages/home.php` : `<li>` sans `<a>`).
