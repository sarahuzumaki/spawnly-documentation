# Documentation technique — InfoLeader

Analyse du site de fin de TP 2 présent dans le dossier `C:\wamp64\www\infoleader`
(contenu identique au commit Git `fb1f466` du 13/10/2026).

**Lecture des sources.** Chaque affirmation est suivie de sa source entre parenthèses ou dans une colonne « Source ».

- Un **chemin** (ex. `admin/includes/auth.php`) désigne un fichier du dépôt.
  Les fonctions citées entre parenthèses se trouvent dans ce fichier.
- **Environnement** désigne une information lue sur le poste WAMP (version, `php.ini`, `httpd-vhosts.conf`).
  Elle n'est pas dans le dépôt.
- **Test** désigne un résultat observé en ouvrant l'adresse sous WAMP (code de réponse, en-têtes, taille mesurée).
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
(`README.md`, `index.php`).
Il comprend un site public et un panneau d'administration réservé à un administrateur unique
(`admin/`, `admin/includes/auth.php` : « Compte administrateur unique »).

### 1.2 Fonctionnalités du site public

| Page | Fichier | Fonctionnalités | Source |
|---|---|---|---|
| Accueil | `index.php` | Bannière, slogan, bouton vers les produits, présentation, vignettes de catégories, 3 engagements, bandeau vers le contact | `index.php` |
| Nos Produits | `products.php` | 6 produits en grille (image, catégorie, nom, description, prix), recherche en temps réel, compteur de résultats | `products.php`, `products.json`, `script.js` |
| Contact | `contact.php` | Formulaire nom / email / message ; validation en JavaScript puis en PHP ; aucun e-mail envoyé ; message enregistré dans un fichier | `script.js` (`validateForm`), `contact.php` |

### 1.3 Fonctionnalités du panneau d'administration

| Fonction | Fichier | Source |
|---|---|---|
| Création du compte au premier lancement | `admin/setup.php` | `admin/setup.php` |
| Connexion avec anti-brute force, fenêtre « Mot de passe oublié » | `admin/index.php` | `admin/index.php`, `admin/includes/ratelimit.php` |
| Mot de passe oublié : code à 8 chiffres, puis nouveau mot de passe | `admin/forgot.php`, `admin/reset.php` | `admin/includes/reset.php` |
| Tableau de bord : 4 outils et informations du compte | `admin/panel.php` | `admin/panel.php` |
| Configuration des e-mails : PHP `mail()` ou Brevo, test d'envoi | `admin/email.php` | `admin/includes/mailconfig.php`, `admin/includes/mailer.php` |
| Mot de passe : modification | `admin/password.php` | `admin/password.php` |
| Utilisateur : adresse email de connexion | `admin/user.php` | `admin/user.php` |
| Maintenance : Système, Journaux, Sauvegarde, Purger `storage/tmp` | `admin/maintenance.php` | `admin/includes/maintenance.php` |
| Déconnexion | `admin/logout.php` | `admin/logout.php` |

### 1.4 Historique du projet

| Étape | Contenu | Source |
|---|---|---|
| TP 1 et TP 2 | Site vitrine, puis panneau d'administration | Historique Git (commit `bd33603`), `README.md` |
| Sécurité du dépôt | Blocage de l'accès web au dossier `.git` | Historique Git (commit `fb1f466`), `.htaccess` |

### 1.5 Points forts

- **Panel protégé** : empreintes de mots de passe, clé SMTP chiffrée, jetons CSRF, anti-brute force,
  sessions liées au compte (`admin/includes/auth.php`, `secrets.php`, `bootstrap.php`, `ratelimit.php`).
- **Code commun regroupé** : le panel réutilise les fonctions de `admin/includes/` et un gabarit HTML commun
  (`admin/includes/layout.php`).
- **Double validation** : chaque formulaire est contrôlé en JavaScript, puis de nouveau en PHP
  (`script.js` et `contact.php` ; `admin/assets/admin.js` et les pages du panel).
- **Retours clairs** : messages de succès et d'erreur, fenêtre de confirmation avant chaque action irréversible
  (`admin/includes/layout.php` : `alert_box` ; `admin/maintenance.php` : `confirm_dialog`).
- **Données protégées** : `storage/`, `vendor/` et `admin/includes/` refusent tout accès depuis le navigateur
  (`<dossier>/.htaccess` : `Require all denied` ; Test : code 403).
- **Une seule bibliothèque**, sans gestionnaire de paquets : PHPMailer (`vendor/phpmailer/`, aucun `composer.json`).
- **Code commenté en français** et données affichées échappées (`htmlspecialchars`, fonction `e()` de `admin/includes/bootstrap.php`).

