# RBAC Testing Guide - Complete

## Quick Test Setup

### Test Users (Password: `password123`)

```
Super Admin:      superadmin@psu.edu.ph
Admin Urdaneta:   admin.urdaneta@psu.edu.ph  
Admin Lingayen:   admin.lingayen@psu.edu.ph
Staff Nurse:      nurse.rosa@psu.edu.ph (Urdaneta Campus)
Staff Doctor:     doctor.pedro@psu.edu.ph (Urdaneta Campus)
Staff Dentist:    dentist.lisa@psu.edu.ph (Lingayen Campus)
Staff Aide:       aide.carlos@psu.edu.ph (Lingayen Campus)
```

---

## Test Scenario 1: Super Admin Access

**Login:** `superadmin@psu.edu.ph / password123`

### Dashboard Sidebar - Should See ✅
- [x] Dashboard
- [x] Patients
- [x] Medical Records
- [x] Dental Records
- [x] Medicines
- [x] Staff Management
- [x] Reports
- [x] Settings

### Medicines Page (`/medicines`)

**Buttons & UI - Should See ✅**
- [x] "Add Medicine" button visible (green)
- [x] Edit icon on each medicine row
- [x] Delete icon on each medicine row
- [x] "New Distribution" button visible

**Tab Navigation - Should See ✅**
- [x] "Overview" tab
- [x] "Stock per Campus" tab
- [x] "Distributions" tab (SUPER ADMIN ONLY)
- [x] "Requests" tab
- [x] "Usage Logs" tab

**Modals - Should See ✅**
- [x] "Add Medicine" modal opens when clicking button
- [x] "Edit Medicine" modal opens when clicking edit icon
- [x] "New Distribution" modal opens when clicking button
- [x] "Approve Request" button visible on pending requests

### Staff Management (`/staff`)

**Should See ✅**
- [x] Staff list with all campuses
- [x] "Add Staff" button
- [x] Edit buttons for each staff
- [x] Delete buttons for each staff

---

## Test Scenario 2: Campus Admin Access

**Login:** `admin.urdaneta@psu.edu.ph / password123`

### Dashboard Sidebar - Should See ✅
- [x] Dashboard
- [x] Patients
- [x] Medical Records
- [x] Dental Records
- [x] Medicines
- [x] Staff Management
- [x] Reports
- [ ] ❌ Settings (Super Admin Only)

### Medicines Page (`/medicines`)

**Buttons & UI - Should See ✅**
- [x] "Add Medicine" button visible (green)
- [x] Edit icons visible
- [x] Delete icons visible
- [ ] ❌ "New Distribution" button NOT visible

**Tab Navigation - Should See ✅**
- [x] "Overview" tab
- [x] "Stock per Campus" tab
- [ ] ❌ "Distributions" tab NOT visible (Super Admin only)
- [x] "Requests" tab (Admin-level only)
- [x] "Usage Logs" tab

**Modals - Should See ✅**
- [x] "Add Medicine" modal works
- [x] "Edit Medicine" modal works
- [ ] ❌ "New Distribution" modal NOT accessible
- [x] "Approve Request" button visible
- [x] Can approve pending requests

### Staff Management (`/staff`)

**Should See ✅**
- [x] Staff list filtered to their campus (Urdaneta only)
- [x] "Add Staff" button
- [x] Edit buttons for each staff
- [x] Delete buttons for each staff

### Campus Filtering

**Should See ✅**
- [x] Can only view staff from "Urdaneta City Campus"
- [x] Patients filtered to Urdaneta campus
- [x] Medical records from Urdaneta campus
- [x] Usage logs from Urdaneta campus

---

## Test Scenario 3: Staff Access

**Login:** `nurse.rosa@psu.edu.ph / password123`

### Dashboard Sidebar - Should See ✅
- [x] Dashboard
- [x] Patients
- [x] Medical Records
- [x] Dental Records
- [x] Medicines
- [ ] ❌ Staff Management NOT visible (Admin+ only)
- [ ] ❌ Reports NOT visible (Admin+ only)
- [ ] ❌ Settings NOT visible (Super Admin only)

### Medicines Page (`/medicines`)

**Buttons & UI - Should See ✅**
- [ ] ❌ "Add Medicine" button NOT visible
- [ ] ❌ Edit icons NOT visible (shows "—" instead)
- [ ] ❌ Delete icons NOT visible (shows "—" instead)
- [ ] ❌ "New Distribution" button NOT visible

**Tab Navigation - Should See ✅**
- [x] "Overview" tab
- [x] "Stock per Campus" tab
- [ ] ❌ "Distributions" tab NOT visible
- [ ] ❌ "Requests" tab NOT visible
- [x] "Usage Logs" tab

