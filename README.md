# renewal
renewal

classic asp

<details>
<summary>학생 등록</summary>

```asp
<!--#include file="/inc/db.asp"-->
<%
Option Explicit

Dim action : action = LCase(Trim(Request("action") & ""))

If action = "insert" Then
    Dim student_name, student_age, phone_no, school_name, parent_phone_no
    Dim addr1, addr2, photo_url

    student_name    = Trim(Request.Form("student_name") & "")
    student_age     = Trim(Request.Form("student_age") & "")
    phone_no        = Trim(Request.Form("phone_no") & "")
    school_name     = Trim(Request.Form("school_name") & "")
    parent_phone_no = Trim(Request.Form("parent_phone_no") & "")
    addr1           = Trim(Request.Form("addr1") & "")
    addr2           = Trim(Request.Form("addr2") & "")
    photo_url       = Trim(Request.Form("photo_url") & "")

    If student_name = "" Then
        Response.Write "<script>alert('이름은 필수입니다.');history.back();</script>"
        Response.End
    End If

    Dim cmd : Set cmd = Server.CreateObject("ADODB.Command")
    With cmd
        .ActiveConnection = Conn
        .CommandType = 1
        .CommandText = "INSERT INTO dbo.Student " & _
                       "(student_name, student_age, phone_no, school_name, parent_phone_no, addr1, addr2, photo_url) " & _
                       "VALUES (?, ?, ?, ?, ?, ?, ?, ?)"

        .Parameters.Append .CreateParameter("@student_name", 202, 1, 50, student_name)
        If student_age = "" Then
            .Parameters.Append .CreateParameter("@student_age", 3, 1, , Null)
        Else
            .Parameters.Append .CreateParameter("@student_age", 3, 1, , CLng(student_age))
        End If
        .Parameters.Append .CreateParameter("@phone_no", 202, 1, 20, phone_no)
        .Parameters.Append .CreateParameter("@school_name", 202, 1, 100, school_name)
        .Parameters.Append .CreateParameter("@parent_phone_no", 202, 1, 20, parent_phone_no)
        .Parameters.Append .CreateParameter("@addr1", 202, 1, 200, addr1)
        .Parameters.Append .CreateParameter("@addr2", 202, 1, 200, addr2)
        .Parameters.Append .CreateParameter("@photo_url", 202, 1, 300, photo_url)

        .Execute
    End With
    Set cmd = Nothing

    Response.Redirect "student_list.asp"
End If
%>
```
</details>


<details> <summary>학생 목록 / 검색</summary>