## 2. Stack technique

| Élément | Valeur | Source |
|---|---|---|
| Environnement | WAMP sous Windows | `README.md` |
| Serveur web | Apache | `.htaccess`, `README.md` |
| Version d'Apache | 2.4.65 | Environnement (section Système de la Maintenance) |
| Langage serveur | PHP | tous les fichiers `.php` |
| Version de PHP | 8.3.28 | Environnement |
| Version minimale | PHP 8.1 : le type de retour `never` est utilisé, et la Maintenance signale une erreur en dessous de 8.1. Le code utilise aussi `match` et `str_starts_with` (PHP 8.0) | `admin/includes/bootstrap.php` (`redirect`), `admin/includes/maintenance.php` (`version_compare`), `admin/reset.php` |
| Extensions PHP | `openssl`, `mbstring`, `json`, `session` | `admin/includes/maintenance.php` (`system_checks`), `admin/includes/secrets.php` (`openssl_encrypt`) |
| Framework | **Absent.** Pages PHP indépendantes et fonctions communes du panel | `admin/includes/` |
| Front-end | HTML5, CSS3, JavaScript sans bibliothèque | `style.css`, `script.js`, `admin/assets/` |
| Bibliothèque | PHPMailer 7.1.1, copiée dans `vendor/` sans Composer | `vendor/phpmailer/phpmailer/VERSION` ; absent : `composer.json` |
| Stockage | Fichiers JSON et texte | `admin/includes/storage.php`, `contact.php`, `products.php` |
| Base de données | **Absent.** Aucune connexion PDO ou mysqli, aucune requête SQL | Recherche de `PDO`, `mysqli`, `SELECT`, `INSERT INTO` : seules des balises HTML `<select>` sont trouvées |
| Versionnage | Git | `.gitignore`, historique Git |

## 3. Arborescence du projet

Source : liste des fichiers suivis par Git (`git ls-files`).

```
infoleader/
├── .htaccess              Renvoie une erreur 404 pour .git, .gitignore et .gitattributes
├── .gitignore             Exclut storage/ sauf storage/.htaccess
├── README.md              Présentation, pages, sécurité, installation
├── index.php              Page d'accueil
├── products.php           Page « Nos Produits » (lit products.json)
├── contact.php            Page « Contact » (validation PHP, enregistrement du message)
├── products.json          Catalogue des 6 produits
├── style.css              Feuille de style du site public
├── script.js              JavaScript du site public
├── assets/
│   ├── banner.jpg         Image de la bannière d'accueil
│   └── products/          Images des produits : casque, clavier, ecran, hub, souris, webcam (.jpg)
├── admin/                 Panneau d'administration
│   ├── index.php          Connexion et fenêtre « Mot de passe oublié »
│   ├── setup.php          Création du compte (premier lancement)
│   ├── forgot.php         Envoi du code de réinitialisation
│   ├── reset.php          Saisie du code et du nouveau mot de passe
│   ├── panel.php          Tableau de bord
│   ├── email.php          Module « Configuration des e-mails »
│   ├── password.php       Module « Mot de passe »
│   ├── user.php           Module « Utilisateur »
│   ├── maintenance.php    Module « Maintenance »
│   ├── logout.php         Déconnexion
│   ├── assets/            admin.css, admin.js
│   └── includes/          Fonctions communes du panel (+ .htaccess)
│       ├── bootstrap.php  Constantes, dossiers, en-têtes, session, CSRF, messages flash
│       ├── storage.php    Lecture et écriture JSON avec verrou, journal des événements
│       ├── auth.php       Compte, règles des mots de passe, session administrateur
│       ├── ratelimit.php  Anti-brute force
│       ├── reset.php      Code de réinitialisation
│       ├── secrets.php    Chiffrement AES-256-GCM
│       ├── mailconfig.php Réglages d'envoi des e-mails
│       ├── mailer.php     Envoi des e-mails, fichier mail.log
│       ├── maintenance.php Contrôles du système, journaux, sauvegarde
│       └── layout.php     Gabarit HTML du panel (en-tête, alertes, titres)
├── storage/.htaccess      Seul fichier versionné de storage/ (données exclues par .gitignore)
└── vendor/                .htaccess et PHPMailer : src/, language/phpmailer.lang-fr.php, LICENSE, VERSION
```

Trois dossiers contiennent un `.htaccess` « `Require all denied` » : `storage/`, `vendor/` et `admin/includes/`.

