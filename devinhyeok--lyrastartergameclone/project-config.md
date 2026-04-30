---
trigger: always_on
description: 일반 C++ 코드 스타일 가이드
---


# 일반 C++ 코드 스타일 가이드

이 문서는 언리얼 엔진 외부의 일반 C++ 코드 작성을 위한 스타일 가이드입니다. 구글 C++ 스타일 가이드를 기반으로 작성되었습니다.

- https://google.github.io/styleguide/cppguide.html

## 목차

1. [파일 구성](mdc:#1-파일-구성)
2. [명명 규칙](mdc:#2-명명-규칙)
3. [형식 지정](mdc:#3-형식-지정)
4. [주석](mdc:#4-주석)
5. [클래스](mdc:#5-클래스)
6. [함수](mdc:#6-함수)
7. [기타 C++ 기능](mdc:#7-기타-c-기능)
8. [일반 원칙](mdc:#8-일반-원칙)

## 1. 파일 구성

### 1.1 파일 확장자

- C++ 소스 파일: `.cpp`
- C++ 헤더 파일: `.h`
- 인라인 함수 정의: `.inl` (선택 사항)

### 1.2 파일 이름

- 모든 파일 이름은 소문자를 사용합니다.
- 단어 사이에는 밑줄(`_`)을 사용합니다. (예: `my_useful_class.cpp`)
- 클래스가 정의된 헤더 파일은 클래스 이름을 사용합니다. (예: 클래스 `MyClass`는 `my_class.h`에 정의)

### 1.3 헤더 파일

- 모든 `.cpp` 파일은 해당 `.h` 파일이 있어야 합니다 (특수 상황 제외).
- 헤더 파일은 자체 완비적(self-contained)이어야 합니다 - 즉, 어떤 다른 파일의 포함 없이도 독립적으로 컴파일 가능해야 합니다.
- 모든 헤더 파일은 `#include` 가드를 포함해야 합니다:

```cpp
#ifndef PROJECT_PATH_FILE_H_
#define PROJECT_PATH_FILE_H_

// 코드 내용...

#endif  // PROJECT_PATH_FILE_H_
```

또는 `#pragma once`를 사용할 수 있습니다:

```cpp
#pragma once

// 코드 내용...
```

### 1.4 Include 순서

1. 관련 헤더 파일: 소스 파일은 해당 헤더를 첫 번째로 포함 (예: `foo.cpp`는 `foo.h`를 첫 번째로 포함)
2. C 시스템 헤더 (`<stdio.h>` 등)
3. C++ 표준 라이브러리 헤더 (`<vector>` 등)
4. 다른 라이브러리 헤더
5. 프로젝트 헤더

각 그룹은 알파벳 순서로 정렬하고, 그룹 사이에는 빈 줄을 넣습니다.

```cpp
// foo.cpp
#include "foo.h"

#include <sys/types.h>
#include <unistd.h>

#include <algorithm>
#include <string>
#include <vector>

#include "base/logging.h"
#include "base/macros.h"
#include "foo/public/other_header.h"
```

## 2. 명명 규칙

### 2.1 일반 규칙

- 이름은 의미 있고 설명적이어야 합니다.
- 축약어와 단축 형태는 일반적으로 통용되는 경우에만 사용합니다.
- 약어는 모두 대문자로 작성하거나 첫 글자만 대문자로 작성합니다. (예: `HTTP` 또는 `Http`, `URL` 또는 `Url`)

### 2.2 파일 명명

- 소문자와 밑줄(`_`) 사용: `my_useful_class.cpp`

### 2.3 타입 명명

- 클래스, 구조체, 타입 별칭, 열거형, 타입 템플릿 매개변수는 파스칼 케이스를 사용합니다:
  - 단어의 첫 글자를 대문자로 작성, 밑줄 사용하지 않음
  - 예: `MyExcitingClass`, `MyImportantEnum`

```cpp
class UrlTable { ... };
struct UrlTableProperties { ... };
using PropertiesMap = map<string, string>;
enum UrlTableError { ... };
```

### 2.4 변수 명명

- 변수명은 소문자와 밑줄로 작성합니다.
- 클래스 멤버 변수는 끝에 밑줄을 추가합니다.

```cpp
string table_name;       // 일반 변수
string table_name_;      // 클래스 멤버 변수
```

### 2.5 상수 명명

- 상수는 `k`로 시작하는 파스칼 케이스를 사용합니다.

```cpp
const int kDaysInAWeek = 7;
const int kAndroidVersions[] = { 10, 11, 12 };
```

### 2.6 함수 명명

- 일반 함수는 파스칼 케이스를 사용합니다. (예: `AddTableEntry()`)
- 접근자와 변경자는 변수 이름처럼 명명하고, Get/Set을 접두어로 사용합니다. (예: `GetCount()`, `SetCount()`)
- 단순 속성 접근의 경우 속성 이름만 사용합니다. (예: `count()`)

```cpp
class MyClass {
public:
    int count() const { return count_; }
    void set_count(int count) { count_ = count; }
    // 더 복잡한 작업을 할 때:
    void AddCount(int count);
    int GetCalculatedCount() const;

private:
    int count_;
};
```

### 2.7 네임스페이스 명명

- 네임스페이스는 소문자로 작성합니다.
- 최상위 네임스페이스는 프로젝트 이름이나 팀 이름을 기반으로 합니다.

```cpp
namespace project_name {
namespace sub_component {
// ...
}  // namespace sub_component
}  // namespace project_name
```

### 2.8 열거형 명명

- 열거형 값은 상수와 동일한 명명 규칙을 따릅니다 (접두어 `k` 사용).

```cpp
enum class UrlTableError {
    kOk = 0,
    kOutOfMemory,
    kMalformedInput,
};
```

### 2.9 매크로 명명

- 매크로는 되도록 사용하지 않습니다.
- 필요할 경우 모두 대문자와 밑줄을 사용합니다.

```cpp
#define ROUND(x) ...
#define PI_ROUNDED 3.0
```

## 3. 형식 지정

### 3.1 줄 길이

- 한 줄은 80자 이내로 제한합니다.

### 3.2 들여쓰기

- 들여쓰기는 공백 2칸을 사용합니다.
- 탭은 사용하지 않습니다.
- 네임스페이스 내용은 들여쓰기하지 않습니다.

```cpp
namespace outer {
namespace inner {
// 네임스페이스 내용은 들여쓰기하지 않습니다.
class MyClass {  // 클래스 내용은 2칸 들여쓰기
 public:
  void Function() {  // 함수 내용은 2칸 들여쓰기
    if (condition) {  // 조건문 내용은 2칸 들여쓰기
      DoSomething();  // 코드 내용은 2칸 들여쓰기
    }
  }
};
}  // namespace inner
}  // namespace outer
```

### 3.3 괄호 스타일

- 여는 중괄호(`{`)는 함수, 클래스, 네임스페이스 선언에서 새 줄에 배치하지 않고, 문장 끝에 함께 둡니다.
- 조건문과 루프의 경우에도 여는 중괄호는 같은 줄에 배치합니다.

```cpp
if (condition) {  // 여는 중괄호는 같은 줄에
  // 코드
} else {  // else도 같은 줄에
  // 코드
}

for (int i = 0; i < kSomeNumber; ++i) {
  // 코드
}

class MyClass {
 public:
  MyClass() : some_var_(0) {}  // 간단한 생성자는 한 줄로

  void Method() {
    // 메서드 코드
  }

 private:
  int some_var_;
};
```

### 3.4 함수 선언 및 정의

- 함수 매개변수가 한 줄에 맞지 않으면, 들여쓰기를 4칸 하고 각 매개변수를 새 줄에 배치합니다.
- 반환 타입은 함수 이름과 같은 줄에 배치합니다.

```cpp
ReturnType ClassName::FunctionName(Type par1, Type par2) {
  // 함수 내용
}

ReturnType LongClassName::ReallyReallyReallyLongFunctionName(
    Type par1,  // 매개변수 4칸 들여쓰기
    Type par2,
    Type par3) {
  // 함수 내용
}
```

### 3.5 조건문

- 조건문은 항상 중괄호를 사용합니다 (한 줄 조건문도 마찬가지).

```cpp
if (condition) {
  DoOneThing();
}

// 한 줄이라도 중괄호 사용
if (condition) {
  DoOneThing();
}

// if-else 체인
if (condition) {
  // 코드
} else if (condition2) {
  // 코드
} else {
  // 코드
}
```

### 3.6 스위치문

- 각 case 문은 switch와 같은 들여쓰기 레벨입니다.
- case 내부 코드는 2칸 들여쓰기합니다.

```cpp
switch (var) {
  case 0:
    DoZero();
    break;
  case 1: {
    // 블록 사용 시 중괄호 필요
    DoOne();
    break;
  }
  default:
    DoDefault();
}
```

### 3.7 포인터와 참조

- 포인터와 참조 기호는 변수 이름이 아닌 타입에 붙입니다.

```cpp
// 올바른 방식
char* c;
const string& str;

// 잘못된 방식
char *c;
const string &str;
```

### 3.8 수평 공백

- 연산자 전후에 공백을 하나씩 추가합니다.
- 콤마 뒤에는 공백을 하나 추가합니다.
- 다음과 같은 경우에는 공백을 추가하지 않습니다:
  - 함수 이름과 여는 괄호 사이
  - 배열 이름과 여는 대괄호 사이
  - 괄호/대괄호의 안쪽

```cpp
// 올바른 사용
sum = a + b;
array[index] = 0;
for (int i = 0; i < 10; ++i) {}
Function(arg1, arg2);
```

### 3.9 수직 공백

- 함수 사이, 클래스 정의 사이에는 빈 줄을 하나 넣습니다.
- 논리적으로 구분되는 코드 블록 사이에는 빈 줄을 넣어 가독성을 높입니다.

## 4. 주석

### 4.1 파일 주석

- 각 소스 파일 상단에 라이센스 표시 후 파일 내용에 대한 간략한 설명을 추가합니다.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/devinhyeok) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-11 -->