```
<!--#include file="/inc/db.asp"-->
<%
Option Explicit

' ===== 검색 파라미터 =====
Dim q, ageMin, ageMax
q      = Trim(Request("q") & "")
ageMin = Trim(Request("ageMin") & "")
ageMax = Trim(Request("ageMax") & "")

' ===== 동적 WHERE 만들기 =====
Dim whereSql : whereSql = " WHERE 1=1 "

Dim cmd : Set cmd = Server.CreateObject("ADODB.Command")
cmd.ActiveConnection = Conn
cmd.CommandType = 1 ' adCmdText

' 키워드 검색 (여러 컬럼 LIKE)
If q <> "" Then
    whereSql = whereSql & " AND ( " & _
        "student_name LIKE ? OR " & _
        "phone_no LIKE ? OR " & _
        "school_name LIKE ? OR " & _
        "parent_phone_no LIKE ? OR " & _
        "addr1 LIKE ? OR " & _
        "addr2 LIKE ? " & _
    ")"

    Dim likeQ : likeQ = "%" & q & "%"

    cmd.Parameters.Append cmd.CreateParameter("@q1", 202, 1, 50,  likeQ)
    cmd.Parameters.Append cmd.CreateParameter("@q2", 202, 1, 20,  likeQ)
    cmd.Parameters.Append cmd.CreateParameter("@q3", 202, 1, 100, likeQ)
    cmd.Parameters.Append cmd.CreateParameter("@q4", 202, 1, 20,  likeQ)
    cmd.Parameters.Append cmd.CreateParameter("@q5", 202, 1, 200, likeQ)
    cmd.Parameters.Append cmd.CreateParameter("@q6", 202, 1, 200, likeQ)
End If

' 나이 최소
If ageMin <> "" Then
    whereSql = whereSql & " AND student_age >= ? "
    cmd.Parameters.Append cmd.CreateParameter("@ageMin", 3, 1, , CLng(ageMin))
End If

' 나이 최대
If ageMax <> "" Then
    whereSql = whereSql & " AND student_age <= ? "
    cmd.Parameters.Append cmd.CreateParameter("@ageMax", 3, 1, , CLng(ageMax))
End If

' ===== 최종 SQL =====
cmd.CommandText = _
    "SELECT TOP 200 student_id, student_name, student_age, phone_no, school_name, parent_phone_no, addr1, addr2, photo_url, created_at " & _
    "FROM dbo.Student " & whereSql & _
    "ORDER BY student_id DESC"

Dim rs : Set rs = cmd.Execute
%>

<!doctype html>
<html lang="ko">
<head>
  <meta charset="utf-8">
  <title>학생 목록</title>
  <style>
    table{border-collapse:collapse;width:100%}
    th,td{border:1px solid #ddd;padding:8px;font-size:14px}
    th{background:#f3f3f3}
    img{max-height:60px}
    .searchbox{padding:10px;border:1px solid #ddd;background:#fafafa;margin-bottom:12px}
    .searchbox input{padding:6px 8px}
  </style>
</head>
<body>
  <h2>학생 목록</h2>

  <div class="searchbox">
    <form method="get" action="student_list.asp">
      키워드:
      <input type="text" name="q" value="<%=Server.HTMLEncode(q)%>" placeholder="이름/전화/학교/주소">
      나이:
      <input type="number" name="ageMin" value="<%=Server.HTMLEncode(ageMin)%>" style="width:80px" placeholder="min">
      ~
      <input type="number" name="ageMax" value="<%=Server.HTMLEncode(ageMax)%>" style="width:80px" placeholder="max">

      <button type="submit">검색</button>
      <a href="student_list.asp">초기화</a>
      &nbsp; | &nbsp;
      <a href="student_new.asp">+ 학생 등록</a>
    </form>
  </div>

  <table>
    <thead>
      <tr>
        <th>ID</th>
        <th>이름</th>
        <th>나이</th>
        <th>전화</th>
        <th>학교</th>
        <th>부모님 전화</th>
        <th>주소</th>
        <th>사진</th>
        <th>등록일</th>
      </tr>
    </thead>
    <tbody>
    <%
    If rs.EOF Then
        Response.Write "<tr><td colspan='9' style='text-align:center;color:#777;padding:20px'>검색 결과 없음</td></tr>"
    Else
        Do Until rs.EOF
            Dim photoUrl : photoUrl = rs("photo_url") & ""
    %>
      <tr>
        <td><%=rs("student_id")%></td>
        <td><%=Server.HTMLEncode(rs("student_name") & "")%></td>
        <td><%=rs("student_age")%></td>
        <td><%=Server.HTMLEncode(rs("phone_no") & "")%></td>
        <td><%=Server.HTMLEncode(rs("school_name") & "")%></td>
        <td><%=Server.HTMLEncode(rs("parent_phone_no") & "")%></td>
        <td>
          <%=Server.HTMLEncode(rs("addr1") & "")%>
          <% If (rs("addr2") & "") <> "" Then %><br><%=Server.HTMLEncode(rs("addr2") & "")%><% End If %>
        </td>
        <td>
          <% If photoUrl <> "" Then %>
            <img src="<%=Server.HTMLEncode(photoUrl)%>" alt="photo">
          <% Else %>-<% End If %>
        </td>
        <td><%=rs("created_at")%></td>
      </tr>
    <%
            rs.MoveNext
        Loop
    End If
    %>
    </tbody>
  </table>

</body>
</html>

<%
rs.Close : Set rs = Nothing
Set cmd = Nothing
Conn.Close : Set Conn = Nothing
%>

```
</details>



asp.net

<details>
<summary>학생 등록</summary>

