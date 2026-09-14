# Spécifications du projet Intranet V1

## 1. Présentation

Intranet V1 est une application web Django monolithique destinée à centraliser des contenus internes, des ressources documentaires, des formulaires, un annuaire et des indicateurs d'activité.

Le dépôt contient :

- un projet Django `intranet` ;
- une application principale `accounts` ;
- une interface Bootstrap personnalisée via `intranet/static/css/app.css` ;
- un JavaScript léger pour le thème dans `intranet/static/js/theme.js`.

Fonctionnalités principales actuellement implémentées :

- fil d'actualité avec publications validées ;
- proposition, brouillons et modération de publications ;
- likes et commentaires sur les publications ;
- bibliothèque de documents groupés par thèmes ;
- bibliothèque de formulaires groupés par thèmes ;
- boîte à idées avec likes, validation staff et espace personnel ;
- annuaire utilisateur avec export XLSX et import CSV/XLS/XLSX ;
- espace personnel utilisateur ;
- indicateurs par société ;
- authentification locale Django et LDAP Active Directory.

## 2. Architecture technique

### 2.1 Stack

- Python ;
- Django `6.0.4` ;
- PostgreSQL par défaut ;
- SQLite possible en local ou en test via `DB_ENGINE=django.db.backends.sqlite3` ;
- Bootstrap `5.3.3` via CDN ;
- WhiteNoise pour les statiques ;
- `openpyxl` pour l'export/import XLSX ;
- `xlrd` pour l'import XLS ;
- `markdown` + `bleach` pour le rendu Markdown sécurisé ;
- `django-python3-ldap` + `ldap3` pour l'authentification LDAP.

### 2.2 Dépendances Python déclarées

`requirements.txt` déclare actuellement :

- `asgiref`
- `bleach`
- `Django`
- `django-python3-ldap`
- `et_xmlfile`
- `ldap3`
- `Markdown`
- `openpyxl`
- `psycopg`
- `psycopg-binary`
- `pyasn1`
- `pyodbc`
- `python-dotenv`
- `sqlparse`
- `tzdata`
- `webencodings`
- `whitenoise`
- `xlrd`

### 2.3 Configuration générale

- `LOGIN_URL = 'login'`
- `LOGIN_REDIRECT_URL = 'home'`
- `LOGOUT_REDIRECT_URL = '/'`
- fichiers statiques servis via WhiteNoise ;
- médias servis via `/media/...` par une route Django explicite ;
- `AccountsConfig.ready()` crée automatiquement les sous-répertoires médias nécessaires.

Répertoires médias créés automatiquement :

- `documents/`
- `formulaires/`
- `publications/`
- `publications/details/`
- `users/profile_pictures/`

## 3. Authentification et sécurité

### 3.1 Accès global

L'application est conçue pour être utilisée par des utilisateurs authentifiés. La page d'accueil `/` est protégée.

Routes publiques :

- `/accounts/login/`

Routes protégées :

- `/`
- toutes les routes `/accounts/...` sauf la connexion.

### 3.2 Authentification locale et LDAP

L'authentification repose sur deux backends :

1. `accounts.auth.ActiveDirectoryLDAPBackend`
2. `django.contrib.auth.backends.ModelBackend`

Le formulaire de connexion personnalisé :

- accepte un identifiant simple ;
- accepte un identifiant `DOMAINE\login` ;
- accepte un identifiant `login@domaine.local` ;
- extrait le `sAMAccountName` local ;
- tente d'abord les variantes locales plausibles ;
- retourne un message spécifique si l'utilisateur LDAP n'existe pas ;
- retourne un message spécifique si LDAP est indisponible et que l'authentification locale échoue.

Règles LDAP importantes :

- le compte Django local est recherché par email (`LDAP_AUTH_USER_LOOKUP_FIELDS = ('email',)`) ;
- LDAP alimente `username`, `first_name`, `last_name` et `email` ;
- `LDAP_AUTH_FORMAT_USERNAME` permet de convertir un identifiant simple en UPN si nécessaire ;
- `LDAP_AUTH_REQUIRED_GROUP_DN`, lorsqu'il est défini, est ajouté au filtre LDAP ;
- les comptes locaux `staff` et `superuser` restent utilisables même si LDAP est absent ou indisponible.

### 3.3 Déconnexion

