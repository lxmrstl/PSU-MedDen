# Campus Filtering Implementation Examples

## Quick Usage Pattern

**In all your controllers that query campus-restricted models, add `.forCampus()` to the query:**

```php
Patient::forCampus()->get()
MedicalRecord::forCampus()->where('status', 'Completed')->get()
DentalRecord::forCampus()->orderBy('visit_date')->paginate(15)
Staff::forCampus()->where('status', 'active')->get()
```

## Example: Patient Controller with Campus Filtering

```php
<?php

namespace App\Http\Controllers;

use App\Models\Patient;
use Illuminate\Http\Request;

class PatientController extends Controller
{
    /**
     * Display list of patients for authenticated user's campus
     * - Super Admin: sees all patients
     * - Admin/Staff: sees only their campus patients
     */
    public function index()
    {
        $patients = Patient::forCampus()
            ->orderBy('name')
            ->paginate(20);

        return view('patients.index', compact('patients'));
    }

    /**
     * Show a specific patient (campus-filtered)
     */
    public function show($id)
    {
        $patient = Patient::forCampus()->findOrFail($id);
        $medicalRecords = $patient->medicalRecords()
            ->forCampus()
            ->latest('visit_date')
            ->get();
        
        $dentalRecords = $patient->dentalRecords()
            ->forCampus()
            ->latest('visit_date')
            ->get();

        return view('patients.show', compact('patient', 'medicalRecords', 'dentalRecords'));
    }

    /**
     * Create new patient (auto-assigned to user's campus)
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email|unique:patients',
            'dob' => 'required|date',
            'gender' => 'required|in:Male,Female',
            'phone' => 'required|string|max:20',
        ]);

        // Automatically assign to authenticated user's campus
        $validated['campus'] = auth()->user()->campus;

        Patient::create($validated);

        return redirect()->route('patients.index')
            ->with('success', 'Patient created successfully');
    }
}
```

## Example: Medical Records Controller with Campus Filtering

```php
<?php

namespace App\Http\Controllers;

use App\Models\MedicalRecord;
use App\Models\Patient;
use Illuminate\Http\Request;

class MedicalRecordController extends Controller
{
    /**
     * Show all medical records for user's campus
     */
    public function index()
    {
        $records = MedicalRecord::forCampus()
            ->with('patient')
            ->orderBy('visit_date', 'desc')
            ->paginate(20);

        return view('medical-records.index', compact('records'));
    }

    /**
     * Show medical record details
     */
    public function show($id)
    {
        $record = MedicalRecord::forCampus()->with('patient')->findOrFail($id);
        return view('medical-records.show', compact('record'));
    }

    /**
     * Create new medical record
     */
    public function store(Request $request)
    {
        $validated = $request->validate([
            'patient_id' => 'required|exists:patients,id',
            'diagnosis' => 'required|string|max:255',
            'treatment' => 'required|string',
            'medications_prescribed' => 'nullable|string',
            'next_followup_date' => 'nullable|date',
        ]);

        // Verify patient belongs to user's campus
        $patient = Patient::forCampus()->findOrFail($validated['patient_id']);

        $record = new MedicalRecord($validated);
        $record->campus = auth()->user()->campus;
        $record->visit_date = now();
        $record->created_by = auth()->id();
        $record->save();

        return redirect()->route('patients.show', $patient->id)
            ->with('success', 'Medical record created successfully');
    }

    /**
     * Update medical record (campus-admin only)
     */
    public function update(Request $request, $id)
    {
        $record = MedicalRecord::forCampus()->findOrFail($id);

        // Only admin-level users can update
        if (!auth()->user()->isAdminLevel()) {
            abort(403, 'Unauthorized');
        }

        $validated = $request->validate([
            'diagnosis' => 'required|string|max:255',
            'treatment' => 'required|string',
            'medications_prescribed' => 'nullable|string',
            'next_followup_date' => 'nullable|date',
        ]);

        $record->update($validated);

        return redirect()->back()
            ->with('success', 'Medical record updated successfully');
    }

    /**
     * Delete medical record (campus-admin only)
     */
    public function destroy($id)
    {
        $record = MedicalRecord::forCampus()->findOrFail($id);

        if (!auth()->user()->isAdminLevel()) {
            abort(403, 'Unauthorized');
        }

        $patientId = $record->patient_id;
        $record->delete();

        return redirect()->route('patients.show', $patientId)
            ->with('success', 'Medical record deleted successfully');
    }
}
```