**Modals - Should NOT See ✅**
- [ ] ❌ "Add Medicine" modal NOT accessible
- [ ] ❌ "Edit Medicine" modal NOT accessible
- [ ] ❌ "New Distribution" modal NOT accessible
- [ ] ❌ "Approve Request" button NOT visible

**Usage Logs - Should See ✅**
- [x] "Usage Logs" tab visible
- [x] Can log medicine usage
- [x] Can see usage history

### Staff Management (`/staff`)

**Access - Should NOT See ✅**
- [ ] ❌ Cannot access `/staff` route (gets 403)
- [ ] ❌ No "Staff" link in sidebar
- [ ] ❌ No "Add Staff" button

---

## API Endpoint Testing

### Test Route Protection

```bash
# Super Admin can DELETE medicine
DELETE /medicines/1
Authorization: Bearer [super_admin_token]
Response: 200 OK ✅

# Campus Admin can DELETE medicine
DELETE /medicines/1
Authorization: Bearer [admin_token]
Response: 200 OK ✅

# Staff cannot DELETE medicine
DELETE /medicines/1
Authorization: Bearer [staff_token]
Response: 403 Forbidden ✅

# Super Admin can POST distribution
POST /distributions
Authorization: Bearer [super_admin_token]
Response: 200 OK ✅

# Campus Admin cannot POST distribution
POST /distributions
Authorization: Bearer [admin_token]
Response: 403 Forbidden ✅

# Staff cannot POST distribution
POST /distributions
Authorization: Bearer [staff_token]
Response: 403 Forbidden ✅
```

---

## Direct URL Testing

### Super Admin
```
GET /medicines                    → 200 ✅ (all visible)
POST /medicines                   → 200 ✅ (create works)
GET /medicines/1/edit             → 200 ✅ (edit form)
PUT /medicines/1                  → 200 ✅ (update works)
DELETE /medicines/1               → 200 ✅ (delete works)
POST /distributions               → 200 ✅ (distributions work)
GET /staff                        → 200 ✅ (staff visible)
```

### Campus Admin
```
GET /medicines                    → 200 ✅
POST /medicines                   → 200 ✅
GET /medicines/1/edit             → 200 ✅
PUT /medicines/1                  → 200 ✅
DELETE /medicines/1               → 200 ✅
POST /distributions               → 403 ✅ (forbidden)
GET /staff                        → 200 ✅ (campus-filtered)
```

### Staff
```
GET /medicines                    → 200 ✅ (view only)
POST /medicines                   → 403 ✅ (forbidden)
GET /medicines/1/edit             → 403 ✅ (forbidden)
PUT /medicines/1                  → 403 ✅ (forbidden)
DELETE /medicines/1               → 403 ✅ (forbidden)
POST /distributions               → 403 ✅ (forbidden)
GET /staff                        → 403 ✅ (forbidden)
POST /usage-logs                  → 200 ✅ (can log usage)
```

---

## Verification Checklist

### Backend Security ✅
- [x] Routes protected with `check.role` middleware
- [x] Controllers enforce authorization with policies
- [x] Unauthorized requests get 403 Forbidden
- [x] Campus filtering works for Admins

### Frontend Security ✅
- [x] Buttons hidden with `@if` blade directives
- [x] Tabs hidden based on role
- [x] Modals wrapped with role checks
- [x] Staff can't see admin buttons

### Database ✅
- [x] Users table has role ENUM column
- [x] Users table has campus ENUM column
- [x] Test users created with correct roles
- [x] No SQL injection vulnerabilities

### Error Handling ✅
- [x] Unauthorized access returns 403
- [x] Campus mismatch handled gracefully
- [x] Invalid IDs return 404
- [x] Missing permissions prevent actions

---

## Troubleshooting

### Issue: "Add Medicine" button visible to Staff
**Solution:** Check that medicines.blade.php line 349 has `@if(Auth::user()->isAdminLevel())`

### Issue: Staff can access `/staff` route
**Solution:** Verify routes/web.php has `check.role:super_admin,admin` middleware on staff routes

### Issue: Campus Admin sees all medicines globally
**Solution:** Verify MedicineController filters by campus for non-Super Admin users

### Issue: Distributions tab visible to Admin
**Solution:** Check medicines.blade.php has `@if(Auth::user()->isSuperAdmin())` on line 468

---

## Files Modified

1. ✅ `routes/web.php` - Added RBAC middleware
2. ✅ `resources/views/medicines.blade.php` - Added role checks to buttons/tabs/modals
3. ✅ `app/Http/Middleware/CheckRole.php` - Route protection
4. ✅ `app/Traits/HasRoles.php` - Permission helpers
5. ✅ `app/Policies/*.php` - Resource authorization

---

## Status: Ready for Testing ✅

All RBAC controls are in place. Use these test scenarios to verify security.
