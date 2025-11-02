# BÁO CÁO HOÀN CHỈNH VỀ DATABASE DBI202_Group3_SE2019 VERSION 1.0

**Ngày tạo:** 02/11/2024  
**Phiên bản:** 1.0  
**Mục đích:** Hệ thống quản lý học tập và mục tiêu học tập cho sinh viên đại học FPT

---

## 📋 MỤC LỤC

1. [Tổng quan về Database](#1-tổng-quan-về-database)
2. [Giải thích SemesterOrder và các trường quan trọng](#2-giải-thích-semesterorder-và-các-trường-quan-trọng)
3. [Chi tiết từng bảng và các trường](#3-chi-tiết-từng-bảng-và-các-trường)
4. [Logic về chương trình học](#4-logic-về-chương-trình-học)
5. [Hệ thống mục tiêu học tập](#5-hệ-thống-mục-tiêu-học-tập)
6. [Cải tiến dữ liệu](#6-cải-tiến-dữ-liệu)

---

## 1. TỔNG QUAN VỀ DATABASE

### 1.1. Mục đích
Database này được thiết kế để:
- **Quản lý sinh viên và cố vấn học tập**: Quan hệ ISA (Is-A) giữa Profile, Student, Advisor
- **Quản lý chương trình học**: Curriculum mapping (Major × Semester × Subject)
- **Quản lý điểm số**: Hệ thống điểm linh hoạt với GradeCategory và GradeDetail
- **Quản lý kỹ năng**: Mapping giữa Subject và Skill, đánh giá kỹ năng sinh viên
- **Quản lý mục tiêu học tập**: Hệ thống toàn diện cho Learning Goals với nhiều loại mục tiêu khác nhau
- **Gamification**: Achievement, Study Streak để khuyến khích sinh viên

### 1.2. Nguyên tắc thiết kế
- **Chuẩn hóa**: Đạt 1NF, 2NF, 3NF, BCNF
- **Loại bỏ redundancy**: Tách bảng theo chức năng, loại bỏ duplicate data
- **Data Integrity**: CHECK constraints, Foreign Keys, Triggers
- **Performance**: Indexes trên các cột thường query
- **Flexibility**: Dễ mở rộng, dễ maintain

---

## 2. GIẢI THÍCH SEMESTERORDER VÀ CÁC TRƯỜNG QUAN TRỌNG

### 2.1. SemesterOrder - Cột quan trọng trong bảng Semester

**Định nghĩa:**
- `SemesterOrder` là số thứ tự của kỳ học trong một năm học
- Giá trị: 1, 2, hoặc 3 (CHECK constraint: BETWEEN 1 AND 3)

**Mapping:**
- **SemesterOrder = 1** → **Spring (SP)** - Học kỳ Xuân (Tháng 1-4)
- **SemesterOrder = 2** → **Summer (SU)** - Học kỳ Hè (Tháng 5-8)
- **SemesterOrder = 3** → **Fall (FA)** - Học kỳ Thu (Tháng 9-12)

**Ví dụ:**
```
SemesterID = 'SP14' → SemesterOrder = 1, AcademicYear = 2014
SemesterID = 'SU14' → SemesterOrder = 2, AcademicYear = 2014
SemesterID = 'FA14' → SemesterOrder = 3, AcademicYear = 2014
```

**Mục đích sử dụng:**
1. **So sánh kỳ học**: Tính toán số kỳ chênh lệch giữa 2 kỳ học
   - Ví dụ: FA14 (Order=3) vs SP15 (Order=1) → Chênh lệch 1 kỳ
2. **Trigger tự động cập nhật Status**: 
   - Xác định sinh viên "Đang theo học", "Tạm nghỉ", "Bảo lưu" dựa trên kỳ cuối cùng
3. **Sắp xếp kỳ học**: ORDER BY AcademicYear, SemesterOrder
4. **Validation**: Đảm bảo logic nghiệp vụ đúng (ví dụ: không thể học Fall trước Spring trong cùng năm)

**Code sử dụng SemesterOrder trong Trigger:**
```sql
-- Trong trg_Student_AutoUpdateStatus
DECLARE @SemesterOrder TABLE (
    SemesterCode VARCHAR(2),
    OrderNum INT
);
INSERT INTO @SemesterOrder VALUES ('SP', 1), ('SU', 2), ('FA', 3);

-- Tính số kỳ chênh lệch
-- Sử dụng để xác định Status: Đang theo học / Tạm nghỉ / Bảo lưu
```

---

## 3. CHI TIẾT TỪNG BẢNG VÀ CÁC TRƯỜNG

### 3.1. LOOKUP TABLES (Bảng tham chiếu)

#### 3.1.1. Role
**Mục đích:** Vai trò người dùng trong hệ thống

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| RoleID | VARCHAR(10) PK | Mã vai trò | Primary Key |
| RoleName | NVARCHAR(50) | Tên vai trò | UNIQUE constraint |
| Description | NVARCHAR(255) NULL | Mô tả vai trò | **THÊM MỚI**: Chi tiết hóa vai trò |
| IsActive | BIT | Kích hoạt | **THÊM MỚI**: Soft delete, quản lý role |

**Vai trò:** Student, Advisor, Admin

---

#### 3.1.2. PersonType (ISA Discriminator)
**Mục đích:** Phân biệt loại người (Student vs Advisor)

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| PersonTypeID | TINYINT PK IDENTITY | ID tự tăng | Primary Key |
| PersonTypeCode | VARCHAR(10) UNIQUE | Mã loại | 'STUDENT', 'ADVISOR' |
| PersonTypeName | NVARCHAR(50) | Tên loại | 'Sinh viên', 'Cố vấn' |
| Description | NVARCHAR(255) NULL | Mô tả | Chi tiết |

**Tại sao thêm bảng này?**
- **Chuẩn hóa ISA**: Thay vì hardcode 'STUDENT'/'ADVISOR' trong code
- **Mở rộng**: Dễ thêm loại người mới (Alumni, Guest, etc.)
- **Data Integrity**: FK constraint đảm bảo giá trị hợp lệ

---

#### 3.1.3. GoalType (Normalized Lookup)
**Mục đích:** Loại mục tiêu học tập (SubjectGoal, GPAGool, SkillGoal, etc.)

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| GoalTypeID | INT PK IDENTITY | ID tự tăng | Primary Key |
| GoalTypeCode | VARCHAR(20) UNIQUE | Mã loại | 'GPAGool', 'SubjectGoal', 'SkillGoal', etc. |
| GoalTypeName | NVARCHAR(100) | Tên loại | Tên đầy đủ |
| Category | NVARCHAR(50) | Phân loại | 'Academic', 'Personal', 'Professional' |
| DefaultUnit | NVARCHAR(20) | Đơn vị mặc định | 'GPA', 'Score', 'Level', '%' |
| CalculationMethod | VARCHAR(50) NULL | Phương pháp tính | 'Average', 'Sum', 'Max', etc. |
| Description | NVARCHAR(500) NULL | Mô tả | Chi tiết |
| IsActive | BIT | Kích hoạt | Soft delete |

**Tại sao tách riêng GoalType?**
- **Loại bỏ redundancy**: Trước đây GoalType, Category, Unit lặp lại trong LearningGoal
- **Chuẩn hóa 3NF**: Loại bỏ transitive dependency
- **Dễ quản lý**: Thêm loại mục tiêu mới không cần sửa bảng LearningGoal

**Các loại GoalType:**
1. **GPAGool**: Mục tiêu GPA (Đơn vị: GPA, Target: 3.0, 3.5, etc.)
2. **SubjectGoal**: Mục tiêu môn học (Đơn vị: Score, Target: 8.0, 9.0, etc.)
3. **SkillGoal**: Mục tiêu kỹ năng (Đơn vị: Level, Target: 3, 4, 5)
4. **SemesterGoal**: Mục tiêu kỳ học (Đơn vị: Credits, Target: 18, 20, etc.)
5. **CreditGoal**: Mục tiêu tín chỉ (Đơn vị: Credits, Target: 120, 150)
6. **AttendanceGoal**: Mục tiêu tham gia (Đơn vị: %, Target: 80%, 90%)
7. **GeneralGoal**: Mục tiêu chung (Đơn vị: Custom)

---

#### 3.1.4. Major
**Mục đích:** Ngành học

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| MajorID | VARCHAR(10) PK | Mã ngành | Primary Key ('SE', 'DM', 'LA') |
| MajorName | NVARCHAR(100) | Tên ngành | 'Software Engineering', etc. |
| MajorCode | VARCHAR(10) UNIQUE NULL | Mã ngành (code) | **THÊM MỚI**: Code riêng (có thể khác ID) |
| Description | NVARCHAR(500) NULL | Mô tả | **THÊM MỚI**: Chi tiết về ngành |
| IsActive | BIT | Kích hoạt | **THÊM MỚI**: Soft delete |

---

#### 3.1.5. Semester (Enhanced)
**Mục đích:** Kỳ học

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| SemesterID | VARCHAR(15) PK | Mã kỳ | Primary Key ('SP14', 'SU14', 'FA14') |
| SemesterName | NVARCHAR(50) | Tên kỳ | 'Spring 2014', 'Summer 2014', 'Fall 2014' |
| StartDate | DATE | Ngày bắt đầu | CHECK: EndDate > StartDate |
| EndDate | DATE | Ngày kết thúc | CHECK: EndDate > StartDate |
| AcademicYear | INT | Năm học | **THÊM MỚI**: 2014, 2015, etc. |
| SemesterOrder | TINYINT | Thứ tự kỳ | **THÊM MỚI**: 1=Spring, 2=Summer, 3=Fall |

**Tại sao thêm AcademicYear và SemesterOrder?**
- **AcademicYear**: Dễ query theo năm (WHERE AcademicYear = 2024)
- **SemesterOrder**: Tính toán chênh lệch kỳ, sắp xếp, validation (xem giải thích ở 2.1)

---

### 3.2. CORE TABLES - AUTHENTICATION & USER MANAGEMENT

#### 3.2.1. Account
**Mục đích:** Tài khoản đăng nhập

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| UserName | VARCHAR(50) PK | Tên đăng nhập | Primary Key |
| Email | VARCHAR(255) UNIQUE | Email | CHECK: LIKE '%@fpt.edu.vn' |
| Password | VARCHAR(255) | Mật khẩu | Hashed password |
| RoleID | VARCHAR(10) FK | Vai trò | FK → Role(RoleID) |
| IsDeleted | BIT | Đã xóa | **THÊM MỚI**: Soft delete |
| DeletedDate | DATETIME NULL | Ngày xóa | **THÊM MỚI**: Audit trail |
| DeletedBy | VARCHAR(50) FK NULL | Người xóa | **THÊM MỚI**: FK → Account(UserName) |
| CreatedDate | DATETIME | Ngày tạo | **THÊM MỚI**: Audit trail |
| LastLoginDate | DATETIME NULL | Lần đăng nhập cuối | **THÊM MỚI**: Track activity |

**Tại sao thêm Soft Delete?**
- **Data Recovery**: Có thể khôi phục tài khoản đã xóa
- **Audit Trail**: Biết ai xóa, khi nào xóa
- **Compliance**: Tuân thủ yêu cầu lưu trữ dữ liệu

---

#### 3.2.2. Profile (ISA Base Class)
**Mục đích:** Thông tin cá nhân (Base class cho Student và Advisor)

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| ID | VARCHAR(10) PK | ID cá nhân | Primary Key ('HS171234', 'GV00123') |
| FirstName | NVARCHAR(50) | Tên | NULL được phép |
| MiddleName | NVARCHAR(50) | Tên đệm | NULL được phép |
| LastName | NVARCHAR(50) | Họ | NULL được phép |
| Address | NVARCHAR(255) | Địa chỉ | NULL được phép |
| SocialNum | VARCHAR(12) UNIQUE | Số CMND/CCCD | CHECK: LEN = 12, UNIQUE |
| StartDate | DATE | Ngày bắt đầu | CHECK: EndDate >= StartDate |
| EndDate | DATE NULL | Ngày kết thúc | NULL = còn hoạt động |
| UserName | VARCHAR(50) UNIQUE FK | Tên đăng nhập | FK → Account(UserName), UNIQUE |
| PersonTypeID | TINYINT FK NULL | Loại người | **THÊM MỚI**: FK → PersonType |
| PersonType | AS (Computed) | Computed column | **THÊM MỚI**: Tự động từ PersonTypeID |

**Tại sao Profile là Base Class?**
- **ISA Hierarchy**: Student và Advisor đều là Person
- **Shared Attributes**: FirstName, LastName, Address, etc. được share
- **Normalization**: Tránh duplicate data giữa Student và Advisor

**PersonType (Computed Column):**
```sql
PersonType AS (
    SELECT PersonTypeCode FROM PersonType WHERE PersonTypeID = Profile.PersonTypeID
)
```
- Giúp query dễ hơn: `WHERE PersonType = 'STUDENT'`
- Tự động sync với PersonTypeID

---

#### 3.2.3. ProfilePersonType (ISA Constraint Table)
**Mục đích:** Đảm bảo mỗi Profile chỉ có 1 PersonType

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| ProfileID | VARCHAR(10) PK FK | ID Profile | FK → Profile(ID) |
| PersonTypeID | TINYINT NOT NULL | Loại người | FK → PersonType(PersonTypeID) |

**Tại sao cần bảng này?**
- **ISA Constraint**: Đảm bảo Profile chỉ là Student HOẶC Advisor (không thể cả hai)
- **Trigger Validation**: `trg_ProfilePersonType_ISA_Constraint` kiểm tra constraint

---

### 3.3. STUDENT & ADVISOR TABLES (ISA Derived Classes)

#### 3.3.1. Student
**Mục đích:** Thông tin sinh viên

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| StudentID | VARCHAR(10) PK | ID sinh viên | Primary Key, FK → Profile(ID) |
| ProfileID | VARCHAR(10) UNIQUE FK | ID Profile | **THÊM MỚI**: FK → Profile(ID), ISA constraint |
| MajorID | VARCHAR(10) FK | Ngành học | FK → Major(MajorID) |
| EnrollmentSemesterID | VARCHAR(15) FK | Kỳ nhập học | **THÊM MỚI**: Kỳ bắt đầu học |
| CurrentSemesterID | VARCHAR(15) FK | Kỳ học hiện tại | **THÊM MỚI**: Kỳ học cuối cùng |
| Status | NVARCHAR(50) | Trạng thái | CHECK: 'Đang theo học', 'Tạm nghỉ', 'Đã ra trường', 'Bảo lưu' |
| EnrollmentDate | DATE | Ngày nhập học | **THÊM MỚI**: Chi tiết |
| GraduationDate | DATE NULL | Ngày tốt nghiệp | **THÊM MỚI**: NULL = chưa tốt nghiệp |

**Tại sao thêm EnrollmentSemesterID và CurrentSemesterID?**
- **EnrollmentSemesterID**: Xác định kỳ nhập học (dùng để tính năm học)
- **CurrentSemesterID**: Kỳ học cuối cùng → Trigger tự động cập nhật Status
- **Business Logic**: 
  - Nếu CurrentSemesterID = FA25 (kỳ hiện tại) → Status = 'Đang theo học'
  - Nếu CurrentSemesterID < FA25 (< 3 kỳ) → Status = 'Tạm nghỉ'
  - Nếu CurrentSemesterID < FA25 (>= 3 kỳ) → Status = 'Bảo lưu'

**Trigger tự động:**
- `trg_Student_AutoUpdateStatus`: Tự động cập nhật Status dựa trên CurrentSemesterID

---

#### 3.3.2. Advisor
**Mục đích:** Thông tin cố vấn học tập

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| AdvisorID | VARCHAR(10) PK | ID cố vấn | Primary Key, FK → Profile(ID) |
| ProfileID | VARCHAR(10) UNIQUE FK | ID Profile | **THÊM MỚI**: FK → Profile(ID), ISA constraint |
| Specialization | NVARCHAR(255) NULL | Chuyên môn | NULL được phép |
| Department | NVARCHAR(100) NULL | Phòng ban | **THÊM MỚI**: Phòng ban làm việc |
| HireDate | DATE | Ngày tuyển dụng | **THÊM MỚI**: Ngày bắt đầu làm việc |
| IsActive | BIT | Kích hoạt | **THÊM MỚI**: Đang làm việc hay không |
| CreatedDate | DATETIME | Ngày tạo | **THÊM MỚI**: Audit trail |

---

### 3.4. ACADEMIC STRUCTURE

#### 3.4.1. Subject (Normalized)
**Mục đích:** Môn học

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| SubjectID | VARCHAR(10) PK | Mã môn | Primary Key ('DBI202', 'PRO192') |
| SubjectName | NVARCHAR(100) | Tên môn | Tên đầy đủ |
| SubjectCode | VARCHAR(10) NULL | Mã môn (code) | **THÊM MỚI**: Code riêng (có thể khác ID) |
| Credits | INT | Số tín chỉ | CHECK: >= 0, DEFAULT: 3 |

**Tại sao LOẠI BỎ MajorID và SemesterID?**
- **Redundancy**: MajorID và SemesterID đã có trong Curriculum
- **Chuẩn hóa**: Một môn học có thể dạy ở nhiều ngành, nhiều kỳ
- **Normalization**: Loại bỏ partial dependency

**Mapping:**
- Major × Subject → Curriculum (bảng này quyết định môn nào, ngành nào, kỳ nào)

---

#### 3.4.2. Curriculum
**Mục đích:** Khung chương trình học (Major × Semester × Subject)

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| MajorID | VARCHAR(10) PK FK | Mã ngành | FK → Major(MajorID) |
| SemesterID | VARCHAR(15) PK FK | Mã kỳ | FK → Semester(SemesterID) |
| SubjectID | VARCHAR(10) PK FK | Mã môn | FK → Subject(SubjectID) |
| IsElective | BIT | Môn bắt buộc? | **REVERSED**: 1 = Bắt buộc, 0 = Tự chọn |
| Credits | INT | Số tín chỉ | **THÊM MỚI**: Override từ Subject (nếu khác) |

**Tại sao đảo ngược IsElective?**
- **Trước**: 0 = bắt buộc, 1 = tự chọn (counter-intuitive)
- **Sau**: 1 = bắt buộc (Required), 0 = tự chọn (Elective) → Dễ hiểu hơn

**Composite Primary Key:**
- (MajorID, SemesterID, SubjectID) UNIQUE
- Một môn học chỉ có thể được assign 1 lần cho 1 ngành trong 1 kỳ

---

#### 3.4.3. SubjectPrerequisite
**Mục đích:** Môn tiên quyết

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| SubjectID | VARCHAR(10) PK FK | Môn học | FK → Subject(SubjectID) |
| PrerequisiteSubjectID | VARCHAR(10) PK FK | Môn tiên quyết | FK → Subject(SubjectID) |
| PrerequisiteType | VARCHAR(20) NULL | Loại tiên quyết | **THÊM MỚI**: 'Required', 'Recommended' |
| MinimumGrade | DECIMAL(4,2) NULL | Điểm tối thiểu | **THÊM MỚI**: CHECK: 0-10, NULL = không yêu cầu |

**Trigger:**
- `trg_SubjectPrerequisite_PreventCycle`: Ngăn chặn circular dependency (A → B → A)

---

### 3.5. GRADE MANAGEMENT

#### 3.5.1. GradeCategory
**Mục đích:** Thành phần điểm (Assignment, Midterm, Final, etc.)

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| CategoryID | INT PK IDENTITY | ID tự tăng | Primary Key |
| SubjectID | VARCHAR(10) FK | Môn học | FK → Subject(SubjectID) |
| CategoryName | NVARCHAR(100) | Tên thành phần | 'Assignment', 'Midterm', 'Final' |
| CategoryPercent | DECIMAL(5,2) | % trọng số | CHECK: > 0 AND <= 100 |
| CategoryOrder | INT | Thứ tự | **THÊM MỚI**: Sắp xếp (1, 2, 3) |
| Condition | NTEXT NULL | Điều kiện | NULL được phép |

**Trigger:**
- `trg_GradeCategory_ValidatePercent`: Đảm bảo tổng CategoryPercent = 100% cho mỗi Subject

**Ví dụ:**
```
DBI202:
- Assignment: 20%
- Midterm: 30%
- Final: 50%
Tổng = 100%
```

---

#### 3.5.2. GradeDetail
**Mục đích:** Chi tiết điểm trong từng Category

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| DetailID | INT PK IDENTITY | ID tự tăng | Primary Key |
| CategoryID | INT FK | Thành phần điểm | FK → GradeCategory(CategoryID) |
| DetailName | NVARCHAR(100) | Tên chi tiết | 'Assignment 1', 'Assignment 2' |
| WeightInCategory | DECIMAL(5,2) | Trọng số trong Category | CHECK: > 0 AND <= 100 |
| DetailOrder | INT | Thứ tự | **THÊM MỚI**: Sắp xếp |

**Trigger:**
- `trg_GradeDetail_ValidateWeight`: Đảm bảo tổng WeightInCategory = 100% cho mỗi Category

**Ví dụ:**
```
Assignment Category (20%):
- Assignment 1: 50% (của 20% = 10% tổng)
- Assignment 2: 50% (của 20% = 10% tổng)
Tổng = 100% (của Category)
```

---

#### 3.5.3. StudentGrade
**Mục đích:** Điểm số của sinh viên

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| GradeID | INT PK IDENTITY | ID tự tăng | Primary Key |
| StudentID | VARCHAR(10) FK | ID sinh viên | FK → Student(StudentID) |
| DetailID | INT FK | Chi tiết điểm | FK → GradeDetail(DetailID) |
| SemesterID | VARCHAR(15) FK | Kỳ học | FK → Semester(SemesterID) |
| SubjectID | VARCHAR(10) FK | Môn học | FK → Subject(SubjectID) |
| Mark | DECIMAL(4,2) | Điểm | CHECK: >= 0 AND <= 10 |
| IsRetake | BIT | Đang học lại? | **THÊM MỚI**: 1 = học lại |
| OriginalGradeID | INT FK NULL | Điểm gốc | **THÊM MỚI**: FK → StudentGrade(GradeID), NULL nếu lần đầu |
| CreatedDate | DATETIME | Ngày tạo | Audit trail |
| ModifiedDate | DATETIME NULL | Ngày sửa | Audit trail |

**Tại sao thêm IsRetake và OriginalGradeID?**
- **Track retake**: Biết sinh viên học lại môn nào
- **Compare**: So sánh điểm lần 1 vs lần 2
- **Statistics**: Thống kê tỷ lệ học lại

**Trigger:**
- `trg_StudentGrade_ValidateSubjectMatch`: Đảm bảo SubjectID khớp với DetailID

---

### 3.6. SKILL MANAGEMENT

#### 3.6.1. Skill
**Mục đích:** Kỹ năng

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| SkillID | INT PK IDENTITY | ID tự tăng | Primary Key |
| SkillName | NVARCHAR(255) | Tên kỹ năng | 'Java Programming', 'Database Design' |
| SkillDescription | NTEXT NULL | Mô tả | NULL được phép |
| SkillCategory | NVARCHAR(50) NULL | Phân loại | **THÊM MỚI**: 'Technical', 'Soft Skills', etc. |
| IsActive | BIT | Kích hoạt | Soft delete |

---

#### 3.6.2. Subject_Skill_Mapping
**Mục đích:** Mapping môn học và kỹ năng

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| SubjectID | VARCHAR(10) PK FK | Môn học | FK → Subject(SubjectID) |
| SkillID | INT PK FK | Kỹ năng | FK → Skill(SkillID) |
| SkillLevel | TINYINT NULL | Mức độ kỹ năng | **THÊM MỚI**: CHECK: 1-5, NULL = không xác định |

**Tại sao thêm SkillLevel?**
- **Gradient**: Một môn học có thể phát triển kỹ năng ở nhiều mức độ
- **Analysis**: Phân tích môn học nào phát triển kỹ năng tốt nhất

---

#### 3.6.3. Student_Skill_Assessment
**Mục đích:** Đánh giá kỹ năng của sinh viên

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| AssessmentID | INT PK IDENTITY | ID tự tăng | Primary Key |
| StudentID | VARCHAR(10) FK | ID sinh viên | FK → Student(StudentID) |
| SkillID | INT FK | Kỹ năng | FK → Skill(SkillID) |
| ProficiencyLevel | INT | Mức độ thành thạo | CHECK: 1-5, DEFAULT: 1 |
| Evidence | NVARCHAR(500) NULL | Bằng chứng | NULL được phép |
| AssessedByAdvisorID | VARCHAR(10) FK NULL | Người đánh giá | **THÊM MỚI**: FK → Advisor(AdvisorID), NULL = tự đánh giá |
| AssessmentDate | DATETIME | Ngày đánh giá | Audit trail |

**Tại sao đổi AssessedBy từ VARCHAR sang AssessedByAdvisorID FK?**
- **Data Integrity**: FK constraint đảm bảo chỉ Advisor mới đánh giá
- **Normalization**: Loại bỏ string-based reference
- **Query**: JOIN dễ hơn, có thể lấy thông tin Advisor

**Constraint:**
- CHECK: AssessedByAdvisorID IS NOT NULL OR Evidence IS NOT NULL
- Phải có ít nhất 1 trong 2: người đánh giá HOẶC bằng chứng

---

## 4. LOGIC VỀ CHƯƠNG TRÌNH HỌC

### 4.1. Curriculum Mapping (Major × Semester × Subject)

**Mô hình:**
```
Major (SE, DM, LA)
  ↓
Semester (SP14, SU14, FA14, ...)
  ↓
Subject (DBI202, PRO192, ...)
  ↓
Curriculum (SE × SP14 × DBI202)
```

**Ví dụ:**
```
SE (Software Engineering):
- SP14: DBI202, PRO192, ... (Năm 1, Kỳ 1)
- SU14: PRJ301, SWE201, ... (Năm 1, Kỳ 2)
- FA14: PRJ392, SWR302, ... (Năm 1, Kỳ 3)
- SP15: ... (Năm 2, Kỳ 1)
```

**Logic:**
1. Mỗi ngành (Major) có khung chương trình riêng
2. Mỗi kỳ học (Semester) có danh sách môn học
3. Mỗi môn học có thể:
   - **Bắt buộc** (IsElective = 1): Tất cả sinh viên ngành đó phải học
   - **Tự chọn** (IsElective = 0): Sinh viên có thể chọn hoặc không

**Môn chung:**
- Một số môn được dạy cho TẤT CẢ các ngành
- Ví dụ: VOV101 (Vovinam), MAS201 (Toán), PHL101 (Triết học)
- Được insert vào Curriculum với tất cả Major × các kỳ đầu

---

### 4.2. Prerequisite Logic (Môn tiên quyết)

**Ví dụ:**
```
DBI202 (Database) → PRJ301 (Project Management)
  ↑
  └── PRO192 (OOP) → DBI202
```

**Logic:**
1. Sinh viên phải học môn tiên quyết TRƯỚC
2. Phải đạt điểm tối thiểu (MinimumGrade) nếu có yêu cầu
3. Không được tạo circular dependency (A → B → A)

**Trigger:**
- `trg_SubjectPrerequisite_PreventCycle`: Sử dụng recursive CTE để detect cycle

---

## 5. HỆ THỐNG MỤC TIÊU HỌC TẬP (LEARNING GOALS)

### 5.1. Tổng quan

Hệ thống mục tiêu học tập được thiết kế rất linh hoạt và toàn diện, hỗ trợ nhiều loại mục tiêu khác nhau:

1. **GPAGool**: Mục tiêu GPA (ví dụ: đạt GPA 3.5)
2. **SubjectGoal**: Mục tiêu môn học (ví dụ: đạt 8.0 điểm DBI202)
3. **SkillGoal**: Mục tiêu kỹ năng (ví dụ: đạt level 4 Java Programming)
4. **SemesterGoal**: Mục tiêu kỳ học (ví dụ: học 20 tín chỉ kỳ này)
5. **CreditGoal**: Mục tiêu tín chỉ (ví dụ: tích lũy 120 tín chỉ)
6. **AttendanceGoal**: Mục tiêu tham gia (ví dụ: đạt 90% attendance)
7. **GeneralGoal**: Mục tiêu chung (tự định nghĩa)

---

### 5.2. Kiến trúc bảng (Normalized)

#### 5.2.1. GoalType (Lookup Table)
**Mục đích:** Loại mục tiêu (đã giải thích ở 3.1.3)

#### 5.2.2. GoalTemplate (Template)
**Mục đích:** Template mục tiêu (có thể reuse)

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| TemplateID | INT PK IDENTITY | ID tự tăng | Primary Key |
| Title | NVARCHAR(255) | Tiêu đề | 'Đạt GPA 3.5', 'Học giỏi DBI202' |
| Description | NTEXT NULL | Mô tả | NULL được phép |
| GoalTypeID | INT FK | Loại mục tiêu | **THÊM MỚI**: FK → GoalType |
| MajorID | VARCHAR(10) FK NULL | Ngành áp dụng | NULL = áp dụng tất cả ngành |
| SubjectID | VARCHAR(10) FK NULL | Môn học | NULL nếu không phải SubjectGoal |
| DefaultTargetValue | DECIMAL(10,2) NULL | Giá trị mục tiêu mặc định | 3.5, 8.0, etc. |
| UnitOverride | NVARCHAR(20) NULL | Đơn vị override | NULL = dùng từ GoalType |
| DifficultyLevel | VARCHAR(20) | Độ khó | **THÊM MỚI**: 'Beginner', 'Intermediate', 'Advanced' |
| EstimatedDuration | INT NULL | Thời gian ước tính (ngày) | **THÊM MỚI**: 30, 60, 90 ngày |
| IsActive | BIT | Kích hoạt | Soft delete |
| CreatedDate | DATETIME | Ngày tạo | Audit trail |

**Tại sao tách GoalType?**
- **Normalization**: Loại bỏ redundancy (GoalType, Category, Unit)
- **Reusability**: Một GoalType có nhiều GoalTemplate

#### 5.2.3. LearningGoal (Core Table - Rút gọn)
**Mục đích:** Mục tiêu học tập của sinh viên

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| GoalID | INT PK IDENTITY | ID tự tăng | Primary Key |
| StudentID | VARCHAR(10) FK | ID sinh viên | FK → Student(StudentID) |
| TemplateID | INT FK NULL | Template | NULL nếu tự tạo |
| GoalTypeID | INT FK | Loại mục tiêu | **THÊM MỚI**: FK → GoalType |
| SemesterID | VARCHAR(15) FK | Kỳ học | FK → Semester(SemesterID) |
| SubjectID | VARCHAR(10) FK NULL | Môn học | NULL nếu không phải SubjectGoal |
| ParentGoalID | INT FK NULL | Mục tiêu cha | **THÊM MỚI**: Hierarchical goals |
| GoalName | NVARCHAR(255) NULL | Tên mục tiêu | NULL = dùng từ Template |
| Description | NTEXT NULL | Mô tả | NULL được phép |
| Status | NVARCHAR(20) | Trạng thái | CHECK: 'Pending', 'In-Progress', 'Completed', 'Cancelled' |
| Priority | VARCHAR(10) | Ưu tiên | CHECK: 'High', 'Medium', 'Low' |
| GoalRank | INT | Thứ hạng | DEFAULT: 0 |
| CreatedDate | DATETIME | Ngày tạo | Audit trail |
| StartDate | DATE NULL | Ngày bắt đầu | CHECK: TargetDate >= StartDate |
| TargetDate | DATE NULL | Ngày mục tiêu | CHECK: TargetDate >= StartDate |
| CompletedDate | DATE NULL | Ngày hoàn thành | CHECK: CompletedDate >= StartDate |

**Tại sao rút gọn từ 19 → 12 trường?**
- **Tách riêng TargetValue**: → GoalTarget table
- **Tách riêng CurrentValue/ProgressPercent**: → GoalSnapshot table
- **Loại bỏ Unit, Category**: → GoalType table
- **Chuẩn hóa**: Loại bỏ redundancy, dễ maintain

**ParentGoalID (Hierarchical Goals):**
- Mục tiêu có thể có mục tiêu con
- Ví dụ: "Đạt GPA 3.5" (Parent) → "Đạt 8.0 DBI202" (Child), "Đạt 8.0 PRO192" (Child)

#### 5.2.4. GoalTarget (Configuration)
**Mục đích:** Cấu hình giá trị mục tiêu

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| GoalID | INT PK FK | ID mục tiêu | FK → LearningGoal(GoalID) |
| TargetValue | DECIMAL(10,2) | Giá trị mục tiêu | 3.5, 8.0, 120, etc. |
| UnitOverride | NVARCHAR(20) NULL | Đơn vị override | NULL = dùng từ GoalType |
| MinimumValue | DECIMAL(10,2) NULL | Giá trị tối thiểu | **THÊM MỚI**: Range validation |
| MaximumValue | DECIMAL(10,2) NULL | Giá trị tối đa | **THÊM MỚI**: Range validation |

**Tại sao tách riêng GoalTarget?**
- **Separation of Concerns**: Cấu hình tách riêng khỏi core info
- **Flexibility**: Dễ thay đổi target mà không ảnh hưởng goal
- **History**: Có thể track lịch sử thay đổi target (nếu cần)

#### 5.2.5. GoalSnapshot (Historical Tracking)
**Mục đích:** Ảnh chụp tiến độ theo thời gian

| Trường | Kiểu | Mô tả | Lý do |
|--------|------|-------|-------|
| SnapshotID | INT PK IDENTITY | ID tự tăng | Primary Key |
| GoalID | INT FK | ID mục tiêu | FK → LearningGoal(GoalID) |
| SnapshotDate | DATE | Ngày chụp | UNIQUE(GoalID, SnapshotDate) |
| CurrentValue | DECIMAL(10,2) | Giá trị hiện tại | Tính từ StudentGrade, Skills, etc. |
| ProgressPercent | DECIMAL(5,2) | % tiến độ | CHECK: 0-100 |
| CalculatedFrom | NVARCHAR(50) NULL | Tính từ | 'FromStudentGrade', 'FromSkills', 'Manual' |
| Note | NVARCHAR(500) NULL | Ghi chú | NULL được phép |

**Tại sao thay thế CurrentValue/ProgressPercent trong LearningGoal?**
- **Historical Tracking**: Lưu lịch sử tiến độ theo thời gian
- **Analysis**: Phân tích xu hướng, tốc độ đạt mục tiêu
- **Normalization**: Loại bỏ tính toán (calculated fields) khỏi core table

**Ví dụ:**
```
Goal: Đạt GPA 3.5
- Snapshot 1 (01/01): CurrentValue = 3.0, ProgressPercent = 85%
- Snapshot 2 (01/02): CurrentValue = 3.2, ProgressPercent = 91%
- Snapshot 3 (01/03): CurrentValue = 3.5, ProgressPercent = 100%
```

#### 5.2.6. Các bảng hỗ trợ khác

**GoalTask:**
- Nhiệm vụ con trong mục tiêu
- FK → TaskTemplate (nếu từ template)
- DueDate, IsCompleted, CompletedDate

**GoalMilestone:**
- Cột mốc trong mục tiêu (25%, 50%, 75%)
- TargetDate, TargetValue, IsCompleted, CompletedDate

**GoalProgress:**
- Bản ghi tiến độ thủ công (khác với Snapshot tự động)
- RecordDate, Value, ProgressPercent, Note

**GoalDependency:**
- Mục tiêu phụ thuộc (Prerequisite, Co-requisite, Related)
- Ví dụ: "Đạt GPA 3.5" phụ thuộc "Đạt 8.0 DBI202"

**GoalRecommendation:**
- Hệ thống đề xuất mục tiêu cho sinh viên
- PriorityScore, IsAccepted, AcceptedDate, DeclinedDate

**GoalComment, GoalReflection:**
- Bình luận và phản ánh về mục tiêu
- IsEdited, EditedDate (track changes)

---

### 5.3. Logic tính toán tiến độ (Progress Calculation)

**Các loại GoalType và cách tính:**

1. **GPAGool**:
   ```sql
   CurrentValue = AVG(StudentGrade.Mark) * 0.4  -- Hệ số 4.0
   ProgressPercent = (CurrentValue / TargetValue) * 100
   ```

2. **SubjectGoal**:
   ```sql
   CurrentValue = AVG(StudentGrade.Mark WHERE SubjectID = Goal.SubjectID)
   ProgressPercent = (CurrentValue / TargetValue) * 100
   ```

3. **SkillGoal**:
   ```sql
   CurrentValue = AVG(Student_Skill_Assessment.ProficiencyLevel WHERE SkillID = Goal.SkillID)
   ProgressPercent = (CurrentValue / TargetValue) * 100
   ```

4. **SemesterGoal, CreditGoal**:
   ```sql
   CurrentValue = SUM(Curriculum.Credits WHERE SemesterID = Goal.SemesterID)
   ProgressPercent = (CurrentValue / TargetValue) * 100
   ```

**Đảm bảo ProgressPercent trong khoảng 0-100:**
```sql
CASE 
    WHEN ProgressPercent < 0 THEN 0
    WHEN ProgressPercent > 100 THEN 100
    ELSE ProgressPercent
END
```

---

## 6. CẢI TIẾN DỮ LIỆU

### 6.1. Nguyên tắc cải tiến

1. **Phân bố đều**: Dữ liệu phân bố đều trên các giá trị
2. **Giá trị biên**: Bao gồm giá trị min, max, edge cases
3. **Dữ liệu thực tế**: Simulate dữ liệu thực tế nhất có thể
4. **Diversity**: Đa dạng các trường hợp, scenarios

### 6.2. Cải tiến StudentGrade

**Trước:**
- Random điểm, không có phân bố rõ ràng

**Sau:**
- **Phân bố đều 10% cho mỗi khoảng điểm**:
  - 0-1: 10%
  - 1-2: 10%
  - 2-3: 10%
  - ...
  - 9-10: 10%
- **Giá trị biên**: Bao gồm điểm 0, 10, và các giá trị giữa
- **Retake logic**: Một số sinh viên học lại (IsRetake = 1)

### 6.3. Cải tiến LearningGoal

**Đa dạng mục tiêu:**
- Mỗi sinh viên có 1/3 số goals (33%)
- Đa dạng GoalType: GPAGool, SubjectGoal, SkillGoal, etc.
- Đa dạng Status: Pending, In-Progress, Completed, Cancelled
- Đa dạng Priority: High, Medium, Low

**ProgressPercent:**
- Tính toán thực tế từ StudentGrade, Skills
- Đảm bảo 0-100%
- Snapshot theo thời gian

### 6.4. Cải tiến Student_Skill_Assessment

**Logic:**
- Gán kỹ năng dựa trên môn học đã học
- ProficiencyLevel dựa trên điểm trung bình:
  - Điểm >= 6.5 → Level 3-5
  - Điểm < 6.5 → Level 1-2
- AssessedByAdvisorID: 33% được advisor đánh giá, 67% tự đánh giá

---

## 7. TỔNG KẾT

### 7.1. Điểm mạnh

✅ **Chuẩn hóa hoàn chỉnh**: 1NF, 2NF, 3NF, BCNF  
✅ **Loại bỏ redundancy**: Tách bảng theo chức năng  
✅ **Data Integrity**: CHECK constraints, Foreign Keys, Triggers  
✅ **ISA Hierarchy**: Profile → Student/Advisor với constraint  
✅ **Hệ thống mục tiêu linh hoạt**: Nhiều loại goals, tracking lịch sử  
✅ **Dữ liệu đa dạng**: Phân bố đều, giá trị biên, thực tế  

### 7.2. Hướng phát triển

🔮 **Tương lai có thể mở rộng:**
- Notification system (thông báo mục tiêu)
- Analytics dashboard (phân tích tiến độ)
- AI Recommendation (đề xuất mục tiêu thông minh)
- Export/Import (xuất nhập dữ liệu)
- API Integration (tích hợp với hệ thống khác)

---

**Kết thúc báo cáo**