La déconnexion repose sur `LogoutView` et redirige vers `/`. Comme `/` est protégé, l'utilisateur non authentifié est ensuite redirigé vers la page de connexion.

## 4. Rôles et permissions

### 4.1 Utilisateur authentifié

Un utilisateur authentifié peut :

- consulter le fil d'actualité ;
- rechercher dans les publications ;
- proposer une publication ;
- consulter ses brouillons de publication ;
- modifier ou supprimer ses propres brouillons ;
- liker une publication ;
- commenter une publication publiée ;
- supprimer ses propres commentaires ;
- consulter les documents autorisés pour sa société ;
- consulter les formulaires autorisés pour sa société ;
- consulter la boîte à idées ;
- proposer une idée ;
- consulter ses propres idées ;
- modifier ses propres idées ;
- supprimer ses propres idées ;
- liker une idée ;
- consulter l'annuaire ;
- consulter le détail d'un profil ;
- exporter l'annuaire en XLSX ;
- consulter les indicateurs ;
- modifier son espace personnel.

### 4.2 Utilisateur staff

Un utilisateur staff possède tous les droits utilisateur standard et peut aussi :

- accéder à l'administration Django ;
- accéder à la modération des publications ;
- accepter ou refuser une publication proposée ;
- modifier ou supprimer une publication publiée ;
- supprimer tous les commentaires de publication ;
- créer, modifier et supprimer des documents ;
- créer, modifier et supprimer des formulaires ;
- importer un annuaire CSV/XLS/XLSX ;
- modifier les indicateurs ;
- supprimer toute idée ;
- valider une idée.

### 4.3 Superuser

Un superuser est un compte local privilégié. Il doit pouvoir se connecter sans dépendre du bon fonctionnement de LDAP.

## 5. Navigation et interface

### 5.1 Layout général

L'interface authentifiée comporte :

- une topbar fixe ;
- une sidebar desktop ;
- une sidebar mobile offcanvas ;
- un accès à l'espace personnel via la photo de profil ;
- un bouton de déconnexion ;
- un bouton de changement de thème ;
- un lien vers l'administration pour les utilisateurs staff.

### 5.2 Thème

Le thème clair/sombre :

- est stocké dans `localStorage` sous la clé `theme` ;
- repose sur `data-bs-theme` sur l'élément `<html>` ;
- est initialisé dès le chargement de la page ;
- est basculé côté client par `intranet/static/js/theme.js`.

Le bouton de thème met à jour son état ARIA et son pictogramme :

- `☀️` en mode clair ;
- `🌙` en mode sombre.

### 5.3 Navigation active

La navigation latérale conserve l'entrée parente active sur les écrans de détail. Exemple :

- une publication publiée laisse `Fil d'actualité` actif ;
- une publication en modération laisse `Modération` actif ;
- une idée détaillée laisse `Boîte à idées` actif ;
- un profil annuaire laisse `Annuaire` actif.

### 5.4 Bouton flottant de retour

Certaines pages affichent un bouton flottant de retour en bas à gauche de l'écran :

- bouton rond ;
- dégradé orange ;
- icône `retour.png` ;
- affichage conditionné à la présence de `floating_back_href` dans le contexte.

Ce bouton est utilisé notamment sur :

- détail de publication publiée ;
- détail de publication en modération ;
- liste des brouillons de publication ;
- détail d'un brouillon de publication ;
- détail et suppression d'idée ;
- liste `Mes idées` ;
- détail annuaire.

### 5.5 Branding actuel

Dans l'état actuel du code :

- le header affiche le logo `LOGO-CELSIUS_blanc.png` ;
- le libellé affiché est `Intranet` ;
- le logo renvoie vers l'accueil.

## 6. Modèle de données

### 6.1 Publication

Champs :

- `title` : `CharField(180)`
- `content` : `TextField`
- `image` : image principale facultative
- `author` : auteur
- `created_at`
- `publie`
- `published_at`
- `refused_at`
- `moderated_by`

Règles :

- une publication créée via l'interface utilisateur n'est pas publiée par défaut ;
- `accept(user)` positionne `publie=True`, renseigne `published_at`, remet `refused_at` à `None` et enregistre `moderated_by` ;
- `refuse(user)` positionne `publie=False`, renseigne `refused_at` et `moderated_by` ;
- ordre par défaut du modèle : `-published_at`, puis `-created_at`.

Extensions image autorisées :

