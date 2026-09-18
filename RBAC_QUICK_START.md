# RBAC Implementation - Quick Start Guide

## What Was Implemented

A complete **Role-Based Access Control (RBAC)** system with three user roles:

1. **Super Admin** - Full system access
2. **Campus Admin** - Campus-level management
3. **Staff** - Clinic staff (nurses, dentists, aides)

## Files Created/Modified

### Core RBAC Files
- ✅ `app/Traits/HasRoles.php` - Role checking helper methods
- ✅ `app/Http/Middleware/CheckRole.php` - Role-based route protection
- ✅ `app/Http/Middleware/CheckCampusAccess.php` - Campus-specific access control
- ✅ `app/Policies/StaffPolicy.php` - Authorization policy for Staff resources
- ✅ `app/Policies/MedicinePolicy.php` - Authorization policy for Medicines
- ✅ `app/Policies/MedicalRecordPolicy.php` - Authorization policy for Medical Records
- ✅ `app/Providers/AppServiceProvider.php` - Registered policies and gates
- ✅ `bootstrap/app.php` - Registered middleware aliases

### Updated Files
- ✅ `app/Models/User.php` - Added HasRoles trait
- ✅ `app/Http/Controllers/MedicineController.php` - Added role-based access checks
- ✅ `resources/views/dashboard.blade.php` - Conditional menu items by role
- ✅ `database/migrations/2026_04_28_000001_update_role_to_enum.php` - Role enum migration
- ✅ `database/migrations/2026_04_28_000002_update_campus_to_enum.php` - Campus enum migration

### Example Files
- ✅ `app/Http/Controllers/StaffControllerExample.php` - Complete RBAC example
- ✅ `database/seeders/RBACTestSeeder.php` - Test users seeder

### Documentation
- ✅ `RBAC_IMPLEMENTATION.md` - Complete implementation guide

## Database Schema

### Users Table (Modified)
```sql
ALTER TABLE users ADD COLUMN role ENUM('super_admin', 'admin', 'staff') DEFAULT 'staff';
ALTER TABLE users ADD COLUMN campus ENUM(
    'Urdaneta City Campus',
    'Lingayen Campus',
    'Binmaley Campus',
    'Bayambang Campus',
    'San Carlos City Campus',
    'Alaminos City Campus',
    'Asingan Campus',
    'Infanta Campus',
    'Sta. Maria Campus'
) NULL;
```

## Test Users (Pre-Seeded)

Use these credentials to test the different roles:

### Super Admin (Full Access)
```
Email: superadmin@psu.edu.ph
Password: password123
Campus: None (can see all)
```

### Campus Admin - Urdaneta
```
Email: admin.urdaneta@psu.edu.ph
Password: password123
Campus: Urdaneta City Campus
```

### Campus Admin - Lingayen
```
Email: admin.lingayen@psu.edu.ph
Password: password123
Campus: Lingayen Campus
```

### Staff - Urdaneta (Nurse)
```
Email: nurse.rosa@psu.edu.ph
Password: password123
Campus: Urdaneta City Campus
```

### Staff - Urdaneta (Doctor)
```
Email: doctor.pedro@psu.edu.ph
Password: password123
Campus: Urdaneta City Campus
```

### Staff - Lingayen (Dentist)
```
Email: dentist.lisa@psu.edu.ph
Password: password123
Campus: Lingayen Campus
```

### Staff - Lingayen (Aide)
```
Email: aide.carlos@psu.edu.ph
Password: password123
Campus: Lingayen Campus
```

## Using RBAC in Your Code

### In Blade Templates

```blade
<!-- Show element only to Super Admin -->
@if(Auth::user()->isSuperAdmin())
    <a href="/settings">Settings</a>
@endif

<!-- Show element to Super Admin and Campus Admin -->
@if(Auth::user()->isAdminLevel())
    <div>Approve Requests</div>
@endif

<!-- Show element to users in their campus -->
@if(Auth::user()->canManageStaff())
    <a href="/staff">Manage Staff</a>
@endif
```

### In Controllers

```php
class MedicineController extends Controller
{
    public function store(Request $request)
    {
        // Using policy authorization
        $this->authorize('create', Medicine::class);
        
        // Or manual check
        if (!auth()->user()->canManageMedicines()) {
            abort(403, 'Unauthorized');
        }
        
        // Create medicine...
    }
}
```