## 4. Points d'entrée et routage

### 4.1 Principe

- **Aucun routeur** : chaque adresse correspond à un fichier PHP, exécuté directement par Apache (arborescence ci-dessus).
- **Pages publiques autonomes** : `index.php`, `products.php` et `contact.php` n'incluent aucun autre fichier PHP.
- **Pages du panel** : elles commencent toutes par `require __DIR__ . '/includes/bootstrap.php'` (`admin/*.php`).
  Ce fichier charge les fonctions communes, envoie les en-têtes de sécurité et démarre la session.
- **Pages protégées** : elles appellent ensuite `require_login()`. Sans connexion valide, le visiteur est redirigé
  vers `admin/index.php` (`admin/includes/auth.php` ; Test : code 302 vers `index.php`).
- **Liens relatifs** : le site fonctionne à `http://localhost/infoleader` comme à `http://infoleader.local` (`README.md`).

### 4.2 Points d'entrée

| Fichier | Méthodes | Accès | Source |
|---|---|---|---|
| `index.php`, `products.php` | GET | Public | — |
| `contact.php` | GET, POST | Public | `contact.php` |
| `admin/setup.php` | GET, POST | Tant qu'aucun compte n'existe, depuis `127.0.0.1` ou `::1` (sinon refus, code 403) | `admin/setup.php`, `bootstrap.php` (`is_local_request`) |
| `admin/index.php` | GET, POST | Public ; renvoie vers `setup.php` sans compte, vers `panel.php` si déjà connecté | `admin/index.php` |
| `admin/forgot.php` | POST | Formulaire de la fenêtre « Mot de passe oublié » (GET : retour à la connexion) | `admin/forgot.php` |
| `admin/reset.php` | GET, POST | Public, si un compte existe | `admin/reset.php` |
| `admin/panel.php`, `email.php`, `password.php`, `user.php`, `maintenance.php` | GET (+ POST pour les modules) | Administrateur connecté | `require_login()` dans chaque fichier |
| `admin/logout.php` | POST | Formulaire de l'en-tête du panel | `admin/logout.php`, `admin/includes/layout.php` (`admin_header`) |

### 4.3 Paramètres d'adresse

| Page | Paramètres | Source |
|---|---|---|
| `admin/email.php` | `tab` = `php` ou `brevo` | `admin/email.php` |
| `admin/maintenance.php` | `section` = `systeme`, `journaux` ou `sauvegarde` ; `onglet` = `connexion` ou `contact` ; `filtre` = `success` ou `failure` ; `page` = numéro | `admin/maintenance.php` |

Une valeur inconnue est remplacée par la valeur par défaut (`in_array` et `isset` dans ces deux fichiers).

### 4.4 Traitement des formulaires

- Chaque formulaire est envoyé en POST à sa propre page.
- **Panel** : le jeton CSRF est vérifié (`csrf_check`). En cas de succès, un message est mémorisé (`flash`)
  puis la page redirige (`redirect`) : actualiser la page ne renvoie pas le formulaire (`admin/*.php`).
- **Contact** : le résultat est affiché directement dans la réponse au POST, sans redirection (`contact.php` : `$success = true`).

### 4.5 Cas particuliers

| Situation | Résultat | Source |
|---|---|---|
| `storage/`, `vendor/`, `admin/includes/` | Code 403 | `<dossier>/.htaccess` ; Test |
| `.git`, `.gitignore`, `.gitattributes` | Code 404 | `.htaccess` (`RedirectMatch 404`) ; Test |
| Page inexistante | Page 404 par défaut d'Apache | **Absent** : aucune page d'erreur dans le dépôt ; Test |
| Dossier sans `index.php` (ex. `/assets/`) | Liste des fichiers du dossier | Environnement : `Options +Indexes` dans `httpd-vhosts.conf` ; Test |

## 5. Stockage des données

### 5.1 Base de données

**Absent.** Le site n'utilise aucune base de données : pas de tables, de relations ni de requêtes SQL
(aucune occurrence de `PDO`, `mysqli`, `SELECT` ou `INSERT INTO` hors balises HTML `<select>`).

### 5.2 Fichiers de données

Les données du panel et les messages de contact sont dans `storage/`, exclu du dépôt (`.gitignore`) et protégé (`storage/.htaccess`).
Les chemins sont définis par des constantes (`admin/includes/bootstrap.php`, sauf mention contraire).

