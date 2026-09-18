# RBAC System - Quick Reference Guide

## 🚀 Quick Start

### Check User Role
```php
// In controller or model
if (Auth::user()->isSuperAdmin()) {
    // Super Admin only code
}

if (Auth::user()->isAdminLevel()) {
    // Admin-level (Super Admin or Campus Admin)
}

if (Auth::user()->isStaff()) {
    // Staff only code
}
```

### Check Permission
```php
// Simple permission check
if (Auth::user()->canManageMedicines()) {
    // User can manage medicines
}

// In controller - authorize method
$this->authorize('manage-medicines');

// In blade template
@can('manage-medicines')
    <!-- Show management UI -->
@endcan
```

### Get Accessible Campuses
```php
$campuses = Auth::user()->getAccessibleCampuses();
// Super Admin: ['Urdaneta', 'Lingayen', 'Binmaley', 'Bayambang', 'San Carlos']
// Others: ['Their Campus']
```

### Filter by Campus
```php
$query = Medicine::query();

if (!Auth::user()->isSuperAdmin()) {
    $query->where('campus', Auth::user()->campus);
}

$medicines = $query->get();
```

---

## 📋 Common Use Cases

### Use Case 1: Show/Hide Dashboard Stats
```blade
<!-- Show all campus stats only to Super Admin -->
@if(Auth::user()->getDashboardScope() === 'all_campuses')
    <div class="university-stats">
        <!-- University-wide statistics -->
    </div>
@else
    <div class="campus-stats">
        <!-- Campus statistics -->
    </div>
@endif
```

### Use Case 2: Medicine Management Page
```php
public function index()
{
    // Check permission
    $this->authorize('viewAny', Medicine::class);

    $medicines = Medicine::query();

    // Filter by role
    if (Auth::user()->isSuperAdmin()) {
        $medicines = $medicines->get(); // All
    } else {
        $medicines = $medicines->where('campus', Auth::user()->campus)->get();
    }

    return view('medicines', ['medicines' => $medicines]);
}

public function create()
{
    $this->authorize('create', Medicine::class); // Only Super Admin
    return view('medicines.create');
}
```

### Use Case 3: Medical Records with Role-Based Actions
```blade
@foreach($medicalRecords as $record)
    <tr>
        <td>{{ $record->patient->name }}</td>
        
        <!-- View - everyone can view their campus records -->
        <td>
            <a href="/records/{{ $record->id }}">View</a>
        </td>
        
        <!-- Edit - only admin-level can edit campus records -->
        @can('update', $record)
            <td>
                <a href="/records/{{ $record->id }}/edit">Edit</a>
            </td>
        @endcan
        
        <!-- Delete - only admin-level can delete -->
        @can('delete', $record)
            <td>
                <form method="POST" action="/records/{{ $record->id }}">
                    @method('DELETE')
                    @csrf
                    <button type="submit">Delete</button>
                </form>
            </td>
        @endcan
    </tr>
@endforeach
```

### Use Case 4: Staff Management (Super Admin & Campus Admin Only)
```php
public function index()
{
    $this->authorize('viewAny', Staff::class); // Only admin-level

    $staff = Staff::query();

    // Super Admin sees all, others see campus staff
    if (!Auth::user()->isSuperAdmin()) {
        $staff->where('campus', Auth::user()->campus);
    }

    return view('staff.index', ['staff' => $staff->get()]);
}
```

### Use Case 5: Medicine Requests (Staff Creates, Admin Approves)
```blade
<!-- Staff can create requests -->
@if(Auth::user()->can('create-medicine-requests'))
    <button class="btn" data-toggle="modal" data-target="#requestModal">
        Request Medicine
    </button>
@endif

<!-- Admin can approve requests -->
@if(Auth::user()->can('approve-medicine-requests'))
    <table>
        <tr>
            <td>{{ $request->medicine->name }}</td>
            <td>
                <button onclick="approveRequest({{ $request->id }})">
                    Approve
                </button>
            </td>
        </tr>
    </table>
@endif
```

---

## 🔑 Key Permission Methods

### Dashboard & Overview
- `canViewDashboard()` - View dashboard
- `getDashboardScope()` - 'all_campuses' or 'campus'

### Patient Management
- `canViewPatients()` - View patient list
- `canViewPatient($campus)` - View specific patient by campus

### Medical Records
- `canViewMedicalRecords()` - View records
- `canCreateMedicalRecords()` - Create records
- `canDeleteMedicalRecords()` - Delete records (admin-level)

### Dental Records
- `canViewDentalRecords()` - View records
- `canCreateDentalRecords()` - Create records
- `canDeleteDentalRecords()` - Delete records (admin-level)

### Medicines
- `canViewMedicinesOverview()` - View overview (everyone)
- `canViewMedicinesStock()` - View stock (admin-level)
- `canManageMedicines()` - Create/edit medicines (admin-level)
- `canViewMedicinesDistributions()` - View distributions (admin-level)
- `canManageMedicinesDistributions()` - Manage distributions (super-admin)
- `canCreateMedicineRequests()` - Create requests (anyone with campus)
- `canViewMedicineRequests()` - View requests (admin-level)
- `canApproveMedicineRequests()` - Approve requests (admin-level)
- `canLogMedicineUsage()` - Log usage (anyone with campus)

