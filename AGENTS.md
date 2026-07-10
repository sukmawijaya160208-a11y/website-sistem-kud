# AGENTS.md

## Project

KUD Sari Subur Wijaya Tani — Laravel 11 app for cooperative management (palm oil). 4 roles: admin, manager, kasir, petani.

## Quick start

```bash
php artisan serve --port=8000
php artisan db:seed --class=KudSeeder
mysql -u root laravelhrmain < laravelhrmain.sql
```

## Auth

- Passwords stored in **plaintext** (not bcrypt). `LoginController::attemptLogin` does `$user->password === $request->password`.
- 4 roles: `admin`, `manager`, `kasir`, `petani` — enum on `users.role` column.
- `CheckRole` middleware: `['auth', 'role:kasir']` — aborts 403 if role doesn't match.

### Seeded accounts

| Role | Email | Password |
|------|-------|----------|
| Admin | admin@gmail.com | password |
| Manager | manager@gmail.com | password |
| Kasir | kasir@gmail.com | password |
| Petani | petani1@gmail.com | password |

## Routes & structure

All app routes in `routes/web.php`:
- `/` — login page
- `/admin/*` — middleware `auth, role:admin`
- `/manager/*` — middleware `auth, role:manager`
- `/kasir/*` — middleware `auth, role:kasir`
- `/petani/*` — middleware `auth, role:petani`

4 controllers under `App\Http\Controllers\Kud\`.

Views under `resources/views/kud/{role}/`, plus shared layouts at `layouts/admin/`.

## Sidebar

Single file `include/admin/sidebar.blade.php` — role-based menus rendered by `@if($role == '...')` blocks. Brand link redirects to role dashboard via `auth()->user()->role`.

## Run seeder

```bash
php artisan db:seed --class=KudSeeder
```

Do **not** run `migrate:fresh` — the `users` table migration is missing `id_kelompok` column (added manually in DB). Re-run seeder instead.

## Common pitfalls

- **`composer dump-autoload` times out** if `optimize-autoloader: true` in `composer.json`. Set to `false` or use `--no-optimize`.
- **Laravel 11 cache config**: `.env` key is `CACHE_STORE`, not `CACHE_DRIVER`.
- **`$s` vs `$setoran` bug**: petani views `slip.blade.php` and `setoran_detail.blade.php` use `$setoran` (controller passes `compact('setoran')`).
- **Old route prefix `kud.*` was renamed to `admin.*`** — views used `kud.setoran.*`, `kud.petani.*`, `kud.harga.*`. Now all use `admin.*`.
- **Orphaned views deleted**: old HRIS `views/admin/`, `views/user/`, shadow `create.blade.php`/`edit.blade.php` files are gone. If missing a view, check `form.blade.php`.
- **`app/Exports/CutiExport.php` disabled** — references deleted `Cutis` model. `PegawaiExport.php` rewritten to use current `User` model columns.

## Key models & relationships

```
User → kelompok (KelompokTani)
User → setoranTbs (SetoranTbs)
User → pembayaran (Pembayaran)
SetoranTbs → user (User)
SetoranTbs → grade (GradeTbs)
SetoranTbs → approver (User)
Pembayaran → user (User)
Pembayaran → setoran (SetoranTbs)
Pembayaran → pembayar (User)
```

## Custom theme

`public/admin/assets/css/custom-theme.css` — modern green KUD theme, loaded after Sneat defaults in `template.blade.php`.

## DB

- MySQL database: `laravelhrmain`
- Migration `0001_01_01_000000_create_users_table.php` missing `id_kelompok` column — added directly in DB.
- Cuti, izin, absensi tables from old HRIS are gone.
