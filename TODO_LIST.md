# Plan d'améliorations — InfoLeader

Ce plan découle de la section 10 « Limites connues et risques » de `DOCUMENTATION.md`.
La colonne **Réf.** renvoie au numéro de la limite concernée (ex. **L1** = limite n° 1).

Il est conçu pour vous, étudiant(e) en BTS SIO 1re année : actions courtes, concrètes et testables dans votre navigateur.

## Sommaire

1. [Cadre à respecter](#1-cadre-à-respecter)
2. [Comment lire ce plan](#2-comment-lire-ce-plan)
3. [Actions par thème](#3-actions-par-thème)
   - [3.1 Sécurité](#31-sécurité)
   - [3.2 Stockage des données](#32-stockage-des-données)
   - [3.3 Qualité du code](#33-qualité-du-code)
   - [3.4 Front-end et accessibilité](#34-front-end-et-accessibilité)
   - [3.5 Conformité et RGPD](#35-conformité-et-rgpd)
   - [3.6 Documentation](#36-documentation)
   - [3.7 Exploitation locale](#37-exploitation-locale)
4. [Définition de Terminé](#4-définition-de-terminé)
5. [Séquencement par jalons](#5-séquencement-par-jalons)
6. [Petit lexique](#6-petit-lexique)

---

## 1. Cadre à respecter

- **WAMP uniquement.** Toutes les actions se font en local, dans `C:\wamp64\www\infoleader`, et se testent
  sur `http://localhost/infoleader` ou `http://infoleader.local`.
  Aucune ne demande de serveur distant, de VPS, de Docker, de Nginx, de PHP-FPM, de nom de domaine réel ni de déploiement automatique.
- **Aucune nouvelle dépendance.** Aucune action n'ajoute Composer, npm, un framework ou un logiciel à installer.
  Tout se fait avec PHP, HTML, CSS, JavaScript et le fichier `.htaccess`, déjà utilisés dans le site.
  Aucune action n'a donc besoin d'être marquée « sous accord du professeur ».
- **Aucun paiement.** Le site ne comporte pas de paiement et aucune action n'en ajoute.
- **Stockage en fichiers JSON : ce n'est pas un défaut.** Il convient à ce projet et toutes les actions le conservent.
  Une base SQLite est seulement proposée en option (action D4), à valider avec votre professeur.
- **Une action à la fois**, testée dans le navigateur avant de passer à la suivante.

## 2. Comment lire ce plan

| Colonne | Signification |
|---|---|
| **Priorité** | **P0** = critique, à faire en premier. **P1** = important. **P2** = optionnel, si vous avez le temps |
| **Complexité** | **simple** : quelques lignes dans 1 ou 2 fichiers. **moyen** : plusieurs fichiers ou un peu de logique. **complexe** : plusieurs heures, nouvelle fonctionnalité |
| **Critère d'acceptation** | Test précis que vous faites vous-même pour vérifier que l'action est réussie |
| **Fichiers concernés** | Fichiers de votre site à modifier ou à créer (« nouveau ») |
| **Fait** | Cochez `[x]` quand la tâche respecte la [Définition de Terminé](#4-définition-de-terminé) |

---

## 3. Actions par thème

### 3.1 Sécurité

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **S1** | **Ajouter un champ piège (*honeypot*) au formulaire de contact.** Ce champ est invisible pour les visiteurs ; un robot le remplit. Si le champ est rempli, le message n'est pas enregistré. | L1 | **P0** | simple | 1) Un message normal est toujours enregistré et apparaît dans Maintenance > Journaux > Messages de contact. 2) Après avoir rempli le champ piège avec l'outil développeur (F12), l'envoi n'est pas enregistré. 3) Le champ n'est ni visible, ni atteignable avec la touche Tab. | `contact.php`, `style.css` |
| [ ] | **S2** | **Protéger les codes de réinitialisation écrits dans `mail.log`.** Dans la copie de débogage d'un mail bien envoyé, remplacer le code par `********`. Ajouter un bouton « Vider mail.log » (avec fenêtre de confirmation) dans Maintenance > Journaux. Sans service d'envoi ou en cas d'échec, le code reste dans `mail.log` : c'est le seul moyen de le récupérer. | L2 | **P0** | moyen | 1) Brevo configuré et débogage coché temporairement : après « Mot de passe oublié », la dernière entrée de `storage/logs/mail.log` affiche `********` au lieu du code. 2) Le code reçu par e-mail fonctionne toujours. 3) « Vider mail.log » demande une confirmation, affiche un message vert, et le fichier est vide. 4) L'événement « Journal vidé » apparaît dans le journal de connexion. | `admin/forgot.php`, `admin/includes/mailer.php`, `admin/includes/maintenance.php`, `admin/maintenance.php` |
| [ ] | **S3** | **Limiter le nombre de messages de contact par adresse IP** (par exemple 5 messages en 10 minutes), avec un compteur dans un fichier JSON de `storage/security/`. | L1 | P1 | moyen | 1) Les 5 premiers messages sont enregistrés. 2) Le 6e envoi en moins de 10 minutes affiche un message d'erreur clair et n'est pas enregistré. 3) Après 10 minutes, l'envoi fonctionne de nouveau. | `contact.php` |
| [ ] | **S4** | **Masquer les informations techniques** dans le `.htaccess` racine : interdire la liste des fichiers des dossiers (`Options -Indexes`), refuser l'accès aux fichiers `.md`, retirer l'en-tête `X-Powered-By` (module `mod_headers`, déjà actif dans WAMP). | L3 | P1 | simple | 1) `/assets/` et `/admin/assets/` affichent « Forbidden » au lieu de la liste des fichiers. 2) `/README.md` est refusé. 3) Dans F12 > Réseau, la réponse de `index.php` n'a plus d'en-tête `X-Powered-By`. 4) Images, CSS et JavaScript se chargent toujours. | `.htaccess` |
| [ ] | **S5** | **Envoyer des en-têtes de sécurité sur les pages publiques**, depuis le `.htaccess` racine, avec les mêmes valeurs que le panel : `X-Content-Type-Options: nosniff`, `X-Frame-Options: DENY`, `Referrer-Policy: same-origin`. | L4 | P1 | simple | 1) Dans F12 > Réseau, `index.php`, `products.php` et `contact.php` envoient ces 3 en-têtes. 2) Le panel fonctionne toujours et garde ses propres en-têtes. | `.htaccess` |
| [ ] | **S6** | **Ne plus afficher les erreurs PHP dans les pages** (`php_flag display_errors off` dans le `.htaccess`). Les erreurs restent enregistrées dans le journal de WAMP (menu WAMP > PHP > PHP error log). À faire en fin de développement : l'affichage des erreurs aide à déboguer. | L5 | P2 | simple | Avec un fichier de test temporaire contenant `<?php echo $inexistante;`, la page n'affiche aucun message et l'erreur figure dans `c:/wamp64/logs/php_error.log`. Supprimez ensuite le fichier de test. | `.htaccess` |
| [ ] | **S7** | **Verrouiller la page d'installation.** Créer un fichier « verrou » dans `storage/config/` lors de la création du compte. S'il existe, `admin/setup.php` refuse l'accès et explique la marche à suivre. | L8 | P2 | simple | Sur une copie de test de `storage/`, après avoir renommé `admin.json`, `admin/setup.php` n'affiche pas le formulaire mais un message explicatif. Remettez ensuite vos vraies données. | `admin/setup.php`, `admin/includes/bootstrap.php` |
| [ ] | **S8** | **Ajouter un en-tête `Content-Security-Policy`**, qui indique au navigateur les seules sources autorisées pour les scripts et les styles (ici : le site lui-même). À faire après **Q5**. | L4 | P2 | moyen | 1) L'en-tête est visible dans F12 > Réseau. 2) Les 3 pages publiques et tous les modules du panel fonctionnent, sans erreur dans la console (F12 > Console). | `.htaccess` |
| [ ] | **S9** | **Compléter l'anti-brute force par un compteur sur le compte** : en plus du compteur par adresse IP, compter les échecs sur l'adresse email visée, avec un blocage court (par exemple 15 minutes). | L6 | P2 | moyen | Après 8 échecs sur l'email du compte, la connexion est refusée pendant 15 minutes, même depuis une autre adresse (testez avec `http://localhost` puis `http://127.0.0.1`). | `admin/includes/ratelimit.php`, `admin/index.php`, `admin/includes/bootstrap.php` |