```asp
using System;
using System.ComponentModel.DataAnnotations;
using System.Data;
using System.Data.SqlClient; // Microsoft.Data.SqlClient 써도 됨
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace YourApp.Pages.Students
{
    public class CreateModel : PageModel
    {
        [BindProperty]
        public StudentCreateForm Form { get; set; } = new();

        public string? ErrorMessage { get; set; }

        public void OnGet()
        {
        }

        public async Task<IActionResult> OnPostAsync()
        {
            // 1) 기본 검증
            if (string.IsNullOrWhiteSpace(Form.StudentName))
            {
                ErrorMessage = "이름은 필수입니다.";
                return Page();
            }

            // 2) Insert
            try
            {
                using var conn = GetConn();
                await conn.OpenAsync();

                using var cmd = conn.CreateCommand();
                cmd.CommandType = CommandType.Text;
                cmd.CommandText = @"
INSERT INTO dbo.Student
(student_name, student_age, phone_no, school_name, parent_phone_no, addr1, addr2, photo_url)
VALUES
(@student_name, @student_age, @phone_no, @school_name, @parent_phone_no, @addr1, @addr2, @photo_url);
";

                cmd.Parameters.Add(new SqlParameter("@student_name", SqlDbType.NVarChar, 50)
                {
                    Value = Form.StudentName.Trim()
                });

                // 나이: null 허용
                cmd.Parameters.Add(new SqlParameter("@student_age", SqlDbType.Int)
                {
                    Value = Form.StudentAge.HasValue ? Form.StudentAge.Value : (object)DBNull.Value
                });

                cmd.Parameters.Add(new SqlParameter("@phone_no", SqlDbType.NVarChar, 20)
                {
                    Value = (object?)Form.PhoneNo?.Trim() ?? DBNull.Value
                });

                cmd.Parameters.Add(new SqlParameter("@school_name", SqlDbType.NVarChar, 100)
                {
                    Value = (object?)Form.SchoolName?.Trim() ?? DBNull.Value
                });

                cmd.Parameters.Add(new SqlParameter("@parent_phone_no", SqlDbType.NVarChar, 20)
                {
                    Value = (object?)Form.ParentPhoneNo?.Trim() ?? DBNull.Value
                });

                cmd.Parameters.Add(new SqlParameter("@addr1", SqlDbType.NVarChar, 200)
                {
                    Value = (object?)Form.Addr1?.Trim() ?? DBNull.Value
                });

                cmd.Parameters.Add(new SqlParameter("@addr2", SqlDbType.NVarChar, 200)
                {
                    Value = (object?)Form.Addr2?.Trim() ?? DBNull.Value
                });

                cmd.Parameters.Add(new SqlParameter("@photo_url", SqlDbType.NVarChar, 300)
                {
                    Value = (object?)Form.PhotoUrl?.Trim() ?? DBNull.Value
                });

                await cmd.ExecuteNonQueryAsync();

                // 성공 시 목록으로
                return RedirectToPage("/Students/Index");
            }
            catch (Exception ex)
            {
                // 운영에서는 로깅 처리 권장
                ErrorMessage = "저장 중 오류가 발생했습니다: " + ex.Message;
                return Page();
            }
        }

        // ===== 폼 DTO =====
        public class StudentCreateForm
        {
            [Required]
            [StringLength(50)]
            public string StudentName { get; set; } = "";

            public int? StudentAge { get; set; }

            [StringLength(20)]
            public string? PhoneNo { get; set; }

            [StringLength(100)]
            public string? SchoolName { get; set; }

            [StringLength(20)]
            public string? ParentPhoneNo { get; set; }

            [StringLength(200)]
            public string? Addr1 { get; set; }

            [StringLength(200)]
            public string? Addr2 { get; set; }

            [StringLength(300)]
            public string? PhotoUrl { get; set; }
        }

        // ===== DB 연결 (여긴 프로젝트에 맞게 구현되어 있다고 치면 됨) =====
        private SqlConnection GetConn()
        {
            // 예시: return new SqlConnection(_config.GetConnectionString("Default"));
            throw new NotImplementedException("GetConn()을 프로젝트 환경에 맞게 연결하세요.");
        }
    }
}

```
</details>