| Fichier | Format | Contenu | Source |
|---|---|---|---|
| `storage/config/admin.json` | JSON | Email, empreinte du mot de passe, dates de création, de connexion (et IP), de connexion précédente (et IP), de changement du mot de passe, de changement de l'email | `admin/setup.php`, `admin/includes/auth.php` (`login_admin`), `admin/user.php` |
| `storage/config/mail.json` | JSON | Mode actif (`php` ou `brevo`) ; pour chaque mode : nom et email de l'expéditeur, destinataire ; pour Brevo : serveur, port, sécurité, identifiant, clé SMTP **chiffrée**, débogage ; dates d'enregistrement | `admin/includes/mailconfig.php` (`validate_brevo`, `save_mail_settings`) |
| `storage/config/app.key` | Texte (base64) | Clé de chiffrement de 32 octets, créée au premier besoin | `admin/includes/secrets.php` (`app_key`) |
| `storage/security/ratelimit.json` | JSON | Par adresse IP : nombre d'échecs, date du dernier échec, prochain essai autorisé, fin de blocage | `admin/includes/ratelimit.php` |
| `storage/security/reset.json` | JSON | Empreinte du code, dates de création et d'expiration, nombre d'essais, adresse IP | `admin/includes/reset.php` (`reset_create`) |
| `storage/logs/auth.log` | Une ligne JSON par événement | Date, type, adresse IP, détail | `admin/includes/storage.php` (`log_event`) |
| `storage/logs/contact.log` | Une ligne JSON par message | Date, IP, nom, email, message | `contact.php` (chemin écrit dans ce fichier) |
| `storage/logs/mail.log` | Texte | Mails non envoyés ou en échec ; copie et trace SMTP si le débogage est actif | `admin/includes/mailer.php` (`mail_log`, `send_mail`) |
| `storage/tmp/` | — | Fichiers temporaires, vidés par « Purger storage/tmp » | `admin/includes/maintenance.php` (`purge_tmp`) |
| `products.json` | JSON, versionné | 6 produits : `name`, `category`, `price` en texte (ex. `"149,99 €"`), `description`, `image` | `products.json`, `products.php` (lecture seule) |

### 5.3 Écriture et accès

- **Verrou** : `json_read` lit avec un verrou partagé ; `json_update` pose un verrou exclusif, lit, modifie puis réécrit
  (`admin/includes/storage.php`, fonction `flock`).
- **Journaux** : ajout de lignes avec `FILE_APPEND | LOCK_EX`
  (`admin/includes/storage.php` : `log_event` ; `admin/includes/mailer.php` : `mail_log` ; `contact.php`).
- **Dossiers** : `storage/config`, `security`, `logs` et `tmp` sont créés s'ils manquent
  (`admin/includes/bootstrap.php` ; `contact.php` pour `logs`).
- **Accès** : les pages du panel passent par les fonctions de `admin/includes/`. `contact.php` et `products.php`
  lisent ou écrivent leur fichier eux-mêmes (`file_put_contents`, `file_get_contents`).
- **Sessions PHP** : leur emplacement n'est pas défini dans le code (absent de `admin/includes/bootstrap.php`).
  Le réglage de WAMP s'applique : `session.save_path = "c:/wamp64/tmp"` (Environnement : `php.ini`).

## 6. Authentification, sessions et sécurité

### 6.1 Compte administrateur

- **Création** : par `admin/setup.php`, depuis `127.0.0.1` ou `::1` uniquement, tant qu'aucun compte n'existe
  (`admin/setup.php` ; `admin/includes/bootstrap.php` : `is_local_request`).
- **Mot de passe** : stocké sous forme d'empreinte avec `password_hash()`, vérifié avec `password_verify()`
  (`admin/includes/auth.php` : `hash_password`, `check_credentials`).
- **Règles** : 20 caractères minimum, une minuscule, une majuscule, un chiffre, un caractère spécial
  (`admin/includes/auth.php` : `password_rules`, `password_errors` ; `bootstrap.php` : `PASSWORD_MIN_LENGTH`).
  Elles sont cochées en direct pendant la saisie (`admin/assets/admin.js`).

### 6.2 Connexion et anti-brute force

- **Message unique** en cas d'erreur : « Identifiant ou mot de passe incorrect » (`admin/index.php`).
- **Pas d'indice** : le mot de passe est toujours vérifié, même si l'email est faux (`admin/includes/auth.php` : `check_credentials`).
- **Attente progressive** : 3, 3, 3, 10, 30, 60 puis 300 secondes après chaque échec (`bootstrap.php` : `RATE_LIMIT_DELAYS`).
  Pendant l'attente, le bouton affiche un compte à rebours (`admin/assets/admin.js`).