### 3.2 Stockage des données

Le stockage actuel en fichiers JSON est adapté au projet. Les actions D1 à D3 le conservent et le rendent plus pratique.

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **D1** | **Stocker les prix en nombre** (ex. `149.99`) et les afficher en euros avec `number_format()`. | L15 | P1 | simple | La page Nos Produits affiche exactement les mêmes prix qu'avant (ex. « 149,99 € »), et `products.json` ne contient plus de symbole « € ». | `products.json`, `products.php` |
| [ ] | **D2** | **Limiter la taille des journaux.** Quand `auth.log`, `contact.log` ou `mail.log` dépasse 1 Mo, le renommer avec la date (ex. `auth-2026-10-15.log`) et repartir d'un fichier vide. | L12 | P1 | moyen | Avec une copie de test d'un journal de plus de 1 Mo, le prochain événement crée l'archive datée et un nouveau fichier ; la Maintenance s'affiche normalement. | `admin/includes/storage.php`, `admin/includes/mailer.php`, `contact.php` |
| [ ] | **D3** | **Gérer les produits depuis le panel**, toujours dans `products.json` : liste, ajout (image choisie parmi celles de `assets/products/`), suppression avec fenêtre de confirmation. | L14 | P2 | complexe | Un produit ajouté dans le panel apparaît sur la page Nos Produits ; un produit supprimé disparaît après confirmation ; chaque action affiche un message vert ou rouge. | Nouveau `admin/products.php`, `admin/panel.php` (5e outil), `products.json` |
| [ ] | **D4** | **Option : passer à une base SQLite.** SQLite est une base de données contenue dans un seul fichier, intégrée à PHP (extension `pdo_sqlite`, déjà fournie avec WAMP) : rien à installer. **À valider avec votre professeur.** | — | P2 | complexe | Le fichier de base se trouve dans `storage/` ; il n'est pas accessible depuis le navigateur ; les pages affichent les mêmes données qu'avant ; « Purger storage/tmp » ne le supprime pas. | `products.php`, `contact.php`, `admin/includes/` (nouveau fichier d'accès à la base) |

### 3.3 Qualité du code

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **Q1** | **Rediriger après l'envoi du formulaire de contact** : après l'enregistrement, rediriger vers `contact.php?envoye=1`, qui affiche le message de succès. | L9 | P1 | simple | Après un envoi réussi, actualiser la page (F5) ne renvoie pas le formulaire, et le message n'apparaît qu'une fois dans la Maintenance. | `contact.php` |
| [ ] | **Q2** | **Écrire une fiche de recette** : la liste des tests à refaire après chaque modification (pages publiques, formulaire de contact, connexion et anti-brute force, 4 modules), avec le résultat attendu pour chacun. | L11 | P1 | simple | En suivant la fiche, vous vérifiez tout le site en une dizaine de minutes ; chaque ligne indique l'action à faire et le résultat attendu. | Nouveau `RECETTE.md` |
| [ ] | **Q3** | **Écrire l'en-tête, le menu et le pied de page une seule fois** dans deux fichiers inclus par les 3 pages publiques (`include`). La page active du menu est indiquée par une variable. | L10 | P1 | moyen | 1) Les 3 pages s'affichent comme avant, avec le bon lien actif. 2) Un lien modifié dans le pied de page change sur les 3 pages. 3) Les fichiers inclus ne s'ouvrent pas seuls dans le navigateur (403). | `index.php`, `products.php`, `contact.php`, nouveaux `includes/header.php`, `includes/footer.php`, `includes/.htaccess` |
| [ ] | **Q4** | **Définir le chemin de `contact.log` à un seul endroit**, dans un petit fichier de configuration commun à `contact.php` et au panel. À faire après **Q3**. | L10 | P2 | simple | Le chemin n'apparaît plus qu'une fois dans le code ; un message envoyé apparaît toujours dans la Maintenance. | `contact.php`, `admin/includes/maintenance.php`, nouveau `includes/config.php` |
| [ ] | **Q5** | **Retirer le JavaScript écrit dans le HTML** : remplacer `onsubmit="return validateForm()"` par un écouteur d'événement dans `script.js`. | L4 | P2 | simple | Le formulaire de contact refuse toujours un envoi incomplet avec les mêmes messages, et `contact.php` ne contient plus d'attribut `onsubmit`. | `contact.php`, `script.js` |
| [ ] | **Q6** | **Créer un script de vérification automatique**, en PHP, lancé en ligne de commande. Il vérifie que chaque page répond avec le bon code : 200 pour les pages, redirection vers la connexion pour le panel, 403 pour `storage/`. | L11 | P2 | moyen | La commande `C:\wamp64\bin\php\php8.3.28\php.exe tests\verifier.php` affiche une ligne « OK » par contrôle et un total. Si vous cassez volontairement une page, le script l'indique en « ÉCHEC ». | Nouveaux `tests/verifier.php`, `tests/.htaccess` |

