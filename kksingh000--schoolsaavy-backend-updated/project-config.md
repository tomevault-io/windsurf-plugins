---
trigger: always_on
description: SchoolSavvy is a **production-ready SaaS platform** built with Laravel 12 & PHP 8.3 for comprehensive school management. This is a **multi-tenant modular system** where schools purchase and activate specific modules. The codebase is **85% complete** with a sophisticated architecture optimized for performance and scalability.
---

# SchoolSavvy SaaS - AI Coding Agent Instructions

## 🎯 Project Overview

SchoolSavvy is a **production-ready SaaS platform** built with Laravel 12 & PHP 8.3 for comprehensive school management. This is a **multi-tenant modular system** where schools purchase and activate specific modules. The codebase is **85% complete** with a sophisticated architecture optimized for performance and scalability.

### 🏗️ Core Architecture

- **Framework**: Laravel 12 with PHP 8.3, MySQL 8.0, Redis caching
- **Authentication**: Laravel Sanctum with multi-role access (Super Admin, School Admin, Teacher, Parent, Student)
- **Deployment**: Docker with RoadRunner (Octane) for high performance
- **API Design**: 100+ REST endpoints with comprehensive validation
- **Database**: Multi-tenant with school_id isolation, 50+ optimized tables

---

## 🧩 Code Architecture Patterns

### 🎨 MVC + Service Layer Pattern

**CRITICAL**: Always follow the established Service-Controller-Model pattern:

```php
// Controllers handle HTTP logic only
class StudentController extends BaseController {
    public function __construct(private StudentService $studentService) {}
    
    public function store(StudentRequest $request) {
        $student = $this->studentService->create($request->validated());
        return $this->successResponse($student, 'Student created successfully');
    }
}

// Services contain ALL business logic
class StudentService extends BaseService {
    protected function initializeModel() {
        $this->model = Student::class;
    }
    
    public function create(array $data) {
        // Business logic, validation, relationships here
        $data['school_id'] = auth()->user()->getSchool()->id;
        return $this->model::create($data);
    }
}
```

### 🔐 Multi-Tenant Security Pattern

**MANDATORY**: Every database query MUST include school isolation:

```php
// ✅ CORRECT - Always filter by school_id
Student::where('school_id', auth()->user()->getSchool()->id)->get();

// ❌ WRONG - Never query without school isolation
Student::all(); // This will leak data between schools
```
**MANDATORY**: Always use Academic Year for all queries and data manipulations.



### 🛡️ BaseController & BaseService Usage

**Always extend** base classes for consistent functionality:

```php
// Controllers - use BaseController for common responses
class MyController extends BaseController {
    // Inherit: successResponse(), errorResponse(), checkModuleAccess()
}

// Services - use BaseService for CRUD operations
class MyService extends BaseService {
    // Inherit: getAll(), find(), create(), update(), delete()
}
```

---

## 📡 API Conventions

### 🎯 Response Format Standard

**ALL API responses** must follow this format:

```php
// Success Response
return $this->successResponse($data, 'Operation successful', 200);
// Returns: {"status": "success", "message": "...", "data": {...}}

// Error Response  
return $this->errorResponse('Error message', $validationErrors, 400);
// Returns: {"status": "error", "message": "...", "errors": {...}}
```

### 🔄 API Route Structure

Follow the established route organization in `routes/api.php`:

```php
// Group routes by feature with middleware
Route::middleware(['auth:sanctum', 'inject.school'])->group(function () {
    // Module-specific routes
    Route::prefix('students')->group(function () {
        Route::get('/', [StudentController::class, 'index']);
        Route::post('/', [StudentController::class, 'store']);
        // Use apiResource for standard CRUD
        Route::apiResource('students', StudentController::class);
    });
});
```

### 📝 Request Validation Pattern

**Always create** Form Request classes for validation:

```php
class StudentRequest extends FormRequest {
    public function rules() {
        return [
            'first_name' => 'required|string|max:255',
            'email' => 'required|email|unique:students,email',
            'class_id' => 'required|exists:classes,id'
        ];
    }
    
    public function messages() {
        return [
            'first_name.required' => 'Student first name is required',
        ];
    }
}
```

---

## 🗄️ Database & Model Patterns

### 🔗 Model Relationships

**Standard relationship patterns** used throughout:

```php
class Student extends Model {
    use HasFactory, SoftDeletes;
    
    protected $fillable = ['school_id', 'first_name', 'last_name', 'email'];
    
    // Always include school relationship
    public function school() {
        return $this->belongsTo(School::class);
    }
    
    // Many-to-many with pivot data
    public function classes() {
        return $this->belongsToMany(ClassRoom::class, 'class_student')
            ->withPivot(['enrollment_date', 'is_active'])
            ->withTimestamps();
    }
    
    // Use consistent scope naming
    public function scopeActive($query) {
        return $query->where('is_active', true);
    }
    
    public function scopeForSchool($query, $schoolId) {
        return $query->where('school_id', $schoolId);
    }

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kksingh000/SchoolSaavy-Backend_updated](https://github.com/kksingh000/SchoolSaavy-Backend_updated) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
