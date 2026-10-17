# API de réservation d'hôtel

API REST construite avec **Symfony 7.2**, **API Platform 4**, **Doctrine ORM** et **PHP 8.2 ou supérieur**. Elle permet de gérer des hôtels, leurs chambres, les utilisateurs et les réservations. L'authentification de l'API repose sur des jetons JWT.

## Fonctionnalités

- Consultation et gestion des hôtels et des chambres.
- Liste personnalisée des chambres avec le nom de leur hôtel.
- Création d'utilisateurs avec mot de passe haché et photo de profil requise lors de l'inscription.
- Authentification par JWT.
- Création de réservations et envoi d'un courriel lors de leur traitement.
- Documentation interactive des opérations API avec Swagger UI.

## Prérequis

- PHP **8.2+** avec les extensions `ctype` et `iconv`.
- Composer.
- Un serveur **MySQL/MariaDB** configuré pour cette application.
- Symfony CLI est facultatif ; le serveur PHP intégré peut aussi être utilisé.
- Docker Compose est facultatif pour démarrer les services définis par le projet.

> **Base de données :** `compose.yaml` démarre PostgreSQL, mais les migrations présentes dans `migrations/` utilisent la syntaxe MySQL. Pour appliquer les migrations telles quelles, configurez une base MySQL/MariaDB. Le service PostgreSQL de Compose ne peut pas être utilisé avec ces migrations sans adapter celles-ci au moteur choisi.

## Installation et démarrage

Depuis la racine du projet :

```bash
composer install
```

Configurez vos paramètres locaux dans `.env.local` (ne placez pas de mots de passe ni de clés privées dans un fichier versionné). Exemple à adapter à votre installation MySQL :

```dotenv
APP_ENV=dev
DATABASE_URL="mysql://UTILISATEUR:MOT_DE_PASSE@127.0.0.1:3306/reservation_hotel?serverVersion=8.0.32&charset=utf8mb4"
MAILER_DSN="smtp://127.0.0.1:1025"
```

Vérifiez également que les clés JWT référencées par `config/packages/lexik_jwt_authentication.yaml` existent et que leur phrase secrète est configurée localement.

Créez la base, appliquez les migrations, puis démarrez le serveur :

```bash
php bin/console doctrine:database:create
php bin/console doctrine:migrations:migrate
```

Avec Symfony CLI :

```bash
symfony server:start --port=8000
```

Ou avec le serveur PHP intégré :

```bash
php -S 127.0.0.1:8000 -t public public/index.php
```

L'API est alors disponible sur `http://127.0.0.1:8000`.

## Services Docker facultatifs

Les fichiers Compose définissent un service PostgreSQL et Mailpit. Le démarrage se fait avec :

```bash
docker compose up -d
docker compose ps
```

Les ports sont attribués par Docker. Pour les consulter :

```bash
docker compose port database 5432
docker compose port mailer 1025
docker compose port mailer 8025
```

Mailpit fournit une interface web pour les courriels de développement. Si vous utilisez ces services, configurez l'URL SMTP locale correspondante dans `MAILER_DSN`. Attention : le service PostgreSQL défini ici ne rend pas les migrations MySQL compatibles avec PostgreSQL.

## Documentation et authentification

- Documentation API / Swagger UI : `http://127.0.0.1:8000/api`
- Connexion : `POST /api/login`, avec un corps JSON contenant `email` et `password`. La réponse contient un jeton `token`.
- Les routes sous `/api` nécessitent une authentification, sauf la route de connexion.
- Pour les routes protégées, envoyez le jeton ainsi :

```http
Authorization: Bearer <votre_token>
```

Exemple de connexion et d'appel authentifié :

```bash
curl -X POST http://127.0.0.1:8000/api/login \
  -H "Content-Type: application/json" \
  -d "{\"email\":\"utilisateur@example.com\",\"password\":\"mot-de-passe\"}"
```

```bash
curl http://127.0.0.1:8000/api/hotels \
  -H "Authorization: Bearer <votre_token>"
```

Le bouton **Authorize** de Swagger UI permet aussi de renseigner le jeton.

## Principales routes

Toutes les routes API Platform ci-dessous sont préfixées par `/api`.

| Route | Fonction |
| --- | --- |
| `POST /api/login` | Authentification et obtention d'un JWT. |
| `/api/hotels` | Consultation, création et opérations de gestion des hôtels. La liste peut être filtrée par nom. |
| `GET /api/hotel_with_rooms` | Liste les numéros de chambres associés au nom de leur hôtel. |
| `/api/rooms` | Opérations API Platform standard sur les chambres. |
| `POST /api/reservations` | Crée une réservation ; le traitement envoie un courriel. |
| `/api/users` | Création d'un utilisateur avec informations de profil et fichier envoyé en multipart. |
| `GET /api/users` | Liste les utilisateurs. |
| `GET /send-email` | Route de test qui envoie un courriel. |

Les méthodes et formats disponibles pour chaque ressource sont détaillés dans Swagger UI. La création d'utilisateur attend les champs `fullname`, `email`, `password` et un fichier `file` en `multipart/form-data`. Les images envoyées sont placées dans `public/uploads/`.

## Commandes Symfony utiles

Créer un utilisateur initial (le mot de passe est haché avant stockage) :

```bash
php bin/console app:create-user utilisateur@example.com "mot-de-passe"
```

Lancer la commande interactive qui demande les identifiants puis récupère un JWT depuis `http://127.0.0.1:8000/api/login` :

```bash
php bin/console app:login
```

Autres commandes utiles :

```bash
php bin/console debug:router
php bin/console doctrine:migrations:status
php bin/console cache:clear
```

## Développement

- Les entités Doctrine se trouvent dans `src/Entity/`.
- Les contrôleurs personnalisés se trouvent dans `src/Controller/`.
- Les migrations Doctrine se trouvent dans `migrations/`.
- Les fichiers téléversés sont stockés dans `public/uploads/`.

Avant un déploiement, configurez des clés JWT et un transport de courriel adaptés à l'environnement, remplacez les adresses de courriel de test définies dans le code, et ne réutilisez pas les paramètres de développement.
