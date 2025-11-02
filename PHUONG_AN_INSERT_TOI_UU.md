# PHƯƠNG ÁN INSERT DỮ LIỆU TỐI ƯU - DATABASE VERSION 1.0

**Ngày tạo:** 02/11/2024  
**Mục tiêu:** Tối ưu insert dữ liệu để có database đầy đủ, đa dạng, không NULL

---

## 🎯 MỤC TIÊU

1. **Tăng số lượng:**
   - **300 sinh viên** (từ 100)
   - **40 cố vấn** (từ 20)

2. **Đảm bảo đầy đủ dữ liệu (KHÔNG NULL):**
   - FirstName, MiddleName, LastName: LUÔN có giá trị
   - Address: LUÔN có giá trị
   - SocialNum: LUÔN có giá trị, UNIQUE
   - EnrollmentDate: LUÔN có giá trị
   - GraduationDate: Có giá trị nếu Status = 'Đã ra trường'
   - CurrentSemesterID: LUÔN có giá trị

3. **Đa dạng trường hợp Status:**
   - **40% Đang theo học** (120 sinh viên): CurrentSemesterID = FA25
   - **30% Tạm nghỉ** (90 sinh viên): CurrentSemesterID < FA25, < 3 kỳ
   - **15% Bảo lưu** (45 sinh viên): CurrentSemesterID < FA25, >= 3 kỳ
   - **15% Đã tốt nghiệp** (45 sinh viên): GraduationDate NOT NULL, Status = 'Đã ra trường'

4. **Tối ưu cho mục tiêu học tập:**
   - Mỗi sinh viên có 3-7 mục tiêu học tập
   - Đa dạng GoalType: GPAGool, SubjectGoal, SkillGoal, SemesterGoal, CreditGoal
   - Đa dạng Status: Pending, In-Progress, Completed
   - Đầy đủ GoalSnapshot, GoalTarget, GoalTask, GoalMilestone

---

## 📊 PHÂN BỐ DỮ LIỆU

### 1. SINH VIÊN (300 sinh viên)

#### Phân bố theo Khóa (Khoa):
- **K14 (2014)**: 20 sinh viên (đã tốt nghiệp)
- **K15 (2015)**: 25 sinh viên (đã tốt nghiệp)
- **K16 (2016)**: 30 sinh viên (một số đã tốt nghiệp, một số bảo lưu)
- **K17 (2017)**: 35 sinh viên (một số bảo lưu, một số tạm nghỉ)
- **K18 (2018)**: 40 sinh viên (một số tạm nghỉ, một số đang học)
- **K19 (2019)**: 45 sinh viên (một số tạm nghỉ, một số đang học)
- **K20 (2020)**: 50 sinh viên (phần lớn đang học)
- **K21 (2021)**: 55 sinh viên (đang học)

#### Phân bố theo Ngành:
- **SE (Software Engineering)**: 40% (120 sinh viên)
- **DM (Digital Marketing)**: 35% (105 sinh viên)
- **LA (Logistics & Supply Chain)**: 25% (75 sinh viên)

#### Phân bố theo Status:
- **Đang theo học**: 40% (120 sinh viên)
  - CurrentSemesterID = FA25
  - GraduationDate = NULL
  - Status được trigger tự động set

- **Tạm nghỉ**: 30% (90 sinh viên)
  - CurrentSemesterID < FA25 (SP25, SU24, FA24)
  - Chênh lệch < 3 kỳ
  - GraduationDate = NULL

- **Bảo lưu**: 15% (45 sinh viên)
  - CurrentSemesterID < FA25 (SP24 trở về trước)
  - Chênh lệch >= 3 kỳ
  - GraduationDate = NULL

- **Đã tốt nghiệp**: 15% (45 sinh viên)
  - CurrentSemesterID: FA19, FA20, FA21
  - GraduationDate: NOT NULL (ngẫu nhiên sau kỳ cuối)
  - Status = 'Đã ra trường'

---

### 2. CỐ VẤN (40 cố vấn)

#### Phân bố theo Ngành:
- **SE**: 15 cố vấn
- **DM**: 15 cố vấn
- **LA**: 10 cố vấn

#### Phân bố theo Chuyên môn:
- Software Engineering, Database Management, Web Development (SE)
- Digital Marketing, Content Marketing, SEO/SEM (DM)
- Translation, Interpreting, Academic Writing (LA)
- Project Management, UI/UX Design, Machine Learning (Chung)

---

### 3. MỤC TIÊU HỌC TẬP

#### Phân bố:
- **100% sinh viên** có ít nhất 1 mục tiêu học tập
- **Mỗi sinh viên** có trung bình 5 mục tiêu (3-7 mục tiêu)
- **Tổng cộng**: ~1500 mục tiêu học tập

#### Phân bố theo GoalType:
- **GPAGool**: 20% (300 mục tiêu)
- **SubjectGoal**: 35% (525 mục tiêu)
- **SkillGoal**: 20% (300 mục tiêu)
- **SemesterGoal**: 15% (225 mục tiêu)
- **CreditGoal**: 10% (150 mục tiêu)

#### Phân bố theo Status:
- **Pending**: 20% (300 mục tiêu)
- **In-Progress**: 50% (750 mục tiêu)
- **Completed**: 25% (375 mục tiêu)
- **Cancelled**: 5% (75 mục tiêu)

