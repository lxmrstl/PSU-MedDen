# Role-Based Access Control (RBAC) Implementation Guide

## Overview
This document describes the complete implementation of Role-Based Access Control in the PSU Medical Management System.

## User Roles

### 1. **Super Admin**
- **Database value:** `super_admin`
- **Campus:** `null` (no campus assignment)
- **Capabilities:**
  - Full access to all pages and features
  - Can view and manage data from ALL campuses
  - Can manage Staff globally
  - Can create/edit/delete medicines
  - Can manage medicine distributions
  - Can approve medicine requests
  - Can access system Settings
  - Can view all Reports

### 2. **Campus Admin**
- **Database value:** `admin`
- **Campus:** One of ['Urdaneta', 'Lingayen', 'Binmaley', 'Bayambang', 'San Carlos']
- **Capabilities:**
  - Can only see and manage data from their assigned campus
  - Can access: Dashboard, Patients, Medical Records, Dental Records
  - Can view medicines and log usage
  - Can request medicine distributions
  - Can manage Staff assigned to their campus only
  - Can view Reports for their campus
  - **Cannot:** Access Settings, View Distributions, Global system management

### 3. **Staff (Clinic Staff)**
- **Database value:** `staff`
- **Campus:** One of ['Urdaneta', 'Lingayen', 'Binmaley', 'Bayambang', 'San Carlos']
- **Capabilities:**
  - Can only see data from their assigned campus
  - Can access: Dashboard, Patients, Medical Records, Dental Records
  - Can view medicines and log usage
  - Can create new patient records and consultations
  - **Cannot:** Access Staff Management, Settings, Distributions, Reports

## Implementation Files

### 1. **Trait: HasRoles** (`app/Traits/HasRoles.php`)
Provides helper methods for checking user permissions:

```php
$user->isSuperAdmin()              // Check if Super Admin
$user->isAdmin()                    // Check if Campus Admin
$user->isStaff()                    // Check if Clinic Staff
$user->isAdminLevel()               // Check if Super Admin or Admin
$user->canManageStaff()             // Check if can manage staff
$user->canManageCampus($campus)     // Check if can manage specific campus
$user->canViewCampus($campus)       // Check if can view campus data
$user->canManageMedicines()         // Check if can manage medicines
$user->canViewDistributions()       // Check if can view distributions
$user->canApproveMedicineRequests() // Check if can approve requests
$user->canLogMedicineUsage()        // Check if can log usage
$user->canAccessSettings()          // Check if can access settings
$user->getAccessibleCampuses()      // Get list of accessible campuses
$user->canCreateRecords()           // Check if can create medical/dental records
$user->canViewReports()             // Check if can view reports
```

### 2. **Middleware: CheckRole** (`app/Http/Middleware/CheckRole.php`)
Restricts routes to specific roles:

```php
// In routes/web.php
Route::middleware('check.role:super_admin,admin')->group(function () {
    // Only Super Admin and Admin can access these routes
});

Route::middleware('check.role:super_admin')->group(function () {
    // Only Super Admin can access these routes
});
```

### 3. **Middleware: CheckCampusAccess** (`app/Http/Middleware/CheckCampusAccess.php`)
Restricts campus-specific routes:

```php
// In routes/web.php
Route::get('/campus/{campus}/patients', function() {
    // Checks if user can access the campus from route parameter
})->middleware('check.campus');
```

### 4. **Policies**

#### StaffPolicy (`app/Policies/StaffPolicy.php`)
```php
// Super Admin can view any staff
// Admin can view staff from their campus
// Staff cannot view other staff
```

#### MedicinePolicy (`app/Policies/MedicinePolicy.php`)
```php
// All authenticated users can view medicines
// Only Super Admin can create medicines
// Super Admin and Admin can update medicines
// Only Super Admin can delete medicines
// Only Super Admin and Admin can view/manage distributions
// Super Admin and Admin can approve requests
// All campus users can log usage
```

#### MedicalRecordPolicy (`app/Policies/MedicalRecordPolicy.php`)
```php
// Super Admin can view any record
// Admin and Staff can only view records from their campus
// Only Admin-level can delete records
```

### 5. **Sidebar Menu (dashboard.blade.php)**
Menu items are conditionally displayed based on user role:

```blade
@if(Auth::user()->canManageStaff())
    <a href="/staff">Staff</a>  <!-- Only shown to Super Admin and Admin -->
@endif

@if(Auth::user()->isAdminLevel())
    <a href="#">Reports</a>      <!-- Only shown to Super Admin and Admin -->
@endif

@if(Auth::user()->isSuperAdmin())
    <a href="#">Settings</a>     <!-- Only shown to Super Admin -->
@endif
```

