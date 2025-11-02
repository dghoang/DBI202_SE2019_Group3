# GIẢI THÍCH VÀ CẢI TIẾN DỮ LIỆU

**Ngày:** 02/11/2024

---

## 📚 GIẢI THÍCH CÁC TRƯỜNG QUAN TRỌNG

### 1. PrerequisiteText trong Curriculum

**Định nghĩa:**
- `PrerequisiteText` là text mô tả các môn tiên quyết của một môn học trong khung chương trình
- Kiểu: `NVARCHAR(500) NULL`

**Mục đích:**
- **Hiển thị nhanh**: Không cần JOIN với SubjectPrerequisite để biết môn tiên quyết
- **Denormalization nhẹ**: Trade-off giữa storage và query performance
- **User-friendly**: Text dễ đọc cho người dùng

**Cách populate:**
- Được tự động cập nhật từ `SubjectPrerequisite` table
- Format: "Tên môn 1 (điểm >= 5.0), Tên môn 2 (điểm >= 6.0)"
- Nếu không có môn tiên quyết → NULL

**Ví dụ:**
```
SubjectID = 'PRJ301'
PrerequisiteText = 'Object-Oriented Programming (điểm >= 5.0), Data Structures & Algorithms (điểm >= 6.0)'
```

**Lý do thêm:**
- **Performance**: Query nhanh hơn khi chỉ cần xem danh sách môn tiên quyết
- **Simplicity**: Không cần JOIN phức tạp cho display
- **Flexibility**: Có thể edit manual nếu cần

---

### 2. AdvisorCapacity.CurrentSlots

**Định nghĩa:**
- `CurrentSlots` là số lượng slot (sinh viên) hiện tại đang được cố vấn quản lý trong một kỳ học
- Kiểu: `INT NOT NULL DEFAULT 0`
- CHECK constraint: `CurrentSlots >= 0`
- Trigger: `CurrentSlots <= MaxSlots`

**Mục đích:**
- **Track capacity**: Theo dõi số lượng sinh viên hiện tại mà cố vấn đang phụ trách
- **Workload management**: Quản lý khối lượng công việc của cố vấn
- **Assignment logic**: Đảm bảo không gán quá nhiều sinh viên cho một cố vấn

**So sánh với MaxSlots:**
- `MaxSlots`: Số lượng slot tối đa cố vấn có thể phụ trách (ví dụ: 30-50)
- `CurrentSlots`: Số lượng slot hiện tại đang sử dụng (0 đến MaxSlots)

**Logic:**
- Khi gán sinh viên cho cố vấn → `CurrentSlots++`
- Khi chuyển sinh viên → `CurrentSlots--`
- Khi kỳ học mới bắt đầu → Reset hoặc tính lại dựa trên assignments

**Ví dụ:**
```
AdvisorID = 'GV00123', SemesterID = 'FA25'
MaxSlots = 35
CurrentSlots = 28  → Còn 7 slot trống
```

**Cách populate dữ liệu:**
- Ban đầu: `CurrentSlots = 0` (chưa gán sinh viên)
- Sau khi gán: `CurrentSlots = COUNT(StudentAdvisorAssignment WHERE AdvisorID = ... AND SemesterID = ...)`

---

## 🎯 THANG ĐIỂM ĐÁNH GIÁ SKILL

### SkillLevel trong Subject_Skill_Mapping

**Định nghĩa:**
- `SkillLevel` là mức độ phát triển kỹ năng mà môn học này cung cấp
- Kiểu: `TINYINT NULL CHECK (SkillLevel BETWEEN 1 AND 5)`
- NULL = Không xác định

**Thang điểm:**
- **Level 1**: Cơ bản (Basic) - Giới thiệu, làm quen
- **Level 2**: Trung bình (Intermediate) - Hiểu và áp dụng cơ bản
- **Level 3**: Trung cấp (Advanced Beginner) - Áp dụng tốt
- **Level 4**: Nâng cao (Advanced) - Thành thạo
- **Level 5**: Chuyên sâu (Expert) - Tinh thông

**Ví dụ:**
```
DBI202 (Database) → Skill 'SQL Programming' → SkillLevel = 3
PRJ301 (Project Management) → Skill 'Project Management' → SkillLevel = 4
```