### In Routes

```php
// Super Admin only
Route::middleware('check.role:super_admin')->group(function () {
    Route::get('/settings', [SettingsController::class, 'index']);
});

// Super Admin and Admin
Route::middleware('check.role:super_admin,admin')->group(function () {
    Route::resource('staff', StaffController::class);
});

// All authenticated users
Route::middleware('auth')->group(function () {
    Route::get('/dashboard', [DashboardController::class, 'index']);
});
```

## Available Helper Methods

All methods below are available on authenticated users:

```php
Auth::user()->isSuperAdmin()              // Check if Super Admin
Auth::user()->isAdmin()                    // Check if Campus Admin
Auth::user()->isStaff()                    // Check if Staff
Auth::user()->isAdminLevel()               // Check if Super Admin or Admin

Auth::user()->canManageStaff()             // Can manage staff
Auth::user()->canManageMedicines()         // Can manage medicines
Auth::user()->canViewDistributions()       // Can view distributions
Auth::user()->canApproveMedicineRequests() // Can approve requests
Auth::user()->canLogMedicineUsage()        // Can log usage
Auth::user()->canAccessSettings()          // Can access settings
Auth::user()->canViewReports()             // Can view reports

Auth::user()->canViewCampus($campus)       // Can view specific campus
Auth::user()->canManageCampus($campus)     // Can manage specific campus
Auth::user()->getAccessibleCampuses()      // Get list of accessible campuses
```

## Access Matrix

| Feature | Super Admin | Campus Admin | Staff |
|---------|-------------|------|-------|
| Dashboard | ✅ | ✅ | ✅ |
| Patients | ✅ | ✅ Campus | ✅ Campus |
| Medical Records | ✅ | ✅ Campus | ✅ Campus |
| Dental Records | ✅ | ✅ Campus | ✅ Campus |
| Medicines (Overview) | ✅ | ✅ | ✅ |
| Medicines (Management) | ✅ | ✅ | ❌ |
| Distributions | ✅ | ❌ | ❌ |
| Usage Logs | ✅ | ✅ Campus | ✅ Campus |
| Requests | ✅ | ✅ Campus | ✅ Request |
| Staff Management | ✅ | ✅ Campus | ❌ |
| Reports | ✅ | ✅ Campus | ❌ |
| Settings | ✅ | ❌ | ❌ |

## Authorization Errors

If users try to access unauthorized resources, they will get:
- **403 Forbidden** error if they lack permissions
- **Redirected to login** if they're not authenticated
- **Sidebar items hidden** based on role

## Testing the System

1. Log in as **Super Admin** - See all menu items and full access
2. Log in as **Campus Admin** - Limited to their campus, no Settings
3. Log in as **Staff** - Only see Dashboard, Patients, Records, Medicines
4. Try accessing `/staff` as Staff - Should get 403 Forbidden

## Next Steps

1. **Customize campus assignment** - Add campus selector to user creation form
2. **Implement audit logging** - Log sensitive operations for compliance
3. **Add role-specific dashboards** - Different widgets for different roles
4. **Create admin panel** - For Super Admin to manage users and roles
5. **Add two-factor authentication** - For admin accounts
6. **Implement activity logs** - Track who accessed what and when

## Important Security Notes

- ⚠️ Always use `$this->authorize()` in controllers (not manual checks)
- ⚠️ Protect sensitive routes with middleware
- ⚠️ Validate campus ownership when filtering data
- ⚠️ Test authorization in your test suite
- ⚠️ Never trust frontend role checks - always verify on backend

## Troubleshooting

**"Call to undefined method isSuperAdmin()"**
- Make sure User model has `use HasRoles;` trait

**Authorization returning 403 unexpectedly**
- Check that policies are registered in AppServiceProvider
- Verify user has correct role and campus assigned

**Middleware not working**
- Ensure middleware aliases are in bootstrap/app.php
- Check route group has proper middleware applied

## Additional Resources

See `RBAC_IMPLEMENTATION.md` for:
- Complete API reference
- Migration examples
- Testing examples
- Security best practices
- Troubleshooting guide
