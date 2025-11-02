# DANH SÁCH ĐẦY ĐỦ TẤT CẢ CÁC BẢNG VÀ TRƯỜNG DỮ LIỆU
## Database: DBI202_Group3_SE2019 Version 1.0

**Ngày:** 02/11/2024  
**Tổng số bảng:** 35 bảng

---

## 📋 MỤC LỤC NHANH

- [PHẦN 1: LOOKUP TABLES (5 bảng)](#phần-1-lookup-tables)
- [PHẦN 2: AUTHENTICATION & USER MANAGEMENT (3 bảng)](#phần-2-authentication--user-management)
- [PHẦN 3: ACADEMIC STRUCTURE (3 bảng)](#phần-3-academic-structure)
- [PHẦN 4: STUDENT MANAGEMENT (5 bảng)](#phần-4-student-management)
- [PHẦN 5: GRADE MANAGEMENT (3 bảng)](#phần-5-grade-management)
- [PHẦN 6: SKILL MANAGEMENT (3 bảng)](#phần-6-skill-management)
- [PHẦN 7: LEARNING GOAL MANAGEMENT (13 bảng)](#phần-7-learning-goal-management)
- [PHẦN 8: GAMIFICATION & ACHIEVEMENTS (3 bảng)](#phần-8-gamification--achievements)
- [PHẦN 9: SYSTEM CONFIGURATION (1 bảng)](#phần-9-system-configuration)

---

# PHẦN 1: LOOKUP TABLES

## 1. ROLE

**Mục đích:** Quản lý vai trò người dùng trong hệ thống

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| RoleID | VARCHAR(10) | ✅ PK | Mã vai trò | 'Student', 'Advisor', 'Admin' |
| RoleName | NVARCHAR(50) | ✅ UNIQUE | Tên vai trò | 'Sinh viên', 'Cố vấn học tập', 'Quản trị viên' |
| Description | NVARCHAR(255) | ❌ | Mô tả vai trò | NULL được phép |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái hoạt động | 1 = Active, 0 = Inactive |

**Relationships:**
- 1:N → Account (RoleID)

**Logic:**
- Lookup table cho vai trò trong hệ thống
- Soft delete với IsActive
- Được sử dụng để phân quyền

---

## 2. PERSONTYPE

**Mục đích:** Phân loại người dùng (ISA discriminator) - Student/Advisor

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| PersonTypeID | TINYINT | ✅ PK | ID tự động | IDENTITY(1,1) |
| PersonTypeCode | VARCHAR(10) | ✅ UNIQUE | Mã loại | 'STUDENT', 'ADVISOR' |
| PersonTypeName | NVARCHAR(50) | ✅ | Tên loại | 'Sinh viên', 'Cố vấn học tập' |
| Description | NVARCHAR(255) | ❌ | Mô tả | NULL được phép |

**Relationships:**
- 1:N → ProfilePersonType (PersonTypeID)

**Logic:**
- ISA discriminator để phân biệt Student và Advisor
- Mỗi Profile chỉ có 1 PersonType (enforced by trigger)

---

## 3. GOALTYPE

**Mục đích:** Loại mục tiêu học tập (chuẩn hóa 3NF)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| GoalTypeID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalTypeCode | VARCHAR(20) | ✅ UNIQUE | Mã loại | 'GPAGool', 'SubjectGoal', 'SkillGoal'... |
| GoalTypeName | NVARCHAR(100) | ✅ | Tên loại | 'Mục tiêu GPA', 'Mục tiêu môn học'... |
| Category | NVARCHAR(50) | ✅ | Danh mục | 'Academic', 'Personal', 'Professional' |
| DefaultUnit | NVARCHAR(20) | ✅ | Đơn vị mặc định | 'GPA', 'Score', 'Level', 'Credits'... |
| CalculationMethod | VARCHAR(50) | ❌ | Phương pháp tính | 'FromStudentGrade', 'FromSkills', 'Manual' |
| Description | NVARCHAR(500) | ❌ | Mô tả | NULL được phép |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |

**Relationships:**
- 1:N → GoalTemplate (GoalTypeID)
- 1:N → LearningGoal (GoalTypeID)

**Logic:**
- Chuẩn hóa 3NF: Loại bỏ redundancy từ LearningGoal
- DefaultUnit được sử dụng khi UnitOverride = NULL

**Các GoalType:**
- GPAGool → DefaultUnit: 'GPA'
- SubjectGoal → DefaultUnit: 'Score'
- SkillGoal → DefaultUnit: 'Level'
- SemesterGoal → DefaultUnit: 'Credits'
- CreditGoal → DefaultUnit: 'Credits'
- AttendanceGoal → DefaultUnit: 'Percent'
- GeneralGoal → DefaultUnit: 'Count'

---

## 4. MAJOR

**Mục đích:** Ngành học

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| MajorID | VARCHAR(10) | ✅ PK | Mã ngành | 'SE', 'DM', 'LA' |
| MajorName | NVARCHAR(100) | ✅ | Tên ngành | 'Kỹ thuật phần mềm', 'Digital Marketing'... |
| MajorCode | VARCHAR(10) | ❌ UNIQUE | Mã ngành (alternate) | Có thể NULL, UNIQUE nếu có |
| Description | NVARCHAR(500) | ❌ | Mô tả ngành | NULL được phép |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |

**Relationships:**
- 1:N → Student (MajorID)
- 1:N → Curriculum (MajorID)
- 1:N → GoalTemplate (MajorID)
- 1:N → MajorAdvisorPool (MajorID)

**Logic:**
- Lookup table cho ngành học
- Soft delete với IsActive

---

## 5. SEMESTER

**Mục đích:** Kỳ học (Spring, Summer, Fall)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| SemesterID | VARCHAR(15) | ✅ PK | Mã kỳ học | Format: 'SP14', 'SU14', 'FA14' |
| SemesterName | NVARCHAR(50) | ✅ | Tên kỳ | 'Spring 2014', 'Summer 2014'... |
| StartDate | DATE | ✅ | Ngày bắt đầu | CHECK: EndDate > StartDate |
| EndDate | DATE | ✅ | Ngày kết thúc | CHECK: EndDate > StartDate |
| AcademicYear | INT | ✅ | Năm học | 2014, 2015, 2016... |
| SemesterOrder | TINYINT | ✅ | Thứ tự kỳ | 1=Spring, 2=Summer, 3=Fall, CHECK: 1-3 |

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
- **SemesterOrder**: Dùng để tính toán số kỳ chênh lệch (trigger `trg_Student_AutoUpdateStatus`)
- Format: SP14 = Spring 2014, FA25 = Fall 2025
- AcademicYear: Dễ query theo năm học

---

# PHẦN 2: AUTHENTICATION & USER MANAGEMENT

## 6. ACCOUNT

**Mục đích:** Tài khoản đăng nhập

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| UserName | VARCHAR(50) | ✅ PK | Tên đăng nhập | UNIQUE |
| Email | VARCHAR(255) | ✅ UNIQUE | Email | CHECK: LIKE '%@fpt.edu.vn' |
| Password | VARCHAR(255) | ✅ | Mật khẩu | Hashed password |
| RoleID | VARCHAR(10) | ✅ FK | Vai trò | FK → Role(RoleID) |
| IsDeleted | BIT | ✅ DEFAULT 0 | Soft delete | 0 = Active, 1 = Deleted |
| DeletedDate | DATETIME | ❌ | Ngày xóa | NULL nếu IsDeleted = 0 |
| DeletedBy | VARCHAR(50) | ❌ FK | Người xóa | FK → Account(UserName), NULL nếu chưa xóa |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |
| LastLoginDate | DATETIME | ❌ | Lần đăng nhập cuối | NULL nếu chưa đăng nhập |

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
- Email phải là @fpt.edu.vn

---

## 7. PROFILE (ISA Superclass)

**Mục đích:** Thông tin cá nhân (superclass cho Student/Advisor)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| ID | VARCHAR(10) | ✅ PK | ID cá nhân | = StudentID hoặc AdvisorID |
| UserName | VARCHAR(50) | ✅ FK UNIQUE | Tên đăng nhập | FK → Account(UserName) |
| FirstName | NVARCHAR(50) | ❌ | Tên | NULL được phép |
| MiddleName | NVARCHAR(50) | ❌ | Tên đệm | NULL được phép |
| LastName | NVARCHAR(50) | ❌ | Họ | NULL được phép |
| Address | NVARCHAR(255) | ❌ | Địa chỉ | NULL được phép |
| SocialNum | VARCHAR(12) | ❌ UNIQUE | Số CMND/CCCD | CHECK: LEN = 12 nếu có |
| StartDate | DATE | ❌ | Ngày bắt đầu | NULL được phép |
| EndDate | DATE | ❌ | Ngày kết thúc | NULL nếu đang hoạt động |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |
| ModifiedDate | DATETIME | ❌ | Ngày sửa | NULL nếu chưa sửa |

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
- Computed column: `PersonType` (từ ProfilePersonType)

---

## 8. PROFILEPERSONTYPE (Junction - ISA Implementation)

**Mục đích:** Junction table để implement ISA relationship

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| ProfileID | VARCHAR(10) | ✅ PK FK | ID Profile | FK → Profile(ID) |
| PersonTypeID | TINYINT | ✅ FK | Loại người | FK → PersonType(PersonTypeID) |

**Relationships:**
- N:1 → Profile (ProfileID)
- N:1 → PersonType (PersonTypeID)

**Logic:**
- Đảm bảo mỗi Profile chỉ có 1 PersonType (ISA constraint)
- Enforced bằng trigger `trg_ProfilePersonType_ISA_Constraint`
- PRIMARY KEY (ProfileID) - mỗi Profile chỉ có 1 PersonType

---

# PHẦN 3: ACADEMIC STRUCTURE

## 9. SUBJECT

**Mục đích:** Môn học

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| SubjectID | VARCHAR(10) | ✅ PK | Mã môn học | 'DBI202', 'PRO192'... |
| SubjectName | NVARCHAR(100) | ✅ | Tên môn học | 'Database Introduction'... |
| SubjectCode | VARCHAR(10) | ❌ UNIQUE | Mã môn (alternate) | UNIQUE nếu có |
| Credits | INT | ✅ DEFAULT 3 | Số tín chỉ | CHECK: Credits >= 0 |
| Description | NTEXT | ❌ | Mô tả môn học | NULL được phép |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |

**Relationships:**
- 1:N → Curriculum (SubjectID)
- 1:N → SubjectPrerequisite (SubjectID, PrerequisiteSubjectID)
- 1:N → GradeCategory (SubjectID)
- 1:N → Subject_Skill_Mapping (SubjectID)
- 1:N → LearningGoal (SubjectID)
- 1:N → GoalTemplate (SubjectID)

**Logic:**
- Normalized: Loại bỏ redundant SemesterID, MajorID
- Môn học được liên kết với Major qua Curriculum

---

## 10. CURRICULUM

**Mục đích:** Khung chương trình (Major × Semester × Subject)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| CurriculumID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| MajorID | VARCHAR(10) | ✅ FK | Mã ngành | FK → Major(MajorID) |
| SemesterID | VARCHAR(15) | ✅ FK | Mã kỳ học | FK → Semester(SemesterID) |
| SubjectID | VARCHAR(10) | ✅ FK | Mã môn học | FK → Subject(SubjectID) |
| IsElective | BIT | ✅ DEFAULT 0 | Loại môn | 1 = Bắt buộc, 0 = Tự chọn |
| Credits | INT | ✅ | Số tín chỉ | Lấy từ Subject.Credits |
| PrerequisiteText | NVARCHAR(500) | ❌ | Text mô tả môn tiên quyết | Tự động cập nhật từ SubjectPrerequisite |

**Constraints:**
- UNIQUE (MajorID, SemesterID, SubjectID)
- IsElective: 1 = Bắt buộc, 0 = Tự chọn (reversed logic)

**Relationships:**
- N:1 → Major (MajorID)
- N:1 → Semester (SemesterID)
- N:1 → Subject (SubjectID)

**Logic:**
- Source of Truth: Major × Semester × Subject
- PrerequisiteText được tự động cập nhật từ SubjectPrerequisite (FOR XML PATH)
- IsElective: Logic reversed (1 = Required, 0 = Elective)

---

## 11. SUBJECTPREREQUISITE

**Mục đích:** Môn học tiên quyết

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| SubjectID | VARCHAR(10) | ✅ PK FK | Mã môn học | FK → Subject(SubjectID) |
| PrerequisiteSubjectID | VARCHAR(10) | ✅ PK FK | Mã môn tiên quyết | FK → Subject(SubjectID) |
| PrerequisiteType | VARCHAR(20) | ✅ DEFAULT 'Required' | Loại tiên quyết | 'Required', 'Recommended' |
| MinimumGrade | DECIMAL(4,2) | ❌ | Điểm tối thiểu | CHECK: 0-10 nếu có |

**Constraints:**
- PRIMARY KEY (SubjectID, PrerequisiteSubjectID)
- CHECK: SubjectID != PrerequisiteSubjectID (không tự reference)
- CHECK: PrerequisiteType IN ('Required', 'Recommended')
- CHECK: MinimumGrade IS NULL OR (MinimumGrade >= 0 AND MinimumGrade <= 10)

**Relationships:**
- N:1 → Subject (SubjectID, PrerequisiteSubjectID)

**Logic:**
- Circular dependency được prevent bằng trigger `trg_SubjectPrerequisite_PreventCycle`
- PrerequisiteType: Required = Bắt buộc, Recommended = Khuyến nghị

---

# PHẦN 4: STUDENT MANAGEMENT

## 12. STUDENT

**Mục đích:** Sinh viên

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| StudentID | VARCHAR(10) | ✅ PK | Mã sinh viên | 'HS171234' |
| ProfileID | VARCHAR(10) | ✅ FK UNIQUE | ID Profile | FK → Profile(ID) |
| EnrollmentSemesterID | VARCHAR(15) | ✅ FK | Kỳ nhập học | FK → Semester(SemesterID) |
| CurrentSemesterID | VARCHAR(15) | ❌ FK | Kỳ học hiện tại | FK → Semester(SemesterID), NULL nếu đã tốt nghiệp |
| MajorID | VARCHAR(10) | ✅ FK | Mã ngành | FK → Major(MajorID) |
| Status | NVARCHAR(50) | ✅ DEFAULT 'Đang theo học' | Trạng thái | 'Đang theo học', 'Đã ra trường', 'Tạm nghỉ', 'Bảo lưu', 'Bị đuổi học' |
| EnrollmentDate | DATE | ✅ | Ngày nhập học | CHECK: GraduationDate >= EnrollmentDate |
| GraduationDate | DATE | ❌ | Ngày tốt nghiệp | NULL nếu chưa tốt nghiệp |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |
| ModifiedDate | DATETIME | ❌ | Ngày sửa | NULL nếu chưa sửa |

**Constraints:**
- CHECK: Status IN ('Đang theo học', 'Đã ra trường', 'Tạm nghỉ', 'Bảo lưu', 'Bị đuổi học')
- CHECK: GraduationDate IS NULL OR GraduationDate >= EnrollmentDate
- CHECK: CurrentSemesterID IS NULL OR CurrentSemesterID >= EnrollmentSemesterID

**Relationships:**
- 1:1 → Profile (ProfileID)
- N:1 → Major (MajorID)
- N:1 → Semester (EnrollmentSemesterID, CurrentSemesterID)
- 1:N → StudentGrade (StudentID)
- 1:N → Student_Skill_Assessment (StudentID)
- 1:N → LearningGoal (StudentID)
- 1:N → StudentAdvisorAssignment (StudentID)
- 1:N → Student_Achievement (StudentID)
- 1:1 → StudyStreak (StudentID)

**Logic:**
- Status được tự động cập nhật bởi trigger `trg_Student_AutoUpdateStatus`:
  - "Đang theo học": CurrentSemesterID = kỳ hiện tại (FA25)
  - "Tạm nghỉ": CurrentSemesterID < kỳ hiện tại (< 3 kỳ liên tục)
  - "Bảo lưu": CurrentSemesterID < kỳ hiện tại (>= 3 kỳ liên tục)
- GraduationDate chỉ có giá trị nếu Status = 'Đã ra trường'

---

## 13. ADVISOR

**Mục đích:** Cố vấn học tập

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| AdvisorID | VARCHAR(10) | ✅ PK | Mã cố vấn | 'GV00123' |
| ProfileID | VARCHAR(10) | ✅ FK UNIQUE | ID Profile | FK → Profile(ID) |
| Specialization | NVARCHAR(255) | ❌ | Chuyên ngành | NULL được phép |
| Department | NVARCHAR(100) | ❌ | Phòng ban | NULL được phép |
| HireDate | DATE | ✅ | Ngày tuyển dụng | Bắt buộc |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |

**Relationships:**
- 1:1 → Profile (ProfileID)
- 1:N → MajorAdvisorPool (AdvisorID)
- 1:N → AdvisorCapacity (AdvisorID)
- 1:N → StudentAdvisorAssignment (AdvisorID)
- 1:N → Student_Skill_Assessment (AssessedByAdvisorID)

**Logic:**
- ISA constraint: Profile không được vừa là Student vừa là Advisor
- Enforced bằng trigger `trg_Advisor_ISA_Constraint`

---

## 14. MAJORADVISORPOOL

**Mục đích:** Pool cố vấn theo ngành

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| MajorID | VARCHAR(10) | ✅ PK FK | Mã ngành | FK → Major(MajorID) |
| AdvisorID | VARCHAR(10) | ✅ PK FK | Mã cố vấn | FK → Advisor(AdvisorID) |
| AssignedDate | DATE | ✅ DEFAULT GETDATE() | Ngày gán | DEFAULT CAST(GETDATE() AS DATE) |

**Relationships:**
- N:1 → Major (MajorID)
- N:1 → Advisor (AdvisorID)

**Logic:**
- Junction table: Nhiều advisor có thể phụ trách 1 ngành
- Mỗi advisor có thể phụ trách nhiều ngành

---

## 15. ADVISORCAPACITY

**Mục đích:** Sức chứa cố vấn (số lượng sinh viên có thể phụ trách)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| CapacityID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| AdvisorID | VARCHAR(10) | ✅ FK | Mã cố vấn | FK → Advisor(AdvisorID) |
| SemesterID | VARCHAR(15) | ✅ FK | Mã kỳ học | FK → Semester(SemesterID) |
| MaxSlots | INT | ✅ | Số slot tối đa | CHECK: MaxSlots > 0 (30-50) |
| CurrentSlots | INT | ✅ DEFAULT 0 | Số slot hiện tại | CHECK: CurrentSlots >= 0, CurrentSlots <= MaxSlots |

**Constraints:**
- UNIQUE (AdvisorID, SemesterID)
- CHECK: MaxSlots > 0
- CHECK: CurrentSlots >= 0
- Trigger: `trg_AdvisorCapacity_ValidateSlots` → CurrentSlots <= MaxSlots

**Relationships:**
- N:1 → Advisor (AdvisorID)
- N:1 → Semester (SemesterID)

**Logic:**
- MaxSlots: Số lượng sinh viên tối đa advisor có thể phụ trách (30-50)
- CurrentSlots: Số lượng sinh viên hiện tại đang phụ trách (tính từ StudentAdvisorAssignment hoặc random 0-80% MaxSlots)
- Được validate bằng trigger để đảm bảo CurrentSlots <= MaxSlots

---

## 16. STUDENTADVISORASSIGNMENT

**Mục đích:** Gán cố vấn cho sinh viên

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| AssignmentID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| StudentID | VARCHAR(10) | ✅ FK | Mã sinh viên | FK → Student(StudentID) |
| AdvisorID | VARCHAR(10) | ✅ FK | Mã cố vấn | FK → Advisor(AdvisorID) |
| SemesterID | VARCHAR(15) | ✅ FK | Mã kỳ học | FK → Semester(SemesterID) |
| AssignedDate | DATE | ✅ DEFAULT GETDATE() | Ngày gán | DEFAULT CAST(GETDATE() AS DATE) |
| EndDate | DATE | ❌ | Ngày kết thúc | NULL nếu đang active, CHECK: EndDate >= AssignedDate |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |

**Constraints:**
- UNIQUE (StudentID, SemesterID) - Mỗi sinh viên chỉ có 1 advisor trong 1 kỳ
- CHECK: EndDate IS NULL OR EndDate >= AssignedDate

**Relationships:**
- N:1 → Student (StudentID)
- N:1 → Advisor (AdvisorID)
- N:1 → Semester (SemesterID)

**Logic:**
- Mỗi sinh viên chỉ có 1 advisor trong 1 kỳ học
- IsActive = 1 nếu assignment đang active

---

# PHẦN 5: GRADE MANAGEMENT

## 17. GRADECATEGORY

**Mục đích:** Thành phần điểm (Assignment, Progress Test, Final Exam...)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| CategoryID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| SubjectID | VARCHAR(10) | ✅ FK | Mã môn học | FK → Subject(SubjectID) |
| CategoryName | NVARCHAR(100) | ✅ | Tên thành phần | 'Assignment', 'Progress Test', 'Final Exam' |
| CategoryPercent | DECIMAL(5,2) | ✅ | Tỷ lệ % | CHECK: 0 < CategoryPercent <= 100 |
| CategoryOrder | INT | ✅ DEFAULT 1 | Thứ tự | Sắp xếp hiển thị |
| Condition | NTEXT | ❌ | Điều kiện | NULL được phép |

**Constraints:**
- UNIQUE (SubjectID, CategoryName)
- CHECK: CategoryPercent > 0 AND CategoryPercent <= 100
- Trigger: `trg_GradeCategory_ValidatePercent` → Tổng CategoryPercent = 100% cho mỗi Subject

**Relationships:**
- N:1 → Subject (SubjectID)
- 1:N → GradeDetail (CategoryID)

**Logic:**
- Tổng CategoryPercent phải = 100% cho mỗi Subject (enforced by trigger)
- CategoryOrder: Sắp xếp thứ tự hiển thị

---

## 18. GRADEDETAIL

**Mục đích:** Chi tiết điểm trong từng thành phần

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| DetailID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| CategoryID | INT | ✅ FK | ID thành phần | FK → GradeCategory(CategoryID) ON DELETE CASCADE |
| DetailName | NVARCHAR(100) | ✅ | Tên chi tiết | 'Assignment 1', 'Assignment 2', 'PT1'... |
| WeightInCategory | DECIMAL(5,2) | ✅ | Trọng số trong thành phần | CHECK: 0 < WeightInCategory <= 100 |
| DetailOrder | INT | ✅ DEFAULT 1 | Thứ tự | Sắp xếp hiển thị |

**Constraints:**
- UNIQUE (CategoryID, DetailName)
- CHECK: WeightInCategory > 0 AND WeightInCategory <= 100
- Trigger: `trg_GradeDetail_ValidateWeight` → Tổng WeightInCategory = 100% cho mỗi Category

**Relationships:**
- N:1 → GradeCategory (CategoryID)
- 1:N → StudentGrade (DetailID)

**Logic:**
- Tổng WeightInCategory phải = 100% cho mỗi Category (enforced by trigger)
- DetailOrder: Sắp xếp thứ tự hiển thị

---

## 19. STUDENTGRADE

**Mục đích:** Điểm số của sinh viên

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| GradeID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| StudentID | VARCHAR(10) | ✅ FK | Mã sinh viên | FK → Student(StudentID) |
| DetailID | INT | ✅ FK | ID chi tiết điểm | FK → GradeDetail(DetailID) |
| SemesterID | VARCHAR(15) | ✅ FK | Mã kỳ học | FK → Semester(SemesterID) |
| SubjectID | VARCHAR(10) | ✅ FK | Mã môn học | FK → Subject(SubjectID) |
| Mark | DECIMAL(4,2) | ✅ | Điểm số | CHECK: 0 <= Mark <= 10 |
| IsRetake | BIT | ✅ DEFAULT 0 | Điểm học lại | 1 = Học lại, 0 = Học lần đầu |
| OriginalGradeID | INT | ❌ FK | ID điểm gốc (nếu học lại) | FK → StudentGrade(GradeID), NULL nếu không học lại |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |
| ModifiedDate | DATETIME | ❌ | Ngày sửa | NULL nếu chưa sửa |

**Constraints:**
- UNIQUE (StudentID, DetailID, SemesterID, SubjectID)
- CHECK: Mark >= 0 AND Mark <= 10
- Trigger: `trg_StudentGrade_ValidateSubjectMatch` → SubjectID phải khớp với DetailID

**Relationships:**
- N:1 → Student (StudentID)
- N:1 → GradeDetail (DetailID)
- N:1 → Semester (SemesterID)
- N:1 → Subject (SubjectID)
- N:1 → StudentGrade (OriginalGradeID - self-reference)

**Logic:**
- IsRetake = 1 nếu đây là điểm học lại
- OriginalGradeID reference đến điểm gốc nếu học lại
- SubjectID phải khớp với SubjectID của GradeDetail (enforced by trigger)

---

# PHẦN 6: SKILL MANAGEMENT

## 20. SKILL

**Mục đích:** Kỹ năng

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| SkillID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| SkillName | NVARCHAR(255) | ✅ | Tên kỹ năng | 'Lập trình Java', 'Database Design'... |
| SkillDescription | NTEXT | ❌ | Mô tả kỹ năng | NULL được phép |
| SkillCategory | NVARCHAR(50) | ❌ | Danh mục kỹ năng | 'Programming', 'Database', 'Marketing'... |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |

**Relationships:**
- 1:N → Subject_Skill_Mapping (SkillID)
- 1:N → Student_Skill_Assessment (SkillID)

**Logic:**
- Lookup table cho kỹ năng
- SkillCategory: Phân loại kỹ năng

---

## 21. SUBJECT_SKILL_MAPPING

**Mục đích:** Mapping môn học với kỹ năng (môn học phát triển kỹ năng nào)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| SubjectID | VARCHAR(10) | ✅ PK FK | Mã môn học | FK → Subject(SubjectID) |
| SkillID | INT | ✅ PK FK | ID kỹ năng | FK → Skill(SkillID) |
| SkillLevel | TINYINT | ❌ | Mức độ phát triển kỹ năng | CHECK: 1-5, NULL được phép |

**Constraints:**
- PRIMARY KEY (SubjectID, SkillID)
- CHECK: SkillLevel BETWEEN 1 AND 5 (nếu có)

**Relationships:**
- N:1 → Subject (SubjectID)
- N:1 → Skill (SkillID)

**Logic:**
- SkillLevel: Mức độ phát triển kỹ năng mà môn học cung cấp (1-5)
  - Level 1: Cơ bản
  - Level 2: Trung bình
  - Level 3: Trung cấp
  - Level 4: Nâng cao
  - Level 5: Chuyên sâu
- Phân bố đều 1-5 để đa dạng dữ liệu

---

## 22. STUDENT_SKILL_ASSESSMENT

**Mục đích:** Đánh giá kỹ năng của sinh viên

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| AssessmentID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| StudentID | VARCHAR(10) | ✅ FK | Mã sinh viên | FK → Student(StudentID) |
| SkillID | INT | ✅ FK | ID kỹ năng | FK → Skill(SkillID) |
| ProficiencyLevel | INT | ✅ DEFAULT 1 | Mức độ thành thạo | CHECK: 1-5 |
| Evidence | NVARCHAR(500) | ❌ | Bằng chứng | NULL được phép nếu có AssessedByAdvisorID |
| AssessedByAdvisorID | VARCHAR(10) | ❌ FK | Người đánh giá | FK → Advisor(AdvisorID), NULL nếu self-assessment |
| AssessmentDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày đánh giá | DEFAULT GETDATE() |

**Constraints:**
- UNIQUE (StudentID, SkillID)
- CHECK: ProficiencyLevel BETWEEN 1 AND 5
- CHECK: AssessedByAdvisorID IS NOT NULL OR Evidence IS NOT NULL (phải có ít nhất 1 trong 2)

**Relationships:**
- N:1 → Student (StudentID)
- N:1 → Skill (SkillID)
- N:1 → Advisor (AssessedByAdvisorID)

**Logic:**
- ProficiencyLevel: Mức độ thành thạo hiện tại (1-5)
  - Level 1: Mới bắt đầu
  - Level 2: Cơ bản
  - Level 3: Trung bình
  - Level 4: Tốt
  - Level 5: Xuất sắc
- Phải có AssessedByAdvisorID (đánh giá bởi advisor) HOẶC Evidence (bằng chứng tự đánh giá)

---

# PHẦN 7: LEARNING GOAL MANAGEMENT

## 23. GOALTEMPLATE

**Mục đích:** Template mục tiêu học tập

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| TemplateID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| Title | NVARCHAR(255) | ✅ | Tiêu đề | 'Mục tiêu GPA tổng thể 3.0'... |
| Description | NTEXT | ❌ | Mô tả | NULL được phép |
| GoalTypeID | INT | ✅ FK | Loại mục tiêu | FK → GoalType(GoalTypeID) |
| MajorID | VARCHAR(10) | ❌ FK | Mã ngành | FK → Major(MajorID), NULL nếu áp dụng tất cả |
| SubjectID | VARCHAR(10) | ❌ FK | Mã môn học | FK → Subject(SubjectID), NULL nếu không phải SubjectGoal |
| DefaultTargetValue | DECIMAL(10,2) | ❌ | Giá trị mục tiêu mặc định | NULL được phép |
| UnitOverride | NVARCHAR(20) | ❌ | Đơn vị override | NULL = dùng DefaultUnit từ GoalType |
| DifficultyLevel | VARCHAR(20) | ❌ | Mức độ khó | 'Beginner', 'Intermediate', 'Advanced' |
| EstimatedDuration | INT | ❌ | Thời gian ước tính (ngày) | NULL được phép |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |

**Relationships:**
- N:1 → GoalType (GoalTypeID)
- N:1 → Major (MajorID)
- N:1 → Subject (SubjectID)
- 1:N → TaskTemplate (TemplateID)
- 1:N → LearningGoal (TemplateID)
- 1:N → GoalRecommendation (TemplateID)

**Logic:**
- Template có thể được dùng lại cho nhiều sinh viên
- UnitOverride: Override đơn vị mặc định từ GoalType.DefaultUnit
- MajorID = NULL: Template áp dụng cho tất cả ngành
- SubjectID = NULL: Template không phải SubjectGoal

---

## 24. TASKTEMPLATE

**Mục đích:** Template nhiệm vụ cho mục tiêu

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| TaskTemplateID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| TemplateID | INT | ✅ FK | ID template mục tiêu | FK → GoalTemplate(TemplateID) ON DELETE CASCADE |
| TaskDescription | NVARCHAR(500) | ✅ | Mô tả nhiệm vụ | 'Theo dõi điểm số hàng tuần'... |
| TaskOrder | INT | ✅ DEFAULT 1 | Thứ tự | Sắp xếp thứ tự thực hiện |
| EstimatedDays | INT | ❌ | Thời gian ước tính (ngày) | NULL được phép |
| IsRequired | BIT | ✅ DEFAULT 1 | Bắt buộc | 1 = Required, 0 = Optional |
| TaskType | VARCHAR(20) | ❌ | Loại nhiệm vụ | 'Study', 'Practice', 'Assignment', 'Exam', 'Project', 'Other' |

**Relationships:**
- N:1 → GoalTemplate (TemplateID)
- 1:N → GoalTask (TaskTemplateID)

**Logic:**
- Mỗi GoalTemplate có thể có nhiều TaskTemplate
- TaskOrder: Sắp xếp thứ tự thực hiện nhiệm vụ

---

## 25. LEARNINGGOAL

**Mục đích:** Mục tiêu học tập cụ thể của sinh viên

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| GoalID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| StudentID | VARCHAR(10) | ✅ FK | Mã sinh viên | FK → Student(StudentID) |
| TemplateID | INT | ❌ FK | ID template | FK → GoalTemplate(TemplateID), NULL nếu custom |
| GoalTypeID | INT | ✅ FK | Loại mục tiêu | FK → GoalType(GoalTypeID) |
| SemesterID | VARCHAR(15) | ✅ FK | Mã kỳ học | FK → Semester(SemesterID) |
| SubjectID | VARCHAR(10) | ❌ FK | Mã môn học | FK → Subject(SubjectID), NULL nếu không phải SubjectGoal |
| ParentGoalID | INT | ❌ FK | ID mục tiêu cha | FK → LearningGoal(GoalID), NULL nếu không có parent |
| GoalName | NVARCHAR(255) | ❌ | Tên mục tiêu | NULL = dùng từ Template |
| Description | NTEXT | ❌ | Mô tả | NULL = dùng từ Template |
| Status | NVARCHAR(20) | ✅ DEFAULT 'Pending' | Trạng thái | 'Pending', 'In-Progress', 'Completed', 'Cancelled' |
| Priority | VARCHAR(10) | ✅ DEFAULT 'Medium' | Ưu tiên | 'High', 'Medium', 'Low' |
| GoalRank | INT | DEFAULT 0 | Xếp hạng | Số nguyên, có thể âm |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | Audit trail |
| StartDate | DATE | ❌ | Ngày bắt đầu | NULL nếu chưa bắt đầu |
| TargetDate | DATE | ❌ | Ngày mục tiêu | NULL nếu chưa đặt |
| CompletedDate | DATE | ❌ | Ngày hoàn thành | NULL nếu chưa hoàn thành |

**Constraints:**
- CHECK: Status IN ('Pending', 'In-Progress', 'Completed', 'Cancelled')
- CHECK: Priority IN ('High', 'Medium', 'Low')
- CHECK: (StartDate IS NULL OR TargetDate IS NULL OR TargetDate >= StartDate)
- CHECK: (CompletedDate IS NULL OR CompletedDate >= StartDate)

**Relationships:**
- N:1 → Student (StudentID)
- N:1 → GoalTemplate (TemplateID)
- N:1 → GoalType (GoalTypeID)
- N:1 → Semester (SemesterID)
- N:1 → Subject (SubjectID)
- N:1 → LearningGoal (ParentGoalID - self-reference)
- 1:1 → GoalTarget (GoalID)
- 1:N → GoalSnapshot (GoalID)
- 1:N → GoalMilestone (GoalID)
- 1:N → GoalProgress (GoalID)
- 1:N → GoalDependency (GoalID, DependsOnGoalID)
- 1:N → GoalTask (GoalID)
- 1:N → GoalComment (GoalID)
- 1:N → GoalReflection (GoalID)

**Logic:**
- TemplateID = NULL: Mục tiêu custom (không dùng template)
- ParentGoalID: Mục tiêu con (sub-goal) của mục tiêu cha
- GoalName = NULL: Dùng Title từ Template
- Status được cập nhật tự động dựa trên tiến độ

---

## 26. GOALTARGET

**Mục đích:** Mục tiêu cụ thể (target values, range)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| GoalID | INT | ✅ PK FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| TargetValue | DECIMAL(10,2) | ✅ | Giá trị mục tiêu | >= 0 |
| UnitOverride | NVARCHAR(20) | ❌ | Đơn vị override | NULL = dùng từ GoalTemplate hoặc GoalType |
| MinimumValue | DECIMAL(10,2) | ❌ | Giá trị tối thiểu | NULL được phép, nếu có: <= TargetValue |
| MaximumValue | DECIMAL(10,2) | ❌ | Giá trị tối đa | NULL được phép, nếu có: >= TargetValue |

**Constraints:**
- CHECK: TargetValue >= 0
- CHECK: (MinimumValue IS NULL OR MinimumValue <= TargetValue)
- CHECK: (MaximumValue IS NULL OR MaximumValue >= TargetValue)

**Relationships:**
- 1:1 → LearningGoal (GoalID)

**Logic:**
- **UnitOverride**: 
  - Ưu tiên: GoalTarget.UnitOverride > GoalTemplate.UnitOverride > GoalType.DefaultUnit
  - 70% giữ nguyên từ template/GoalType
  - 30% override sang đơn vị khác (đa dạng: GPA_4, GPA_10, Letter, Proficiency...)
- **MinimumValue**: 65%-75% của DefaultTargetValue (giá trị tối thiểu chấp nhận)
- **MaximumValue**: 125%-135% của DefaultTargetValue (giá trị tối đa đặt ra)
- Tạo range hợp lý cho mục tiêu

---

## 27. GOALSNAPSHOT

**Mục đích:** Snapshot tiến độ (historical tracking)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| SnapshotID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| SnapshotDate | DATE | ✅ DEFAULT GETDATE() | Ngày snapshot | UNIQUE với GoalID |
| CurrentValue | DECIMAL(10,2) | ✅ | Giá trị hiện tại | Tính từ StudentGrade/Skills |
| ProgressPercent | DECIMAL(5,2) | ❌ | Phần trăm tiến độ | CHECK: 0-100 |
| CalculatedFrom | NVARCHAR(50) | ❌ | Nguồn tính toán | 'FromStudentGrade', 'FromSkills', 'Manual' |
| Note | NVARCHAR(500) | ❌ | Ghi chú | NULL được phép |

**Constraints:**
- UNIQUE (GoalID, SnapshotDate) - Mỗi goal chỉ có 1 snapshot trong 1 ngày
- CHECK: ProgressPercent BETWEEN 0 AND 100 (nếu có)

**Relationships:**
- N:1 → LearningGoal (GoalID)

**Logic:**
- Snapshot hiện tại: SnapshotDate = GETDATE()
- Historical snapshots: 2-4 snapshots/goal với ngày khác nhau (7-99 days ago)
- CurrentValue được tính từ:
  - GPAGool: AVG(StudentGrade.Mark) * 0.4
  - SubjectGoal: AVG(StudentGrade.Mark) WHERE SubjectID = ...
  - SkillGoal: AVG(Student_Skill_Assessment.ProficiencyLevel)
- ProgressPercent: (CurrentValue / TargetValue) * 100 (clamped 0-100)

---

## 28. GOALMILESTONE

**Mục đích:** Cột mốc mục tiêu

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| MilestoneID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| MilestoneName | NVARCHAR(255) | ✅ | Tên cột mốc | 'Milestone 1: Đạt 25% tiến độ'... |
| TargetDate | DATE | ❌ | Ngày mục tiêu | NULL được phép |
| TargetValue | DECIMAL(10,2) | ❌ | Giá trị mục tiêu | NULL được phép |
| IsCompleted | BIT | ✅ DEFAULT 0 | Đã hoàn thành | 1 = Completed, 0 = Not completed |
| CompletedDate | DATE | ❌ | Ngày hoàn thành | NULL nếu chưa hoàn thành |

**Constraints:**
- CHECK: CompletedDate IS NULL OR TargetDate IS NULL OR CompletedDate >= TargetDate

**Relationships:**
- N:1 → LearningGoal (GoalID)

**Logic:**
- Mỗi goal có 3 milestones (25%, 50%, 75% tiến độ)
- IsCompleted = 1 nếu goal.Status = 'Completed'
- CompletedDate >= TargetDate nếu IsCompleted = 1

---

## 29. GOALPROGRESS

**Mục đích:** Tiến độ mục tiêu (historical tracking)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| ProgressID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| RecordDate | DATE | ✅ | Ngày ghi nhận | Bắt buộc |
| Value | DECIMAL(10,2) | ✅ | Giá trị | Tăng dần theo thời gian |
| ProgressPercent | DECIMAL(5,2) | ❌ | Phần trăm tiến độ | CHECK: 0-100 |
| Note | NVARCHAR(500) | ❌ | Ghi chú | NULL được phép |

**Constraints:**
- CHECK: ProgressPercent BETWEEN 0 AND 100 (nếu có)

**Relationships:**
- N:1 → LearningGoal (GoalID)

**Logic:**
- 5-8 records/goal (tăng từ 3-5)
- Value tăng dần theo thời gian (simulate progress)
- ProgressPercent được clamp 0-100

---

## 30. GOALDEPENDENCY

**Mục đích:** Phụ thuộc giữa các mục tiêu

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| DependencyID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) |
| DependsOnGoalID | INT | ✅ FK | ID mục tiêu phụ thuộc | FK → LearningGoal(GoalID) |
| DependencyType | VARCHAR(20) | ❌ | Loại phụ thuộc | 'Prerequisite', 'Co-requisite', 'Related' |

**Constraints:**
- CHECK: GoalID != DependsOnGoalID (không tự phụ thuộc)
- CHECK: DependencyType IN ('Prerequisite', 'Co-requisite', 'Related')

**Relationships:**
- N:1 → LearningGoal (GoalID, DependsOnGoalID)

**Logic:**
- 30% goals có dependency
- DependencyType:
  - Prerequisite: Mục tiêu này cần hoàn thành trước
  - Co-requisite: Hai mục tiêu nên thực hiện đồng thời
  - Related: Hai mục tiêu có liên quan với nhau
- Tránh circular dependency: GoalID < DependsOnGoalID

---

## 31. GOALTASK

**Mục đích:** Nhiệm vụ của mục tiêu

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| TaskID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| TaskTemplateID | INT | ❌ FK | ID template nhiệm vụ | FK → TaskTemplate(TaskTemplateID) |
| TaskDescription | NVARCHAR(500) | ✅ | Mô tả nhiệm vụ | Lấy từ TaskTemplate hoặc custom |
| DueDate | DATE | ❌ | Ngày hết hạn | NULL được phép |
| IsCompleted | BIT | ✅ DEFAULT 0 | Đã hoàn thành | 1 = Completed, 0 = Not completed |
| CompletedDate | DATE | ❌ | Ngày hoàn thành | NULL nếu chưa hoàn thành |

**Relationships:**
- N:1 → LearningGoal (GoalID)
- N:1 → TaskTemplate (TaskTemplateID)
- 1:N → Reminder (TaskID)

**Logic:**
- 85% goals có tasks
- IsCompleted = 1 nếu goal.Status = 'Completed'
- CompletedDate = DueDate ± 5 days nếu IsCompleted = 1

---

## 32. GOALCOMMENT

**Mục đích:** Comment trên mục tiêu

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| CommentID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| UserID | VARCHAR(10) | ✅ FK | ID người comment | FK → Profile(ID) |
| CommentText | NTEXT | ✅ | Nội dung comment | Bắt buộc |
| Timestamp | DATETIME | ✅ DEFAULT GETDATE() | Thời gian comment | DEFAULT GETDATE() |
| IsEdited | BIT | ✅ DEFAULT 0 | Đã chỉnh sửa | 1 = Edited, 0 = Not edited |
| EditedDate | DATETIME | ❌ | Ngày chỉnh sửa | NULL nếu chưa chỉnh sửa |

**Relationships:**
- N:1 → LearningGoal (GoalID)
- N:1 → Profile (UserID)

**Logic:**
- 60% goals có comment
- 50% từ student, 50% từ advisor
- Mỗi goal có 1-2 comments
- 20% comments được edit (có EditedDate)

---

## 33. GOALRECOMMENDATION

**Mục đích:** Đề xuất mục tiêu thông minh

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| RecommendationID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| StudentID | VARCHAR(10) | ✅ FK | Mã sinh viên | FK → Student(StudentID) |
| TemplateID | INT | ❌ FK | ID template | FK → GoalTemplate(TemplateID) |
| RecommendedTargetValue | DECIMAL(10,2) | ❌ | Giá trị mục tiêu đề xuất | NULL được phép |
| Reason | NVARCHAR(500) | ❌ | Lý do đề xuất | NULL được phép |
| PriorityScore | DECIMAL(5,2) | ❌ | Điểm ưu tiên | CHECK: 0-100 |
| CreatedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày tạo | DEFAULT GETDATE() |
| IsAccepted | BIT | ✅ DEFAULT 0 | Đã chấp nhận | 1 = Accepted, 0 = Pending/Declined |
| AcceptedDate | DATETIME | ❌ | Ngày chấp nhận | NULL nếu chưa chấp nhận |
| DeclinedDate | DATETIME | ❌ | Ngày từ chối | NULL nếu chưa từ chối |

**Relationships:**
- N:1 → Student (StudentID)
- N:1 → GoalTemplate (TemplateID)

**Logic:**
- 40% students có recommendations
- 33% Accepted, 33% Pending, 33% Declined
- PriorityScore: 60-100
- 6 loại Reason khác nhau

---

## 34. GOALREFLECTION

**Mục đích:** Suy ngẫm về mục tiêu

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| ReflectionID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| GoalID | INT | ✅ FK | ID mục tiêu | FK → LearningGoal(GoalID) ON DELETE CASCADE |
| ReflectionText | NTEXT | ✅ | Nội dung suy ngẫm | Bắt buộc |
| Timestamp | DATETIME | ✅ DEFAULT GETDATE() | Thời gian | DEFAULT GETDATE() |
| IsEdited | BIT | ✅ DEFAULT 0 | Đã chỉnh sửa | 1 = Edited, 0 = Not edited |
| EditedDate | DATETIME | ❌ | Ngày chỉnh sửa | NULL nếu chưa chỉnh sửa |

**Relationships:**
- N:1 → LearningGoal (GoalID)

**Logic:**
- 50% goals có reflection
- Đa dạng theo status:
  - Completed: 4 loại reflection
  - In-Progress: 3 loại reflection
  - Pending: 1 loại reflection
- 25% reflections được edit

---

## 35. REMINDER

**Mục đích:** Nhắc nhở

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| ReminderID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| TaskID | INT | ✅ FK | ID nhiệm vụ | FK → GoalTask(TaskID) ON DELETE CASCADE |
| ReminderTime | DATETIME | ✅ | Thời gian nhắc nhở | Bắt buộc |
| Message | NVARCHAR(255) | ❌ | Nội dung nhắc nhở | NULL được phép |
| Status | VARCHAR(10) | ✅ DEFAULT 'Pending' | Trạng thái | 'Pending', 'Sent', 'Failed' |

**Constraints:**
- CHECK: Status IN ('Pending', 'Sent', 'Failed')

**Relationships:**
- N:1 → GoalTask (TaskID)

**Logic:**
- Nhắc nhở cho các nhiệm vụ sắp đến hạn

---

# PHẦN 8: GAMIFICATION & ACHIEVEMENTS

## 36. ACHIEVEMENT

**Mục đích:** Thành tích

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| BadgeID | INT | ✅ PK | ID tự động | IDENTITY(1,1) |
| BadgeName | NVARCHAR(100) | ✅ UNIQUE | Tên thành tích | 'Xuất sắc học tập'... |
| BadgeCode | VARCHAR(50) | ❌ UNIQUE | Mã thành tích | UNIQUE nếu có |
| CriteriaDescription | NVARCHAR(500) | ❌ | Mô tả tiêu chí | NULL được phép |
| BadgeIconURL | NVARCHAR(255) | ❌ | URL icon | NULL được phép |
| IsActive | BIT | ✅ DEFAULT 1 | Trạng thái | 1 = Active, 0 = Inactive |

**Relationships:**
- 1:N → Student_Achievement (BadgeID)

**Logic:**
- Lookup table cho thành tích
- Soft delete với IsActive

---

## 37. STUDENT_ACHIEVEMENT

**Mục đích:** Thành tích của sinh viên

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| StudentID | VARCHAR(10) | ✅ PK FK | Mã sinh viên | FK → Student(StudentID) |
| BadgeID | INT | ✅ PK FK | ID thành tích | FK → Achievement(BadgeID) |
| DateEarned | DATETIME | ✅ DEFAULT GETDATE() | Ngày đạt được | DEFAULT GETDATE() |

**Relationships:**
- N:1 → Student (StudentID)
- N:1 → Achievement (BadgeID)

**Logic:**
- Junction table: Sinh viên có thể có nhiều thành tích
- DateEarned: Ngày sinh viên đạt được thành tích

---

## 38. STUDYSTREAK

**Mục đích:** Chuỗi học tập (số ngày học liên tục)

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| StudentID | VARCHAR(10) | ✅ PK FK | Mã sinh viên | FK → Student(StudentID) |
| CurrentStreak | INT | ✅ DEFAULT 0 | Chuỗi hiện tại | CHECK: >= 0 |
| LongestStreak | INT | ✅ DEFAULT 0 | Chuỗi dài nhất | CHECK: >= 0, >= CurrentStreak |
| LastActiveDate | DATE | ❌ | Ngày hoạt động cuối | NULL nếu chưa có |

**Constraints:**
- CHECK: CurrentStreak >= 0
- CHECK: LongestStreak >= 0
- CHECK: LongestStreak >= CurrentStreak

**Relationships:**
- 1:1 → Student (StudentID)

**Logic:**
- CurrentStreak: Số ngày học liên tục hiện tại
- LongestStreak: Chuỗi học tập dài nhất từ trước đến nay
- LastActiveDate: Ngày cuối cùng sinh viên hoạt động (học, làm bài tập...)

---

# PHẦN 9: SYSTEM CONFIGURATION

## 39. SYSTEMCONFIGURATION

**Mục đích:** Cấu hình hệ thống

| Trường | Kiểu | NOT NULL | Mô tả | Logic & Ràng buộc |
|--------|------|----------|-------|-------------------|
| ConfigKey | VARCHAR(50) | ✅ PK | Khóa cấu hình | 'CurrentSemester', 'MaxAdvisorSlots'... |
| ConfigValue | NVARCHAR(500) | ✅ | Giá trị cấu hình | 'FA25', '50'... |
| Description | NVARCHAR(255) | ❌ | Mô tả | NULL được phép |
| ModifiedDate | DATETIME | ✅ DEFAULT GETDATE() | Ngày sửa | DEFAULT GETDATE() |

**Relationships:**
- Không có FK

**Logic:**
- Key-value store cho cấu hình hệ thống
- Ví dụ: CurrentSemester = 'FA25' (được dùng trong trigger `trg_Student_AutoUpdateStatus`)

---

## 🔗 ERD DIAGRAM

### Text Format ERD

```
┌─────────────────┐
│     ROLE        │
├─────────────────┤
│ RoleID (PK)     │
│ RoleName        │
│ Description     │
│ IsActive        │
└────────┬────────┘
         │ 1
         │
         │ N
┌────────▼────────┐
│    ACCOUNT      │
├─────────────────┤
│ UserName (PK)   │
│ Email           │
│ Password        │
│ RoleID (FK)     │
│ IsDeleted       │
│ DeletedDate     │
│ DeletedBy (FK)  │
└────────┬────────┘
         │ 1
         │
         │ 1
┌────────▼────────┐
│    PROFILE      │ (ISA Superclass)
├─────────────────┤
│ ID (PK)         │
│ UserName (FK)   │
│ FirstName       │
│ MiddleName      │
│ LastName        │
│ Address         │
│ SocialNum       │
└────┬───────┬────┘
     │       │
     │ 1     │ 1
     │       │
     │       │
┌────▼───┐ ┌─▼──────────┐
│ STUDENT│ │  ADVISOR   │
├────────┤ ├────────────┤
│StudentID│ │AdvisorID  │
│ProfileID│ │ProfileID  │
│Enroll.. │ │Specializ..│
│Current..│ │Department │
│MajorID  │ │HireDate   │
│Status   │ └───────────┘
│Enroll.. │
│GradDate │
└────┬────┘
     │
     │ N
     │
┌────▼─────────────────┐
│   LEARNINGGOAL       │
├──────────────────────┤
│ GoalID (PK)          │
│ StudentID (FK)       │
│ TemplateID (FK)      │
│ GoalTypeID (FK)      │
│ Status               │
│ Priority             │
└────┬─────────────────┘
     │
     │ 1
     │
┌────▼──────────┐
│  GOALTARGET   │
├───────────────┤
│ GoalID (PK)   │
│ TargetValue   │
│ UnitOverride  │ ⭐ TRÁNH NULL
│ MinimumValue  │ ⭐ TRÁNH NULL
│ MaximumValue  │ ⭐ TRÁNH NULL
└───────────────┘

[Và nhiều bảng khác...]
```

---

## 📊 TỔNG KẾT CÁC BẢNG

| Phần | Số bảng | Tên các bảng |
|------|---------|--------------|
| 1. Lookup Tables | 5 | Role, PersonType, GoalType, Major, Semester |
| 2. Auth & User | 3 | Account, Profile, ProfilePersonType |
| 3. Academic | 3 | Subject, Curriculum, SubjectPrerequisite |
| 4. Student Management | 5 | Student, Advisor, MajorAdvisorPool, AdvisorCapacity, StudentAdvisorAssignment |
| 5. Grade Management | 3 | GradeCategory, GradeDetail, StudentGrade |
| 6. Skill Management | 3 | Skill, Subject_Skill_Mapping, Student_Skill_Assessment |
| 7. Goal Management | 13 | GoalTemplate, TaskTemplate, LearningGoal, GoalTarget, GoalSnapshot, GoalMilestone, GoalProgress, GoalDependency, GoalTask, GoalComment, GoalRecommendation, GoalReflection, Reminder |
| 8. Gamification | 3 | Achievement, Student_Achievement, StudyStreak |
| 9. System Config | 1 | SystemConfiguration |
| **TỔNG CỘNG** | **39** | |

---

**File này được tạo để hỗ trợ làm ERD và báo cáo database hoàn chỉnh**

