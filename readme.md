# Clang/AST Libary 
Note thư viện lại để triển khai thuận toán, đọc k hiểu thì lên đọc docs.
<img width="540" height="406" alt="image" src="https://github.com/user-attachments/assets/57667e42-7510-49ce-b68a-699b49a79929" />

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
- FuTag:
```
class FutagAnalyzer : public Checker<check::ASTDecl<TranslationUnitDecl>> {
    ...
    void checkASTDecl(const TranslationUnitDecl *TUD, AnalysisManager &Mgr,
BugReporter &BR) const;
}

TranslationUnitDecl là nút gốc của toàn bộ cây AST
Thông báo rõ checkASTDecl() được gọi một lần cho mỗi Translation Unit (file 
.c/.cpp) trong dự án, không phải một lần duy nhất cho toàn bộ thư viện.

```

### "clang/StaticAnalyzer/Core/CheckerManager.h" 
- Chức năng: thu thập các Checker được đăng ký (register...check).

```
void ento::registerFutagAnalyzer(CheckerManager &Mgr) {
    Mgr.registerChecker<FutagAnalyzer>();
}
```

### "clang/StaticAnalyzer/Core/PathSensitive/AnalysisManager.h"
- Dễ hiểu thì "AnalysisManager" là thằng quyết lưu Function,... cho Checker 

```
int add(int a, int b);
```
- Khi Clang Static Analyzer được dùng để phân tích hàm add():
    1. Tạo 1 đối tượng AnalysisManager (gọi tắt là Mrg)
    2. Ra lệnh cho Mrg quản lý toàn bộ mọi thứ liên quan đến hàm add() này

- Checker như FutagAnalyzer nhận vào Mgr và trích xuất thông tin để xử lý:
    - Mgr.getCFG(func)
    - Mgr.getASTContext()
    - ...

## Thư viện công cụ hỗ trợ (Tool & Basic)
### "clang/Basic/SourceManager.h"

- Chức năng: "SourceManager" là "Người Quản lý Bản đồ". Header này định nghĩa lớp SourceManager, có nhiệm vụ quản lý tất cả thông tin liên quan đến vị trí của mã nguồn. Nó biết chính xác mỗi ký tự, mỗi khai báo đến từ file nào, dòng bao nhiêu, cột bao nhiêu.
- Ứng dụng chính để lọc các hàm hệ thống...
```
bool checkSystemHeader(FD *func) { // Giả sử func trỏ đén printf <stdio..h>
    SourceLocation loc = func->getLocation(); // Lấy "tọa độ" của hàm
    SourceManager &sm = Mgr.getSourceManager();
    bool is_system = sm.isInSystemHeader(loc); // Hỏi "bản đồ"
    std::string filename = sm.getFilename(loc).str(); // Lấy tên file từ "tọa độ")
}
```

### "clang/Analysis/AnalysisDeclContext.h"
### "clang/Tooling/Tooling.h"
```
./my_tool my_file.c -- -I/some/path

CommonOptionsParser OptionsParser(...);
ClangTool Tool(OptionsParser.getCompilations(), 
               OptionsParser.getSourcePathList());
Tool.run(...);
```