- `jpg`
- `jpeg`
- `png`
- `gif`
- `webp`

### 6.2 PublicationDetailImage

Image complémentaire liée à une publication.

Champs :

- `publication`
- `image`
- `created_at`

Règles :

- une publication peut avoir plusieurs images complémentaires ;
- ordre du modèle : `id` croissant ;
- les images complémentaires sont créées une par une à l'enregistrement du formulaire.

### 6.3 PublicationLike

Champs :

- `publication`
- `user`
- `created_at`

Règles :

- unicité sur `(publication, user)` ;
- le clic de toggle ajoute ou supprime le like ;
- ordre du modèle : `-created_at`.

### 6.4 PublicationCommentaire

Champs :

- `publication`
- `user`
- `content`
- `created_at`

Règles :

- les commentaires sont associés à une publication publiée dans la vue publique ;
- la liste visible est triée du plus récent au plus ancien ;
- la page détail affiche 3 commentaires par défaut ;
- `?commentaires=all` affiche l'intégralité.

### 6.5 ThemeDocument

Champs :

- `title` : unique
- `author`
- `created_at`

Ordre : `title`.

### 6.6 Document

Champs :

- `title`
- `is_common`
- `societes`
- `theme`
- `file`
- `file_hash`
- `uploaded_by`
- `created_at`

Sociétés disponibles :

- `Alain Postic`
- `CBSM`
- `LFPB`
- `TSO`

Extensions autorisées :

- `pdf`
- `docx`
- `doc`
- `xls`
- `xlsx`

Règles :

- ordre du modèle : `title` ;
- `is_common=True` rend le document visible à tous ;
- `is_common=False` active le filtrage par une ou plusieurs sociétés ;
- `Commun` est un mode de visibilité, pas une société ;
- le hash SHA-256 est calculé à la sauvegarde ;
- le hash est recalculé si le fichier change ;
- `file_hash` est unique ;
- la propriété `extensions` retourne l'extension réelle du fichier.

### 6.7 ThemeFormulaire

Champs :

- `title` : unique
- `author`
- `created_at`

Ordre : `title`.

### 6.8 Formulaire

Champs :

- `title`
- `is_common`
- `societes`
- `theme`
- `file`
- `file_hash`
- `uploaded_by`
- `created_at`

Sociétés disponibles :

- `Alain Postic`
- `CBSM`
- `LFPB`
- `TSO`

Extensions autorisées :

- `doc`
- `docx`

Règles :

- ordre du modèle : `title` ;
- `is_common=True` rend le formulaire visible à tous ;
- `is_common=False` active le filtrage par une ou plusieurs sociétés ;
- `Commun` est un mode de visibilité, pas une société ;
- le hash SHA-256 est calculé à la sauvegarde ;
- le hash est recalculé si le fichier change ;
- `file_hash` est unique.

### 6.9 Idee

Champs :

- `title`
- `content`
- `uploaded_by`
- `created_at`
- `validate`

Règles :

- ordre du modèle : `title` ;
- les vues principales réordonnent les idées par `-created_at` ;
- `validate` indique la validation staff.

### 6.10 IdeeLike

Champs :

- `idee`
- `user`
- `created_at`

Règles :

- unicité sur `(idee, user)` ;
- ordre du modèle : `-created_at` ;
- le clic de toggle ajoute ou supprime le like.

### 6.11 UserProfile

Champs :

- `user`
- `profile_picture`
- `default_photo` : `man` ou `woman`
- `societe`
- `poste`
- `tel_poste`
- `sda`
- `tel_mobile`
- `birthday`
- `bio`

Règles :

- un profil est créé automatiquement via signal `post_save` à la création d'un utilisateur ;
- la photo de profil réutilise les mêmes extensions image autorisées que les publications ;
- `default_profile_picture_path` renvoie :
  - `img/base_user.png` pour `man`
  - `img/base_user_woman.png` pour `woman`

### 6.12 Indicateur

Champs :

- `nom`
- `societe`
- `valeur`
- `explication`
- `updated_at`

Types possibles :

- `effectif`
- `absenteisme`
- `taux_at_mp`

Sociétés possibles :

- `Alain Postic`
- `CBSM`
- `LFPB`
- `TSO`

Règles :

- unicité sur `(nom, societe)` ;
- ordre du modèle : `societe`, puis `nom` ;
- `valeur` est un `DecimalField(max_digits=8, decimal_places=2)`.

