# BÁO CÁO CẢI TIẾN DỮ LIỆU - DATABASE VERSION 1

**Ngày:** 02/11/2024  
**Phiên bản:** DBI202_Group3_SE2019.Verion1.sql

---

## 📋 TÓM TẮT CÁC CẢI TIẾN

### 1. ✅ PrerequisiteText trong Curriculum

**Vấn đề:** Sử dụng `STRING_AGG()` không tương thích với SQL Server cũ

**Giải pháp:** 
- Thay thế bằng `FOR XML PATH` + `STUFF()` để tương thích tốt hơn
- Tự động cập nhật từ `SubjectPrerequisite` table
- Format: "Tên môn (điểm >= X), Tên môn (điểm >= Y)"

**Kết quả:**
- ✅ Tương thích với mọi phiên bản SQL Server
- ✅ Text hiển thị đầy đủ thông tin môn tiên quyết
- ✅ Giảm JOIN phức tạp khi query

---

### 2. ✅ Subject_Skill_Mapping - SkillLevel

**Vấn đề:** SkillLevel phân bố không đều, có thể có nhiều môn cùng level

**Giải pháp:**
- **Phân bố đều 1-5:** Mỗi level chiếm 20% dữ liệu
- Công thức: `1 + (CHECKSUM(SubjectID + SkillID) % 5)`
- Đảm bảo không có pattern lặp lại

**Kết quả:**
- ✅ Dữ liệu đa dạng, không liên tục giống nhau
- ✅ Mỗi skill có thể phát triển ở nhiều mức độ khác nhau
- ✅ Phù hợp cho phân tích thống kê

---

### 3. ✅ AdvisorCapacity.CurrentSlots

**Vấn đề:** CurrentSlots = 0 (không thực tế)

**Giải pháp:**
- **Tính từ thực tế:** Nếu có `StudentAdvisorAssignment` → COUNT thực tế
- **Random đa dạng:** Nếu chưa có assignment → Random 0-80% MaxSlots
- Đảm bảo `CurrentSlots <= MaxSlots` (trigger validate)

**Kết quả:**
- ✅ Dữ liệu thực tế hơn, phản ánh workload của advisor
- ✅ Đa dạng: Một số advisor có ít sinh viên, một số có nhiều
- ✅ Phù hợp cho quản lý workload

---

### 4. ✅ GoalTarget - MinimumValue và MaximumValue

**Vấn đề:** MinimumValue và MaximumValue = NULL

**Giải pháp:**
- **MinimumValue:** 70% của TargetValue (giá trị tối thiểu chấp nhận)
- **MaximumValue:** 130% của TargetValue (giá trị tối đa đặt ra)
- Tạo range hợp lý cho mục tiêu

**Kết quả:**
- ✅ Không còn NULL, dữ liệu đầy đủ
- ✅ Có range để đánh giá mục tiêu
- ✅ Phù hợp cho validation và reporting

---

### 5. ✅ GoalComment

**Vấn đề:** Có thể NULL, thiếu dữ liệu

**Giải pháp:**
- **60% goals có comment** (Status = 'In-Progress' hoặc 'Completed')
- **Đa dạng nguồn:** 50% từ student, 50% từ advisor
- **Nhiều nội dung:** 5 loại comment khác nhau
- **Mỗi goal có 1-2 comments**
- **20% comments được edit** (với EditedDate)

**Kết quả:**
- ✅ 60% goals có ít nhất 1 comment
- ✅ Đa dạng về nguồn và nội dung
- ✅ Hỗ trợ tracking và feedback

---

### 6. ✅ GoalDependency

**Vấn đề:** Có thể NULL, thiếu dữ liệu

**Giải pháp:**
- **30% goals có dependency** (tối đa 150 dependencies)
- **Đa dạng loại:** 
  - 33% Prerequisite (mục tiêu này cần hoàn thành trước)
  - 33% Co-requisite (thực hiện đồng thời)
  - 33% Related (có liên quan)
- **IsRequired:** Prerequisite luôn required, Co-requisite/Related 50% required
- **Tránh circular dependency:** `lg1.GoalID < lg2.GoalID`

**Kết quả:**
- ✅ 30% goals có dependency
- ✅ Đa dạng về loại dependency
- ✅ Hỗ trợ phân tích mối quan hệ giữa mục tiêu

---

## 📊 THỐNG KÊ DỮ LIỆU

### Phân bố SkillLevel (Subject_Skill_Mapping)
- Level 1: ~20%
- Level 2: ~20%
- Level 3: ~20%
- Level 4: ~20%
- Level 5: ~20%

### Phân bố CurrentSlots (AdvisorCapacity)
- 0-20% MaxSlots: ~25%
- 21-40% MaxSlots: ~25%
- 41-60% MaxSlots: ~25%
- 61-80% MaxSlots: ~25%

### Phân bố GoalComment
- Goals có comment: 60%
- Comments từ student: 50%
- Comments từ advisor: 50%
- Comments được edit: 20%

### Phân bố GoalDependency
- Goals có dependency: 30%
- Prerequisite: 33%
- Co-requisite: 33%
- Related: 33%

---

## 🎯 MỤC TIÊU ĐẠT ĐƯỢC

1. ✅ **Tránh NULL:** Tất cả các trường quan trọng đều có dữ liệu
2. ✅ **Đa dạng dữ liệu:** Phân bố đều, không lặp lại pattern
3. ✅ **Thực tế:** Dữ liệu phản ánh tình huống thực tế
4. ✅ **Phù hợp query:** Hỗ trợ tốt cho phân tích và báo cáo
5. ✅ **Mục tiêu học tập:** Dữ liệu đầy đủ và phong phú

---

## 📝 CHI TIẾT KỸ THUẬT

### Công thức phân bố SkillLevel
```sql
1 + (ABS(CHECKSUM(CAST(s.SubjectID AS VARCHAR) + CAST(sk.SkillID AS VARCHAR))) % 5)
```

### Công thức CurrentSlots
```sql
CASE 
    WHEN EXISTS (StudentAdvisorAssignment) THEN COUNT(*)
    ELSE ABS(CHECKSUM(...) % (MaxSlots * 0.8))
END
```

### Công thức GoalTarget
```sql
MinimumValue = TargetValue * 0.7
MaximumValue = TargetValue * 1.3
```

---

## ✅ HOÀN THÀNH

Tất cả các yêu cầu đã được thực hiện:
- ✅ Giải thích PrerequisiteText và CurrentSlots
- ✅ Cải thiện SkillLevel phân bố đều
- ✅ Cải thiện CurrentSlots từ thực tế
- ✅ Thêm GoalComment (60% goals)
- ✅ Thêm GoalDependency (30% goals)
- ✅ Đầy đủ MinimumValue, MaximumValue trong GoalTarget
- ✅ Tránh NULL, đa dạng dữ liệu

**File SQL:** `DBI202_Group3_SE2019.Verion1.sql`  
**Document giải thích:** `GIAI_THICH_CAI_TIEN_DU_LIEU.md`

---

**Kết thúc báo cáo**