<details> <summary>학생 목록 / 검색</summary>

```
using System;
using System.Collections.Generic;
using System.Data;
using System.Data.SqlClient;
using System.Text;
using System.Threading.Tasks;
using Microsoft.AspNetCore.Mvc;
using Microsoft.AspNetCore.Mvc.RazorPages;

namespace YourApp.Pages.Students
{
    public class IndexModel : PageModel
    {
        // ===== QueryString 바인딩 =====
        [BindProperty(SupportsGet = true)]
        public string? Q { get; set; }

        [BindProperty(SupportsGet = true)]
        public int? AgeMin { get; set; }

        [BindProperty(SupportsGet = true)]
        public int? AgeMax { get; set; }

        public List<StudentRow> Students { get; private set; } = new();

        public async Task OnGetAsync()
        {
            using var conn = GetConn();
            await conn.OpenAsync();

            using var cmd = conn.CreateCommand();
            cmd.CommandType = CommandType.Text;

            // ===== 동적 WHERE + 파라미터 =====
            var where = new StringBuilder(" WHERE 1=1 ");

            if (!string.IsNullOrWhiteSpace(Q))
            {
                where.Append(@"
 AND (
    student_name LIKE @q OR
    phone_no LIKE @q OR
    school_name LIKE @q OR
    parent_phone_no LIKE @q OR
    addr1 LIKE @q OR
    addr2 LIKE @q
 )
");
                cmd.Parameters.Add(new SqlParameter("@q", SqlDbType.NVarChar, 200)
                {
                    Value = "%" + Q.Trim() + "%"
                });
            }

            if (AgeMin.HasValue)
            {
                where.Append(" AND student_age >= @ageMin ");
                cmd.Parameters.Add(new SqlParameter("@ageMin", SqlDbType.Int) { Value = AgeMin.Value });
            }

            if (AgeMax.HasValue)
            {
                where.Append(" AND student_age <= @ageMax ");
                cmd.Parameters.Add(new SqlParameter("@ageMax", SqlDbType.Int) { Value = AgeMax.Value });
            }

            cmd.CommandText = $@"
SELECT TOP 200
    student_id,
    student_name,
    student_age,
    phone_no,
    school_name,
    parent_phone_no,
    addr1,
    addr2,
    photo_url,
    created_at
FROM dbo.Student
{where}
ORDER BY student_id DESC;
";

            using var rd = await cmd.ExecuteReaderAsync();

            while (await rd.ReadAsync())
            {
                Students.Add(new StudentRow
                {
                    StudentId = rd.GetInt32(0),
                    StudentName = rd.IsDBNull(1) ? "" : rd.GetString(1),
                    StudentAge = rd.IsDBNull(2) ? (int?)null : rd.GetInt32(2),
                    PhoneNo = rd.IsDBNull(3) ? null : rd.GetString(3),
                    SchoolName = rd.IsDBNull(4) ? null : rd.GetString(4),
                    ParentPhoneNo = rd.IsDBNull(5) ? null : rd.GetString(5),
                    Addr1 = rd.IsDBNull(6) ? null : rd.GetString(6),
                    Addr2 = rd.IsDBNull(7) ? null : rd.GetString(7),
                    PhotoUrl = rd.IsDBNull(8) ? null : rd.GetString(8),
                    CreatedAt = rd.IsDBNull(9) ? (DateTime?)null : rd.GetDateTime(9),
                });
            }
        }

        public class StudentRow
        {
            public int StudentId { get; set; }
            public string StudentName { get; set; } = "";
            public int? StudentAge { get; set; }
            public string? PhoneNo { get; set; }
            public string? SchoolName { get; set; }
            public string? ParentPhoneNo { get; set; }
            public string? Addr1 { get; set; }
            public string? Addr2 { get; set; }
            public string? PhotoUrl { get; set; }
            public DateTime? CreatedAt { get; set; }
        }

        private SqlConnection GetConn()
        {
            throw new NotImplementedException("GetConn()을 프로젝트 환경에 맞게 연결하세요.");
        }
    }
}


```
</details>