## 7. Parcours fonctionnels

### 7.1 Fil d'actualité

Route : `/`

Fonctionnalités :

- liste les publications publiées ;
- pagination par 6 ;
- recherche via le paramètre `p` sur le titre et le contenu ;
- affiche le nombre total de publications publiées ;
- affiche les anniversaires du jour ;
- permet d'accéder à `Mes propositions` ;
- permet de proposer une publication.

Sur les cartes :

- image principale ou placeholder `No_Image_Placeholder.png` ;
- date de publication ;
- auteur ;
- compteur de likes si non nul ;
- extrait Markdown tronqué.

### 7.2 Proposition de publication

Route : `/accounts/publications/proposer/`

Fonctionnalités :

- formulaire accessible à tout utilisateur authentifié ;
- champs : titre, contenu, image principale, images supplémentaires ;
- le contenu est saisi en Markdown ;
- les images supplémentaires sont multi-fichiers ;
- l'auteur est l'utilisateur connecté ;
- les images complémentaires sont créées après sauvegarde ;
- message de succès après soumission ;
- redirection vers l'accueil.

### 7.3 Détail d'une publication publiée

Route : `/accounts/publications/<pk>/`

Fonctionnalités :

- affiche le titre, le contenu, les métadonnées et les images ;
- agrège image principale + images complémentaires via `get_publication_images(...)` ;
- si plus d'une image est disponible, une modale Bootstrap avec carousel est proposée ;
- affiche le bouton de like seulement si l'utilisateur n'est pas l'auteur ;
- affiche le compteur de likes si non nul ;
- affiche les actions staff `Modifier la publication` et `Supprimer la publication` ;
- affiche un bouton flottant de retour vers `/`.

Commentaires :

- formulaire de commentaire sur la page ;
- affichage de 3 commentaires par défaut ;
- lien `Charger plus` avec `?commentaires=all#commentaires` ;
- suppression possible pour l'auteur du commentaire ou un staff ;
- retour après création ou suppression sur l'ancre `#commentaires`.

### 7.4 Modération des publications

Routes :

- `/accounts/publications/moderation/`
- `/accounts/publications/<pk>/moderation/`
- `/accounts/publications/<pk>/accepter/`
- `/accounts/publications/<pk>/refuser/`

Accès : staff uniquement.

Fonctionnalités :

- liste les publications `publie=False` et `refused_at IS NULL` ;
- compte les éléments à traiter ;
- permet d'ouvrir le détail ;
- permet d'accepter ou refuser directement depuis la liste ;
- le détail réutilise le template de publication avec un mode `is_moderation_preview` ;
- le détail affiche un bouton flottant de retour vers la liste de modération.

### 7.5 Brouillons de publication

Routes :

- `/accounts/publications/brouillons/`
- `/accounts/publications/brouillons/<pk>`
- `/accounts/publications/brouillons/<pk>/modifier/`
- `/accounts/publications/brouillons/<pk>/supprimer/`

Fonctionnalités :

- liste les publications de l'utilisateur courant encore non publiées et non refusées ;
- détail accessible uniquement à l'auteur ;
- modification du titre, du contenu, de l'image principale et des images complémentaires ;
- suppression sélective d'images complémentaires existantes lors de la modification ;
- suppression après page de confirmation ;
- bouton flottant vers `/` sur la liste ;
- bouton flottant vers `/accounts/publications/brouillons/` sur le détail.

### 7.6 Documents

Routes :

- `/accounts/documents/`
- `/accounts/documents/ajouter/`
- `/accounts/documents/<pk>/modifier/`
- `/accounts/documents/<pk>/supprimer/`

Liste :

- accessible aux utilisateurs authentifiés ;
- groupement par thème ;
- recherche par titre via `q` ;
- sections repliables par thème ;
- sections ouvertes automatiquement si une recherche est active ;
- affichage du nombre de documents visibles dans le contexte courant ;
- filtrage par société pour les non-staff :
  - visible si `is_common=True`
  - ou si `is_common=False` et qu'au moins une société associée correspond à la société du profil utilisateur
- aucun filtrage pour les staff ;
- ouverture des fichiers dans un nouvel onglet ;
- icône adaptée à l'extension.

Création / modification :