### 3.4 Front-end et accessibilité

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **F1** | **Corriger les contrastes de couleur.** Au survol, garder le bouton principal dans une couleur foncée au lieu du vert clair ; foncer le texte des pastilles « À surveiller » ; foncer les bordures des champs. | L18 | P1 | simple | Avec un vérificateur de contraste (ex. outil développeur du navigateur), on obtient au moins 4,5:1 pour les textes et 3:1 pour les bordures de champ. | `style.css`, `admin/assets/admin.css` |
| [ ] | **F2** | **Agrandir les zones cliquables** : bouton « Afficher », lien « Mot de passe oublié ? », liens du pied de page, bouton du menu mobile. | L19 | P1 | simple | Dans l'outil développeur, chacun de ces éléments mesure au moins 44 px de haut (ou 24 px au minimum avec un espacement suffisant). | `style.css`, `admin/assets/admin.css` |
| [ ] | **F3** | **Afficher l'erreur sous le champ concerné**, en plus du bandeau qui disparaît après 6 s. Le message sous le champ reste affiché tant que l'erreur n'est pas corrigée. | L20 | P1 | moyen | Dans les modules Mot de passe et Utilisateur, après une erreur de saisie, le bandeau disparaît mais le message reste sous le champ, et le curseur est placé dans le premier champ en erreur. | `admin/assets/admin.js`, `admin/assets/admin.css` |
| [ ] | **F4** | **Respecter le réglage « réduire les animations »** (`prefers-reduced-motion`) : supprimer fondus et défilement animé pour les personnes qui l'ont activé. | L21 | P2 | simple | Avec l'option « Effets d'animation » désactivée dans Windows, les sections apparaissent sans fondu et le lien « En savoir plus » saute directement à la section. | `style.css`, `admin/assets/admin.css`, `script.js` |
| [ ] | **F5** | **Ajouter un lien d'évitement** « Aller au contenu », visible quand on navigue au clavier (touche Tab). Plus simple après **Q3**. | L22 | P2 | simple | Sur chaque page, la première pression sur Tab fait apparaître le lien ; Entrée amène au contenu principal. | `index.php`, `products.php`, `contact.php` (ou `includes/header.php` après Q3), `admin/includes/layout.php`, `style.css`, `admin/assets/admin.css` |
| [ ] | **F6** | **Rendre les vignettes de catégories cliquables** : chaque vignette devient un lien vers la page produits filtrée (ex. `products.php?q=clavier`) ; `script.js` lit le paramètre `q` et remplit le champ de recherche existant. | L23 | P2 | simple | Un clic sur « Claviers » affiche la page produits avec « clavier » déjà saisi dans la recherche et seuls les claviers visibles. | `index.php`, `script.js`, `style.css` |
| [ ] | **F7** | **Préremplir le formulaire de contact** depuis le lien « Un conseil ? » avec le nom du produit. | L16 | P2 | simple | Après un clic sur « Un conseil ? » d'un produit, le message commence par « Au sujet de : » suivi du nom du produit. | `products.php`, `contact.php` |

