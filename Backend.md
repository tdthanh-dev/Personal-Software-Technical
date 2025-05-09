# Hướng dẫn triển khai Backend cho AIDIMS

## 1. Tổng quan công nghệ

### 1.1. Công nghệ chính
- **Ngôn ngữ**: C#
- **Framework**: .NET Framework
- **ORM**: Entity Framework 6
- **Kiến trúc**: ASP.NET MVC
- **Cơ sở dữ liệu**: PostgreSQL
- **API Documentation**: Swagger/OpenAPI

### 1.2. Các thư viện và package chính
- **Swashbuckle**: Tích hợp Swagger cho ASP.NET
- **Npgsql**: Provider kết nối PostgreSQL cho .NET
- **Npgsql.EntityFramework**: Entity Framework provider cho PostgreSQL
- **AutoMapper**: Mapping giữa các object
- **Newtonsoft.Json**: Xử lý JSON
- **log4net**: Logging
- **fo-dicom**: Thư viện xử lý file DICOM

## 2. Cấu trúc dự án

### 2.1. Cấu trúc Solution

```
AIDIMS.Backend/
├── AIDIMS.API                  # Project ASP.NET MVC/Web API
├── AIDIMS.Core                 # Business logic core
├── AIDIMS.Data                 # Data access layer (EF, Repository)
├── AIDIMS.Domain               # Domain models, interfaces
├── AIDIMS.Services             # Service layer
├── AIDIMS.DICOM                # DICOM processing utilities
├── AIDIMS.Common               # Shared utilities, extensions
└── AIDIMS.Tests                # Unit tests, integration tests
```

### 2.2. Chi tiết từng module

#### AIDIMS.Domain
- **Entities**: Các entity chính (Patient, DicomFile, User, Role...)
- **DTOs**: Data Transfer Objects
- **Interfaces**: Interface cho repository và service
- **Enums**: Enum và constant

#### AIDIMS.Data
- **ApplicationDbContext**: EF DbContext
- **Repositories**: Repository pattern implementation
- **Migrations**: EF Code First Migrations
- **Configurations**: Entity configurations

#### AIDIMS.Core
- **Managers**: Business logic managers
- **Validators**: Validation logic
- **Processors**: Business processors

#### AIDIMS.Services
- **PatientService**: Quản lý thông tin bệnh nhân
- **DicomService**: Quản lý file DICOM
- **UserService**: Quản lý người dùng và phân quyền
- **NotificationService**: Quản lý thông báo

#### AIDIMS.DICOM
- **DicomProcessor**: Xử lý và đọc file DICOM
- **DicomConverter**: Chuyển đổi định dạng
- **DicomMetadataExtractor**: Trích xuất metadata

#### AIDIMS.API
- **Controllers**: API Controllers
- **Filters**: Action Filters
- **App_Start**: Cấu hình ứng dụng
- **Global.asax**: Application startup
- **SwaggerConfig**: Cấu hình Swagger

## 3. Thiết kế cơ sở dữ liệu

### 3.1. Các bảng chính

#### Patients
```sql
CREATE TABLE patients (
    id SERIAL PRIMARY KEY,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    date_of_birth DATE NOT NULL,
    gender VARCHAR(10),
    address TEXT,
    phone VARCHAR(20),
    email VARCHAR(100),
    medical_record_number VARCHAR(50) UNIQUE NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### DicomFiles
```sql
CREATE TABLE dicom_files (
    id SERIAL PRIMARY KEY,
    patient_id INTEGER REFERENCES patients(id),
    study_instance_uid VARCHAR(64) NOT NULL,
    series_instance_uid VARCHAR(64) NOT NULL,
    sop_instance_uid VARCHAR(64) NOT NULL,
    modality VARCHAR(10) NOT NULL,
    study_date DATE,
    acquisition_date DATE,
    storage_path TEXT NOT NULL,
    file_size BIGINT NOT NULL,
    physician_id INTEGER REFERENCES users(id),
    thumbnail_path TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    CONSTRAINT uk_sop_instance UNIQUE(sop_instance_uid)
);
```

#### Users
```sql
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    username VARCHAR(50) UNIQUE NOT NULL,
    password_hash TEXT NOT NULL,
    email VARCHAR(100) UNIQUE NOT NULL,
    first_name VARCHAR(100) NOT NULL,
    last_name VARCHAR(100) NOT NULL,
    role_id INTEGER REFERENCES roles(id),
    is_active BOOLEAN DEFAULT TRUE,
    last_login_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### Roles
