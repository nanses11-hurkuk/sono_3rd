# [프로그램 명세서] 전자결재 다국어 처리 공통 모듈 (PRG_EA_ML_COMMON)

## 1. 프로그램 개요
- **프로그램 ID**: PRG_EA_ML_COMMON
- **프로그램명**: 전자결재 사용자별 로케일 기반 다국어 처리 공통 모듈
- **모듈**: 전자결재 (smarts4j.coviapproval)
- **프로그램 유형**: Java (Service/DAO), JavaScript (Utility), JSP (UI)
- **설명**: 전자결재 시스템 내 모든 UI 레이블, 시스템 메시지 및 DB 데이터를 사용자의 설정 언어에 맞춰 동적으로 처리하는 공통 로직을 정의함.

---

## 2. 관련 기술 표준 (Reference)
본 프로그램은 `\smarts4j.docs.260324` 기술 표준 문서를 준수함.
- **아키텍처**: `02_Architecture\COMMON_DEPENDENCY_SUMMARY.md` (`DicHelper`, `getLocaleColumn`)
- **데이터베이스**: `02_Architecture\assets\data\sys_object_user_schema.csv` (`LanguageCode`)
- **UI 컨트롤**: `01_Common_UI_Standard\COMMON_CONTROLS_SPEC.md` (`kind=dictionary`)

---

## 3. 프로그램 로직 상세 (Logic Detail)

### 3.1 사용자 언어 정보 획득 (Session Management)
- **로직**: 세션(Session)에 저장된 `LanguageCode` 정보를 참조하여 현재 사용자의 선호 언어를 확인한다.
- **구현 방식**:
    - Java: `SessionHelper.getSession("LanguageCode")` 호출.
    - JavaScript: `Common.getSession("LanguageCode")` 호출.

### 3.2 UI 레이블 및 메시지 변환 (Dictionary Search)
- **로직**: 하드코딩된 문자열 대신 다국어 사전(`sys_base_dictionary`)의 키(Key)를 사용하여 메시지를 출력한다.
- **구현 방식**:
    - **JSP/Server Side**: `<spring:message code='Cache.lbl_Title'/>` 또는 `DicHelper.getDic("lbl_Title")` 사용.
    - **Client Side**: `Common.getDic("lbl_Title")` 스크립트 함수를 사용하여 실시간 변환.

### 3.3 DB 데이터 동적 컬럼 조회 (SQL Dynamic Query)
- **로직**: 조직도 정보나 기초 코드 조회 시 사용자의 언어에 맞는 컬럼(Name_Ko, Name_En 등)을 선택한다.
- **구현 방식**:
    - MyBatis XML 쿼리 작성 시 `getLocaleColumn()`을 통해 반환된 컬럼명을 맵핑.
    - 예시 SQL: `SELECT ${localeColumn} AS DisplayName FROM sys_object_user WHERE UserCode = #{userCode}`

### 3.4 다국어 입력 컨트롤 적용 (UI Control)
- **로직**: 다국어 지원이 필요한 입력 필드(예: 결재선 명칭 등)에 다국어 입력 팝업 기능을 연결한다.
- **구현 방식**:
    - `kind=dictionary` 속성을 가진 input 컨트롤을 배치하고 `coviCmn.openDicLayerPopup`을 연결함.

---

## 4. 입출력 데이터 정의 (I/O Definition)

### 4.1 입력 (Input)
| 항목명 | 변수명 | 타입 | 설명 |
| :--- | :--- | :---: | :--- |
| 언어 코드 | langCode | String | 사용자의 세션 언어 (ko, en, vi 등) |
| 다국어 키 | dicKey | String | 다국어 사전에 정의된 고유 ID (lbl_XXX, msg_XXX) |

### 4.2 출력 (Output)
| 항목명 | 변수명 | 타입 | 설명 |
| :--- | :--- | :---: | :--- |
| 변환 텍스트 | translatedText | String | 현재 언어에 맞게 변환된 최종 문자열 |
| 로케일 컬럼 | localeColumn | String | DB 조회 시 사용할 컬럼명 (DisplayName_En 등) |

---

## 5. 예외 처리 및 제약 사항
1. **사전 키 누락**: 다국어 사전에 존재하지 않는 키 호출 시, 기본값(한국어 또는 키 자체)을 반환하도록 처리 (`DicHelper` 기본 동작).
2. **[GAP] 결재 본문 처리**: 결재 문서 본문(HTML) 및 작성자가 입력한 제목은 동적 번역 대상에서 제외됨 (기술 표준 미정의 영역).
3. **캐시 의존성**: 다국어 데이터는 서버 메모리 캐시에 적재되므로, 사전 데이터 변경 후 반드시 캐시 초기화(`Redis` 등) 로직을 실행해야 함.

---
**작성일**: 2026-03-24
**작성자**: 개발팀 (Lead Developer)
**검토자**: PM (ghkim2)
