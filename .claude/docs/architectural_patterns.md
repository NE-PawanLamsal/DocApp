# Architectural Patterns & Design Decisions

## Backend (Laravel)

### MVC + RESTful API
The backend serves two consumers: a web dashboard (Blade/MVC) and the Flutter mobile app (JSON API). Controllers handle both. API routes are all prefixed under `/api` in `routes/api.php`. Web routes live in `routes/web.php`.

Authentication is split: web uses session-based Jetstream/Fortify; mobile uses Sanctum token auth (`middleware('auth:sanctum')`).

### Controller Responsibilities
Each controller maps closely to a resource:
- `UsersController` — login, register, logout, user profile, favorites (`doc_app_Laravel/app/Http/Controllers/UsersController.php`)
- `AppointmentsController` — list and book appointments (`doc_app_Laravel/app/Http/Controllers/AppointmentsController.php`)
- `DocsController` — doctor dashboard, reviews/ratings (`doc_app_Laravel/app/Http/Controllers/DocsController.php`)

No dedicated service layer — business logic sits directly in controllers.

### Eloquent Relationships
Defined on models (`doc_app_Laravel/app/Models/`):
- `User` hasOne `Doctors`, hasMany `Appointments`, hasMany `Reviews`
- `Appointments` belongsTo `User`
- `Reviews` belongsTo `User`

The `users` table has a `type` column (`'user'` | `'doctor'`) to differentiate roles. Doctor-specific data is in a separate `doctors` table linked via `doc_id` FK.

### Favorites as JSON
User favorites (doctor IDs) are stored as a JSON-encoded array in `user_details.fav` (a single column), not a join table. Updated wholesale on each `/api/fav` POST.

### Date Format Convention
Appointment dates are stored as strings in `n/j/Y` format (e.g., `1/15/2023`), not ISO 8601. Filtering logic on the dashboard uses PHP `date('n/j/Y')` to match today's appointments. Be consistent with this format when writing queries or migrations.

## Mobile (Flutter)

### Provider for Global State
A single `AuthModel` (extends `ChangeNotifier`) holds the authenticated user's data and is provided at the root of the widget tree (`main.dart`). Screens read from it via `context.read<AuthModel>()` / `context.watch<AuthModel>()`. No Riverpod, Bloc, or GetX — just Provider.

Reference: `doctor_appointment_app/lib/models/auth_model.dart`

### Single HTTP Provider
All network calls are centralized in `DioProvider` (`doctor_appointment_app/lib/providers/dio_provider.dart`). It contains static-style methods for each API operation (login, register, getUser, getAppointments, bookAppointment, sendReview, updateFav). No repository layer — screens call `DioProvider` methods directly.

Dio is configured with a base URL from `Config.serverApiUrl` (defined in `lib/utils/config.dart`). Auth token is appended per-request from SharedPreferences.

### Token Persistence
After login/register, the Sanctum token is stored in `SharedPreferences` under a known key. On app start, `main.dart` reads the stored token and pre-populates `AuthModel` to restore session. On logout, the token is cleared.

### Named Route Navigation
All screen transitions use named routes defined in `main.dart`. Arguments are passed via `RouteSettings.arguments` and extracted with `ModalRoute.of(context)!.settings.arguments`. No go_router or auto_route.

### Config as a Central Constants File
`doctor_appointment_app/lib/utils/config.dart` is the single source of truth for:
- `Config.serverApiUrl` — backend base URL (must be updated for different environments)
- Theme colors (primary, secondary, etc.)
- Standard spacing/sizing values used across widgets

### Reusable Widget Convention
UI components that appear on multiple screens are extracted to `lib/components/`. Examples: `DoctorCard`, `AppointmentCard`, `CustomAppBar`, `LoginForm`. Screens compose these components rather than duplicating markup.

## API Contract

### Authentication
All protected endpoints require `Authorization: Bearer {token}` header. The token is issued by Sanctum on login/register and has no explicit expiry configured.

### Response Shape
- Success: HTTP 200/201 with JSON body (varies per endpoint, no envelope wrapper)
- Validation failure: Laravel's default validation exception JSON
- No standardized error envelope across endpoints

### Appointment Slots
Time slots are hardcoded in the Flutter booking UI (9 AM–5 PM, weekdays only). The backend does not enforce slot availability — it accepts any `date`/`time` string posted to `/api/book`. Duplicate booking prevention (if any) must be handled client-side or added to the controller.
