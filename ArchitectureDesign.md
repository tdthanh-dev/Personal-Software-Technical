# Thiết kế Kiến trúc Hệ thống AIDIMS

## 1. Kiến trúc Tổng thể

Hệ thống AIDIMS được xây dựng theo kiến trúc nhiều tầng (multi-tier architecture) kết hợp với microservices cho module AI.

```mermaid
graph TD
    Client[Người dùng] --> WebApp[Ứng dụng Web]
    WebApp --> APIGateway[API Gateway]
    APIGateway --> AuthService[Dịch vụ Xác thực]
    APIGateway --> PatientService[Dịch vụ Quản lý Bệnh nhân]
    APIGateway --> DicomService[Dịch vụ Quản lý DICOM]
    APIGateway --> AIService[Dịch vụ Phân tích AI]
    
    AuthService --> Database[(Cơ sở dữ liệu)]
    PatientService --> Database
    DicomService --> Database
    DicomService --> BlobStorage[(Blob Storage)]
    AIService --> AIModels[(AI Models)]
    AIService --> Cache[(Redis Cache)]
    
    style WebApp fill:#90CAF9,stroke:#1565C0
    style APIGateway fill:#90CAF9,stroke:#1565C0
    style AuthService fill:#A5D6A7,stroke:#2E7D32
    style PatientService fill:#A5D6A7,stroke:#2E7D32
    style DicomService fill:#A5D6A7,stroke:#2E7D32
    style AIService fill:#A5D6A7,stroke:#2E7D32
    style Database fill:#FFCC80,stroke:#EF6C00
    style BlobStorage fill:#FFCC80,stroke:#EF6C00
    style AIModels fill:#FFCC80,stroke:#EF6C00
    style Cache fill:#FFCC80,stroke:#EF6C00
```

## 2. Kiến trúc Chi tiết Backend

```mermaid
graph TD
    subgraph "AIDIMS.API"
        APIControllers[Controllers]
        Filters[Filters]
        ApiConfig[Configuration]
        SwaggerDoc[Swagger]
    end
    
    subgraph "AIDIMS.Services"
        PatientSvc[PatientService]
        DicomSvc[DicomService]
        UserSvc[UserService]
        NotificationSvc[NotificationService]
    end
    
    subgraph "AIDIMS.Core"
        BL[Business Logic]
        Validation[Validators]
        Processors[Processors]
    end
    
    subgraph "AIDIMS.DICOM"
        DicomProcessor[DicomProcessor]
        DicomConverter[DicomConverter]
        MetadataExtractor[MetadataExtractor]
    end
    
    subgraph "AIDIMS.Data"
        Repositories[Repositories]
        EFContext[DbContext]
        Migrations[Migrations]
        EFConfig[Configuration]
    end
    
    subgraph "AIDIMS.Domain"
        Entities[Entities]
        DTOs[DTOs]
        Interfaces[Interfaces]
        Enums[Enums]
    end
    
    APIControllers --> PatientSvc
    APIControllers --> DicomSvc
    APIControllers --> UserSvc
    APIControllers --> NotificationSvc
    
    PatientSvc --> BL
    DicomSvc --> BL
    UserSvc --> BL
    NotificationSvc --> BL
    
    DicomSvc --> DicomProcessor
    
    BL --> Validation
    BL --> Processors
    
    DicomProcessor --> MetadataExtractor
    DicomProcessor --> DicomConverter
    
    BL --> Repositories
    
    Repositories --> EFContext
    
    EFContext --> Entities
    
    PatientSvc --> DTOs
    DicomSvc --> DTOs
    UserSvc --> DTOs
    
    EFContext --> EFConfig
    
    style APIControllers fill:#90CAF9,stroke:#1565C0
    style PatientSvc fill:#A5D6A7,stroke:#2E7D32
    style DicomSvc fill:#A5D6A7,stroke:#2E7D32
    style UserSvc fill:#A5D6A7,stroke:#2E7D32
    style NotificationSvc fill:#A5D6A7,stroke:#2E7D32
    style BL fill:#FFCC80,stroke:#EF6C00
    style Repositories fill:#CE93D8,stroke:#7B1FA2
    style EFContext fill:#CE93D8,stroke:#7B1FA2
    style Entities fill:#F48FB1,stroke:#C2185B
    style DicomProcessor fill:#81D4FA,stroke:#0288D1
```

## 3. Luồng Xử lý Dữ liệu

### 3.1. Luồng Upload và Phân tích DICOM

