# Medicine Dispensing Feature - Implementation Guide

## Overview
The Medicine Dispensing feature allows staff members to dispense medicines during patient visits. When a medical record is created, staff can select medicines from the inventory, specify quantities and dosages, and the system will automatically deduct the stock from the medicine inventory.

---

## Files Created/Modified

### 1. **Database Migration**
**File:** `database/migrations/2024_01_01_000000_create_medicine_dispenses_table.php`

Creates the `medicine_dispenses` table with the following fields:
- `id` - Primary key
- `medical_record_id` - Links to medical records
- `patient_id` - Links to patients
- `medicine_id` - Links to medicines
- `quantity` - Amount dispensed
- `dosage` - e.g., "500mg", "2 tablets"
- `instructions` - e.g., "Take twice daily"
- `dispensed_by` - Staff member who dispensed
- `dispensed_at` - Timestamp
- Indexes for fast queries

### 2. **Models**

#### **MedicineDispense Model**
**File:** `app/Models/MedicineDispense.php`

Key features:
- Relationships with MedicalRecord, Patient, Medicine, User
- `forCampus()` scope to filter by user's campus (Super Admin sees all)
- `getFormattedDateAttribute()` accessor for date formatting

#### **Updated MedicalRecord Model**
**File:** `app/Models/MedicalRecord.php` (updated)

Added:
```php
public function medicineDispenses()
{
    return $this->hasMany(MedicineDispense::class);
}
```

#### **Updated Medicine Model**
**File:** `app/Models/Medicine.php` (updated)

Added:
```php
public function medicineDispenses()
{
    return $this->hasMany(MedicineDispense::class);
}
```

### 3. **Controllers**

#### **MedicalRecordsController Updates**
**File:** `app/Http/Controllers/MedicalRecordsController.php` (updated)

New imports:
```php
use App\Models\MedicineDispense;
use Illuminate\Support\Facades\DB;
```

**Modified `store()` method:**
- Now accepts `medicines_dispensed` array in request
- Validates medicine data (medicine_id, quantity, dosage, instructions)
- Uses database transaction for data integrity
- Creates MedicineDispense records
- Automatically decrements medicine stock using `$medicine->decrement()`
- Handles insufficient stock errors

**New `getAvailableMedicines()` method:**
- Returns list of active medicines with stock levels
- Shows low stock warnings
- Used by frontend to populate medicine selection dropdown

#### **MedicineDispensingController**
**File:** `app/Http/Controllers/MedicineDispensingController.php` (new)

Methods:
- `index()` - Shows dispensing logs page with statistics
- `getData()` - Returns filtered dispensing records (JSON)
- `getStatistics()` - Returns total dispensed, this month count, top medicines
- `exportCSV()` - Exports dispensing logs to CSV file

---

## 4. **Routes**

**File:** `routes/web.php` (updated)

Added routes:
```php
// Get available medicines
Route::get('/medical-records/medicines', [MedicalRecordsController::class, 'getAvailableMedicines'])
    ->middleware('auth');

// Dispensing logs page
Route::get('/medicine-dispensing', [MedicineDispensingController::class, 'index'])
    ->middleware('auth')
    ->name('medicine-dispensing');

// Dispensing logs API endpoints
Route::get('/medicine-dispensing/data', [MedicineDispensingController::class, 'getData'])
    ->middleware('auth');
Route::get('/medicine-dispensing/statistics', [MedicineDispensingController::class, 'getStatistics'])
    ->middleware('auth');
Route::get('/medicine-dispensing/export', [MedicineDispensingController::class, 'exportCSV'])
    ->middleware('auth')
    ->name('medicine-dispensing.export');
```

---

## 5. **Views**

#### **Updated Medical Records Modal**
**File:** `resources/views/medical-records.blade.php` (updated)

Added "Medications Dispensed" section:
- Shows in the "Add Medical Record" modal
- Dynamic rows for adding multiple medicines
- Shows current stock levels
- Low stock warnings
- Medicine, quantity, dosage, and instructions fields
- "Add Medicine" button to add more rows
- "Remove" (✕) button to remove rows