- **Essai trop rapide** : refusé sans vérifier le mot de passe (`admin/index.php` : événement `login_rejected`).
- **Blocage** : au 8e échec, l'adresse IP est bloquée 14 400 s, soit 4 h
  (`bootstrap.php` : `RATE_LIMIT_BLOCK_AFTER`, `RATE_LIMIT_BLOCK_DURATION` ; `ratelimit.php` : `rate_limit_fail`).
- **Oubli** : compteur effacé après 86 400 s sans échec (`RATE_LIMIT_FORGET_AFTER`), et après une connexion réussie
  (`ratelimit.php` : `rate_limit_clear`).
- **Périmètre** : connexion (`admin/index.php`), code de réinitialisation (`admin/reset.php`),
  mot de passe actuel des modules Mot de passe et Utilisateur (`auth.php` : `verify_current_password`).
- **IP bloquée** : la demande de code est refusée (`admin/forgot.php`).

### 6.3 Mot de passe oublié

- Code de 8 chiffres tiré avec `random_int` (`admin/includes/reset.php` : `reset_create`).
- Valable 300 s, 5 essais au maximum, un envoi toutes les 60 s
  (`bootstrap.php` : `RESET_CODE_TTL`, `RESET_CODE_MAX_ATTEMPTS`, `RESET_RESEND_DELAY` ; `admin/forgot.php`).
- Seule l'empreinte du code est stockée (`admin/includes/reset.php`).
- Envoyé au destinataire défini dans « Configuration des e-mails », sinon à l'email du compte
  (`admin/includes/mailer.php` : `mail_recipient`).

### 6.4 Sessions

- **Cookie** `IL_ADMIN`, `HttpOnly`, `SameSite=Strict`, `Secure` en HTTPS, supprimé à la fermeture du navigateur
  (`admin/includes/bootstrap.php` : `session_set_cookie_params`).
- **Mode strict** et cookies seuls : `session.use_strict_mode`, `session.use_only_cookies` (`bootstrap.php`).
- **Nouvel identifiant** à la connexion : `session_regenerate_id(true)` (`admin/includes/auth.php` : `login_admin`).
- **Expiration** : 1 800 s d'inactivité ou 28 800 s au total
  (`bootstrap.php` : `SESSION_IDLE_TIMEOUT`, `SESSION_MAX_LIFETIME` ; `auth.php` : `is_logged_in`).
- **Lien avec le compte** : la session garde une empreinte du compte et devient invalide s'il change
  (`auth.php` : `account_fingerprint`, `is_logged_in`).
- **Autres sessions** déconnectées après un changement de mot de passe ou d'email (`auth.php` : `refresh_admin_session`).
- **Déconnexion** par un formulaire POST protégé par jeton CSRF (`admin/logout.php`).

### 6.5 Autres protections