### 3.5 Conformité et RGPD

Le **RGPD** (Règlement général sur la protection des données) impose d'informer les personnes dont on enregistre les données,
de n'en garder que le nécessaire et de ne pas les conserver indéfiniment.
Le formulaire de contact enregistre un nom, un email, un message et une adresse IP (`contact.php`).

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **R1** | **Informer les visiteurs** : courte mention sous le formulaire de contact (données collectées, usage, durée de conservation) et page « Mentions légales et confidentialité » liée dans le pied de page. | L17 | P1 | simple | La mention est visible sous le formulaire ; le lien du pied de page ouvre la nouvelle page depuis les 3 pages du site. | `contact.php`, `index.php`, `products.php` (ou `includes/footer.php` après Q3), nouveau `mentions-legales.php` |
| [ ] | **R2** | **Ne plus enregistrer l'adresse IP** des visiteurs avec leur message : elle n'est pas utile pour y répondre. | L17 | P2 | simple | Un nouveau message de contact apparaît dans la Maintenance sans adresse IP, et sa ligne dans `contact.log` ne contient plus de champ `ip`. | `contact.php`, `admin/maintenance.php` |
| [ ] | **R3** | **Limiter la durée de conservation des messages** (par exemple 12 mois) : bouton « Supprimer les messages de plus de 12 mois » dans la Maintenance, avec fenêtre de confirmation. | L17 | P2 | moyen | Sur une copie de test de `contact.log` contenant un message daté de plus de 12 mois, le bouton le supprime après confirmation ; les messages récents restent. | `admin/includes/maintenance.php`, `admin/maintenance.php` |
| [ ] | **R4** | **Documenter le cookie.** Le seul cookie, `IL_ADMIN`, sert à la connexion au panel : il est strictement nécessaire, donc aucun bandeau de consentement n'est requis. L'indiquer dans la page créée en **R1**. | L17 | P2 | simple | La page « Mentions légales et confidentialité » mentionne le cookie `IL_ADMIN`, son rôle et sa durée (fermeture du navigateur). | `mentions-legales.php` |