## Usage Examples

### In Blade Templates

```blade
<!-- Show element only to Super Admin -->
@if(Auth::user()->isSuperAdmin())
    <a href="/settings">Settings</a>
@endif

<!-- Show element to Admin-level users -->
@if(Auth::user()->isAdminLevel())
    <div>Approve these requests</div>
@endif

<!-- Show element to Staff and Admin (but not Super Admin) -->
@if(Auth::user()->isStaff() || Auth::user()->isAdmin())
    <button>Log Medicine Usage</button>
@endif

<!-- Show element only to users with campus access -->
@if(Auth::user()->canCreateRecords())
    <button>Create Medical Record</button>
@endif

<!-- Check campus access -->
@foreach(Auth::user()->getAccessibleCampuses() as $campus)
    <option>{{ $campus }}</option>
@endforeach
```

### In Controllers

```php
class MedicineController extends Controller
{
    public function store(Request $request)
    {
        // Using authorize() method with Policy
        $this->authorize('create', Medicine::class);
        
        // Or check manually using helper
        if (!auth()->user()->canManageMedicines()) {
            abort(403, 'Unauthorized');
        }
        
        // Create medicine...
    }

    public function approveRequest($id)
    {
        // Only Super Admin and Admin can approve
        $this->authorize('approveRequests', Medicine::class);
        
        $request = Request::find($id);
        
        // Admin can only approve for their campus
        if (auth()->user()->isAdmin()) {
            if ($request->campus !== auth()->user()->campus) {
                abort(403, 'Cannot approve requests for other campuses');
            }
        }
        
        // Approve the request...
    }
}
```

### In Routes (web.php)

```php
use App\Http\Middleware\CheckRole;
use App\Http\Middleware\CheckCampusAccess;

// Super Admin only routes
Route::middleware(['auth', 'check.role:super_admin'])->group(function () {
    Route::post('/medicines/distributions', [MedicineController::class, 'storeDistribution']);
    Route::resource('settings', SettingsController::class);
});

// Admin and Super Admin routes
Route::middleware(['auth', 'check.role:super_admin,admin'])->group(function () {
    Route::resource('staff', StaffController::class);
    Route::get('/reports', [ReportController::class, 'index']);
    Route::post('/medicines/requests/approve/{id}', [MedicineController::class, 'approveRequest']);
});

// Campus-specific routes
Route::middleware(['auth', 'check.campus'])->group(function () {
    Route::get('/campus/{campus}/patients', [PatientController::class, 'index']);
});

// All authenticated users
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
    Route::post('/medicines/usage-logs', [MedicineController::class, 'storeUsageLog']);
});
```

## Database Schema

### Users Table
```sql
CREATE TABLE users (
    id BIGINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL,
    password VARCHAR(255) NOT NULL,
    role ENUM('super_admin', 'admin', 'staff') DEFAULT 'staff',
    campus ENUM('Urdaneta', 'Lingayen', 'Binmaley', 'Bayambang', 'San Carlos') NULL,
    email_verified_at TIMESTAMP NULL,
    remember_token VARCHAR(100) NULL,
    created_at TIMESTAMP,
    updated_at TIMESTAMP
);

-- Indices for better query performance
CREATE INDEX idx_role ON users(role);
CREATE INDEX idx_campus ON users(campus);
```

## Migration Files

### Add Role and Campus Columns (if starting from scratch)

File: `database/migrations/YYYY_MM_DD_HHMMSS_add_role_and_campus_to_users_table.php`

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->enum('role', ['super_admin', 'admin', 'staff'])
              ->default('staff')
              ->after('password');
        
        $table->enum('campus', ['Urdaneta', 'Lingayen', 'Binmaley', 'Bayambang', 'San Carlos'])
              ->nullable()
              ->after('email');
    });
}
```

### Update Existing Role to Enum

File: `database/migrations/2026_04_28_000001_update_role_to_enum.php`

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->enum('role', ['super_admin', 'admin', 'staff'])
              ->default('staff')
              ->change();
    });
}
```

### Update Existing Campus to Enum

File: `database/migrations/2026_04_28_000002_update_campus_to_enum.php`

```php
public function up()
{
    Schema::table('users', function (Blueprint $table) {
        $table->enum('campus', ['Urdaneta', 'Lingayen', 'Binmaley', 'Bayambang', 'San Carlos'])
              ->nullable()
              ->change();
    });
}
```