spring data jpa 
<details>
<summary>컨트롤러</summary>

```asp
package com.example.demo.student;

import com.example.demo.student.dto.StudentCreateRequest;
import com.example.demo.student.dto.StudentResponse;
import org.springframework.data.domain.*;
import org.springframework.web.bind.annotation.*;

@RestController
@RequestMapping("/api/students")
public class StudentController {

    private final StudentService service;

    public StudentController(StudentService service) {
        this.service = service;
    }

    // 등록
    @PostMapping
    public Long create(@RequestBody StudentCreateRequest req) {
        return service.create(req);
    }

    // 조회/검색
    @GetMapping
    public Page<StudentResponse> list(
            @RequestParam(required = false) String name,
            @RequestParam(required = false) String school,
            @RequestParam(required = false) Integer ageMin,
            @RequestParam(required = false) Integer ageMax,
            @RequestParam(defaultValue = "0") int page,
            @RequestParam(defaultValue = "20") int size
    ) {
        Pageable pageable = PageRequest.of(page, size);
        return service.search(name, school, ageMin, ageMax, pageable);
    }

}

```
</details>

<details>
<summary>서비스</summary>

```asp
package com.example.demo.student;

import com.example.demo.student.dto.StudentCreateRequest;
import com.example.demo.student.dto.StudentResponse;
import org.springframework.data.domain.*;
import org.springframework.stereotype.Service;
import org.springframework.transaction.annotation.Transactional;

@Service
@Transactional
public class StudentService {

    private final StudentRepository repo;

    public StudentService(StudentRepository repo) {
        this.repo = repo;
    }

    public Long create(StudentCreateRequest req) {
        Student s = Student.create(
                req.getStudentName(),
                req.getStudentAge(),
                req.getPhoneNo(),
                req.getSchoolName(),
                req.getParentPhoneNo(),
                req.getAddr1(),
                req.getAddr2(),
                req.getPhotoUrl()
        );
        return repo.save(s).getStudentId();
    }

    @Transactional(readOnly = true)
    public Page<StudentResponse> search(String q, Integer ageMin, Integer ageMax, Pageable pageable) {
        return repo.search(q, ageMin, ageMax, pageable).map(StudentResponse::from);
    }
}

```
</details>


<details>
<summary>레포지토리(구현)</summary>

```asp
import com.querydsl.core.BooleanBuilder;
import com.querydsl.jpa.impl.JPAQueryFactory;
import org.springframework.data.domain.*;
import jakarta.persistence.EntityManager;

import static com.example.demo.student.QStudent.student;

public class StudentRepositoryImpl implements StudentRepositoryCustom {

    private final JPAQueryFactory queryFactory;

    public StudentRepositoryImpl(EntityManager em) {
        this.queryFactory = new JPAQueryFactory(em);
    }

    @Override
    public Page<Student> search(
            String name,
            String school,
            Integer ageMin,
            Integer ageMax,
            Pageable pageable
    ) {

        BooleanBuilder where = new BooleanBuilder();

        // ===== Classic ASP의 IF + AND 와 동일 =====
        if (name != null && !name.isBlank()) {
            where.and(student.studentName.contains(name));
        }

        if (school != null && !school.isBlank()) {
            where.and(student.schoolName.contains(school));
        }

        if (ageMin != null) {
            where.and(student.studentAge.goe(ageMin));
        }

        if (ageMax != null) {
            where.and(student.studentAge.loe(ageMax));
        }

        // ===== 데이터 조회 =====
        var content = queryFactory
                .selectFrom(student)
                .where(where)
                .orderBy(student.studentId.desc())
                .offset(pageable.getOffset())
                .limit(pageable.getPageSize())
                .fetch();

        // ===== 전체 카운트 =====
        long total = queryFactory
                .select(student.count())
                .from(student)
                .where(where)
                .fetchOne();

        return new PageImpl<>(content, pageable, total);
    }
}


```
</details>




where문 asp -> jpa
<details>
<summary>조건 1개</summary>