### 3.6 Documentation

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **DOC1** | **Expliquer la sauvegarde et la restauration** de `storage/` dans le `README.md` : quels fichiers copier, comment les remettre, pourquoi `storage/` n'est pas dans Git, et pourquoi `app.key` est indispensable pour relire la clé SMTP. | L7, L13 | P1 | simple | En suivant uniquement le README, vous copiez `storage/` dans un dossier de sauvegarde, puis vous le restaurez ; la connexion au panel et le test d'envoi d'e-mail fonctionnent toujours. | `README.md` |
| [ ] | **DOC2** | **Mettre à jour `DOCUMENTATION.md`** après chaque action terminée (fonctionnement, sécurité, limites). | toutes | P1 | simple | Pour chaque action cochée, la documentation décrit le nouveau comportement, et la limite correspondante est retirée ou modifiée. | `DOCUMENTATION.md` |
| [ ] | **DOC3** | **Tenir un journal des modifications** : date, action, fichiers modifiés. | — | P2 | simple | Chaque action terminée a une ligne datée dans le journal. | Nouveau `CHANGELOG.md` |

### 3.7 Exploitation locale

| Fait | ID | Action | Réf. | Priorité | Complexité | Critère d'acceptation | Fichiers concernés |
|---|---|---|---|---|---|---|---|
| [ ] | **E1** | **Surveiller la taille des journaux** dans Maintenance > Système : une ligne par journal (`auth.log`, `contact.log`, `mail.log`) avec sa taille, et la pastille « À surveiller » au-delà de 1 Mo. | L12 | P1 | simple | La section Système affiche les 3 journaux et leur taille ; avec une copie de test de plus de 1 Mo, la pastille passe à « À surveiller ». | `admin/includes/maintenance.php` |
| [ ] | **E2** | **Consulter `mail.log` dans le panel** : nouvel onglet « E-mails » dans Maintenance > Journaux. | L2 | P2 | moyen | L'onglet affiche les dernières entrées de `mail.log`, de la plus récente à la plus ancienne ; un fichier vide affiche « Aucune entrée ». | `admin/includes/maintenance.php`, `admin/maintenance.php` |
| [ ] | **E3** | **Compléter la sauvegarde JSON** avec les messages de contact (option à cocher dans la fenêtre d'export). | L13 | P2 | moyen | Avec l'option cochée, le fichier exporté contient une rubrique « messages_de_contact » ; sans l'option, il est identique à aujourd'hui ; aucun secret n'y apparaît. | `admin/includes/maintenance.php`, `admin/maintenance.php` |

---

## 4. Définition de Terminé

Une tâche est **terminée** seulement si **toutes** les conditions suivantes sont vraies :

1. **Le critère d'acceptation** de la tâche est vérifié dans votre navigateur, sur `http://localhost/infoleader` ou `http://infoleader.local`.
2. **Aucune régression** (aucune fonctionnalité existante cassée). Vous avez vérifié :
   - les 3 pages publiques s'affichent et le formulaire de contact enregistre un message ;
   - la connexion au panel fonctionne, avec l'attente de 3 s après une erreur ;
   - les 4 modules (Configuration des e-mails, Mot de passe, Utilisateur, Maintenance) s'ouvrent et fonctionnent.
3. **Aucune erreur** : aucun message PHP dans les pages, rien de nouveau dans la console du navigateur (F12 > Console)
   ni dans le journal d'erreurs de WAMP (menu WAMP > PHP > PHP error log).
4. **Les tests sont faits sur des données d'essai** : copiez le dossier `storage/` avant de tester, puis remettez vos vraies données.
5. **Aucune nouvelle dépendance** n'a été ajoutée, et le site fonctionne toujours uniquement sur WAMP.
6. **Le code est commenté** en français, comme le reste du projet.
7. **Toute action destructive** du panel (suppression, purge) passe par une fenêtre de confirmation,
   et chaque action affiche un message vert (succès) ou rouge (erreur).
8. **La documentation est à jour** (`DOCUMENTATION.md`, et `README.md` si besoin).
9. **La tâche est cochée** dans ce fichier.
10. **Un commit Git** décrit la modification (ex. « S1 : champ piège sur le formulaire de contact ») ;
    `storage/` n'y figure pas (il est exclu par `.gitignore`).

---

## 5. Séquencement par jalons

Un **jalon** est une étape du projet. Terminez un jalon avant de commencer le suivant.

| Jalon | Objectif | Actions, dans l'ordre | Pourquoi maintenant |
|---|---|---|---|
| **Jalon 1 — Corriger le critique** | Supprimer les deux risques sérieux | **S1**, **S2** | Formulaire public ouvert aux robots ; codes de réinitialisation lisibles dans un fichier |
| **Jalon 2 — Sécuriser et fiabiliser** | Disposer d'une fiche de tests, puis fermer les points faibles simples | **Q2**, **Q1**, **S3**, **S4**, **S5**, **E1**, **DOC1** | La fiche de recette sert à vérifier toutes les tâches suivantes ; les autres actions sont courtes et réduisent les risques |
| **Jalon 3 — Code, accessibilité et RGPD** | Rendre le site plus simple à maintenir, utilisable par tous et conforme | **Q3**, **F1**, **F2**, **F3**, **R1** | Code dupliqué sur 3 pages ; contrastes et zones cliquables mesurés insuffisants ; information des visiteurs obligatoire |
| **Jalon 4 — Données** | Rendre les données plus pratiques | **D1**, **D2** ; **DOC2** en continu, après chaque tâche | Prix exploitables, journaux de taille maîtrisée |
| **Jalon 5 — Optionnel** | Aller plus loin si le temps le permet | **Q5** puis **S8** ; **Q4** ; **R2**, **R3**, **R4** ; **F4** à **F7** ; **S6**, **S7**, **S9** ; **E2**, **E3** ; **Q6** ; **DOC3** ; **D3** ; **D4** (avec l'accord de votre professeur) | Améliorations utiles mais non bloquantes |

---

## 6. Petit lexique

| Terme | Explication courte |
|---|---|
| **Honeypot** (champ piège) | Champ de formulaire caché aux humains. Un robot le remplit, ce qui permet de rejeter son envoi. |
| **Adresse IP** | Numéro qui identifie un appareil sur le réseau. En local : `127.0.0.1` ou `::1`. |
| **Anti-brute force** | Protection qui ralentit puis bloque les essais répétés de mot de passe. |
| **CSRF** | Attaque où un autre site déclenche une action à votre place. Un jeton secret dans chaque formulaire l'empêche. |
| **`.htaccess`** | Fichier de réglages d'Apache placé dans un dossier du site : il s'applique à ce dossier et à ses sous-dossiers. |
| **En-tête HTTP** | Information envoyée par le serveur avec chaque page (invisible à l'écran, visible dans F12 > Réseau). |
| **Content-Security-Policy (CSP)** | En-tête qui liste les seules sources de scripts et de styles autorisées par le navigateur. |
| **Liste des fichiers d'un dossier** | Page générée par Apache quand un dossier n'a pas de page d'accueil : elle montre tous ses fichiers. |
| **Journal (log)** | Fichier où le site note ce qui se passe : connexions, envois d'e-mails, messages. |
| **Rotation des journaux** | Archiver un journal devenu trop gros et repartir d'un fichier vide. |
| **Redirection après envoi** | Après un formulaire, le serveur renvoie vers une autre adresse : actualiser la page ne renvoie plus le formulaire. |
| **`include`** | Instruction PHP qui insère le contenu d'un autre fichier : un même morceau de page est écrit une seule fois. |
| **Recette** | Série de tests qui vérifie que le site fait bien ce qui est attendu. |
| **Régression** | Fonctionnalité qui marchait et qui ne marche plus après une modification. |
| **Contraste** | Différence de luminosité entre un texte et son fond. Un contraste trop faible rend la lecture difficile. |
| **WCAG** | Règles internationales d'accessibilité des sites web (contrastes, tailles, clavier...). |
| **Lien d'évitement** | Lien caché qui apparaît au clavier et permet d'aller directement au contenu. |
| **`prefers-reduced-motion`** | Réglage du système qui demande aux sites de limiter les animations. |
| **RGPD** | Règlement européen sur la protection des données personnelles. |
| **SQLite** | Base de données contenue dans un seul fichier, intégrée à PHP, sans serveur à installer. |
| **Commit** | Enregistrement d'une étape du projet dans l'historique Git. |
