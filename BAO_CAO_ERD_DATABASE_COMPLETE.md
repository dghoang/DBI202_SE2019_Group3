# BÁO CÁO ERD VÀ DATABASE HOÀN CHỈNH
## Hệ thống Quản lý Học tập và Mục tiêu Học tập

**Phiên bản:** 1.0  
**Ngày:** 02/11/2024  
**Database:** DBI202_Group3_SE2019

---

## 📋 MỤC LỤC

1. [Tổng quan Database](#tổng-quan-database)
2. [Danh sách TẤT CẢ các bảng](#danh-sách-tất-cả-các-bảng)
3. [Chi tiết từng bảng với trường và logic](#chi-tiết-từng-bảng)
4. [ERD Diagram](#erd-diagram)
5. [Relationships và Constraints](#relationships-và-constraints)
6. [Business Logic](#business-logic)

---

## 📊 TỔNG QUAN DATABASE

### Mục đích
Hệ thống quản lý học tập và mục tiêu học tập toàn diện cho sinh viên đại học, bao gồm:
- Quản lý sinh viên và cố vấn học tập
- Quản lý chương trình học, môn học, điểm số
- Quản lý kỹ năng và đánh giá
- Quản lý mục tiêu học tập chi tiết
- Gamification và achievements

### Chuẩn hóa
- ✅ **1NF**: Tất cả các trường đều atomic
- ✅ **2NF**: Loại bỏ partial dependencies
- ✅ **3NF**: Loại bỏ transitive dependencies
- ✅ **BCNF**: Loại bỏ anomalies

### Tổng số bảng: **35 bảng**

---

## 📋 DANH SÁCH TẤT CẢ CÁC BẢNG

### PHẦN 1: LOOKUP TABLES (5 bảng)
1. **Role** - Vai trò người dùng
2. **PersonType** - Loại người dùng (ISA discriminator)
3. **GoalType** - Loại mục tiêu học tập
4. **Major** - Ngành học
5. **Semester** - Kỳ học

### PHẦN 2: AUTHENTICATION & USER MANAGEMENT (3 bảng)
6. **Account** - Tài khoản đăng nhập
7. **Profile** - Thông tin cá nhân (ISA superclass)
8. **ProfilePersonType** - Junction table cho ISA relationship

### PHẦN 3: ACADEMIC STRUCTURE (3 bảng)
9. **Subject** - Môn học
10. **Curriculum** - Khung chương trình (Major × Semester × Subject)
11. **SubjectPrerequisite** - Môn học tiên quyết

### PHẦN 4: STUDENT MANAGEMENT (5 bảng)
12. **Student** - Sinh viên
13. **Advisor** - Cố vấn học tập
14. **MajorAdvisorPool** - Pool cố vấn theo ngành
15. **AdvisorCapacity** - Sức chứa cố vấn
16. **StudentAdvisorAssignment** - Gán cố vấn cho sinh viên

### PHẦN 5: GRADE MANAGEMENT (3 bảng)
17. **GradeCategory** - Thành phần điểm
18. **GradeDetail** - Chi tiết điểm
19. **StudentGrade** - Điểm số sinh viên

### PHẦN 6: SKILL MANAGEMENT (3 bảng)
20. **Skill** - Kỹ năng
21. **Subject_Skill_Mapping** - Mapping môn học với kỹ năng
22. **Student_Skill_Assessment** - Đánh giá kỹ năng sinh viên

### PHẦN 7: LEARNING GOAL MANAGEMENT (13 bảng)
23. **GoalTemplate** - Template mục tiêu
24. **TaskTemplate** - Template nhiệm vụ
25. **LearningGoal** - Mục tiêu học tập
26. **GoalTarget** - Mục tiêu cụ thể (target values)
27. **GoalSnapshot** - Snapshot tiến độ (historical tracking)
28. **GoalMilestone** - Cột mốc mục tiêu
29. **GoalProgress** - Tiến độ mục tiêu
30. **GoalDependency** - Phụ thuộc giữa mục tiêu
31. **GoalTask** - Nhiệm vụ của mục tiêu
32. **GoalComment** - Comment trên mục tiêu
33. **GoalRecommendation** - Đề xuất mục tiêu
34. **GoalReflection** - Suy ngẫm về mục tiêu
35. **Reminder** - Nhắc nhở

### PHẦN 8: GAMIFICATION & ACHIEVEMENTS (3 bảng)
36. **Achievement** - Thành tích
37. **Student_Achievement** - Thành tích của sinh viên
38. **StudyStreak** - Chuỗi học tập

### PHẦN 9: SYSTEM CONFIGURATION (1 bảng)
39. **SystemConfiguration** - Cấu hình hệ thống

---

## 📝 CHI TIẾT TỪNG BẢNG

### 1. ROLE (Lookup Table)

**Mục đích:** Quản lý vai trò người dùng trong hệ thống

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| RoleID | VARCHAR(10) | Khóa chính, mã vai trò | ✅ | 'Student', 'Advisor', 'Admin' |
| RoleName | NVARCHAR(50) | Tên vai trò | ✅ | UNIQUE constraint |
| Description | NVARCHAR(255) | Mô tả vai trò | ❌ | Có thể NULL |
| IsActive | BIT | Trạng thái hoạt động | ✅ | DEFAULT 1 |

**Relationships:**
- 1:N → Account (RoleID)

**Logic:**
- Được sử dụng để phân quyền trong hệ thống
- Soft delete với IsActive

---

### 2. PERSONTYPE (ISA Discriminator)

**Mục đích:** Phân loại người dùng (Student/Advisor) cho ISA relationship

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| PersonTypeID | TINYINT | Khóa chính | ✅ | IDENTITY(1,1) |
| PersonTypeCode | VARCHAR(10) | Mã loại | ✅ | 'STUDENT', 'ADVISOR' |
| PersonTypeName | NVARCHAR(50) | Tên loại | ✅ | 'Sinh viên', 'Cố vấn' |
| Description | NVARCHAR(255) | Mô tả | ❌ | Có thể NULL |

**Relationships:**
- 1:N → ProfilePersonType (PersonTypeID)

**Logic:**
- ISA discriminator: Mỗi Profile chỉ có 1 PersonType
- Enforced bằng trigger `trg_ProfilePersonType_ISA_Constraint`

---

### 3. GOALTYPE (Normalized Lookup)

**Mục đích:** Loại mục tiêu học tập (chuẩn hóa từ LearningGoal)

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| GoalTypeID | INT | Khóa chính | ✅ | IDENTITY(1,1) |
| GoalTypeCode | VARCHAR(20) | Mã loại | ✅ | 'GPAGool', 'SubjectGoal', 'SkillGoal'... |
| GoalTypeName | NVARCHAR(100) | Tên loại | ✅ | 'Mục tiêu GPA', 'Mục tiêu môn học'... |
| Category | NVARCHAR(50) | Danh mục | ✅ | 'Academic', 'Personal', 'Professional' |
| DefaultUnit | NVARCHAR(20) | Đơn vị mặc định | ✅ | 'GPA', 'Score', 'Level', 'Credits'... |
| CalculationMethod | VARCHAR(50) | Phương pháp tính | ❌ | 'FromStudentGrade', 'FromSkills'... |
| Description | NVARCHAR(500) | Mô tả | ❌ | Có thể NULL |
| IsActive | BIT | Trạng thái | ✅ | DEFAULT 1 |

**Relationships:**
- 1:N → GoalTemplate (GoalTypeID)
- 1:N → LearningGoal (GoalTypeID)

**Logic:**
- Chuẩn hóa 3NF: Loại bỏ redundancy từ LearningGoal
- DefaultUnit được sử dụng khi UnitOverride = NULL

**Các loại GoalType:**
- GPAGool: Mục tiêu GPA (DefaultUnit: 'GPA')
- SubjectGoal: Mục tiêu môn học (DefaultUnit: 'Score')
- SkillGoal: Mục tiêu kỹ năng (DefaultUnit: 'Level')
- SemesterGoal: Mục tiêu kỳ học (DefaultUnit: 'Credits')
- CreditGoal: Mục tiêu tích lũy tín chỉ (DefaultUnit: 'Credits')
- AttendanceGoal: Mục tiêu tham gia (DefaultUnit: 'Percent')
- GeneralGoal: Mục tiêu chung (DefaultUnit: 'Count')

---

### 4. MAJOR

**Mục đích:** Ngành học

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| MajorID | VARCHAR(10) | Khóa chính | ✅ | 'SE', 'DM', 'LA' |
| MajorName | NVARCHAR(100) | Tên ngành | ✅ | 'Kỹ thuật phần mềm'... |
| MajorCode | VARCHAR(10) | Mã ngành | ❌ | UNIQUE, có thể NULL |
| Description | NVARCHAR(500) | Mô tả | ❌ | Có thể NULL |
| IsActive | BIT | Trạng thái | ✅ | DEFAULT 1 |

**Relationships:**
- 1:N → Student (MajorID)
- 1:N → Curriculum (MajorID)
- 1:N → GoalTemplate (MajorID)

---

### 5. SEMESTER

**Mục đích:** Kỳ học (Spring, Summer, Fall)

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| SemesterID | VARCHAR(15) | Khóa chính | ✅ | Format: 'SP14', 'SU14', 'FA14' |
| SemesterName | NVARCHAR(50) | Tên kỳ | ✅ | 'Spring 2014', 'Summer 2014'... |
| StartDate | DATE | Ngày bắt đầu | ✅ | CHECK: EndDate > StartDate |
| EndDate | DATE | Ngày kết thúc | ✅ | CHECK: EndDate > StartDate |
| AcademicYear | INT | Năm học | ✅ | 2014, 2015... |
| SemesterOrder | TINYINT | Thứ tự kỳ | ✅ | 1=Spring, 2=Summer, 3=Fall |

**Constraints:**
- CHECK: SemesterID format ('SP[0-9][0-9]', 'SU[0-9][0-9]', 'FA[0-9][0-9]')
- CHECK: EndDate > StartDate
- CHECK: SemesterOrder BETWEEN 1 AND 3

**Relationships:**
- 1:N → Student (EnrollmentSemesterID, CurrentSemesterID)
- 1:N → Curriculum (SemesterID)
- 1:N → LearningGoal (SemesterID)
- 1:N → StudentGrade (SemesterID)
- 1:N → AdvisorCapacity (SemesterID)
- 1:N → StudentAdvisorAssignment (SemesterID)

**Logic:**
- SemesterOrder dùng để tính toán số kỳ chênh lệch (trigger `trg_Student_AutoUpdateStatus`)
- Format: SP14 = Spring 2014, FA25 = Fall 2025

---

### 6. ACCOUNT

**Mục đích:** Tài khoản đăng nhập

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| UserName | VARCHAR(50) | Khóa chính | ✅ | UNIQUE |
| Email | VARCHAR(255) | Email | ✅ | UNIQUE, CHECK: LIKE '%@fpt.edu.vn' |
| Password | VARCHAR(255) | Mật khẩu | ✅ | Hashed password |
| RoleID | VARCHAR(10) | Vai trò | ✅ | FK → Role(RoleID) |
| IsDeleted | BIT | Soft delete | ✅ | DEFAULT 0 |
| DeletedDate | DATETIME | Ngày xóa | ❌ | NULL nếu IsDeleted = 0 |
| DeletedBy | VARCHAR(50) | Người xóa | ❌ | FK → Account(UserName) |
| CreatedDate | DATETIME | Ngày tạo | ✅ | DEFAULT GETDATE() |
| LastLoginDate | DATETIME | Lần đăng nhập cuối | ❌ | NULL nếu chưa đăng nhập |

**Constraints:**
- CHECK: Email LIKE '%@fpt.edu.vn'
- CHECK: (IsDeleted = 0) OR (IsDeleted = 1 AND DeletedDate IS NOT NULL)

**Relationships:**
- N:1 → Role (RoleID)
- 1:1 → Profile (UserName)
- N:1 → Account (DeletedBy - self-reference)

**Logic:**
- Soft delete: IsDeleted = 1, DeletedDate IS NOT NULL
- DeletedBy reference chính account đã xóa

---

### 7. PROFILE (ISA Superclass)

**Mục đích:** Thông tin cá nhân (superclass cho Student/Advisor)

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| ID | VARCHAR(10) | Khóa chính | ✅ | = StudentID hoặc AdvisorID |
| UserName | VARCHAR(50) | Tên đăng nhập | ✅ | FK → Account(UserName), UNIQUE |
| FirstName | NVARCHAR(50) | Tên | ❌ | Có thể NULL |
| MiddleName | NVARCHAR(50) | Tên đệm | ❌ | Có thể NULL |
| LastName | NVARCHAR(50) | Họ | ❌ | Có thể NULL |
| Address | NVARCHAR(255) | Địa chỉ | ❌ | Có thể NULL |
| SocialNum | VARCHAR(12) | Số CMND/CCCD | ❌ | UNIQUE, CHECK: LEN = 12 |
| StartDate | DATE | Ngày bắt đầu | ❌ | Có thể NULL |
| EndDate | DATE | Ngày kết thúc | ❌ | NULL nếu đang hoạt động |
| CreatedDate | DATETIME | Ngày tạo | ✅ | DEFAULT GETDATE() |
| ModifiedDate | DATETIME | Ngày sửa | ❌ | NULL nếu chưa sửa |

**Constraints:**
- CHECK: EndDate IS NULL OR EndDate >= StartDate
- CHECK: SocialNum IS NULL OR LEN(SocialNum) = 12

**Relationships:**
- N:1 → Account (UserName)
- 1:1 → Student (ID → ProfileID)
- 1:1 → Advisor (ID → ProfileID)
- 1:1 → ProfilePersonType (ID → ProfileID)
- 1:N → GoalComment (ID → UserID)

**Logic:**
- ISA superclass: Profile có thể là Student hoặc Advisor (không được cả hai)
- Enforced bằng triggers: `trg_Student_ISA_Constraint`, `trg_Advisor_ISA_Constraint`

---

### 8. PROFILEPERSONTYPE (Junction - ISA Implementation)

**Mục đích:** Junction table để implement ISA relationship

**Trường dữ liệu:**
| Trường | Kiểu | Mô tả | NOT NULL | Logic |
|--------|------|-------|----------|-------|
| ProfileID | VARCHAR(10) | Khóa chính | ✅ | FK → Profile(ID) |
| PersonTypeID | TINYINT | Loại người | ✅ | FK → PersonType(PersonTypeID) |

**Constraints:**
- PRIMARY KEY (ProfileID) - Mỗi Profile chỉ có 1 PersonType

**Relationships:**
- N:1 → Profile (ProfileID)
- N:1 → PersonType (PersonTypeID)

**Logic:**
- Đảm bảo mỗi Profile chỉ có 1 PersonType (ISA constraint)
- Enforced bằng trigger `trg_ProfilePersonType_ISA_Constraint`

---

Tiếp tục ở phần sau...