#### **Medicine Dispensing Logs Page**
**File:** `resources/views/medicine-dispensing.blade.php` (new)

Features:
- Dashboard with stat cards (Total, This Month, Top Medicine)
- Date range filtering
- Export to CSV
- Paginated table showing all dispensing records
- Responsive design matching system UI

---

## How to Use

### 1. **Add Medical Record with Medicine Dispensing**

1. Go to **Medical Records** page
2. Click **"Add Medical Record"** button
3. Fill in patient information, diagnosis, treatment, etc.
4. In the **"Medications Dispensed"** section:
   - Click **"+ Add Medicine"** to add medicine rows
   - Select medicine (shows current stock)
   - Enter quantity to dispense
   - Enter dosage (optional, e.g., "500mg", "2 tablets")
   - Enter instructions (optional, e.g., "Take twice daily")
   - Click **✕** to remove a row if needed
5. Click **"Add Record"** to save

**What happens:**
- Medical record is created
- For each medicine dispensed:
  - MedicineDispense record is created
  - Medicine stock is automatically reduced
  - If stock is insufficient, an error is shown

### 2. **View Dispensing Logs**

1. Go to **Medicines** → **Medicine Dispensing Logs**
2. View statistics:
   - Total medicines dispensed
   - Dispensed this month
   - Most frequently dispensed medicine
3. Filter by:
   - Date range (start & end date)
   - Specific medicine
4. Export to CSV by clicking **"Export CSV"** button

---

## Request/Response Examples

### Create Medical Record with Medicines

**Request:**
```json
{
    "patient_name": "John Doe",
    "campus": "Urdaneta",
    "visit_date": "2024-01-15",
    "chief_complaint": "Fever and cough",
    "diagnosis": "Acute Respiratory Infection",
    "treatment": "Rest, fluids, monitor temperature",
    "physician": "Dr. Smith",
    "status": "Completed",
    "medications_prescribed": "Paracetamol 500mg daily",
    "clinical_notes": "Patient appears in good general condition",
    "medicines_dispensed": [
        {
            "medicine_id": 5,
            "quantity": 20,
            "dosage": "500mg",
            "instructions": "Take 1 tablet twice daily after meals"
        },
        {
            "medicine_id": 8,
            "quantity": 10,
            "dosage": "2 tablets",
            "instructions": "Take twice daily"
        }
    ]
}
```

### Get Available Medicines

**Response:**
```json
[
    {
        "id": 5,
        "name": "Paracetamol",
        "stock": 450,
        "low_stock_alert": 50,
        "category": "Pain Reliever",
        "unit": "tablets",
        "is_low_stock": false
    },
    {
        "id": 8,
        "name": "Cough Syrup",
        "stock": 15,
        "low_stock_alert": 20,
        "category": "Cough Medicine",
        "unit": "bottles",
        "is_low_stock": true
    }
]
```

### Get Dispensing Data

**Response:**
```json
[
    {
        "id": 1,
        "medical_record_id": 42,
        "patient_name": "John Doe",
        "medicine_name": "Paracetamol",
        "category": "Pain Reliever",
        "quantity": 20,
        "unit": "tablets",
        "dosage": "500mg",
        "instructions": "Take 1 tablet twice daily after meals",
        "dispensed_by": "Maria Santos",
        "dispensed_at": "Jan 15, 2024 10:30"
    }
]
```

---

## Key Features

### ✅ Stock Deduction
- Automatically deducts dispensed quantity from medicine inventory
- Prevents dispensing if insufficient stock
- Uses database transactions for atomicity

### ✅ Low Stock Warnings
- Shows warning if medicine is below low_stock_alert threshold
- Visual indicator in medicine selection dropdown
- Helps staff track when to reorder

### ✅ Audit Trail
- Records who dispensed what medicine
- Timestamp for every dispensing
- Linked to medical records for complete history