---

## 🔧 LOGIC IMPLEMENTATION

### 1. Sinh viên - Logic Status

```sql
-- Phân bố Status dựa trên Khoa và CurrentSemesterID
CASE 
    -- Đã tốt nghiệp (K14-K16)
    WHEN @Khoa <= 16 THEN
        SET @Status = N'Đã ra trường'
        SET @GraduationDate = DATEADD(day, ABS(CHECKSUM(...)) % 180, LastSemesterEndDate)
        SET @EndDate = @GraduationDate
        
    -- Bảo lưu (K17-K18, CurrentSemesterID cũ)
    WHEN @Khoa <= 18 AND CurrentSemesterID < 'SP24' THEN
        SET @Status = N'Bảo lưu'
        SET @GraduationDate = NULL
        SET @EndDate = NULL
        
    -- Tạm nghỉ (K17-K19, CurrentSemesterID gần)
    WHEN @Khoa <= 19 AND CurrentSemesterID >= 'SP24' AND CurrentSemesterID < 'FA25' THEN
        SET @Status = N'Tạm nghỉ'
        SET @GraduationDate = NULL
        SET @EndDate = NULL
        
    -- Đang theo học (K18-K21, CurrentSemesterID = FA25)
    ELSE
        SET @Status = N'Đang theo học'
        SET @GraduationDate = NULL
        SET @EndDate = NULL
END
```

### 2. Đảm bảo KHÔNG NULL

```sql
-- FirstName, MiddleName, LastName: LUÔN có giá trị
SET @FirstName = ISNULL((SELECT TOP 1 Name FROM #FirstNames ORDER BY NEWID()), N'Unknown')
SET @MiddleName = ISNULL((SELECT TOP 1 Name FROM #MiddleNames ORDER BY NEWID()), N'')
SET @LastName = ISNULL((SELECT TOP 1 Name FROM #LastNames ORDER BY NEWID()), N'Unknown')

-- Address: LUÔN có giá trị
SET @Address = CASE ABS(CHECKSUM(NEWID())) % 20
    WHEN 0 THEN N'Hà Nội, Việt Nam'
    WHEN 1 THEN N'TP. Hồ Chí Minh, Việt Nam'
    WHEN 2 THEN N'Đà Nẵng, Việt Nam'
    ...
    ELSE N'Việt Nam'
END

-- CurrentSemesterID: LUÔN có giá trị
IF @CurrentSemesterID IS NULL
    SET @CurrentSemesterID = @EnrollmentSemesterID
```

### 3. Mục tiêu học tập - Logic phân bố

```sql
-- Mỗi sinh viên có 3-7 mục tiêu
DECLARE @GoalCount INT = 3 + (ABS(CHECKSUM(s.StudentID)) % 5)

-- Phân bố GoalType
-- GPAGool: 20%
-- SubjectGoal: 35%
-- SkillGoal: 20%
-- SemesterGoal: 15%
-- CreditGoal: 10%

-- Phân bố Status
-- Pending: 20%
-- In-Progress: 50%
-- Completed: 25%
-- Cancelled: 5%
```

---

## 📝 CHECKLIST THỰC HIỆN

### Phase 1: Sinh viên & Cố vấn
- [ ] Tăng số lượng sinh viên lên 300
- [ ] Tăng số lượng cố vấn lên 40
- [ ] Đảm bảo KHÔNG NULL cho tất cả trường bắt buộc
- [ ] Phân bố Status đa dạng (4 trường hợp)
- [ ] Đảm bảo GraduationDate cho sinh viên đã tốt nghiệp

### Phase 2: StudentGrade
- [ ] Đảm bảo sinh viên đang học có điểm đầy đủ
- [ ] Sinh viên đã tốt nghiệp có điểm đầy đủ đến kỳ cuối
- [ ] Sinh viên tạm nghỉ/bảo lưu có điểm đến kỳ cuối học

### Phase 3: Mục tiêu học tập
- [ ] 100% sinh viên có mục tiêu học tập
- [ ] Mỗi sinh viên có 3-7 mục tiêu
- [ ] Đa dạng GoalType
- [ ] Đa dạng Status
- [ ] Đầy đủ GoalSnapshot, GoalTarget, GoalTask, GoalMilestone

### Phase 4: Student_Skill_Assessment
- [ ] Đảm bảo sinh viên có kỹ năng dựa trên môn học đã học
- [ ] Phân bố ProficiencyLevel hợp lý

---

## 🎯 KẾT QUẢ MONG ĐỢI

1. **Database đầy đủ:**
   - Không có NULL values không mong muốn
   - Tất cả trường bắt buộc có giá trị

2. **Đa dạng dữ liệu:**
   - 4 trường hợp Status khác nhau
   - Phân bố đều các khoa, ngành
   - Đa dạng mục tiêu học tập

3. **Sẵn sàng query:**
   - Có thể query theo bất kỳ tiêu chí nào
   - Dữ liệu hợp lý, phản ánh thực tế
   - Đủ dữ liệu để phân tích

4. **Tối ưu cho mục tiêu học tập:**
   - ~1500 mục tiêu học tập
   - Đa dạng GoalType, Status
   - Đầy đủ tracking (Snapshot, Task, Milestone)

---

**Kết thúc phương án**