```asp
Dim studentId
studentId = CLng(Request("student_id"))

sql = "SELECT * FROM dbo.Student WHERE student_id = ?"

cmd.Parameters.Append cmd.CreateParameter("@student_id", 3, 1, , studentId)

Set rs = cmd.Execute

If rs.EOF Then
    Response.Write "학생 없음"
    Response.End
End If

' 사용 예
student_name = rs("student_name")
student_age  = rs("student_age")


============================================================================================================
============================================================================================================

controller/
└─ StudentController.java

service/
└─ StudentService.java

repository/
└─ StudentRepository.java

dto/
└─ StudentResponse.java   // record



컨트롤러 (StudentController.java)
@GetMapping("/{studentId}")
public StudentResponse getOne(@PathVariable Long studentId) {
    return service.findOne(studentId);
}

서비스 (StudentService.java)
@Transactional(readOnly = true)
public StudentResponse findOne(Long studentId) {
    Student s = repo.findById(studentId)
            .orElseThrow(() -> new IllegalArgumentException("학생 없음"));
    return StudentResponse.from(s);
}

레포지토리 (StudentRepository.java)
public interface StudentRepository extends JpaRepository<Student, Long> { }

응답dto (StudentResponse.java)
@Getter
@AllArgsConstructor
public class StudentResponse {

    private final Long studentId;
    private final String studentName;
    private final Integer studentAge;
    private final String phoneNo;
    private final String schoolName;
    private final String parentPhoneNo;
    private final String addr1;
    private final String addr2;
    private final String photoUrl;
    private final LocalDateTime createdAt;

    public static StudentResponse from(Student s) {
        return new StudentResponse(
                s.getStudentId(),
                s.getStudentName(),
                s.getStudentAge(),
                s.getPhoneNo(),
                s.getSchoolName(),
                s.getParentPhoneNo(),
                s.getAddr1(),
                s.getAddr2(),
                s.getPhotoUrl(),
                s.getCreatedAt()
        );
    }
}
응답 dto 자바 17 이상 (record)
public record StudentResponse(
        Long studentId,
        String studentName,
        Integer studentAge,
        String phoneNo,
        String schoolName,
        String parentPhoneNo,
        String addr1,
        String addr2,
        String photoUrl,
        LocalDateTime createdAt
) {
    public static StudentResponse from(Student s) {
        return new StudentResponse(
                s.getStudentId(),
                s.getStudentName(),
                s.getStudentAge(),
                s.getPhoneNo(),
                s.getSchoolName(),
                s.getParentPhoneNo(),
                s.getAddr1(),
                s.getAddr2(),
                s.getPhotoUrl(),
                s.getCreatedAt()
        );
    }
}
```
</details>



<details>
<summary>조건 여러 개</summary>

```asp
whereSql = "WHERE 1=1"

If name <> "" Then
    whereSql = whereSql & " AND student_name LIKE '%" & name & "%'"
End If

If ageMin <> "" Then
    whereSql = whereSql & " AND student_age >= " & ageMin
End If

============================================================================================================
============================================================================================================
BooleanBuilder where = new BooleanBuilder();

if (name != null && !name.isBlank()) {
    where.and(student.studentName.contains(name));
}

if (ageMin != null) {
    where.and(student.studentAge.goe(ageMin));
}


```
</details>




<details> <summary>간단한 프로시저 -> jpa</summary>

```
ALTER PROCEDURE [dbo].[airpod_proc_delete]
	-- Add the parameters for the stored procedure here
	@user_id varchar(20),
	@airpod_seq int
AS
BEGIN
	-- SET NOCOUNT ON added to prevent extra result sets from
	-- interfering with SELECT statements.
	SET NOCOUNT ON;
	
    BEGIN
        RETURN 1;
    END

    DELETE FROM tb_airpod
    WHERE airpod_seq = @airpod_seq;

    RETURN 0;
END


============================================================================================================
============================================================================================================
컨트롤러
@DeleteMapping("/{airpodSeq}")
public void delete(@PathVariable int airpodSeq) { service.delete(airpodSeq); }

서비스
@Transactional
public void delete(int airpodSeq) { repo.deleteById(airpodSeq); }

```
</details>

복잡한 프로시저는 mybatis







