# RBAC Routes & Views - Fixes Applied ✅

## Date: April 28, 2026

### Summary of Changes

Fixed and enhanced the **Routes & Views** to properly enforce **Role-Based Access Control (RBAC)** across the application.

---

## 1. Routes Updated (`routes/web.php`)

### ✅ Medicines Routes - Now Protected

**View (All authenticated users)**
```php
Route::get('/medicines', [MedicineController::class, 'index'])->middleware('auth');
```

**Create/Edit/Delete (Super Admin & Campus Admin Only)**
```php
Route::middleware(['auth', 'check.role:super_admin,admin'])->group(function () {
    Route::post('/medicines', [MedicineController::class, 'store']);
    Route::get('/medicines/{id}/edit', [MedicineController::class, 'edit']);
    Route::put('/medicines/{id}', [MedicineController::class, 'update']);
    Route::delete('/medicines/{id}', [MedicineController::class, 'destroy']);
});
```

### ✅ Distributions Routes - Super Admin Only

```php
Route::post('/distributions', [MedicineController::class, 'storeDistribution'])
    ->middleware(['auth', 'check.role:super_admin']);
```

### ✅ Requests Approval - Admin Only

```php
Route::post('/requests/{id}/approve', [MedicineController::class, 'approveRequest'])
    ->middleware(['auth', 'check.role:super_admin,admin']);
```

### ✅ Usage Logs - All Campus Users

```php
Route::post('/usage-logs', [MedicineController::class, 'storeUsageLog'])
    ->middleware('auth');
```

### ✅ Staff Management - Admin Only

```php
Route::middleware(['auth', 'check.role:super_admin,admin'])->group(function () {
    Route::resource('staff', StaffController::class);
});
```

---

## 2. Medicines View Updated (`resources/views/medicines.blade.php`)

### ✅ Add Medicine Button - Hidden from Staff

**Before:**
```blade
<button class="btn btn-primary btn-lg" data-bs-toggle="modal" data-bs-target="#addMedicineModal">
    <i class="fas fa-plus me-2"></i>Add Medicine
</button>
```

**After:**
```blade
@if(Auth::user()->isAdminLevel())
<button class="btn btn-primary btn-lg" data-bs-toggle="modal" data-bs-target="#addMedicineModal">
    <i class="fas fa-plus me-2"></i>Add Medicine
</button>
@endif
```

### ✅ Edit/Delete Buttons - Hidden from Staff

**Before:**
```blade
<div class="btn-group btn-group-sm" role="group">
    <button type="button" class="btn btn-outline-primary btn-sm" onclick="editMedicine(...)">
        <i class="fas fa-edit"></i>
    </button>
    <button type="button" class="btn btn-outline-danger btn-sm" onclick="deleteMedicine(...)">
        <i class="fas fa-trash"></i>
    </button>
</div>
```

**After:**
```blade
@if(Auth::user()->isAdminLevel())
<div class="btn-group btn-group-sm" role="group">
    <button type="button" class="btn btn-outline-primary btn-sm" onclick="editMedicine(...)">
        <i class="fas fa-edit"></i>
    </button>
    <button type="button" class="btn btn-outline-danger btn-sm" onclick="deleteMedicine(...)">
        <i class="fas fa-trash"></i>
    </button>
</div>
@else
<span class="text-muted">—</span>
@endif
```

### ✅ Tab Navigation - Role-Based Visibility

**Distributions Tab**
- Shows only for Super Admin

```blade
@if(Auth::user()->isSuperAdmin())
<li class="nav-item">
    <a class="nav-link fw-600" id="distributions-tab" data-bs-toggle="tab" href="#distributions">
        Distributions
    </a>
</li>
@endif
```

**Requests Tab**
- Shows only for Super Admin & Campus Admin

```blade
@if(Auth::user()->isAdminLevel())
<li class="nav-item">
    <a class="nav-link fw-600" id="requests-tab" data-bs-toggle="tab" href="#requests">
        Requests
    </a>
</li>
@endif
```

### ✅ Approve Request Button - Admin Only

**Before:**
```blade
@if($request->status === 'Pending')
    <button type="button" class="btn btn-success btn-sm" onclick="approveRequest(...)">
        Approve
    </button>
@else
    <span class="text-muted">—</span>
@endif
```

**After:**
```blade
@if(Auth::user()->isAdminLevel() && $request->status === 'Pending')
    <button type="button" class="btn btn-success btn-sm" onclick="approveRequest(...)">
        Approve
    </button>
@else
    <span class="text-muted">—</span>
@endif
```

### ✅ Modals - Protected with Role Checks

All admin-only modals wrapped in role checks:

```blade
@if(Auth::user()->isAdminLevel())
<!-- Add Medicine Modal -->
<div class="modal fade" id="addMedicineModal" ...>
    ...
</div>

<!-- Edit Medicine Modal -->
<div class="modal fade" id="editMedicineModal" ...>
    ...
</div>
@endif

@if(Auth::user()->isSuperAdmin())
<!-- Distribution Modal -->
<div class="modal fade" id="distributionModal" ...>
    ...
</div>
@endif
```