## Seeding Test Users

Add to `database/seeders/DatabaseSeeder.php`:

```php
use App\Models\User;

public function run()
{
    // Super Admin (no campus)
    User::create([
        'name' => 'Super Admin',
        'email' => 'superadmin@psu.edu.ph',
        'password' => Hash::make('password'),
        'role' => 'super_admin',
        'campus' => null,
    ]);

    // Campus Admin for Urdaneta
    User::create([
        'name' => 'Admin Urdaneta',
        'email' => 'admin.urdaneta@psu.edu.ph',
        'password' => Hash::make('password'),
        'role' => 'admin',
        'campus' => 'Urdaneta',
    ]);

    // Staff for Urdaneta
    User::create([
        'name' => 'Staff Urdaneta',
        'email' => 'staff.urdaneta@psu.edu.ph',
        'password' => Hash::make('password'),
        'role' => 'staff',
        'campus' => 'Urdaneta',
    ]);

    // Campus Admin for Lingayen
    User::create([
        'name' => 'Admin Lingayen',
        'email' => 'admin.lingayen@psu.edu.ph',
        'password' => Hash::make('password'),
        'role' => 'admin',
        'campus' => 'Lingayen',
    ]);

    // Staff for Lingayen
    User::create([
        'name' => 'Staff Lingayen',
        'email' => 'staff.lingayen@psu.edu.ph',
        'password' => Hash::make('password'),
        'role' => 'staff',
        'campus' => 'Lingayen',
    ]);
}
```

## Authorization Gate Definitions

Registered in `AppServiceProvider.php`:

```php
Gate::define('admin-level', fn($user) => $user->isAdminLevel());
Gate::define('super-admin', fn($user) => $user->isSuperAdmin());
Gate::define('manage-staff', fn($user) => $user->canManageStaff());
Gate::define('manage-medicines', fn($user) => $user->canManageMedicines());
Gate::define('access-settings', fn($user) => $user->canAccessSettings());

// Usage in Controllers/Blade
if (Gate::allows('super-admin')) { ... }
if (Gate::denies('manage-staff')) { ... }
@can('admin-level') ... @endcan
@cannot('access-settings') ... @endcannot
```

## Testing

```php
// In tests
$superAdmin = User::factory()->create(['role' => 'super_admin', 'campus' => null]);
$admin = User::factory()->create(['role' => 'admin', 'campus' => 'Urdaneta']);
$staff = User::factory()->create(['role' => 'staff', 'campus' => 'Urdaneta']);

// Assert helpers work
$this->assertTrue($superAdmin->isSuperAdmin());
$this->assertTrue($admin->isAdmin());
$this->assertTrue($staff->isStaff());

// Test authorization
$response = $this->actingAs($staff)->post('/medicines', [...]);
$response->assertStatus(403); // Should be forbidden

$response = $this->actingAs($admin)->post('/staff', [...]);
$response->assertStatus(200); // Should succeed
```

## Security Best Practices

1. **Always use $this->authorize()** in controllers instead of manual checks when possible
2. **Validate campus ownership** when users access campus-specific data
3. **Implement audit logging** for sensitive operations (staff creation, request approval)
4. **Use middleware** to protect entire route groups rather than individual controller methods
5. **Test authorization** in your test suite to ensure roles work correctly
6. **Use Gates/Policies** instead of hardcoding role checks
7. **Set default role to 'staff'** when creating new users to be safe
8. **Validate campus enum values** to prevent invalid data entry

## Troubleshooting

### Issue: "Call to undefined method isSuperAdmin()"
**Solution:** Ensure the `HasRoles` trait is used in the User model:
```php
use App\Traits\HasRoles;
class User extends Authenticatable {
    use HasRoles;
}
```

### Issue: Authorization returns 403 unexpectedly
**Solution:** Check that the policy is registered in AppServiceProvider:
```php
$gate->policy(Medicine::class, MedicinePolicy::class);
```

### Issue: Middleware not working on routes
**Solution:** Ensure middleware aliases are registered in bootstrap/app.php:
```php
$middleware->alias([
    'check.role' => CheckRole::class,
    'check.campus' => CheckCampusAccess::class,
]);
```

### Issue: Enum migration fails
**Solution:** Ensure your MySQL version supports ENUM columns (5.7+), or use a string column with check constraint:
```php
$table->string('role'); // Alternative to enum
$table->check('role IN ("super_admin", "admin", "staff")');
```
