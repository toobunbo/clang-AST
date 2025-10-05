# Clang/AST Libary 

## Nhóm thư viện nền tảng AST
### "clang/AST/Decl.h"
-   Chức năng: (Declaration) Sau khi Clang tạo ra cây AST, ta sẽ làm việc với các con trỏ trỏ đến các đối tượng này (FunctionDecl *, RecordDecl *...) để đọc thông tin từ chúng.
``` code C
// DemoDecl.h
struct Point { int x; }; #RecordDecl
int global_var; # VarDecl

int distance(Point p) {  FunctionDecl
    int temp;
    return 0;
}
```
### "clang/AST/type.h"
- Chức năng: hỗ trợ phân tích kiểu dữ liệu chi tiết. Ví dụ getType() lấy về một đối tượng QualType chứa PointerType(), PointeeType(),...
```
const char* name;
name->getType() // Tạo một QualType đại diện cho char * name 

isPointerType() -> true
getPointeeType().getAsString() -> "const char"
getPointeeType().isConstQualified() -> true 
```

### "clang/AST/RecursiveASTVisitor.h"
- Chức năng: cung cấp template class supervippro **RecursiveASTVisitor**. 
```
struct LocalVisitor : public RecursiveASTVisitor<LocalVisitor> {
    bool VisitFunctionDecl(FunctionDecl *FD) { //body }
    bool VisitRecordDecl(RecordDecl *RD) {}
    bool VisitTypedefDecl(TypedefDecl *TD) {}
    bool VisitEnumDecl(EnumDecl *ED) {}
}

Bắt đầu với một nút trong cây AST (Futag bắt đầu bằng TUD): {
    1. Kiểm tra loại nút (FuncitonDecl,...)
    2. Kiểm tra xem LocalVisitor có định nghĩa `VisitFunctionDecl` không
    3. Nếu có kích hoạt VisitFunctionDecl
    4. Tiếp tục Đi xuống: Vì kết quả là true, RecursiveASTVisitor sẽ tự động lấy tất cả các nút con của FunctionDecl (ví dụ: các ParmVarDecl, CompoundStmt) và lặp lại từ bước 2.1 cho mỗi nút con đó.
}
```
### "clang/ASTMatchers/ASTMatchers.h" và ASTFinder.h
###  "clang/AST/ODRHash.h"

## Framework phân tích tĩnh
### "clang/StaticAnalyzer/Core/Checker.h"

` Quan trọng nhất `

- Chức năng: bất kỳ lớp nào kế thừa từ Checker đều được Clang Static Analyzer công nhận là một plugin phân tích hợp lệ.
- Clang sẽ tự động gọi các hàm callback tương ứng cho bạn nếu được xem là plugin hợp lệ.
```
// Đăng ký sự kiện "kiểm tra SAU KHI đi qua một câu lệnh"
class MyIfStmtChecker : public Checker<check::PostStmt<IfStmt>> {
public:
    // 2. Lợi ích: Clang sẽ TỰ ĐỘNG gọi hàm này cho MỌI câu lệnh `if`
    void checkPostStmt(const IfStmt *IS, CheckerContext &C) const {
        // ... logic phân tích câu lệnh if của bạn ở đây ...
        // Bạn không cần tự mình đi tìm các câu lệnh if.
    }
};
```
## Bộ công cụ xây dựng tool