- staff uniquement ;
- menu de visibilité `Commun` ou `Filtrer` ;
- si `Filtrer` est choisi, sélection d'une ou plusieurs sociétés ;
- choix d'un thème existant ou création d'un nouveau thème via l'option `Nouveau theme` ;
- affichage conditionnel du champ `new_theme_title` côté client ;
- détection des doublons par hash de fichier.

Suppression :

- page de confirmation générique.

### 7.7 Formulaires

Routes :

- `/accounts/formulaires/`
- `/accounts/formulaires/ajouter/`
- `/accounts/formulaires/<pk>/modifier/`
- `/accounts/formulaires/<pk>/supprimer/`

Liste :

- accessible aux utilisateurs authentifiés ;
- groupement par thème ;
- recherche par titre via `q` ;
- sections repliables par thème ;
- filtrage par société identique à celui des documents ;
- ouverture des fichiers dans un nouvel onglet.

Création / modification :

- staff uniquement ;
- menu de visibilité `Commun` ou `Filtrer` ;
- si `Filtrer` est choisi, sélection d'une ou plusieurs sociétés ;
- même logique de thème que pour les documents ;
- détection des doublons par hash.

### 7.8 Boîte à idées

Routes :

- `/accounts/idee/`
- `/accounts/idee/ajouter/`
- `/accounts/idee/mes_idees/`
- `/accounts/idee/<pk>/`
- `/accounts/idee/<pk>/like/`
- `/accounts/idee/<pk>/modifier/`
- `/accounts/idee/<pk>/supprimer/`
- `/accounts/idee/<pk>/valide/`

Liste :

- pagination par 6 ;
- tri par date décroissante dans la vue ;
- compteur de likes ;
- indicateur visuel si l'idée est validée ;
- accès à `Mes idées` et au formulaire de création.

Détail :

- rendu Markdown du contenu ;
- like possible si l'utilisateur n'est pas l'auteur ;
- actions auteur : modifier, supprimer ;
- action staff : suppression ;
- action staff supplémentaire : validation si l'idée n'est pas encore validée ;
- bouton flottant vers la liste des idées.

Mes idées :

- liste paginée des idées de l'utilisateur ;
- compteur d'idées personnelles ;
- bouton flottant vers la liste générale des idées.

### 7.9 Annuaire

Routes :

- `/accounts/annuaire/`
- `/accounts/annuaire/<pk>/`
- `/accounts/annuaire/export/`
- `/accounts/annuaire/import/`

Liste :

- accessible aux utilisateurs authentifiés ;
- tri par prénom ;
- recherche par prénom, nom ou nom d'utilisateur via `a` ;
- compteur de profils ;
- bouton d'export visible dans le header staff ;
- formulaire d'import visible pour les staff.

Détail :

- affiche société, poste, email, téléphone poste, téléphone fixe, téléphone mobile, anniversaire, biographie ;
- affiche `Non renseigné` pour chaque champ absent ;
- affiche `Aucune information complementaire.` si email, téléphone poste, téléphone fixe, téléphone mobile, anniversaire et biographie sont tous absents ;
- affiche la photo réelle ou l'avatar par défaut ;
- ajoute un bouton flottant vers la liste annuaire.

Copie dans le presse-papiers :

- email, téléphone poste, téléphone fixe et mobile sont rendus comme boutons ;
- clic déclenche une copie via `navigator.clipboard.writeText(...)` si possible ;
- fallback vers `document.execCommand("copy")` sinon ;
- notification toast locale `Copié dans le presse-papiers` ou `Copie impossible` ;
- une icône de copie apparaît au survol ou au focus clavier ;
- l'icône change selon le thème :
  - `copy_dark.png` en mode sombre
  - `copy_light.png` en mode clair

Export XLSX :

- utilise `openpyxl` ;
- nom de fichier `annuaire.xlsx` ;
- onglet `Annuaire` ;
- largeur des colonnes ajustée au contenu ;
- ordre des colonnes :
  1. Nom
  2. Prenom
  3. Nom d'utilisateur
  4. Societe
  5. Poste
  6. Email
  7. Telephone poste
  8. Telephone fixe
  9. Telephone mobile
  10. Anniversaire
  11. Administrateur

Import CSV/XLS/XLSX :

