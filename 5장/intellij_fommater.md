
# ✅ **Google Java Style – IntelliJ Code Style XML**

> ✔ 들여쓰기 2 spaces

> ✔ 최대 줄 길이 100

> ✔ import 와일드카드 금지

> ✔ 구글 자바 스타일의 모든 기본 규칙 반영

파일명 예시: **GoogleJavaStyle.xml**

```xml
<code_scheme name="GoogleJavaStyle" version="173">
  <option name="CLASS_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
  <option name="NAMES_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
  <option name="RIGHT_MARGIN" value="100" />
  <option name="WRAP_WHEN_TYPING_REACHES_RIGHT_MARGIN" value="true" />

  <JavaCodeStyleSettings>
    <option name="DO_NOT_IMPORT_INNER_CLASSES" value="true" />
    <option name="CLASS_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
    <option name="NAMES_COUNT_TO_USE_IMPORT_ON_DEMAND" value="999" />
    <option name="INSERT_INNER_CLASS_IMPORTS" value="true" />

    <option name="IMPORT_LAYOUT_TABLE">
      <value>
        <package name="java" withSubpackages="true" static="false" />
        <emptyLine />
        <package name="javax" withSubpackages="true" static="false" />
        <emptyLine />
        <package name="" withSubpackages="true" static="false" />
        <emptyLine />
        <package name="" withSubpackages="true" static="true" />
      </value>
    </option>
  </JavaCodeStyleSettings>

  <codeStyleSettings language="JAVA">
    <option name="KEEP_FIRST_COLUMN_COMMENT" value="true" />
    <option name="KEEP_SIMPLE_BLOCKS_IN_ONE_LINE" value="false" />
    <option name="KEEP_SIMPLE_METHODS_IN_ONE_LINE" value="false" />
    <option name="KEEP_SIMPLE_CLASSES_IN_ONE_LINE" value="false" />

    <!-- 들여쓰기 2칸 -->
    <option name="INDENT_SIZE" value="2" />
    <option name="TAB_SIZE" value="2" />
    <option name="CONTINUATION_INDENT_SIZE" value="4" />

    <!-- 공백 규칙 -->
    <option name="SPACE_AFTER_KEYWORD" value="true" />
    <option name="SPACE_BEFORE_METHOD_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_METHOD_CALL_PARENTHESES" value="false" />
    <option name="SPACE_WITHIN_METHOD_PARENTHESES" value="false" />

    <!-- 줄바꿈 규칙 -->
    <option name="METHOD_BRACE_STYLE" value="1" />
    <option name="CLASS_BRACE_STYLE" value="1" />
    <option name="BRACE_STYLE" value="1" />
    <option name="ELSE_ON_NEW_LINE" value="false" />
    <option name="WHILE_ON_NEW_LINE" value="false" />
    <option name="CATCH_ON_NEW_LINE" value="false" />
    <option name="FINALLY_ON_NEW_LINE" value="false" />

    <!-- annotation 위치 -->
    <option name="DO_NOT_WRAP_AFTER_SINGLE_ANNOTATION" value="true" />

    <!-- 연속 줄 들여쓰기 -->
    <option name="ALIGN_MULTILINE_PARAMETERS" value="false" />
    <option name="ALIGN_MULTILINE_METHOD_BRACKETS" value="false" />

    <!-- switch/case -->
    <option name="INDENT_CASE_FROM_SWITCH" value="true" />
  </codeStyleSettings>
</code_scheme>
```

---

# 👍 **설치 방법 요약**

1. 위 XML 전체 복사
2. `GoogleJavaStyle.xml` 로 저장
3. IntelliJ에서
   **Settings → Editor → Code Style → Java → 우측 톱니 → Import Scheme → IntelliJ IDEA code style XML**
4. 방금 저장한 XML 선택

---


