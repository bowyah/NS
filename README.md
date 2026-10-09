# Nella Sourcing (full stack)

Integrity and Service. Laravel (PHP) backend, MySQL database, Python AI service, HTML, CSS, and JavaScript frontend.

This repository is an overlay. You add it on top of a fresh Laravel project with Laravel Breeze, which supplies registration, login, email verification, password reset, and login rate limiting.

## Status

Written but not yet run against a live Laravel and MySQL install. The Python service is tested (5 passing tests). Run the PHP test suite and fix any version differences before going live (step 7).

## 1. Requirements

PHP 8.2+, Composer, MySQL 8 or MariaDB 10.6+, Node 18+, Python 3.10+.

## 2. Install

```bash
composer create-project laravel/laravel nella-sourcing
cd nella-sourcing
composer require laravel/breeze --dev
php artisan breeze:install blade
cp -r /path/to/this/repo/* /path/to/this/repo/.gitignore .      # overwrites routes/web.php, bootstrap/app.php, User model
cat .env.nella.example >> .env                                  # then edit the values, and remove duplicate keys above
php artisan key:generate
php artisan migrate --seed
npm install && npm run build
php artisan serve
```

## 3. Python AI service

```bash
cd python/ai_service
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
export NELLA_AI_TOKEN="same value as in Laravel .env"
uvicorn app:app --host 127.0.0.1 --port 8001
```

Image analysis is optional. Set `ANTHROPIC_API_KEY` in the service environment to enable it (not tested here). Without it, analysis uses keyword rules only. The service refuses all requests if no token is set.

## 4. What is built

* Accounts: registration, login, logout, email verification, password reset, rate limited login and lockout (Breeze), password hashing, session management
* Roles: client, agent, manager, admin, enforced by Laravel policies and route middleware
* Persistent memory in MySQL: profile, preferences, request history, frequent products, previous suppliers and orders, welcome back message, edit, delete, export, account deletion
* Sourcing requests (NS-SRC-2026-000125), quotations (NS-QUO), orders (NS-ORD-2026-00458) with race safe numbering
* Quotation flow: agent drafts, manager releases, client approves, order is created, every status change is stored
* Supplier database with Verified, Pending, Unverified. New suppliers are always Unverified and only managers can verify
* Shipments with tracking events, payments ledger, messaging, notifications, documents, reviews
* Secure uploads: content type sniffing, random names, private storage, authorised downloads
* Audit log with encrypted IP and coarse device, failed login and lockout alerts
* Admin dashboard and user and role management
* Security headers including a strict Content Security Policy, CSRF on all forms and fetch calls, Eloquent queries only, escaped output

## 5. Not included, needs your decision

* Payment gateway. Payments are recorded by staff only. Add a processor (for example Stripe or a local mobile money provider) before taking money online.
* Two factor authentication. Add Laravel Fortify or a Google2FA package.
* WhatsApp alerts. In app notifications exist. Add the WhatsApp Business API for outbound messages.
* Marketplace integrations (Alibaba, 1688). These need partner access and terms review.
* Multiple items per quotation. The schema supports it, the form quotes the first item.

## 6. Production checklist

1. HTTPS only, `APP_DEBUG=false`, `SESSION_SECURE_COOKIE=true`.
2. Point the web server at `public/`. Files are stored in `storage/app/private`, which is not web accessible.
3. Run the AI service on 127.0.0.1 behind a firewall.
4. Set a real mail provider so verification and reset emails are delivered.
5. Remove `ADMIN_PASSWORD` from `.env` after the first seed.
6. Backups: nightly `mysqldump --single-transaction nella_sourcing | gzip > backup.sql.gz`, plus a copy of `storage/app/private`, both stored off server and encrypted. Test a restore monthly: create an empty database, load the dump, run `php artisan migrate:status`.
7. Run `php artisan test` and `cd python/ai_service && pytest`.

## 7. Structure

* `database/migrations` all tables, foreign keys, indexes
* `app/Models`, `app/Policies`, `app/Services`, `app/Http`
* `resources/views`, `public/css`, `public/js`
* `python/ai_service` Nella AI
* `tests/Feature` access control, workflow, upload, privacy tests