---

## 3. Access Control Summary

### Super Admin Access ✅
- ✅ View all medicines
- ✅ Create, Edit, Delete medicines
- ✅ View and manage distributions
- ✅ Approve medicine requests
- ✅ View and manage staff
- ✅ View usage logs from all campuses

### Campus Admin Access ✅
- ✅ View all medicines
- ✅ Create, Edit, Delete medicines (global)
- ✅ Cannot manage distributions (Super Admin only)
- ✅ Approve medicine requests for their campus
- ✅ View and manage staff in their campus
- ✅ View usage logs for their campus

### Staff Access ✅
- ✅ View medicines overview
- ✅ Cannot create/edit/delete medicines
- ✅ Cannot see distributions tab
- ✅ Cannot see requests tab
- ✅ Can log medicine usage
- ✅ Cannot manage staff

---

## 4. Route Protection Matrix

| Route | Method | Super Admin | Admin | Staff | Public |
|-------|--------|---|---|---|---|
| `/medicines` | GET | ✅ | ✅ | ✅ | ❌ |
| `/medicines` | POST | ✅ | ✅ | ❌ | ❌ |
| `/medicines/{id}/edit` | GET | ✅ | ✅ | ❌ | ❌ |
| `/medicines/{id}` | PUT | ✅ | ✅ | ❌ | ❌ |
| `/medicines/{id}` | DELETE | ✅ | ✅ | ❌ | ❌ |
| `/distributions` | POST | ✅ | ❌ | ❌ | ❌ |
| `/requests/{id}/approve` | POST | ✅ | ✅ | ❌ | ❌ |
| `/usage-logs` | POST | ✅ | ✅ | ✅ | ❌ |
| `/staff` | GET | ✅ | ✅ | ❌ | ❌ |
| `/staff` | POST | ✅ | ✅ | ❌ | ❌ |
| `/staff/{id}` | GET | ✅ | ✅ | ❌ | ❌ |
| `/staff/{id}` | PUT | ✅ | ✅ | ❌ | ❌ |
| `/staff/{id}` | DELETE | ✅ | ✅ | ❌ | ❌ |

---

## 5. Files Modified

| File | Changes |
|------|---------|
| `routes/web.php` | Added `check.role` middleware to CRUD routes |
| `resources/views/medicines.blade.php` | Added role checks to buttons, tabs, and modals |

---

## 6. Testing Instructions

### Test Super Admin:
1. Login as `superadmin@psu.edu.ph / password123`
2. Go to `/medicines`
3. **Verify:**
   - ✅ "Add Medicine" button visible
   - ✅ Edit/Delete buttons visible on medicines
   - ✅ "Stock per Campus" tab visible
   - ✅ "Distributions" tab visible
   - ✅ "Requests" tab visible
   - ✅ "New Distribution" button visible
   - ✅ "Approve" buttons visible on pending requests
   - ✅ Staff link visible in sidebar

### Test Campus Admin:
1. Login as `admin.urdaneta@psu.edu.ph / password123`
2. Go to `/medicines`
3. **Verify:**
   - ✅ "Add Medicine" button visible
   - ✅ Edit/Delete buttons visible
   - ✅ "Stock per Campus" tab visible
   - ✅ "Distributions" tab **NOT** visible
   - ✅ "Requests" tab visible
   - ✅ "New Distribution" button **NOT** visible
   - ✅ "Approve" buttons visible on pending requests
   - ✅ Staff link visible in sidebar

### Test Staff:
1. Login as `nurse.rosa@psu.edu.ph / password123`
2. Go to `/medicines`
3. **Verify:**
   - ✅ "Add Medicine" button **NOT** visible
   - ✅ Edit/Delete buttons **NOT** visible (shows "—" instead)
   - ✅ "Stock per Campus" tab visible
   - ✅ "Distributions" tab **NOT** visible
   - ✅ "Requests" tab **NOT** visible
   - ✅ "Usage Logs" tab visible
   - ✅ Can log medicine usage
   - ✅ Staff link **NOT** visible in sidebar

---

## 7. Security Improvements

✅ **Backend Protection** - Routes enforce role checks at middleware level
✅ **Frontend Protection** - UI buttons hidden based on role
✅ **Modal Protection** - Admin-only modals wrapped in role checks
✅ **Tab Visibility** - Tabs conditionally shown based on role
✅ **Error Prevention** - Staff cannot accidentally access admin features

---

## 8. Notes

- **Route-level protection** is PRIMARY defense (backend)
- **View-level hiding** is SECONDARY UX enhancement
- Users attempting to POST/PUT/DELETE without permission will get **403 Forbidden** from middleware
- Try navigating directly to `/medicines/1/edit` as Staff user to verify backend protection

---

## Status: ✅ COMPLETE

All RBAC controls have been properly implemented in routes and views.