---

### ProficiencyLevel trong Student_Skill_Assessment

**Định nghĩa:**
- `ProficiencyLevel` là mức độ thành thạo hiện tại của sinh viên với một kỹ năng
- Kiểu: `INT NOT NULL DEFAULT 1 CHECK (ProficiencyLevel BETWEEN 1 AND 5)`

**Thang điểm:**
- **Level 1**: Mới bắt đầu
- **Level 2**: Cơ bản
- **Level 3**: Trung bình
- **Level 4**: Tốt
- **Level 5**: Xuất sắc

**Logic tính toán:**
- Dựa trên điểm trung bình của các môn học liên quan
- Điểm >= 8.0 → Level 4-5
- Điểm >= 6.5 → Level 3
- Điểm < 6.5 → Level 1-2

---

## 🔧 CẢI TIẾN DỮ LIỆU

### 1. Subject_Skill_Mapping - Tránh dữ liệu liên tục giống nhau

**Vấn đề hiện tại:**
- Có thể có nhiều môn học liên quan đến cùng một skill với cùng SkillLevel

**Giải pháp:**
- Phân bố SkillLevel đa dạng: 1, 2, 3, 4, 5
- Mỗi môn học có thể phát triển skill ở nhiều mức độ khác nhau
- Một skill có thể được phát triển bởi nhiều môn học ở các level khác nhau

**Logic cải tiến:**
```sql
-- Phân bố SkillLevel đa dạng
SkillLevel = CASE (ABS(CHECKSUM(SubjectID + SkillID)) % 5)
    WHEN 0 THEN 1  -- 20% Level 1
    WHEN 1 THEN 2  -- 20% Level 2
    WHEN 2 THEN 3  -- 20% Level 3
    WHEN 3 THEN 4  -- 20% Level 4
    ELSE 5         -- 20% Level 5
END
```

---

### 2. AdvisorCapacity.CurrentSlots - Tính toán từ thực tế

**Cải tiến:**
- Thay vì để CurrentSlots = 0, tính từ `StudentAdvisorAssignment`
- Phân bố đều: Một số advisor có ít sinh viên, một số có nhiều (nhưng < MaxSlots)

**Logic:**
```sql
CurrentSlots = 
    CASE 
        WHEN COUNT(StudentAdvisorAssignment) > 0 THEN 
            COUNT(StudentAdvisorAssignment WHERE AdvisorID = ... AND SemesterID = ...)
        ELSE ABS(CHECKSUM(...) % (MaxSlots * 0.8))  -- 0-80% MaxSlots
    END
```

---

### 3. GoalComment - Đảm bảo có dữ liệu

**Vấn đề:** Có thể NULL

**Giải pháp:**
- Mỗi goal có ít nhất 1 comment (từ student hoặc advisor)
- Phân bố: 60% goals có comment
- Đa dạng: Comment từ student, advisor, với timestamp khác nhau

---

### 4. GoalDependency - Đảm bảo có dữ liệu

**Vấn đề:** Có thể NULL

**Giải pháp:**
- 30% goals có dependency
- Đa dạng: Prerequisite, Co-requisite, Related
- Đảm bảo không circular dependency

---

### 5. GoalTarget - Đầy đủ MinimumValue, MaximumValue

**Vấn đề:** MinimumValue, MaximumValue = NULL

**Giải pháp:**
- MinimumValue: 70% của TargetValue
- MaximumValue: 130% của TargetValue
- Tạo range hợp lý cho mục tiêu

---

## 📊 KẾ HOẠCH CẢI TIẾN

1. ✅ Subject_Skill_Mapping: SkillLevel phân bố đều 1-5
2. ✅ Student_Skill_Assessment: ProficiencyLevel đa dạng, Evidence đầy đủ
3. ✅ AdvisorCapacity: CurrentSlots tính từ assignments thực tế
4. ✅ GoalComment: 60% goals có comment, đa dạng
5. ✅ GoalDependency: 30% goals có dependency
6. ✅ GoalTarget: MinimumValue, MaximumValue đầy đủ
7. ✅ PrerequisiteText: Cập nhật từ SubjectPrerequisite

---

**Kết thúc giải thích**