| Protection | Détail | Source |
|---|---|---|
| CSRF | Jeton secret dans chaque formulaire du panel, comparé avec `hash_equals` ; sinon arrêt avec le code 400 | `admin/includes/bootstrap.php` (`csrf_token`, `csrf_field`, `csrf_check`) |
| Échappement | Données affichées passées dans `htmlspecialchars` | `contact.php`, `products.php`, `admin/includes/bootstrap.php` (`e`) |
| Injection d'en-têtes mail | Retours à la ligne refusés dans « Configuration des e-mails » | `admin/includes/mailconfig.php` (`has_line_break`) |
| Clé SMTP | Chiffrée en AES-256-GCM, jamais réaffichée | `admin/includes/secrets.php`, `admin/email.php` |
| Trace SMTP | Lignes d'authentification masquées | `admin/includes/mailer.php` (`redact_smtp_line`) |
| Export JSON | Mot de passe, clé SMTP et jeton d'API retirés | `admin/includes/maintenance.php` (`backup_data`) |
| Purge | Seuls les fichiers situés dans `storage/tmp` sont supprimés, liens symboliques exclus | `admin/includes/maintenance.php` (`purge_tmp`) |
| Adresse IP | Seule `REMOTE_ADDR` est utilisée (en-têtes `X-Forwarded-For` ignorés) | `admin/includes/bootstrap.php` (`client_ip`) |
| En-têtes (panel) | `X-Frame-Options: DENY`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: same-origin`, `Cache-Control: no-store` ; balise `robots` `noindex, nofollow` | `admin/includes/bootstrap.php`, `admin/includes/layout.php` (`page_start`) |
| En-têtes (site public) | **Absent** : seul `X-Powered-By` est envoyé | Test (`index.php`, `contact.php`) |
| Fichiers sensibles | `storage/`, `vendor/`, `admin/includes/` refusés (403) ; `.git` masqué (404) | `<dossier>/.htaccess`, `.htaccess` ; Test |

### 6.6 Journal des événements

- **17 types** : connexion réussie, échec, tentative refusée, IP bloquée, code refusé, mot de passe actuel incorrect,
  compte créé, déconnexion, code demandé, mot de passe réinitialisé, mot de passe modifié, email modifié,
  configuration des e-mails, test d'envoi, fichiers temporaires purgés, journal vidé, sauvegarde exportée
  (`admin/includes/maintenance.php` : `event_types`).
- **Affichage** : trois cartes dans la Maintenance (connexions réussies, échecs / blocages, total des événements),
  20 événements par page (`admin/maintenance.php` ; `admin/includes/maintenance.php` : `log_filters`, `LOG_PER_PAGE`).

## 7. Front-end

### 7.1 HTML

- **Balises sémantiques** : `header`, `nav`, `main`, `section`, `article`, `aside`, `footer`, `dialog` ; langue `fr`
  (`index.php`, `contact.php`, `admin/index.php`, `admin/includes/layout.php`).
- **Site public** : l'en-tête, le menu et le pied de page sont recopiés dans chacune des 3 pages
  (`index.php`, `products.php`, `contact.php`).
- **Panel** : gabarit commun produit par des fonctions PHP (`admin/includes/layout.php` : `page_start`, `page_end`, `admin_header`).
- **Accessibilité** : 55 attributs `aria-*` (`aria-label`, `aria-current`, `aria-expanded`...), libellés reliés aux champs,
  textes alternatifs (fichiers `.php` du site et du panel).

### 7.2 CSS

- **Deux feuilles** : `style.css` (site) et `admin/assets/admin.css` (panel).
- **Charte** : variables `--bleu: #0056b3`, `--gris: #2d2d2d`, `--blanc: #ffffff`, `--vert: #4caf50` (`style.css` et `admin.css` : `:root`).
- **Polices** : « Segoe UI », Roboto, Arial, Helvetica, sans-serif (`style.css`, `admin.css` : `body`).
- **Responsive « mobile-first »** : styles pour mobile, puis `@media (min-width: …)` à 600, 769 et 1100 px (`style.css`)
  et à 700 et 1000 px (`admin.css`).
- **Ressources externes** : **absent** (aucune police, feuille ou image chargée depuis un autre site).

### 7.3 JavaScript

Écrit sans bibliothèque, avec `var` et `function` (`script.js`, `admin/assets/admin.js`).
`admin.js` utilise aussi `Array.from`, les expressions régulières Unicode `\p{…}` et l'élément `<dialog>`,
qui demandent un navigateur à jour.

| Fichier | Rôle |
|---|---|
| `script.js` | Menu mobile, défilement fluide, fondu des sections, couleur du bouton au survol, filtre des produits, validation du contact |
| `admin/assets/admin.js` | Messages qui disparaissent après 6 s, bouton bloqué pendant l'attente, fenêtres `<dialog>`, afficher / masquer le mot de passe, règles du mot de passe, contrôles avant envoi, sécurité SMTP liée au port, compte à rebours du code |

### 7.4 Thème clair / sombre

**Absent** : aucune règle `prefers-color-scheme` dans `style.css` ni `admin/assets/admin.css`.

## 8. Intégrations externes

| Service | Usage | Source |
|---|---|---|
| **Brevo** : serveur SMTP `smtp-relay.brevo.com`, ports 587 et 2525 (STARTTLS) ou 465 (SSL) | Envoi du code de réinitialisation et des mails de test | `admin/includes/mailconfig.php`, `admin/includes/mailer.php` |
| **PHPMailer 7.1.1** | Client SMTP, messages d'erreur en français | `vendor/phpmailer/phpmailer/`, `mailer.php` (`setLanguage('fr')`) |
| Fonction PHP `mail()` | Mode « PHP mail » : prévu pour un hébergeur, sans serveur d'envoi sous WAMP | `admin/includes/mailer.php`, `admin/email.php` (message d'information) |

**API externe et paiement : absents.** Aucun appel `curl_*`, `fsockopen` ni `file_get_contents('http…')` ;
la seule connexion sortante est la connexion SMTP ouverte par PHPMailer.
L'export de la Maintenance indique : « Aucune API n'est configurée dans cette version du site » (`admin/includes/maintenance.php` : `backup_data`).