- staff uniquement dans l'interface ;
- POST uniquement ;
- formats acceptés : `.csv`, `.xls`, `.xlsx` ;
- en CSV :
  - UTF-8 avec BOM accepté
  - fallback Windows-1252 accepté
  - séparateur détecté automatiquement entre `;` et `,`
- en XLSX : lecture via `openpyxl`
- en XLS : lecture via `xlrd`
- le fichier doit reprendre exactement les colonnes de l'export dans le même ordre ;
- lignes vides ignorées ;
- ligne sans `username` ignorée ;
- recherche d'un utilisateur existant :
  1. par `username` insensible à la casse
  2. puis par `email` insensible à la casse
- création si aucun utilisateur n'existe ;
- mise à jour seulement avec les valeurs non vides ;
- le champ `Administrateur` accepte notamment `Oui`, `Non`, `Yes`, `No`, `True`, `False`, `1`, `0`, `admin`, `administrateur` ;
- dates d'anniversaire acceptées :
  - `JJ/MM/AAAA`
  - `AAAA-MM-JJ`
  - `JJ-MM-AAAA`

Documentation détaillée annuaire : `import_excel-csv.md`.

### 7.10 Indicateurs

Routes :

- `/accounts/indicateurs/`
- `/accounts/indicateurs/modifier/`

Liste :

- accessible aux utilisateurs authentifiés ;
- affiche les 3 indicateurs pour les 4 sociétés ;
- affiche `0` quand aucune valeur n'existe ;
- `Effectif` est affiché sans décimales ;
- les autres indicateurs sont affichés avec `%` ;
- l'explication éventuelle est affichée sous la valeur ;
- bouton `Gestion des indicateurs` visible pour les staff.

Modification :

- accessible aux staff ;
- formulaire basé sur `IndicateursForm` ;
- si le couple `(nom, societe)` existe, la vue recharge sa valeur et son explication ;
- un script client remplit automatiquement la valeur et l'explication en fonction du couple choisi ;
- la soumission crée ou met à jour l'entrée.

### 7.11 Espace personnel

Route : `/accounts/espace-personnel/`

Fonctionnalités :

- modification de la photo de profil ;
- choix de l'avatar par défaut ;
- modification de la date d'anniversaire ;
- modification de la biographie ;
- le numéro de poste n'est pas éditable par l'utilisateur depuis cet écran ;
- bouton de retour vers l'accueil ;
- message de succès après enregistrement.

## 8. Formulaires applicatifs

### 8.1 PublicationForm

Champs :

- `title`
- `content`
- `image`
- `detail_images`

Particularités :

- `detail_images` accepte plusieurs fichiers ;
- validation des extensions image ;
- aide de saisie Markdown dans le placeholder.

### 8.2 PublicationCommentForm

Champ :

- `content`

Widget :

- `Textarea(rows=4)`.

### 8.3 DocumentForm

Champs :

- `title`
- `societe`
- `theme`
- `new_theme_title`
- `file`

Particularités :

- `theme` est un `ChoiceField` ;
- ajout de l'option technique `__new__` ;
- création du thème si `__new__` est choisi ;
- calcul et contrôle du hash dans `clean_file()`.

### 8.4 FormulaireForm

Champs :

- `title`
- `societe`
- `theme`
- `new_theme_title`
- `file`

Particularités :

- même logique que `DocumentForm` ;
- doublons rejetés par hash.

### 8.5 IdeeForm

Champs :

- `title`
- `content`

Particularités :

- placeholder d'aide Markdown.

### 8.6 EspacePersoForm

Champs :

- `profile_picture`
- `default_photo`
- `birthday`
- `bio`

Particularités :

- `birthday` utilise un `input type="date"`.

### 8.7 IndicateursForm

Champs :

- `nom`
- `societe`
- `valeur`
- `explication`

Particularités :

- les valeurs négatives sont interdites ;
- `validate_unique()` est neutralisé et l'unicité effective est gérée par la vue qui charge l'instance existante.

## 9. URLs principales