```mermaid
sequenceDiagram
    participant Technician as Kỹ thuật viên
    participant WebApp as Ứng dụng Web
    participant DicomAPI as DICOM API
    participant DicomProc as DICOM Processor
    participant DB as Database
    participant Storage as Blob Storage
    participant AIService as AI Service
    participant Doctor as Bác sĩ
    
    Technician->>WebApp: Upload DICOM file
    WebApp->>DicomAPI: POST /api/dicom/upload
    DicomAPI->>DicomProc: Xử lý DICOM
    DicomProc->>DicomProc: Extract metadata
    DicomProc->>Storage: Lưu file DICOM
    DicomProc->>DB: Lưu metadata
    DicomAPI->>AIService: Yêu cầu phân tích
    AIService->>AIService: Phân tích hình ảnh
    AIService->>DB: Lưu kết quả phân tích
    AIService-->>DicomAPI: Trả về kết quả
    DicomAPI-->>WebApp: Trả về response
    WebApp-->>Technician: Hiển thị thông báo thành công
    AIService->>Doctor: Gửi thông báo có kết quả mới
```

### 3.2. Luồng Xem và Chẩn đoán

```mermaid
sequenceDiagram
    participant Doctor as Bác sĩ
    participant WebApp as Ứng dụng Web
    participant DicomAPI as DICOM API
    participant DB as Database
    participant Storage as Blob Storage
    
    Doctor->>WebApp: Xem danh sách bệnh nhân
    WebApp->>DicomAPI: GET /api/patients
    DicomAPI->>DB: Truy vấn danh sách
    DB-->>DicomAPI: Trả về dữ liệu
    DicomAPI-->>WebApp: Trả về danh sách
    WebApp-->>Doctor: Hiển thị danh sách
    
    Doctor->>WebApp: Chọn bệnh nhân
    WebApp->>DicomAPI: GET /api/patients/{id}
    DicomAPI->>DB: Truy vấn thông tin chi tiết
    DB-->>DicomAPI: Trả về dữ liệu
    DicomAPI-->>WebApp: Trả về thông tin
    WebApp-->>Doctor: Hiển thị thông tin
    
    Doctor->>WebApp: Xem hình ảnh DICOM
    WebApp->>DicomAPI: GET /api/dicom/patient/{patientId}
    DicomAPI->>DB: Truy vấn metadata
    DB-->>DicomAPI: Trả về metadata
    DicomAPI->>Storage: Lấy file DICOM
    Storage-->>DicomAPI: Trả về file
    DicomAPI-->>WebApp: Trả về dữ liệu hình ảnh
    WebApp-->>Doctor: Hiển thị hình ảnh
    
    Doctor->>WebApp: Thêm chú thích/ghi chú
    WebApp->>DicomAPI: POST /api/dicom/annotations
    DicomAPI->>DB: Lưu chú thích
    DB-->>DicomAPI: Xác nhận lưu
    DicomAPI-->>WebApp: Trả về xác nhận
    WebApp-->>Doctor: Hiển thị xác nhận
```

## 4. Mô hình Cơ sở Dữ liệu

```mermaid
erDiagram
    Patients ||--o{ DicomFiles : "có"
    Patients {
        int id PK
        string first_name
        string last_name
        date date_of_birth
        string gender
        string address
        string phone
        string email
        string medical_record_number UK
        datetime created_at
        datetime updated_at
    }
    
    DicomFiles ||--o{ DicomAnnotations : "có"
    DicomFiles ||--o{ AIAnalysisResults : "có"
    DicomFiles {
        int id PK
        int patient_id FK
        string study_instance_uid
        string series_instance_uid
        string sop_instance_uid UK
        string modality
        date study_date
        date acquisition_date
        string storage_path
        long file_size
        int physician_id FK
        string thumbnail_path
        datetime created_at
        datetime updated_at
    }
    
    Users ||--o{ DicomAnnotations : "tạo"
    Users ||--o{ DicomFiles : "phụ trách"
    Users {
        int id PK
        string username UK
        string password_hash
        string email UK
        string first_name
        string last_name
        int role_id FK
        boolean is_active
        datetime last_login_at
        datetime created_at
        datetime updated_at
    }
    
    Roles ||--o{ Users : "có"
    Roles {
        int id PK
        string name UK
        string description
        datetime created_at
        datetime updated_at
    }
    
    DicomAnnotations {
        int id PK
        int dicom_file_id FK
        int user_id FK
        jsonb annotation_data
        datetime created_at
        datetime updated_at
    }
    
    AIModels ||--o{ AIAnalysisResults : "tạo"
    AIModels {
        int id PK
        string name
        string version
        string description
        datetime created_at
    }
    
    AIAnalysisResults {
        int id PK
        int dicom_file_id FK
        int model_id FK
        jsonb result_data
        float confidence_score
        float processing_time
        datetime created_at
    }
```

