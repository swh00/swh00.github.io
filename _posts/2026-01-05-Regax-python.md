---
layout: post
title: "파이썬 정규표현식(RegEx) 정리"
date: 2026-01-05
categories: [개발, 파이썬]
tags: [python, regex, study]
---

개발을 하다 보면 복잡한 문자열을 다뤄야 할 때가 많습니다. 로그 파일에서 특정 에러 코드만 뽑아내거나, 사용자가 입력한 이메일 형식이 맞는지 검증해야 하는 순간들이죠. 이럴 때 **정규표현식(Regular Expression)**은 선택이 아닌 필수입니다.

오늘은 파이썬에서 정규표현식을 다루는 `re` 모듈의 핵심 사용법을 정리해 보겠습니다.

<br>

## 1. 정규표현식이란?

정규표현식(줄여서 정규식, RegEx)은 **특정한 규칙을 가진 문자열의 집합을 표현하는 언어**입니다. 
파이썬뿐만 아니라 JavaScript, Java, C++ 등 대부분의 언어에서 지원하며, 문법도 거의 유사합니다.

파이썬에서는 기본 라이브러리인 `re` 모듈을 제공합니다.

```python
import re
```

<br>

## 2. 핵심 메타 문자 (Meta Characters)

정규식의 꽃은 바로 메타 문자입니다. 자주 쓰이는 것들을 표로 정리해 보았습니다.

| 메타 문자 | 설명 | 예시 |
|:---:|:---|:---|
| `.` | 줄바꿈(`\n`)을 제외한 모든 문자 1개 | `a.b` → aab, a0b (O) / abc (X) |
| `^` | 문자열의 시작 | `^Life` |
| `$` | 문자열의 끝 | `short$` |
| `*` | 앞의 문자가 0번 이상 반복 | `ca*t` → ct, cat, caaat |
| `+` | 앞의 문자가 1번 이상 반복 | `ca+t` → cat, caaat (O) / ct (X) |
| `?` | 앞의 문자가 있거나 없음 (0 or 1) | `ab?c` → ac, abc |
| `{m,n}` | m번 이상 n번 이하 반복 | `a{1,3}` |
| `[]` | 괄호 안의 문자 중 하나 | `[abc]` → a, b, c 중 하나 |
| `|` | 또는 (OR) | `a|b` |

### 💡 자주 쓰는 문자 클래스
복잡한 `[0-9]`, `[a-zA-Z]` 대신 아래 단축 문자를 많이 사용합니다.

- `\d`: 숫자 (`[0-9]`)
- `\w`: 문자 + 숫자 + `_` (underscore) (`[a-zA-Z0-9_]`)
- `\s`: 공백 문자(스페이스, 탭, 엔터 등)
- `\D`, `\W`, `\S`: 위 문자들의 반대 (대문자는 NOT을 의미)

<br>

## 3. 파이썬 `re` 모듈 핵심 함수

파이썬에서 가장 많이 사용하는 4가지 함수입니다.

### 3.1. `re.match()` vs `re.search()`

가장 많이 혼동하는 두 함수입니다.

- **`match()`**: 문자열의 **처음부터** 패턴이 일치하는지 확인합니다.
- **`search()`**: 문자열 **전체**를 훑어서 첫 번째로 일치하는 부분을 찾습니다.

```python
import re

text = "3 little pigs"

# 1. match
# 문자열 시작이 숫자(\d)이므로 매칭 성공
m = re.match(r'\d+', text)
print(m.group())  # 결과: 3

# 2. match 실패 케이스
text2 = "The 3 little pigs"
m = re.match(r'\d+', text2)
print(m)  # 결과: None (시작이 T 이므로)

# 3. search
# 문자열 중간에 있는 숫자를 찾아냄
s = re.search(r'\d+', text2)
print(s.group())  # 결과: 3
```

> **Tip:** 파이썬에서 정규식을 쓸 때는 문자열 앞에 `r`을 붙여 `r'패턴'` 형태(Raw String)로 쓰는 것이 좋습니다. 그래야 백슬래시(`\`)가 이스케이프 문자로 처리되지 않고 그대로 정규식 문자로 인식됩니다.

### 3.2. `re.findall()`

매칭되는 모든 문자열을 **리스트(List)**로 반환합니다. 가장 활용도가 높습니다.

```python
text = "010-1234-5678 and 02-987-6543"

# 전화번호 패턴 추출 (\d+-\d+-\d+)
phones = re.findall(r'\d+-\d+-\d+', text)
print(phones)
# 결과: ['010-1234-5678', '02-987-6543']
```

### 3.3. `re.sub()`

찾은 패턴을 다른 문자열로 **치환(Replace)**합니다. 개인정보 마스킹 등에 유용합니다.

```python
text = "My phone is 010-1234-5678."

# 숫자를 모두 '*'로 변경
masked = re.sub(r'\d', '*', text)
print(masked)
# 결과: My phone is ***-****-****.
```

<br>

## 4. 그룹핑 (Grouping)

패턴 안에서 `()`를 사용하면 특정 부분만 따로 추출할 수 있습니다. `match`나 `search` 객체의 `.group(인덱스)` 메서드를 사용합니다.

- `group(0)`: 매칭된 전체 문자열
- `group(n)`: n번째 그룹

```python
text = "Kim 010-1234-5678"
pattern = r'(\w+)\s+(\d+-\d+-\d+)'

m = re.search(pattern, text)

print(m.group(0)) # 전체: Kim 010-1234-5678
print(m.group(1)) # 이름: Kim
print(m.group(2)) # 번호: 010-1234-5678
```

<br>

## 5. 성능 팁: `re.compile()`

같은 패턴을 반복문 안에서 여러 번 사용해야 한다면, 매번 패턴을 해석하는 것보다 **컴파일**을 해두는 것이 성능상 유리합니다.

```python
# 패턴 컴파일 객체 생성
email_regex = re.compile(r'[\w\.-]+@[\w\.-]+')

users = ["user1@test.com", "fail-email", "admin@korea.kr"]

for user in users:
    # 컴파일된 객체의 search 메서드 사용
    if email_regex.search(user):
        print(f"{user}: Valid Email")
    else:
        print(f"{user}: Invalid")
```

<br>

## 마무리

정규표현식은 처음엔 외계어처럼 보이지만, 익숙해지면 코드 몇 십 줄을 단 한 줄로 줄여주는 강력한 무기가 됩니다. 

복잡한 패턴이 필요할 때는 [regex101.com](https://regex101.com/) 같은 사이트에서 실시간으로 테스트해보며 작성하는 것을 추천합니다.

---
**Reference**
- [Python 공식 문서 - re 모듈](https://docs.python.org/3/library/re.html)
