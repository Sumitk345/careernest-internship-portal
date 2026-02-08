# Database Architecture

## Supported Databases
- Primary: MySQL (production)
- ORM: JPA/Hibernate (portable to other relational databases)

## Entity-Relationship Diagram (Mermaid)

```mermaid
erDiagram
    USERS {
        BIGINT user_id PK
        VARCHAR name
        VARCHAR email UNIQUE
        VARCHAR password
        ENUM role
        BOOLEAN is_verified
        TIMESTAMP created_at
        ENUM provider
        VARCHAR provider_id
    }

    STUDENT_PROFILE {
        BIGINT student_id PK, FK -> USERS.user_id
        VARCHAR education
        TEXT skills
        VARCHAR resume_url
        VARCHAR profile_pic_url
        TEXT bio
    }

    COMPANY_PROFILE {
        BIGINT company_id PK, FK -> USERS.user_id
        VARCHAR company_name
        VARCHAR industry
        VARCHAR website
        TEXT description
        VARCHAR headquarters
        VARCHAR founded
        VARCHAR company_size
        VARCHAR phone
        VARCHAR email
        VARCHAR logo
        ENUM status
    }

    INTERNSHIP {
        BIGINT internship_id PK
        BIGINT company_id FK -> COMPANY_PROFILE.company_id
        VARCHAR title
        VARCHAR location
        VARCHAR stipend
        VARCHAR skills_required
        VARCHAR duration
        DATE deadline
        VARCHAR status
    }

    APPLICATION {
        BIGINT application_id PK
        BIGINT student_id FK -> STUDENT_PROFILE.student_id
        BIGINT internship_id FK -> INTERNSHIP.internship_id
        VARCHAR resume_url
        ENUM status
        TIMESTAMP applied_date
    }

    APPLICATION_STAGE {
        BIGINT stage_id PK
        BIGINT application_id FK -> APPLICATION.application_id
        ENUM status
        TEXT notes
        TIMESTAMP stage_date
        BIGINT updated_by FK -> USERS.user_id
    }

    CERTIFICATE {
        BIGINT certificate_id PK
        BIGINT student_id FK -> STUDENT_PROFILE.student_id
        BIGINT internship_id FK -> INTERNSHIP.internship_id
        DATE issue_date
        VARCHAR file_url
    }

    NOTIFICATION {
        BIGINT notification_id PK
        BIGINT user_id FK -> USERS.user_id
        VARCHAR message
        TIMESTAMP created_at
    }

    EMAIL_VERIFICATION_OTP {
        BIGINT id PK
        VARCHAR email UNIQUE
        VARCHAR otp_code
        TIMESTAMP created_at
        TIMESTAMP expires_at
        BOOLEAN used
        INT attempt_count
    }

    USERS ||--|| STUDENT_PROFILE : "mapsId"
    USERS ||--|| COMPANY_PROFILE : "mapsId"
    COMPANY_PROFILE ||--o{ INTERNSHIP : "has"
    STUDENT_PROFILE ||--o{ APPLICATION : "applies"
    INTERNSHIP ||--o{ APPLICATION : "receives"
    APPLICATION ||--o{ APPLICATION_STAGE : "progresses"
    STUDENT_PROFILE ||--o{ CERTIFICATE : "earns"
    INTERNSHIP ||--o{ CERTIFICATE : "issues"
    USERS ||--o{ NOTIFICATION : "receives"
```

## Tables, Fields, and Indices

### users
- Columns: user_id PK, name, email UNIQUE, password, role ENUM(STUDENT, COMPANY, ADMIN), is_verified, created_at, provider ENUM(LOCAL, GOOGLE, ...), provider_id
- Indices: `email` (UNIQUE)

### student_profile
- Columns: student_id PK (FK users.user_id), education, skills, resume_url, profile_pic_url, bio
- Indices: `student_id` (PK/FK)

### company_profile
- Columns: company_id PK (FK users.user_id), company_name, industry, website, description, headquarters, founded, company_size, phone, email, logo, status ENUM(PENDING, APPROVED)
- Indices: `company_id` (PK/FK), recommended: `(status)`, `(company_name)`

### internship
- Columns: internship_id PK, company_id FK, title, location, stipend, skills_required, duration, deadline DATE, status
- Indices: `(company_id)`, recommended: `(deadline)`, `(title)`, `(location)`

### application
- Columns: application_id PK, student_id FK, internship_id FK, resume_url, status ENUM, applied_date TIMESTAMP
- Indices: `(student_id)`, `(internship_id)`, recommended composite: `(student_id, internship_id)`

### application_stage
- Columns: stage_id PK, application_id FK, status ENUM, notes, stage_date TIMESTAMP, updated_by FK
- Indices: `(application_id)`, recommended: `(stage_date DESC)`

### certificate
- Columns: certificate_id PK, student_id FK, internship_id FK, issue_date DATE, file_url
- Indices: `(student_id)`, `(internship_id)`, recommended: `(issue_date)`

### notification
- Columns: notification_id PK, user_id FK, message, created_at TIMESTAMP
- Indices: `(user_id)`, `(created_at DESC)`

### email_verification_otp
- Columns: id PK, email UNIQUE, otp_code, created_at, expires_at, used, attempt_count
- Indices: `email` (UNIQUE), recommended: `(expires_at)`, `(used)`