### Staff Management
- `canViewStaff()` - View staff list (admin-level)
- `canManageStaff()` - Add/edit/delete staff (admin-level)
- `canAddStaff()` - Add staff
- `canEditStaff()` - Edit staff
- `canDeleteStaff()` - Delete staff

### Reports & System
- `canViewReports()` - View reports (admin-level)
- `getReportsScope()` - 'university' or 'campus'
- `canAccessSettings()` - Access settings (super-admin)
- `canManageUserRoles()` - Manage roles (super-admin)

### Campus Access
- `canViewCampus($campus)` - Can view campus
- `canManageCampus($campus)` - Can manage campus
- `getAccessibleCampuses()` - Array of accessible campuses

---

## 🎯 Authorization Gates

All gates are available via `Gate::allows()`, `$user->can()`, or Blade `@can`.

```php
// Role gates
'super-admin'           // Is Super Admin
'campus-admin'          // Is Campus Admin
'staff'                 // Is Staff
'admin-level'           // Is Admin-level

// Feature gates
'view-dashboard'        // Can view dashboard
'view-patients'         // Can view patients
'view-medical-records'  // Can view medical records
'create-medical-records'// Can create medical records
'delete-medical-records'// Can delete medical records
'view-dental-records'   // Can view dental records
'create-dental-records' // Can create dental records
'delete-dental-records' // Can delete dental records
'view-medicines'        // Can view medicines
'manage-medicines'      // Can manage medicines
'view-medicines-distributions' // Can view distributions
'manage-medicines-distributions' // Can manage distributions
'view-medicine-requests'// Can view requests
'create-medicine-requests' // Can create requests
'approve-medicine-requests' // Can approve requests
'view-medicine-usage-logs' // Can view usage logs
'log-medicine-usage'    // Can log usage
'view-staff'            // Can view staff
'manage-staff'          // Can manage staff
'add-staff'             // Can add staff
'edit-staff'            // Can edit staff
'delete-staff'          // Can delete staff
'view-clearances'       // Can view clearances
'approve-clearances'    // Can approve clearances
'view-reports'          // Can view reports
'access-settings'       // Can access settings
'manage-user-roles'     // Can manage user roles
'switch-campuses'       // Can switch campuses
```

---

## 📁 File Structure

```
app/
├── Traits/
│   └── HasRoles.php                    (50+ permission methods)
├── Policies/
│   ├── MedicinePolicy.php              (15 methods)
│   ├── MedicalRecordPolicy.php         (10 methods)
│   ├── DentalRecordPolicy.php          (10 methods)
│   └── StaffPolicy.php                 (12 methods)
├── Http/
│   ├── Middleware/
│   │   ├── CheckRole.php               (Role validation)
│   │   └── CheckCampusAccess.php       (Campus validation)
│   └── Controllers/
│       └── *Controller.php             (Authorization checks)
└── Providers/
    └── AppServiceProvider.php          (60+ gates)

tests/
├── Unit/
│   └── RBACUnitTest.php                (70+ tests)
└── Feature/
    └── RBACFeatureTest.php             (50+ scenarios)
```

---

## 🧪 Testing

### Run Unit Tests
```bash
php artisan test tests/Unit/RBACUnitTest.php
```

### Run Feature Tests
```bash
php artisan test tests/Feature/RBACFeatureTest.php
```

### Run All RBAC Tests
```bash
php artisan test tests/Unit/RBACUnitTest.php tests/Feature/RBACFeatureTest.php
```

### Run Single Test Method
```bash
php artisan test tests/Unit/RBACUnitTest.php --filter super_admin_has_correct_role
```

---

## 🔒 Best Practices

1. **Always authorize in controllers**
   ```php
   $this->authorize('action', Model::class);
   ```

2. **Use gates in views**
   ```blade
   @can('manage-medicines')
       <!-- Content -->
   @endcan
   ```

3. **Filter queries by campus**
   ```php
   if (!$user->isSuperAdmin()) {
       $query->where('campus', $user->campus);
   }
   ```

4. **Never trust frontend validation**
   - Always validate permissions in backend
   - Use policies and gates consistently

5. **Use role checking for UI control**
   ```blade
   @if($user->isSuperAdmin())
       <!-- Show university-wide controls -->
   @elseif($user->isAdmin())
       <!-- Show campus controls -->
   @else
       <!-- Show staff interface -->
   @endif
   ```

6. **Document permission requirements**
   ```php
   /**
    * Only Super Admin can access this
    * @throws \Illuminate\Auth\Access\AuthorizationException
    */
   public function dangerousAction()
   {
       $this->authorize('super-admin');
   }
   ```

---

## 🆘 Troubleshooting

### User sees 403 Forbidden
- Check their role in database
- Verify their campus assignment
- Check policy method returns true

### Feature appears for wrong role
- Check Blade template conditions
- Verify @can directive uses correct gate
- Check policy file logic

### Campus filtering not working
- Verify user has campus assigned
- Check query has campus filter
- Verify Super Admin bypass logic

### Policy not being called
- Check policy is registered in AppServiceProvider
- Verify model uses policy
- Check gate is defined

---

## 📞 Need Help?

1. Check the comprehensive documentation in `RBAC_SPECIFICATION_COMPLETE.md`
2. Review test files for usage examples
3. Check existing controller implementations
4. Run tests to verify system works

---

**Last Updated:** April 28, 2026  
**Quick Reference Version:** 1.0