| Nom | Méthode | URL | Accès |
| --- | --- | --- | --- |
| `home` | GET | `/` | Authentifié |
| `login` | GET/POST | `/accounts/login/` | Public |
| `logout` | POST | `/accounts/logout/` | Authentifié |
| `document_list` | GET | `/accounts/documents/` | Authentifié |
| `document_create` | GET/POST | `/accounts/documents/ajouter/` | Staff |
| `document_modif` | GET/POST | `/accounts/documents/<pk>/modifier/` | Staff |
| `document_delete` | GET/POST | `/accounts/documents/<pk>/supprimer/` | Staff |
| `publication_create` | GET/POST | `/accounts/publications/proposer/` | Authentifié |
| `publication_detail` | GET/POST | `/accounts/publications/<pk>/` | Authentifié |
| `publication_like` | POST | `/accounts/publications/<pk>/like/` | Authentifié |
| `publication_commentaire_delete` | GET/POST | `/accounts/publications/detail/commentaire/<pk>/supprimer/` | Auteur du commentaire ou staff |
| `publication_modif` | GET/POST | `/accounts/publications/<pk>/modifier/` | Staff |
| `publication_delete` | GET/POST | `/accounts/publications/<pk>/supprimer/` | Staff |
| `publication_moderation` | GET | `/accounts/publications/moderation/` | Staff |
| `publication_moderation_detail` | GET | `/accounts/publications/<pk>/moderation/` | Staff |
| `publication_accept` | POST | `/accounts/publications/<pk>/accepter/` | Staff |
| `publication_refuse` | POST | `/accounts/publications/<pk>/refuser/` | Staff |
| `publication_perso` | GET | `/accounts/publications/brouillons/` | Authentifié |
| `publication_perso_details` | GET | `/accounts/publications/brouillons/<pk>` | Auteur |
| `publication_modif_perso` | GET/POST | `/accounts/publications/brouillons/<pk>/modifier/` | Auteur |
| `publication_delete_perso` | GET/POST | `/accounts/publications/brouillons/<pk>/supprimer/` | Auteur |
| `idee_list` | GET | `/accounts/idee/` | Authentifié |
| `idee_create` | GET/POST | `/accounts/idee/ajouter/` | Authentifié |
| `idee_perso` | GET | `/accounts/idee/mes_idees/` | Authentifié |
| `idee_detail` | GET | `/accounts/idee/<pk>/` | Authentifié |
| `idee_like` | POST | `/accounts/idee/<pk>/like/` | Authentifié |
| `idee_modif` | GET/POST | `/accounts/idee/<pk>/modifier/` | Authentifié |
| `idee_delete` | GET/POST | `/accounts/idee/<pk>/supprimer/` | Auteur ou staff |
| `idee_valide` | POST | `/accounts/idee/<pk>/valide/` | Staff |
| `espace_personnel` | GET/POST | `/accounts/espace-personnel/` | Authentifié |
| `annuaire` | GET | `/accounts/annuaire/` | Authentifié |
| `annuaire_detail` | GET | `/accounts/annuaire/<pk>/` | Authentifié |
| `annuaire_export` | GET | `/accounts/annuaire/export/` | Authentifié |
| `annuaire_import` | POST | `/accounts/annuaire/import/` | Staff |
| `formulaire_list` | GET | `/accounts/formulaires/` | Authentifié |
| `formulaire_create` | GET/POST | `/accounts/formulaires/ajouter/` | Staff |
| `formulaire_modif` | GET/POST | `/accounts/formulaires/<pk>/modifier/` | Staff |
| `formulaire_delete` | GET/POST | `/accounts/formulaires/<pk>/supprimer/` | Staff |
| `indicateurs` | GET | `/accounts/indicateurs/` | Authentifié |
| `indicateurs_update` | GET/POST | `/accounts/indicateurs/modifier/` | Staff |
| `admin:index` | GET | `/administration/` | Staff / superuser |

## 10. Rendu Markdown

Le filtre template `markdown_to_html` :

- convertit le texte avec `markdown` ;
- utilise les extensions `extra` et `sane_lists` ;
- nettoie le HTML produit avec `bleach` ;
- renvoie une chaîne marquée safe après nettoyage.

Tags autorisés :

- `a`
- `blockquote`
- `br`
- `code`
- `em`
- `h1`
- `h2`
- `h3`
- `h4`
- `h5`
- `h6`
- `hr`
- `li`
- `ol`
- `p`
- `pre`
- `strong`
- `u`
- `ul`

Attributs autorisés :

- pour `a` : `href`, `title`

Protocoles autorisés :

- `http`
- `https`
- `mailto`

## 11. Variables d'environnement

### 11.1 Générales