## 9. Configuration

### 9.1 Constantes du panel

| Fichier | Réglages |
|---|---|
| `admin/includes/bootstrap.php` | Fuseau `Europe/Paris`. Chemins de `storage/` et de ses fichiers. Session : 1 800 s d'inactivité, 28 800 s au maximum. Mot de passe : 20 caractères minimum. Code : 300 s, 5 essais, 60 s entre deux envois. Anti-brute force : délais, blocage après 8 échecs pendant 14 400 s, oubli après 86 400 s |
| `admin/includes/mailconfig.php` | Fichier `mail.json`, serveur Brevo par défaut, ports et sécurité associée |
| `admin/includes/secrets.php` | Fichier `app.key` |
| `admin/includes/maintenance.php` | Fichier `contact.log`, pagination (20 événements, 10 messages) |

### 9.2 Pages publiques

Les chemins sont écrits directement dans les pages : `__DIR__ . '/products.json'` (`products.php`)
et `__DIR__ . '/storage/logs'` (`contact.php`).
Le chemin de `contact.log` est donc défini deux fois (`contact.php` et `admin/includes/maintenance.php` : `CONTACT_LOG_FILE`).

### 9.3 Réglages saisis dans le panel

Compte administrateur et envoi des e-mails : `storage/config/admin.json` et `storage/config/mail.json`, non versionnés (`.gitignore`).

### 9.4 Variables d'environnement

**Absent** : aucun fichier `.env`, aucun appel à `getenv` ou `$_ENV`.

### 9.5 Configuration hors du dépôt

| Élément | Valeur | Source |
|---|---|---|
| Virtual host `infoleader.local` | `Options +Indexes +FollowSymLinks`, `AllowOverride All`, `Require local` | Environnement : `httpd-vhosts.conf` ; procédure dans `README.md` |
| Fichier `hosts` de Windows | `127.0.0.1 infoleader.local` | `README.md` |
| `php.ini` | `display_errors = On`, `expose_php = On`, `log_errors = On` vers `c:/wamp64/logs/php_error.log` | Environnement |

## 10. Limites connues et risques

Classées de la plus importante à la moins importante, dans chaque thème.
Le virtual host n'accepte que la machine locale (`Require local`, Environnement) : les risques de sécurité
concernent surtout les bonnes pratiques à acquérir.

### 10.1 Sécurité

1. **Formulaire de contact sans protection anti-abus.** `contact.php` n'a ni jeton CSRF, ni champ piège, ni limite d'envois.
   Un robot peut envoyer des messages en masse et remplir `storage/logs/contact.log`.
2. **Codes de réinitialisation en clair dans `mail.log`.** Le code est écrit dans le corps du mail (`admin/forgot.php`).
   Sans service d'envoi, en cas d'échec ou si le débogage est actif, le mail complet est copié dans `storage/logs/mail.log`
   (`admin/includes/mailer.php` : `send_mail`). Ce fichier n'est ni consultable ni vidable dans le panel
   (`admin/includes/maintenance.php` : `purge_log` ne traite que `auth` et `contact`).
3. **Informations techniques visibles depuis le navigateur** (Test) :
   - liste des fichiers de `/assets/`, `/assets/products/` et `/admin/assets/` (Environnement : `Options +Indexes`) ;
   - `README.md` lisible (code 200), comme tout fichier `.md` déposé à la racine ;
   - en-tête `X-Powered-By: PHP/8.3.28`, qui révèle la version de PHP (Environnement : `expose_php = On`).
4. **Pages publiques sans en-têtes de sécurité** : `index.php` et `contact.php` n'envoient que `X-Powered-By` (Test).
   Aucune page n'envoie d'en-tête `Content-Security-Policy` (absent de `admin/includes/bootstrap.php`).
5. **Erreurs PHP affichées dans les pages.** `display_errors = On` (Environnement) et aucun gestionnaire d'erreurs
   (`set_error_handler`, `set_exception_handler` : absents). Par exemple, si `storage/` n'est plus accessible en écriture,
   l'exception de `json_update` n'est pas interceptée (`admin/includes/storage.php`) : le message et le chemin du serveur s'affichent.
6. **Anti-brute force fondé sur l'adresse IP seule** (`admin/includes/ratelimit.php` : clé `$ip`).
   Sur un réseau partagé, un blocage touche tous les utilisateurs. En local, `127.0.0.1` et `::1` sont deux clés différentes.