### ✅ Campus-Based Access
- Super Admin sees all dispensing logs
- Admins/Staff see only their campus
- Filters automatically applied by user role

### ✅ Reporting
- View total dispensed count
- Track monthly dispensing
- Top medicines report
- Export to CSV for analysis

---

## Database Schema

```sql
CREATE TABLE medicine_dispenses (
    id BIGINT PRIMARY KEY AUTO_INCREMENT,
    medical_record_id BIGINT NOT NULL,
    patient_id BIGINT NOT NULL,
    medicine_id BIGINT NOT NULL,
    quantity INT NOT NULL,
    dosage VARCHAR(255) NULL,
    instructions TEXT NULL,
    dispensed_by BIGINT NOT NULL,
    dispensed_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    created_at TIMESTAMP,
    updated_at TIMESTAMP,
    
    FOREIGN KEY (medical_record_id) REFERENCES medical_records(id) ON DELETE CASCADE,
    FOREIGN KEY (patient_id) REFERENCES patients(id) ON DELETE CASCADE,
    FOREIGN KEY (medicine_id) REFERENCES medicines(id) ON DELETE RESTRICT,
    FOREIGN KEY (dispensed_by) REFERENCES users(id) ON DELETE RESTRICT,
    
    INDEX (medical_record_id),
    INDEX (patient_id),
    INDEX (medicine_id),
    INDEX (dispensed_by),
    INDEX (dispensed_at)
);
```

---

## Security & Validation

1. **Authentication Required** - All routes protected by `auth` middleware
2. **Campus Filtering** - Users only see their campus data
3. **Stock Validation** - Prevents over-dispensing
4. **Database Transactions** - Ensures data consistency
5. **Request Validation** - Validates all medicine dispensing data
6. **Authorization** - Role-based access control via policies

---

## Testing the Feature

1. **Test Medicine Dispensing:**
   ```
   - Create medical record
   - Add multiple medicines
   - Verify stock decreases
   - Check dispensing log page
   ```

2. **Test Stock Deduction:**
   ```
   - Note current medicine stock
   - Dispense medicine in medical record
   - Go to medicines page
   - Confirm stock reduced by dispensed quantity
   ```

3. **Test Low Stock Alert:**
   ```
   - Set low_stock_alert to 100
   - Current stock to 90
   - Open Add Medical Record
   - Add that medicine - should show warning
   ```

4. **Test Campus Filtering:**
   ```
   - Login as staff from Campus A
   - Create record with medicine
   - Login as staff from Campus B
   - Should NOT see Campus A's dispensing
   ```

---

## Troubleshooting

**Issue:** Stock not decreasing
- **Solution:** Ensure medicines_dispensed array is properly formatted in request
- Check database transaction isn't rolled back due to errors
- Verify medicine_id exists and has sufficient stock

**Issue:** Low stock warning not showing
- **Solution:** Check if low_stock_alert value is set on medicine
- Ensure current stock is <= low_stock_alert

**Issue:** Can't find dispensing logs page
- **Solution:** Ensure you're logged in and routes are registered
- Run `php artisan route:list` to verify routes

**Issue:** Medical record created but medicines not dispensed
- **Solution:** Check medicines_dispensed array was included in request
- Verify medicine quantities are valid integers >= 1

---

## Future Enhancements

1. **Batch Dispensing** - Dispense same medicine multiple times
2. **Dispensing Returns** - Reverse dispensing for returned medicines
3. **Expiry Tracking** - Track medicine expiration dates
4. **Dispensing Receipt** - Print/email receipt for patient
5. **Advanced Reports** - Charts, trends, cost analysis
6. **Integration** - Sync with hospital pharmacy system

---

## Support & Questions

For issues or questions about the Medicine Dispensing feature:
1. Check the logs: `storage/logs/laravel.log`
2. Verify database migration ran: `php artisan migrate:status`
3. Test API endpoints manually using Postman
4. Review browser console for JavaScript errors

---

**Created:** June 3, 2026  
**Version:** 1.0  
**Status:** Production Ready
