# DocApp

A doctor appointment booking system. Patients browse doctors by specialty, book time slots, rate doctors, and manage favorites. Doctors access a web dashboard to view appointments and reviews.

## Tech Stack

| Layer | Technology |
|---|---|
| Backend | Laravel 9, PHP 8.0+, MySQL |
| API Auth | Laravel Sanctum (Bearer tokens) |
| Admin UI | Blade templates, Tailwind CSS 3, Alpine.js |
| Asset Build | Vite 4 |
| Mobile | Flutter (Dart SDK 2.18+) |
| State (mobile) | Provider 6 |
| HTTP (mobile) | Dio 4 |

## Project Structure

```
DocApp/
├── doc_app_Laravel/          # Laravel backend + web dashboard
│   ├── app/Http/Controllers/ # AppointmentsController, DocsController, UsersController
│   ├── app/Models/           # User, Appointments, Doctors, UserDetails, Reviews
│   ├── routes/api.php        # All API endpoint definitions
│   └── database/migrations/  # 11 migration files
└── doctor_appointment_app/   # Flutter mobile client
    └── lib/
        ├── screens/          # Full-page widgets (home, booking, appointments, etc.)
        ├── components/       # Reusable widgets (DoctorCard, AppointmentCard, etc.)
        ├── providers/        # dio_provider.dart — all API calls
        ├── models/           # auth_model.dart — global auth state
        └── utils/config.dart # Theme colors, sizing constants, app URL
```

## Essential Commands

### Laravel Backend
```bash
# First-time setup
composer install
cp .env.example .env
php artisan key:generate
php artisan migrate

# Development
npm run dev          # Vite dev server (Blade assets)
npm run build        # Production asset build

# Testing
vendor/bin/phpunit
```

### Flutter Mobile
```bash
flutter pub get      # Install dependencies
flutter run          # Run on connected device/emulator
flutter build apk    # Android release build
flutter test         # Run unit tests
flutter analyze      # Static analysis
```

## Key Entry Points

- Backend API routes: `doc_app_Laravel/routes/api.php`
- Flutter app init + routing: `doctor_appointment_app/lib/main.dart`
- All API calls (mobile): `doctor_appointment_app/lib/providers/dio_provider.dart`
- Global auth state (mobile): `doctor_appointment_app/lib/models/auth_model.dart`
- App base URL + theme: `doctor_appointment_app/lib/utils/config.dart`

## Environment

Backend config lives in `doc_app_Laravel/.env` (copy from `.env.example`). Key variables: `DB_*` (MySQL), `APP_URL`. The Flutter app's base URL is hardcoded in `lib/utils/config.dart`.

## Additional Documentation

Check these files when working on specific areas:

| Topic | File |
|---|---|
| Architecture, patterns, conventions | `.claude/docs/architectural_patterns.md` |
