# RBAC System - Complete Specification & Implementation Guide

## Overview

This document provides a comprehensive specification of the Role-Based Access Control (RBAC) system for the PSU Medical Management System. The system implements three distinct roles with hierarchical permissions and campus-based access restrictions.

## Table of Contents

1. [Roles & Responsibilities](#roles--responsibilities)
2. [Feature Access Matrix](#feature-access-matrix)
3. [Permissions & Capabilities](#permissions--capabilities)
4. [Technical Implementation](#technical-implementation)
5. [Usage Examples](#usage-examples)
6. [API Reference](#api-reference)

---

## Roles & Responsibilities

### 1. **Super Admin (Highest Access)**

**Role Name:** `super_admin` | **Campus:** `null` (all campuses)

**Who They Are:**
- University-wide administrator (maybe from main IT/Admin office)
- Top system administrator with complete control

**Main Responsibilities:**
- Full system control across all campuses
- System configuration & maintenance
- User role management
- Financial & administrative oversight
- University-wide reporting

**Access Level:** Full system access across all campuses

---

### 2. **Campus Admin (Campus-level Administrator / Clinic Head)**

**Role Name:** `admin` | **Campus:** One specific PSU campus (Urdaneta, Lingayen, etc.)

**Who They Are:**
- Head of clinic per campus
- Campus medical officer
- Campus administrator responsible for operations

**Main Responsibilities:**
- Manages their assigned campus operations only
- Staff management for their campus
- Patient & medical record management
- Medicine stock & requests for their campus
- Campus-specific reporting

**Access Level:** Limited to their assigned campus

---

### 3. **Staff (Clinic Nurse, Dentist, Medical Aide, etc.)**

**Role Name:** `staff` | **Campus:** One specific PSU campus

**Who They Are:**
- Regular clinic personnel
- Nurses, dentists, medical aides, support staff
- Frontline clinical workers

**Main Responsibilities:**
- Daily clinical operations & patient consultations
- Creating medical/dental records during consultations
- Logging medicine usage
- Requesting medicines when stock is low
- Patient encounter documentation

**Access Level:** Limited - focused on daily clinical work only

---

## Feature Access Matrix

| Feature | Super Admin | Campus Admin | Staff |
|---------|:-----------:|:-----------:|:-----:|
| **DASHBOARD** | | | |
| View Dashboard | ✅ All campuses | ✅ Their campus | ✅ Their campus |
| View All Campus Statistics | ✅ | ❌ | ❌ |
| **PATIENTS** | | | |
| View All Patients | ✅ All campuses | ✅ Their campus | ✅ Their campus |
| Add New Patients | ✅ | ✅ | ❌ |
| Edit Patient Info | ✅ | ✅ | ❌ |
| Delete Patients | ✅ | ✅ | ❌ |
| **MEDICAL RECORDS** | | | |
| View Medical Records | ✅ All | ✅ Campus only | ✅ Campus only |
| Add New Medical Records | ✅ | ✅ | ✅ |
| Create Consultations | ✅ | ✅ | ✅ |
| Edit Medical Records | ✅ | ✅ Campus only | ❌ |
| Delete Medical Records | ✅ | ✅ Campus only | ❌ |
| **DENTAL RECORDS** | | | |
| View Dental Records | ✅ All | ✅ Campus only | ✅ Campus only |
| Add New Dental Records | ✅ | ✅ | ✅ |
| Edit Dental Records | ✅ | ✅ Campus only | ❌ |
| Delete Dental Records | ✅ | ✅ Campus only | ❌ |
| **MEDICINES** | | | |
| View Overview | ✅ | ✅ | ✅ |
| View Stock per Campus | ✅ | ✅ | ❌ |
| View Distributions | ✅ | ✅ | ❌ |
| Manage Distributions | ✅ | ❌ | ❌ |
| Create/Edit Medicines | ✅ | ✅ | ❌ |
| Delete Medicines | ✅ | ❌ | ❌ |
| View Medicine Requests | ✅ | ✅ | ❌ |
| Create Requests | ✅ | ✅ | ✅ |
| Approve Requests | ✅ | ✅ | ❌ |
| View Usage Logs | ✅ | ✅ | ✅ |
| Log Medicine Usage | ✅ | ✅ | ✅ |
| **STAFF MANAGEMENT** | | | |
| View Staff List | ✅ All | ✅ Campus only | ❌ |
| Add Staff | ✅ | ✅ | ❌ |
| Edit Staff | ✅ | ✅ Campus only | ❌ |
| Delete Staff | ✅ | ✅ Campus only | ❌ |
| Manage Other Admins | ✅ | ❌ | ❌ |
| **CLEARANCES** | | | |
| View Clearances | ✅ | ✅ | ❌ |
| Approve Clearances | ✅ | ✅ | ❌ |
| **REPORTS** | | | |
| View University-wide Reports | ✅ | ❌ | ❌ |
| View Campus Reports | ✅ | ✅ | ❌ |
| Export Data | ✅ | ✅ | ❌ |
| **SYSTEM SETTINGS** | | | |
| Access System Settings | ✅ | ❌ | ❌ |
| Manage User Roles | ✅ | ❌ | ❌ |
| Manage User Accounts | ✅ | ❌ | ❌ |
| Switch Between All Campuses | ✅ | ❌ | ❌ |

---

## Permissions & Capabilities

### Role Checking Methods

```php
// Check if user is Super Admin
if (Auth::user()->isSuperAdmin()) { }

// Check if user is Campus Admin
if (Auth::user()->isAdmin()) { }

// Check if user is Staff
if (Auth::user()->isStaff()) { }

// Check if user is Admin-level (Super Admin OR Campus Admin)
if (Auth::user()->isAdminLevel()) { }
```

### Dashboard Access

```php
// Check if user can view dashboard
if (Auth::user()->canViewDashboard()) { }

// Get dashboard scope
$scope = Auth::user()->getDashboardScope();
// Returns: 'all_campuses' for Super Admin, 'campus' for others
```

### Patient Access

```php
// Check if user can view patients
if (Auth::user()->canViewPatients()) { }

// Check if user can view a specific patient
if (Auth::user()->canViewPatient($patientCampus)) { }
```

### Medical Records

```php
// View medical records
if (Auth::user()->canViewMedicalRecords()) { }

// Create medical records
if (Auth::user()->canCreateMedicalRecords()) { }

// Delete medical records (Admin-level only)
if (Auth::user()->canDeleteMedicalRecords()) { }
```

### Dental Records

```php
// View dental records
if (Auth::user()->canViewDentalRecords()) { }

// Create dental records
if (Auth::user()->canCreateDentalRecords()) { }

// Delete dental records (Admin-level only)
if (Auth::user()->canDeleteDentalRecords()) { }
```

### Medicines

```php
// Overview - everyone
if (Auth::user()->canViewMedicinesOverview()) { }

// Stock per campus - Admin-level only
if (Auth::user()->canViewMedicinesStock()) { }

// Full management - Admin-level only
if (Auth::user()->canManageMedicines()) { }

// Distributions - Admin-level only
if (Auth::user()->canViewMedicinesDistributions()) { }

// Manage distributions - Super Admin only
if (Auth::user()->canManageMedicinesDistributions()) { }

// View requests - Admin-level only
if (Auth::user()->canViewMedicineRequests()) { }

// Create requests - Anyone with campus assigned
if (Auth::user()->canCreateMedicineRequests()) { }

// Approve requests - Admin-level only
if (Auth::user()->canApproveMedicineRequests()) { }

// Log usage - Anyone with campus assigned
if (Auth::user()->canLogMedicineUsage()) { }
```

### Staff Management

```php
// View staff - Admin-level only
if (Auth::user()->canViewStaff()) { }

// Manage staff (add/edit/delete) - Admin-level only
if (Auth::user()->canManageStaff()) { }

// Can add staff
if (Auth::user()->canAddStaff()) { }

// Can edit staff
if (Auth::user()->canEditStaff()) { }

// Can delete staff
if (Auth::user()->canDeleteStaff()) { }
```

### Reports & Clearances

```php
// View reports - Admin-level only
if (Auth::user()->canViewReports()) { }

// Get reports scope
$scope = Auth::user()->getReportsScope();
// Returns: 'university' for Super Admin, 'campus' for Admin

// View clearances - Admin-level only
if (Auth::user()->canViewClearances()) { }

// Approve clearances - Admin-level only
if (Auth::user()->canApproveClearances()) { }
```

### System Settings

```php
// Access settings - Super Admin only
if (Auth::user()->canAccessSettings()) { }

// Manage user roles - Super Admin only
if (Auth::user()->canManageUserRoles()) { }

// Switch campuses - Super Admin only
if (Auth::user()->canSwitchCampuses()) { }
```

### Campus Access

```php
// Check if user can view a specific campus
if (Auth::user()->canViewCampus($campus)) { }

// Check if user can manage a specific campus
if (Auth::user()->canManageCampus($campus)) { }

// Get accessible campuses
$campuses = Auth::user()->getAccessibleCampuses();
// Returns array of campus names
```

---

## Technical Implementation

### Database Schema

#### Users Table
```sql
ALTER TABLE users ADD COLUMN role ENUM('super_admin', 'admin', 'staff') DEFAULT 'staff';
ALTER TABLE users ADD COLUMN campus VARCHAR(255) NULLABLE;
```

### Traits

#### HasRoles Trait
Located: `app/Traits/HasRoles.php`

Provides 50+ permission-checking methods used throughout the application.

### Policies

Authorization policies define what actions users can perform on specific models:

| Policy | File | Model |
|--------|------|-------|
| MedicinePolicy | `app/Policies/MedicinePolicy.php` | Medicine |
| MedicalRecordPolicy | `app/Policies/MedicalRecordPolicy.php` | MedicalRecord |
| DentalRecordPolicy | `app/Policies/DentalRecordPolicy.php` | DentalRecord |
| StaffPolicy | `app/Policies/StaffPolicy.php` | Staff |

### Middleware

#### CheckRole Middleware
Location: `app/Http/Middleware/CheckRole.php`

Protects routes by role:
```php
// Only Super Admin
Route::get('/admin', function() {})->middleware('check.role:super_admin');

// Super Admin and Campus Admin
Route::get('/admin-panel', function() {})->middleware('check.role:super_admin,admin');

// Staff and above
Route::get('/clinic', function() {})->middleware('check.role:staff,admin,super_admin');
```

#### CheckCampusAccess Middleware
Location: `app/Http/Middleware/CheckCampusAccess.php`

Validates campus-specific access.

### AppServiceProvider

Registers all policies and gates in `app/Providers/AppServiceProvider.php`.

Available Gates: 50+ authorization gates for checking permissions in views and controllers.

---

## Usage Examples

### In Controllers

```php
class MedicineController extends Controller
{
    public function index()
    {
        // Check permission
        $this->authorize('viewAny', Medicine::class);

        // Only show medicines for accessible campuses
        $user = auth()->user();
        if ($user->isSuperAdmin()) {
            $medicines = Medicine::all();
        } else {
            $medicines = Medicine::where('campus', $user->campus)->get();
        }

        return view('medicines', ['medicines' => $medicines]);
    }

    public function update(Request $request, Medicine $medicine)
    {
        // Check if user can update this medicine
        $this->authorize('update', $medicine);

        // Update logic
        $medicine->update($request->validated());
    }
}
```

### In Blade Templates

```blade
@if(Auth::user()->isSuperAdmin())
    <div>Super Admin Dashboard</div>
@elseif(Auth::user()->isAdmin())
    <div>Campus Admin Dashboard</div>
@else
    <div>Staff Dashboard</div>
@endif

@can('manage-medicines')
    <a href="/medicines/create">Add Medicine</a>
@endcan

@can('view', $medicalRecord)
    <div>View Medical Record</div>
@endcan
```

### Checking Gates

```php
// Using gate
if (Gate::allows('manage-staff')) {
    // User can manage staff
}

// Using user method
if (Auth::user()->can('manage-staff')) {
    // User can manage staff
}

// Using authorize
$this->authorize('manage-staff');
```

---

## API Reference

### User Model Methods

#### Role Checking
- `isSuperAdmin(): bool`
- `isAdmin(): bool`
- `isStaff(): bool`
- `isAdminLevel(): bool`

#### Dashboard
- `canViewDashboard(): bool`
- `getDashboardScope(): string`

#### Patients
- `canViewPatients(): bool`
- `canViewPatient($campus): bool`

#### Medical Records
- `canViewMedicalRecords(): bool`
- `canCreateMedicalRecords(): bool`
- `canAddMedicalRecords(): bool`
- `canDeleteMedicalRecords(): bool`

#### Dental Records
- `canViewDentalRecords(): bool`
- `canCreateDentalRecords(): bool`
- `canAddDentalRecords(): bool`
- `canDeleteDentalRecords(): bool`

#### Medicines
- `canViewMedicinesOverview(): bool`
- `canViewMedicinesStock(): bool`
- `canManageMedicines(): bool`
- `canViewMedicinesDistributions(): bool`
- `canManageMedicinesDistributions(): bool`
- `canViewMedicineRequests(): bool`
- `canCreateMedicineRequests(): bool`
- `canApproveMedicineRequests(): bool`
- `canViewMedicineUsageLogs(): bool`
- `canLogMedicineUsage(): bool`

#### Staff Management
- `canViewStaff(): bool`
- `canManageStaff(): bool`
- `canAddStaff(): bool`
- `canEditStaff(): bool`
- `canDeleteStaff(): bool`

#### Clearances
- `canViewClearances(): bool`
- `canApproveClearances(): bool`

#### Reports
- `canViewReports(): bool`
- `getReportsScope(): string`

#### System
- `canAccessSettings(): bool`
- `canManageUserRoles(): bool`
- `canSwitchCampuses(): bool`

#### Campus Access
- `canViewCampus($campus): bool`
- `canManageCampus($campus): bool`
- `getAccessibleCampuses(): array`

#### Utility
- `getRoleDisplayName(): string`
- `getRoleDescription(): string`
- `hasHighPrivilege(): bool`

---

## Campus List

The system supports 5 PSU campuses:

1. **Urdaneta**
2. **Lingayen**
3. **Binmaley**
4. **Bayambang**
5. **San Carlos**

---

## Testing the RBAC System

### Test Users

```
Super Admin:
Email: superadmin@psu.edu.ph
Password: password123

Campus Admin (Urdaneta):
Email: admin.urdaneta@psu.edu.ph
Password: password123

Campus Admin (Lingayen):
Email: admin.lingayen@psu.edu.ph
Password: password123

Staff Member:
Email: nurse.rosa@psu.edu.ph
Password: password123
```

### Testing Access

1. Log in as each role
2. Navigate to different features
3. Verify that unauthorized access is blocked
4. Confirm role-specific features are visible/hidden appropriately

---

## Implementation Checklist

- [x] HasRoles trait created with 50+ permission methods
- [x] DentalRecordPolicy created
- [x] MedicinePolicy updated with comprehensive methods
- [x] MedicalRecordPolicy updated with comprehensive methods
- [x] StaffPolicy updated with comprehensive methods
- [x] AppServiceProvider updated with 60+ gates
- [x] CheckRole middleware configured
- [x] CheckCampusAccess middleware configured
- [x] Controllers updated with authorization checks
- [x] Comprehensive documentation created
- [ ] Unit tests created
- [ ] Feature tests created

---

## Best Practices

1. **Always authorize actions** - Use `$this->authorize()` in controllers
2. **Use gates in views** - Use `@can` and `@gate` directives in Blade
3. **Campus filtering** - Always filter data by user's campus when not Super Admin
4. **Consistent naming** - Use the permission methods instead of duplicating logic
5. **Test all roles** - Test features with each role to ensure proper access control
6. **Defensive coding** - Never trust the user's role; always verify in backend

---

## Troubleshooting

### User cannot see feature that should be accessible

1. Check the role in database
2. Verify campus assignment (for Admin/Staff)
3. Check blade template for correct `@can` directive
4. Verify policy method returns true for user role

### Unauthorized access not blocked

1. Verify middleware is applied to route
2. Check policy method logic
3. Check campus comparison logic

### Confusion about permissions

1. Refer to the Feature Access Matrix above
2. Check the specific permission method in HasRoles trait
3. Consult the API Reference section

---

## Support & Questions

For questions about the RBAC system:

1. Check this documentation
2. Review the relevant policy file
3. Check the HasRoles trait methods
4. Run unit/feature tests

---

**Last Updated:** April 28, 2026  
**Version:** 1.0  
**Status:** Complete Implementation