## Example: Staff Controller with Campus Filtering

```php
<?php

namespace App\Http\Controllers;

use App\Models\Staff;
use Illuminate\Http\Request;

class StaffController extends Controller
{
    /**
     * List all staff for user's campus
     */
    public function index()
    {
        $staff = Staff::forCampus()
            ->where('status', 'active')
            ->orderBy('full_name')
            ->paginate(20);

        return view('staff.index', compact('staff'));
    }

    /**
     * Show staff member details
     */
    public function show($id)
    {
        $staff = Staff::forCampus()->findOrFail($id);
        return view('staff.show', compact('staff'));
    }

    /**
     * Create new staff member (admin-only)
     */
    public function store(Request $request)
    {
        // Verify user is admin or super-admin
        if (!auth()->user()->isAdminLevel()) {
            abort(403, 'Unauthorized');
        }

        $validated = $request->validate([
            'full_name' => 'required|string|max:255',
            'position' => 'required|string|max:255',
            'contact_number' => 'required|string|max:20',
            'specialization' => 'nullable|string|max:255',
            'hire_date' => 'required|date',
        ]);

        // Assign to authenticated user's campus
        $validated['campus'] = auth()->user()->campus;

        Staff::create($validated);

        return redirect()->route('staff.index')
            ->with('success', 'Staff member created successfully');
    }

    /**
     * Update staff member (admin-only)
     */
    public function update(Request $request, $id)
    {
        $staff = Staff::forCampus()->findOrFail($id);

        if (!auth()->user()->isAdminLevel()) {
            abort(403, 'Unauthorized');
        }

        $validated = $request->validate([
            'full_name' => 'required|string|max:255',
            'position' => 'required|string|max:255',
            'contact_number' => 'required|string|max:20',
            'specialization' => 'nullable|string|max:255',
        ]);

        $staff->update($validated);

        return redirect()->back()
            ->with('success', 'Staff member updated successfully');
    }

    /**
     * Delete staff member (admin-only)
     */
    public function destroy($id)
    {
        $staff = Staff::forCampus()->findOrFail($id);

        if (!auth()->user()->isAdminLevel()) {
            abort(403, 'Unauthorized');
        }

        $staff->delete();

        return redirect()->route('staff.index')
            ->with('success', 'Staff member deleted successfully');
    }
}
```

## Key Points

1. **Always use `.forCampus()`** when querying Patient, MedicalRecord, DentalRecord, or Staff models
2. **Super Admin** automatically sees all data (no campus filter applied)
3. **Admin/Staff** automatically see only their campus data
4. **New records** should be assigned the current user's campus
5. **Verification** happens automatically - if a user tries to access data from another campus, `findOrFail()` will throw a 404 error

## Testing Campus Filtering in Controllers

```php
// Test that admin only sees their campus data
public function test_urdaneta_admin_only_sees_urdaneta_patients()
{
    $urdanetaAdmin = User::factory()->create([
        'role' => 'admin',
        'campus' => 'Urdaneta City Campus'
    ]);

    $urdanetaPatient = Patient::factory()->create([
        'campus' => 'Urdaneta City Campus'
    ]);

    $lingayenPatient = Patient::factory()->create([
        'campus' => 'Lingayen Campus'
    ]);

    $this->actingAs($urdanetaAdmin);
    $response = $this->get('/patients');

    $this->assertContains($urdanetaPatient->name, $response->getContent());
    $this->assertNotContains($lingayenPatient->name, $response->getContent());
}

// Test that super admin sees all patients
public function test_super_admin_sees_all_patients()
{
    $superAdmin = User::factory()->create([
        'role' => 'super_admin',
        'campus' => null
    ]);

    $urdanetaPatient = Patient::factory()->create(['campus' => 'Urdaneta City Campus']);
    $lingayenPatient = Patient::factory()->create(['campus' => 'Lingayen Campus']);

    $this->actingAs($superAdmin);
    $response = $this->get('/patients');

    $this->assertContains($urdanetaPatient->name, $response->getContent());
    $this->assertContains($lingayenPatient->name, $response->getContent());
}
```