7. **Clé de chiffrement à côté des données chiffrées** : `storage/config/app.key` et `storage/config/mail.json`
   (`admin/includes/secrets.php`, `admin/includes/mailconfig.php`). Le chiffrement protège une copie partielle,
   pas un accès complet à `storage/`.
8. **Recréation du compte.** Si `storage/config/admin.json` disparaît, une personne présente sur la machine peut créer
   un nouveau compte via `admin/setup.php` (`admin_exists`, `is_local_request`).

### 10.2 Fiabilité et qualité du code

9. **Message de contact enregistré deux fois.** Après un envoi réussi, la page répond directement au POST, sans redirection
   (`contact.php` : `$success = true`). Actualiser la page (F5) renvoie le formulaire et enregistre le message une seconde fois.
10. **Code dupliqué.** L'en-tête, le menu et le pied de page sont recopiés dans les 3 pages publiques
    (`index.php`, `products.php`, `contact.php`) : chaque modification doit être faite 3 fois.
    Le chemin de `contact.log` est aussi défini à deux endroits (section 9.2).
11. **Aucun test automatisé** : aucun fichier de test, aucune intégration continue (absents du dépôt, pas de dossier `tests/` ni `.github/`).
12. **Journaux sans limite de taille.** `auth.log`, `contact.log` et `mail.log` ne sont jamais réduits.
    Les deux premiers sont relus en entier à chaque affichage de la Maintenance (`admin/includes/maintenance.php` : `read_json_lines`).
13. **Sauvegarde partielle** : l'export contient la configuration, pas les messages ni les journaux
    (`admin/includes/maintenance.php` : `backup_data`).

### 10.3 Données et fonctionnalités

14. **Catalogue figé** : produits dans `products.json`, sans gestion depuis le panel (`products.php` : lecture seule).
15. **Prix stockés en texte** (`"149,99 €"` dans `products.json`) : impossible de trier, filtrer ou calculer.
16. **Lien « Un conseil ? »** : ouvre le contact sans indiquer le produit (`products.php` : `href="contact.php"`).

### 10.4 Données personnelles (RGPD)

17. **Messages de contact conservés sans limite et sans information.** Nom, email, message et adresse IP sont enregistrés
    (`contact.php`) sans durée de conservation : seule une suppression manuelle de tous les messages existe
    (`admin/maintenance.php` : « Vider les messages »). Le formulaire n'informe pas le visiteur de cette collecte,
    et le site n'a pas de page de mentions légales ou de confidentialité (pied de page : 3 liens seulement).

### 10.5 Accessibilité et ergonomie

Les ratios de contraste sont calculés selon la formule des WCAG 2.1 à partir des couleurs des feuilles de style.
Le minimum (niveau AA) est de 4,5:1 pour le texte et 3:1 pour les bordures de champ.

18. **Contrastes insuffisants** :
    - bouton principal au survol, texte blanc sur vert `#4caf50` : **2,78:1** (`style.css` : `.btn-cta:hover`, `.btn-cta.hovered`) ;
    - pastille « À surveiller », `#b26a00` sur `#fff6e5` : **3,95:1** (`admin/assets/admin.css` : `.pill-warn`) ;
    - bordures des champs, `#e2e7ee` sur `#ffffff` ou `#f5f7fa` : **1,24:1** et **1,16:1** (`--gris-bord` dans `style.css` et `admin.css`).
19. **Zones cliquables petites** (44 × 44 px recommandés). Hauteurs mesurées dans le navigateur (Test) :
    - « Afficher » le mot de passe : 30 px (`admin.css` : `.toggle-password`, `padding: 6px 10px`) ;
    - « Mot de passe oublié ? » : 18 px (`admin.css` : `.link-button`, sans marge intérieure) ;
    - liens du pied de page : 26 px (`style.css` : `.footer-nav a`) ;
    - bouton du menu mobile : 40 × 42 px (`style.css` : `.menu-toggle`, `padding: 8px`).
20. **Messages d'erreur temporaires** : ils disparaissent après 6 s (`admin/includes/layout.php` : `alert_box`, `data-autohide="6000"` ;
    `admin/assets/admin.js` : `autohide`), sans rappel permanent sous les champs.
21. **Animations non désactivables** : aucune règle `prefers-reduced-motion` dans `style.css` ni `admin.css`
    (fondu des sections et défilement fluide : `script.js`).
22. **Lien d'évitement « Aller au contenu »** : absent des pages publiques et du gabarit du panel.
23. **Vignettes de catégories non cliquables** : les 6 catégories de l'accueil (image et nom) ne mènent pas aux produits
    correspondants (`index.php` : `<li>` sans `<a>`).