## 5. Kiến trúc Triển khai

```mermaid
graph TD
    subgraph "Production Environment"
        LB[Load Balancer]
        
        subgraph "Web Servers"
            WS1[Web Server 1]
            WS2[Web Server 2]
        end
        
        subgraph "API Servers"
            API1[API Server 1]
            API2[API Server 2]
        end
        
        subgraph "AI Servers"
            AI1[AI Server 1]
            AI2[AI Server 2]
        end
        
        subgraph "Database"
            PGMaster[(PostgreSQL Master)]
            PGSlave[(PostgreSQL Slave)]
        end
        
        subgraph "Storage"
            BS[(Blob Storage)]
        end
        
        subgraph "Cache & Queue"
            Redis[(Redis)]
            RabbitMQ[(RabbitMQ)]
        end
    end
    
    LB --> WS1
    LB --> WS2
    
    WS1 --> API1
    WS1 --> API2
    WS2 --> API1
    WS2 --> API2
    
    API1 --> AI1
    API1 --> AI2
    API2 --> AI1
    API2 --> AI2
    
    API1 --> PGMaster
    API2 --> PGMaster
    PGMaster --> PGSlave
    
    API1 --> BS
    API2 --> BS
    
    API1 --> Redis
    API2 --> Redis
    API1 --> RabbitMQ
    API2 --> RabbitMQ
    
    AI1 --> RabbitMQ
    AI2 --> RabbitMQ
    AI1 --> BS
    AI2 --> BS
    
    style LB fill:#90CAF9,stroke:#1565C0
    style WS1 fill:#A5D6A7,stroke:#2E7D32
    style WS2 fill:#A5D6A7,stroke:#2E7D32
    style API1 fill:#FFCC80,stroke:#EF6C00
    style API2 fill:#FFCC80,stroke:#EF6C00
    style AI1 fill:#CE93D8,stroke:#7B1FA2
    style AI2 fill:#CE93D8,stroke:#7B1FA2
    style PGMaster fill:#F48FB1,stroke:#C2185B
    style PGSlave fill:#F48FB1,stroke:#C2185B
    style BS fill:#81D4FA,stroke:#0288D1
    style Redis fill:#81D4FA,stroke:#0288D1
    style RabbitMQ fill:#81D4FA,stroke:#0288D1
```

## 6. Luồng CI/CD

```mermaid
graph LR
    subgraph "Development"
        Dev[Developer]
        LocalBuild[Local Build]
        GitRepo[GitHub Repository]
    end
    
    subgraph "CI/CD Pipeline"
        Build[Build]
        UnitTest[Unit Tests]
        CodeQuality[Code Quality]
        IntTest[Integration Tests]
        Package[Package]
    end
    
    subgraph "Deployment"
        DeployDev[Deploy to Dev]
        TestDev[Test in Dev]
        DeployStaging[Deploy to Staging]
        TestStaging[Test in Staging]
        DeployProd[Deploy to Production]
        Monitor[Monitor]
    end
    
    Dev -->|commit| LocalBuild
    LocalBuild -->|push| GitRepo
    GitRepo -->|trigger| Build
    Build --> UnitTest
    UnitTest --> CodeQuality
    CodeQuality --> IntTest
    IntTest --> Package
    Package --> DeployDev
    DeployDev --> TestDev
    TestDev --> DeployStaging
    DeployStaging --> TestStaging
    TestStaging --> DeployProd
    DeployProd --> Monitor
    
    style Dev fill:#90CAF9,stroke:#1565C0
    style GitRepo fill:#90CAF9,stroke:#1565C0
    style Build fill:#A5D6A7,stroke:#2E7D32
    style UnitTest fill:#A5D6A7,stroke:#2E7D32
    style CodeQuality fill:#A5D6A7,stroke:#2E7D32
    style IntTest fill:#A5D6A7,stroke:#2E7D32
    style Package fill:#A5D6A7,stroke:#2E7D32
    style DeployDev fill:#FFCC80,stroke:#EF6C00
    style DeployStaging fill:#FFCC80,stroke:#EF6C00
    style DeployProd fill:#FFCC80,stroke:#EF6C00
    style Monitor fill:#FFCC80,stroke:#EF6C00
``` 