- `SECRET_KEY`
- `DEBUG`
- `ALLOWED_HOSTS`
- `CSRF_TRUSTED_ORIGINS`
- `LANGUAGE_CODE`
- `TIME_ZONE`
- `MEDIA_ROOT`
- `CSRF_COOKIE_SECURE`
- `SESSION_COOKIE_SECURE`
- `DJANGO_LOG_LEVEL`

### 11.2 Base de données

- `DB_ENGINE`
- `DB_NAME`
- `DB_USER`
- `DB_PASSWORD`
- `DB_HOST`
- `DB_PORT`

### 11.3 LDAP

- `LDAP_AUTH_URL`
- `LDAP_AUTH_USE_TLS`
- `LDAP_AUTH_CONNECT_USE_SSL`
- `LDAP_AUTH_CONNECT_TIMEOUT`
- `LDAP_AUTH_RECEIVE_TIMEOUT`
- `LDAP_AUTH_SEARCH_BASE`
- `LDAP_AUTH_OBJECT_CLASS`
- `LDAP_AUTH_CONNECTION_USERNAME`
- `LDAP_AUTH_CONNECTION_PASSWORD`
- `LDAP_AUTH_ACTIVE_DIRECTORY_DOMAIN`
- `LDAP_AUTH_DEFAULT_UPN_SUFFIX`
- `LDAP_AUTH_REQUIRED_GROUP_DN`
- `LDAP_AUTH_ENABLED`

## 12. Administration

Administration Django disponible sur :

- `/administration/`

Éléments administrables :

- utilisateurs Django avec profil inline ;
- publications ;
- documents ;
- sociétés de filtrage ;
- thèmes de documents ;
- formulaires ;
- thèmes de formulaires ;
- idées ;
- likes d'idées ;
- likes de publications ;
- commentaires de publications ;
- indicateurs.

Le formulaire de login admin utilise `LdapAwareAdminAuthenticationForm`.

## 13. Tests

Commande de référence :

```bash
python intranet/manage.py test accounts
```

Commande possible avec SQLite :

```bash
SECRET_KEY=test DB_ENGINE=django.db.backends.sqlite3 DB_NAME=/tmp/intranet-test.sqlite3 python intranet/manage.py test accounts
```

Les tests existants couvrent notamment :

- authentification locale et LDAP ;
- rendu Markdown sécurisé ;
- flux publications, modération, likes et commentaires ;
- annuaire, export et import ;
- idées et likes ;
- documents et formulaires ;
- indicateurs ;
- espace personnel.

## 14. Points d'attention et anomalies connues

Points observables dans l'état actuel du code :

- `IdeeModifPersoView` ne filtre pas son queryset par auteur ; une URL connue permet donc potentiellement de modifier une idée qui n'appartient pas à l'utilisateur courant ;
- les vues de toggle de like (`PublicationLikeToggleView`, `IdeeLikeToggleView`) n'empêchent pas côté serveur l'auto-like ; la restriction est portée par le template ;
- `annuaire_export` n'est protégé que par authentification au niveau de l'URL, alors que le bouton d'export n'est affiché qu'aux staff dans l'interface ;
- plusieurs classes d'administration portent encore le nom Python `DocumentAdmin` alors qu'elles enregistrent d'autres modèles ;
- `views.py` importe `Model` sans usage ;
- certains libellés visibles restent partiellement sans accents (`actualite`, `Moderation`, `Creee`, etc.) ;
- des tests d'authentification admin visent encore `/admin/login/` alors que l'administration réelle est montée sur `/administration/`.

## 15. Critères d'acceptation globaux

L'application est considérée conforme à son état actuel si :

- un utilisateur non connecté ne peut pas accéder aux pages protégées ;
- un utilisateur connecté peut consulter le fil d'actualité, les ressources, l'annuaire, les idées et les indicateurs ;
- une publication proposée n'apparaît pas dans le fil tant qu'elle n'est pas acceptée ;
- un staff peut accepter ou refuser une publication ;
- un brouillon de publication ne peut être consulté que par son auteur ;
- les commentaires de publication sont visibles par défaut à hauteur de 3 éléments, puis extensibles ;
- les documents et formulaires sont filtrés par société pour les non-staff ;
- l'import annuaire n'écrase pas les données existantes avec des cellules vides ;
- le détail annuaire permet la copie de l'email et des numéros ;
- les pages concernées affichent le bouton flottant de retour ;
- la sidebar garde l'entrée parente active sur les pages de détail principales.