```sql
CREATE TABLE roles (
    id SERIAL PRIMARY KEY,
    name VARCHAR(50) UNIQUE NOT NULL,
    description TEXT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### DicomAnnotations
```sql
CREATE TABLE dicom_annotations (
    id SERIAL PRIMARY KEY,
    dicom_file_id INTEGER REFERENCES dicom_files(id),
    user_id INTEGER REFERENCES users(id),
    annotation_data JSONB NOT NULL,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
    updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

#### AIAnalysisResults
```sql
CREATE TABLE ai_analysis_results (
    id SERIAL PRIMARY KEY,
    dicom_file_id INTEGER REFERENCES dicom_files(id),
    model_id INTEGER REFERENCES ai_models(id),
    result_data JSONB NOT NULL,
    confidence_score FLOAT,
    processing_time FLOAT,
    created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

### 3.2. Entity Framework Code First

Mẫu code cho Entity và DbContext:

**Patient Entity**:
```csharp
public class Patient
{
    public int Id { get; set; }
    public string FirstName { get; set; }
    public string LastName { get; set; }
    public DateTime DateOfBirth { get; set; }
    public string Gender { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }
    public string Email { get; set; }
    public string MedicalRecordNumber { get; set; }
    public DateTime CreatedAt { get; set; }
    public DateTime UpdatedAt { get; set; }

    // Navigation properties
    public virtual ICollection<DicomFile> DicomFiles { get; set; }
}
```

**ApplicationDbContext**:
```csharp
public class ApplicationDbContext : DbContext
{
    public ApplicationDbContext() : base("name=AIDIMSConnection")
    {
    }

    public DbSet<Patient> Patients { get; set; }
    public DbSet<DicomFile> DicomFiles { get; set; }
    public DbSet<User> Users { get; set; }
    public DbSet<Role> Roles { get; set; }
    public DbSet<DicomAnnotation> DicomAnnotations { get; set; }
    public DbSet<AIAnalysisResult> AIAnalysisResults { get; set; }

    protected override void OnModelCreating(DbModelBuilder modelBuilder)
    {
        // Configuration
        modelBuilder.Configurations.Add(new PatientConfiguration());
        modelBuilder.Configurations.Add(new DicomFileConfiguration());
        // Cấu hình các entity khác...

        base.OnModelCreating(modelBuilder);
    }
}
```

## 4. Triển khai API

### 4.1. API Controllers

#### PatientController
```csharp
[RoutePrefix("api/patients")]
public class PatientController : ApiController
{
    private readonly IPatientService _patientService;

    public PatientController(IPatientService patientService)
    {
        _patientService = patientService;
    }

    [HttpGet]
    [Route("")]
    public IHttpActionResult GetAll([FromUri]PatientFilterDto filter)
    {
        var patients = _patientService.GetPatients(filter);
        return Ok(patients);
    }

    [HttpGet]
    [Route("{id:int}")]
    public IHttpActionResult GetById(int id)
    {
        var patient = _patientService.GetPatientById(id);
        if (patient == null)
            return NotFound();

        return Ok(patient);
    }

    [HttpPost]
    [Route("")]
    public IHttpActionResult Create(PatientCreateDto patientDto)
    {
        if (!ModelState.IsValid)
            return BadRequest(ModelState);

        var createdPatient = _patientService.CreatePatient(patientDto);
        return Created($"api/patients/{createdPatient.Id}", createdPatient);
    }

    // PUT, DELETE và các API khác...
}
```

#### DicomController
```csharp
[RoutePrefix("api/dicom")]
public class DicomController : ApiController
{
    private readonly IDicomService _dicomService;

    public DicomController(IDicomService dicomService)
    {
        _dicomService = dicomService;
    }

    [HttpPost]
    [Route("upload")]
    public async Task<IHttpActionResult> UploadDicomFile()
    {
        if (!Request.Content.IsMimeMultipartContent())
            return BadRequest("Unsupported media type");

        var provider = new MultipartFormDataStreamProvider(Path.GetTempPath());
        await Request.Content.ReadAsMultipartAsync(provider);

        var patientId = provider.FormData["patientId"];
        var files = provider.FileData;

        var results = await _dicomService.ProcessDicomFilesAsync(files, patientId);
        return Ok(results);
    }

    [HttpGet]
    [Route("patient/{patientId:int}")]
    public IHttpActionResult GetDicomFilesByPatient(int patientId)
    {
        var files = _dicomService.GetDicomFilesByPatient(patientId);
        return Ok(files);
    }

    // API khác cho việc quản lý DICOM...
}
```

### 4.2. Tích hợp Swagger

Cấu hình Swagger trong **SwaggerConfig.cs**:

```csharp
public class SwaggerConfig
{
    public static void Register()
    {
        var thisAssembly = typeof(SwaggerConfig).Assembly;

        GlobalConfiguration.Configuration
            .EnableSwagger(c =>
            {
                c.SingleApiVersion("v1", "AIDIMS API")
                    .Description("API for DICOM Image Management System")
                    .Contact(cc => cc
                        .Name("AIDIMS Development Team")
                        .Email("team@aidims.com"));

                c.IncludeXmlComments(GetXmlCommentsPath());
                c.DescribeAllEnumsAsStrings();
                c.OperationFilter<AddAuthorizationHeaderParameterOperationFilter>();
            })
            .EnableSwaggerUi(c =>
            {
                c.DocumentTitle("AIDIMS API Documentation");
                c.EnableValidator();
            });
    }

    private static string GetXmlCommentsPath()
    {
        return Path.Combine(AppDomain.CurrentDomain.BaseDirectory, "AIDIMS.API.xml");
    }
}
```

## 5. Bước triển khai

### 5.1. Thiết lập dự án ban đầu

1. **Tạo solution và các project**:
   ```
   md AIDIMS.Backend
   cd AIDIMS.Backend
   dotnet new sln -n AIDIMS
   dotnet new classlib -n AIDIMS.Domain
   dotnet new classlib -n AIDIMS.Data
   dotnet new classlib -n AIDIMS.Core
   dotnet new classlib -n AIDIMS.Services
   dotnet new classlib -n AIDIMS.DICOM
   dotnet new classlib -n AIDIMS.Common
   dotnet new webapi -n AIDIMS.API
   dotnet new xunit -n AIDIMS.Tests
   ```

2. **Thêm các project vào solution**:
   ```
   dotnet sln add AIDIMS.Domain/AIDIMS.Domain.csproj
   dotnet sln add AIDIMS.Data/AIDIMS.Data.csproj
   dotnet sln add AIDIMS.Core/AIDIMS.Core.csproj
   dotnet sln add AIDIMS.Services/AIDIMS.Services.csproj
   dotnet sln add AIDIMS.DICOM/AIDIMS.DICOM.csproj
   dotnet sln add AIDIMS.Common/AIDIMS.Common.csproj
   dotnet sln add AIDIMS.API/AIDIMS.API.csproj
   dotnet sln add AIDIMS.Tests/AIDIMS.Tests.csproj
   ```

3. **Cài đặt các package cần thiết**:
   ```
   dotnet add AIDIMS.Data package Npgsql.EntityFramework
   dotnet add AIDIMS.API package Swashbuckle.AspNetCore
   dotnet add AIDIMS.Services package AutoMapper
   dotnet add AIDIMS.DICOM package fo-dicom
   ```

### 5.2. Tạo Database và Migration

1. **Cấu hình kết nối**:
   - Thêm connection string vào Web.config/App.config
   ```xml
   <connectionStrings>
     <add name="AIDIMSConnection" connectionString="Server=localhost;Port=5432;Database=aidims;User Id=postgres;Password=yourpassword;" providerName="Npgsql" />
   </connectionStrings>
   ```

2. **Tạo Initial Migration**:
   ```
   Enable-Migrations -ContextTypeName ApplicationDbContext -MigrationsDirectory Migrations
   Add-Migration InitialCreate
   Update-Database
   ```

### 5.3. Triển khai API Core

1. **Xây dựng Repositories**:
   - Tạo IRepository<T> interface
   - Tạo Repository<T> base class
   - Implement các repository cụ thể

2. **Xây dựng Services**:
   - Tạo IService interfaces
   - Implement các service

3. **Xây dựng Controllers**:
   - Tạo các controller API
   - Cấu hình routing

4. **Cấu hình Dependency Injection**:
   - Sử dụng Unity, Ninject hoặc Autofac

### 5.4. Xây dựng Module DICOM

1. **Implement DicomProcessor**:
   - Sử dụng fo-dicom để đọc file DICOM
   - Extract metadata
   - Lưu trữ file và metadata

2. **Implement DicomViewer API**:
   - API để lấy hình ảnh DICOM
   - API để chuyển đổi định dạng (DICOM sang PNG/JPEG)

### 5.5. Cấu hình Swagger

1. **Thêm XML Documentation**:
   - Bật XML documentation trong project properties
   - Thêm XML comments cho API

2. **Cấu hình Swagger UI**:
   - Tùy chỉnh giao diện
   - Thêm OAuth2 nếu cần

## 6. Mục tiêu ưu tiên và lộ trình

### 6.1. Sprint 1: Cấu trúc cơ bản và Database (2 tuần)
- Thiết lập cấu trúc dự án
- Thiết kế và tạo database schema
- Xây dựng Entity Framework models
- Tạo Repository pattern

### 6.2. Sprint 2: DICOM Processing và API cơ bản (2 tuần)
- Tích hợp fo-dicom
- Xử lý và lưu trữ file DICOM
- Xây dựng API cơ bản (CRUD Patients)
- Tích hợp Swagger

### 6.3. Sprint 3: API đầy đủ (2 tuần)
- Hoàn thiện tất cả các API
- Tạo API Documentation
- Thêm authentication và authorization
- Thiết lập CI/CD pipeline

## 7. Lưu ý và best practices

### 7.1. Security
- Sử dụng HTTPS
- Implement JWT authentication
- Sử dụng Entity Framework để chống SQL Injection
- Validate input

### 7.2. Performance
- Implement caching (Redis)
- Paging cho large result sets
- Async/await cho IO operations
- Tối ưu Entity Framework queries

### 7.3. Testing
- Unit tests cho business logic
- Integration tests cho repositories
- End-to-end tests cho API

### 7.4. CI/CD
- Automated build và test
- Code quality checks
- Automated deployment

## 8. Tài nguyên và references

### 8.1. Thư viện và tools
- Entity Framework: https://docs.microsoft.com/en-us/ef/
- ASP.NET MVC: https://docs.microsoft.com/en-us/aspnet/mvc/
- Swagger: https://swagger.io/
- fo-dicom: https://github.com/fo-dicom/fo-dicom

### 8.2. Tài liệu
- PostgreSQL Documentation: https://www.postgresql.org/docs/
- DICOM Standard: https://www.dicomstandard.org/

## 9. Cấu trúc cây thư mục chi tiết

Dưới đây là cấu trúc cây thư mục chi tiết của dự án AIDIMS:

```
AIDIMS.Backend/
│
├── AIDIMS.sln                                # Solution file
│
├── AIDIMS.API/                               # API Layer
│   ├── App_Start/                            # Startup configuration
│   │   ├── AutofacConfig.cs                  # Dependency Injection config
│   │   ├── RouteConfig.cs                    # Route config
│   │   ├── WebApiConfig.cs                   # Web API config
│   │   └── SwaggerConfig.cs                  # Swagger configuration
│   │
│   ├── Controllers/                          # API Controllers
│   │   ├── AuthController.cs                 # Authentication controller
│   │   ├── PatientController.cs              # Patient management
│   │   ├── DicomController.cs                # DICOM file management
│   │   ├── UserController.cs                 # User management
│   │   └── NotificationController.cs         # Notification management
│   │
│   ├── Filters/                              # API Filters
│   │   ├── AuthenticationFilter.cs           # JWT Authentication
│   │   ├── ExceptionFilter.cs                # Global exception handling
│   │   └── LoggingFilter.cs                  # Request/response logging
│   │
│   ├── Models/                               # View Models
│   │   ├── AuthModels.cs                     # Auth models (login, register)
│   │   └── ErrorModels.cs                    # Error response models
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── Global.asax                           # Application entry point
│   ├── Global.asax.cs                        # Application startup logic
│   ├── Web.config                            # Configuration file
│   ├── Web.Debug.config                      # Debug configuration
│   ├── Web.Release.config                    # Release configuration
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.API.csproj                     # Project file
│
├── AIDIMS.Domain/                            # Domain Layer
│   ├── Entities/                             # Domain Entities
│   │   ├── Patient.cs                        # Patient entity
│   │   ├── DicomFile.cs                      # DICOM file entity
│   │   ├── User.cs                           # User entity
│   │   ├── Role.cs                           # Role entity
│   │   ├── DicomAnnotation.cs                # Annotation entity
│   │   ├── AIModel.cs                        # AI model entity
│   │   └── AIAnalysisResult.cs               # Analysis result entity
│   │
│   ├── DTOs/                                 # Data Transfer Objects
│   │   ├── PatientDto.cs                     # Patient DTOs
│   │   ├── DicomFileDto.cs                   # DICOM file DTOs
│   │   ├── UserDto.cs                        # User DTOs
│   │   └── AnalysisResultDto.cs              # Analysis result DTOs
│   │
│   ├── Interfaces/                           # Domain Interfaces
│   │   ├── Repositories/                     # Repository interfaces
│   │   │   ├── IRepository.cs                # Generic repository interface
│   │   │   ├── IPatientRepository.cs         # Patient repository interface
│   │   │   └── IDicomFileRepository.cs       # DICOM repository interface
│   │   │
│   │   └── Services/                         # Service interfaces
│   │       ├── IPatientService.cs            # Patient service interface
│   │       ├── IDicomService.cs              # DICOM service interface
│   │       └── IUserService.cs               # User service interface
│   │
│   ├── Enums/                                # Enumerations
│   │   ├── UserRoles.cs                      # User roles enum
│   │   ├── DicomModality.cs                  # DICOM modality types
│   │   └── NotificationType.cs               # Notification types
│   │
│   ├── Constants/                            # Constants
│   │   └── SystemConstants.cs                # System-wide constants
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.Domain.csproj                  # Project file
│
├── AIDIMS.Data/                              # Data Access Layer
│   ├── Context/                              # EF Context
│   │   ├── ApplicationDbContext.cs           # Main DB context
│   │   └── DesignTimeDbContextFactory.cs     # For EF migrations
│   │
│   ├── Repositories/                         # Repository implementations
│   │   ├── Repository.cs                     # Generic repository
│   │   ├── PatientRepository.cs              # Patient repository
│   │   ├── DicomFileRepository.cs            # DICOM file repository
│   │   ├── UserRepository.cs                 # User repository
│   │   └── AIAnalysisResultRepository.cs     # AI results repository
│   │
│   ├── Configurations/                       # EF Configurations
│   │   ├── PatientConfiguration.cs           # Patient config
│   │   ├── DicomFileConfiguration.cs         # DICOM file config
│   │   ├── UserConfiguration.cs              # User config
│   │   └── RoleConfiguration.cs              # Role config
│   │
│   ├── Migrations/                           # EF Migrations
│   │   ├── Configuration.cs                  # Migration configuration
│   │   └── {timestamp}_InitialCreate.cs      # Initial migration
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.Data.csproj                    # Project file
│
├── AIDIMS.Core/                              # Business Logic Layer
│   ├── Managers/                             # Business logic
│   │   ├── PatientManager.cs                 # Patient business logic
│   │   ├── DicomManager.cs                   # DICOM business logic
│   │   └── UserManager.cs                    # User business logic
│   │
│   ├── Validators/                           # Validation logic
│   │   ├── PatientValidator.cs               # Patient validation
│   │   ├── DicomFileValidator.cs             # DICOM validation
│   │   └── UserValidator.cs                  # User validation
│   │
│   ├── Processors/                           # Domain processors
│   │   └── NotificationProcessor.cs          # Notification processing
│   │
│   ├── Security/                             # Security components
│   │   ├── PasswordHasher.cs                 # Password hashing
│   │   └── JwtTokenGenerator.cs              # JWT token generation
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.Core.csproj                    # Project file
│
├── AIDIMS.Services/                          # Service Layer
│   ├── Services/                             # Service implementations
│   │   ├── PatientService.cs                 # Patient service
│   │   ├── DicomService.cs                   # DICOM service
│   │   ├── UserService.cs                    # User service
│   │   └── NotificationService.cs            # Notification service
│   │
│   ├── Profiles/                             # AutoMapper profiles
│   │   ├── PatientProfile.cs                 # Patient mapping profile
│   │   ├── DicomFileProfile.cs               # DICOM mapping profile
│   │   └── UserProfile.cs                    # User mapping profile
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.Services.csproj                # Project file
│
├── AIDIMS.DICOM/                             # DICOM Processing Layer
│   ├── Processing/                           # DICOM processing
│   │   ├── DicomProcessor.cs                 # Main DICOM processor
│   │   ├── DicomConverter.cs                 # Format converter
│   │   └── DicomValidator.cs                 # DICOM validation
│   │
│   ├── Metadata/                             # Metadata extraction
│   │   ├── DicomMetadataExtractor.cs         # Extract metadata
│   │   └── DicomTagMapping.cs                # DICOM tag mapping
│   │
│   ├── Storage/                              # Storage management
│   │   ├── DicomStorageManager.cs            # Storage management
│   │   └── ThumbnailGenerator.cs             # Create thumbnails
│   │
│   ├── Viewers/                              # Viewing utilities
│   │   └── DicomImageRenderer.cs             # Render DICOM for web
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.DICOM.csproj                   # Project file
│
├── AIDIMS.Common/                            # Common Utilities
│   ├── Extensions/                           # Extension methods
│   │   ├── StringExtensions.cs               # String extensions
│   │   ├── DateTimeExtensions.cs             # DateTime extensions
│   │   └── EnumerableExtensions.cs           # IEnumerable extensions
│   │
│   ├── Helpers/                              # Helper classes
│   │   ├── FileHelper.cs                     # File operations
│   │   ├── JsonHelper.cs                     # JSON serialization
│   │   └── LogHelper.cs                      # Logging helper
│   │
│   ├── Logging/                              # Logging
│   │   └── Logger.cs                         # Logger implementation
│   │
│   ├── Utils/                                # Utilities
│   │   ├── Guard.cs                          # Parameter validation
│   │   └── AsyncHelper.cs                    # Async utilities
│   │
│   ├── Properties/                           # Project properties
│   │   └── AssemblyInfo.cs
│   │
│   ├── packages.config                       # NuGet packages
│   └── AIDIMS.Common.csproj                  # Project file
│
└── AIDIMS.Tests/                             # Test Projects
    ├── Unit/                                 # Unit Tests
    │   ├── Core/                             # Core tests
    │   │   ├── PatientManagerTests.cs        # Patient logic tests
    │   │   └── UserManagerTests.cs           # User logic tests
    │   │
    │   ├── Services/                         # Services tests
    │   │   ├── PatientServiceTests.cs        # Patient service tests
    │   │   └── DicomServiceTests.cs          # DICOM service tests
    │   │
    │   └── DICOM/                            # DICOM tests
    │       └── DicomProcessorTests.cs        # DICOM processor tests
    │
    ├── Integration/                          # Integration Tests
    │   ├── API/                              # API tests
    │   │   ├── PatientControllerTests.cs     # Patient API tests
    │   │   └── DicomControllerTests.cs       # DICOM API tests
    │   │
    │   └── Data/                             # Data layer tests
    │       └── RepositoryTests.cs            # Repository tests
    │
    ├── TestData/                             # Test data
    │   ├── SampleDicomFiles/                 # Sample DICOM files
    │   └── TestDataGenerator.cs              # Generate test data
    │
    ├── Helpers/                              # Test helpers
    │   ├── TestDbContext.cs                  # In-memory DB context
    │   └── MockHelper.cs                     # Mocking helper
    │
    ├── Properties/                           # Project properties
    │   └── AssemblyInfo.cs
    │
    ├── packages.config                       # NuGet packages
    └── AIDIMS.Tests.csproj                   # Project file
``` 