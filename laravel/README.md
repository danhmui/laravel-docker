# Laravel Docker setup

Laravel 13 with PHP 8.3, Nginx, and MySQL 8.0.

Copy `.env.example` to `.env`, then set distinct nonempty `DB_PASSWORD`
and `MYSQL_ROOT_PASSWORD` values. Never commit `.env`.

MySQL uses database/user `laravel`, internal host `db`, and port `3306`.
Host access is limited to `127.0.0.1`; change `FORWARD_DB_PORT` if port
3306 is already occupied. Database files persist in the `dbdata` volume.
MySQL initialization variables only apply to an empty data volume;
changing `.env` does not update existing database users or passwords.

Start containers:

```sh
docker compose up -d --build
```

Install PHP dependencies and generate the application key inside the PHP
container. Generate the key only for a new environment; preserve existing keys.

```sh
docker compose exec app composer install
docker compose exec app php artisan key:generate
docker compose exec app php artisan migrate
docker compose exec app chown -R www-data:www-data storage bootstrap/cache
docker compose exec app php artisan migrate:status
docker compose exec app php artisan test
```

Default migrations create the users, sessions, cache, and jobs tables.
Wait for MySQL to finish initialization before running migrations.
Application URL: http://localhost:8080.

If Composer extraction times out on a Windows bind mount, retry with:

```sh
docker compose exec -e COMPOSER_PROCESS_TIMEOUT=1200 app composer install
```
