# Campus Filtering System

## Overview

The campus filtering system automatically restricts data visibility based on the authenticated user's campus assignment. 

**Rules:**
- **Super Admin**: Can see all data from all campuses
- **Admin**: Can only see data from their assigned campus
- **Staff**: Can only see data from their assigned campus

## Models with Campus Filtering

The following models include campus filtering scopes:

1. **Patient** - Filters patients by campus
2. **MedicalRecord** - Filters medical records by campus  
3. **DentalRecord** - Filters dental records by campus
4. **Staff** - Filters staff members by campus

## Usage in Controllers

### Basic Usage - Apply Filter Automatically

```php
// In your controller
use App\Models\Patient;

class PatientController extends Controller
{
    public function index()
    {
        // Automatically filtered by authenticated user's campus
        $patients = Patient::forCampus()->get();
        
        return view('patients.index', compact('patients'));
    }
    
    public function show($id)
    {
        $patient = Patient::forCampus()->findOrFail($id);
        return view('patients.show', compact('patient'));
    }
}
```

### Chaining with Other Queries

```php
// Combine with other filters
$activePatients = Patient::forCampus()
    ->where('status', 'active')
    ->orderBy('name')
    ->paginate(15);

// Get medical records for a specific patient in the user's campus
$records = MedicalRecord::forCampus()
    ->where('patient_id', $patientId)
    ->latest('visit_date')
    ->get();

// Get staff members in user's campus
$staff = Staff::forCampus()
    ->where('status', 'active')
    ->orderBy('full_name')
    ->get();
```

## How It Works

### Example Scenarios

**Scenario 1: Urdaneta Admin Viewing Patients**
```php
$user = auth()->user(); // Campus: 'Urdaneta City Campus'
$patients = Patient::forCampus()->get();

// Result: Only patients with campus='Urdaneta City Campus' are returned
```

**Scenario 2: Super Admin Viewing Patients**
```php
$user = auth()->user(); // Role: 'super_admin', Campus: null
$patients = Patient::forCampus()->get();

// Result: All patients from all campuses are returned
```

**Scenario 3: Staff Member Viewing Medical Records**
```php
$user = auth()->user(); // Campus: 'Lingayen Campus'
$records = MedicalRecord::forCampus()->get();

// Result: Only medical records with campus='Lingayen Campus' are returned
```

## Scope Implementation

Each model includes a `scopeForCampus()` method:

```php
public function scopeForCampus($query)
{
    $user = auth()->user();

    // Super Admin can see all records
    if ($user && $user->isSuperAdmin()) {
        return $query;
    }

    // Admin and Staff can only see their campus records
    if ($user && $user->campus) {
        return $query->where('campus', $user->campus);
    }

    // Unauthenticated or no campus assigned - show nothing
    return $query->whereRaw('1 = 0');
}
```

## Important Notes

1. **Always Apply the Scope**: Remember to chain `.forCampus()` when querying these models in your controllers
2. **Relationships**: When loading related data, the scope only applies to direct queries
3. **Super Admin**: Super admins have `campus = null`, so they bypass the campus filter automatically
4. **Unauthenticated Users**: If no user is authenticated or they have no campus, no records are returned

## Example: Complete Patient Controller

```php
namespace App\Http\Controllers;

use App\Models\Patient;
use Illuminate\Http\Request;

class PatientController extends Controller
{
    public function index()
    {
        // Get only patients from authenticated user's campus
        $patients = Patient::forCampus()
            ->orderBy('name')
            ->paginate(20);
            
        return view('patients.index', compact('patients'));
    }

    public function show(Patient $patient)
    {
        // Verify the patient belongs to user's campus
        // (scopeForCampus already handles this)
        return view('patients.show', compact('patient'));
    }

    public function create()
    {
        return view('patients.create');
    }

    public function store(Request $request)
    {
        $validated = $request->validate([
            'name' => 'required|string|max:255',
            'email' => 'required|email',
            'dob' => 'required|date',
            'gender' => 'required|in:Male,Female',
            'phone' => 'required|string',
        ]);

        // Automatically assign current user's campus
        $validated['campus'] = auth()->user()->campus;

        Patient::create($validated);

        return redirect()->route('patients.index')
            ->with('success', 'Patient created successfully');
    }
}
```

## Testing Campus Filtering

Test that campus filtering works correctly:

```php
// Unit test example
public function test_admin_can_only_see_their_campus_patients()
{
    $urdanetaAdmin = User::factory()->create([
        'role' => 'admin',
        'campus' => 'Urdaneta City Campus'
    ]);
    
    $lingayenPatient = Patient::factory()->create([
        'campus' => 'Lingayen Campus'
    ]);
    
    $this->actingAs($urdanetaAdmin);
    
    $patients = Patient::forCampus()->get();
    
    $this->assertNotContains($lingayenPatient->id, $patients->pluck('id'));
}
```

## Bypassing Campus Filtering (Not Recommended)

If you absolutely need to bypass campus filtering (only for super admin operations):

```php
// Get ALL records regardless of campus (use with caution)
$allPatients = Patient::all(); // Without forCampus()
```

**Note**: This bypasses the security check. Use only for admin-only operations.
