# [프로그램 상세 명세서] 전자결재 전사 다국어 동적 처리 모듈 (PRG_EA_ML_DETAIL)

## 1. 프로그램 개요
- **프로그램 ID**: PRG_EA_ML_DETAIL
- **프로그램명**: 전자결재 사용자별 언어 로케일 기반 동적 렌더링 시스템
- **모듈**: 전자결재 (smarts4j.coviapproval) 및 공통 코어 (smarts4j.covicore)
- **적용 기술**: Java 8, MyBatis 3, Spring Framework, JavaScript (CoviCommon UI)
- **핵심 목표**: 사용자의 `LanguageCode` 프로필을 추적하여 UI 레이블, 시스템 메시지, DB 데이터를 실시간으로 국지화(Localization)하여 표시함.

---

## 2. 계층별 로직 상세 분석 (Layered Logic Analysis)

### 2.1 데이터 계층 (Database Layer)
- **사용자 프로필 로직**:
    - `sys_object_user` 테이블의 `LanguageCode` 필드(varchar(5))를 조회하여 사용자의 기본 언어(ko, en, ja, zh, e1~e6)를 결정함.
- **다국어 사전 저장 로직**:
    - `sys_base_dictionary` 테이블에 저장된 `DicID`를 Key로 하며, 각 언어별 컬럼(Ko, En, Ja, Zh, Vi 등)에 저장된 값을 추출함.
- **동적 컬럼 선택 로직 (`getLocaleColumn`)**:
    - 사용자의 언어 설정에 따라 물리적 컬럼명을 동적으로 반환함.
    - 예: 사용자가 '영어'인 경우 `DisplayName` 조회 요청 시 `DisplayName_En` 컬럼을 맵핑함.

### 2.2 서비스 계층 (Server Side - Java)
- **언어 세션 로딩**:
    - `SessionHelper.getSession("LanguageCode")`를 통해 현재 로그인 사용자의 로케일을 상시 유지함.
- **사전 데이터 조회 (`DicHelper`)**:
    - 로직: `DicHelper.getDic(domainID, key)` 호출 시 Redis 캐시 또는 DB에서 해당 로케일에 맞는 문자열을 반환함.
    - 예외: 요청한 로케일에 데이터가 없을 경우 기본 로케일(Ko) 데이터를 반환하도록 설계됨.
- **전자결재 특화 로직**:
    - `smarts4j.coviapproval` 내의 관리자 설정(기초코드, 직무 등)은 저장 시 `kind=dictionary` 표준 컨트롤을 통해 다국어 사전과 연동되어 저장됨.

### 2.3 프레젠테이션 계층 (Client Side - UI/UX)
- **동적 메시지 렌더링 (`Common.getDic`)**:
    - 클라이언트 스크립트 실행 시 `Common.getDic("사전키")` 호출을 통해 실시간으로 사용자의 브라우저/세션 언어에 맞는 텍스트를 DOM에 출력함.
- **JSP 태그 라이브러리 연동**:
    - `<spring:message code='Cache.lbl_...'>` 태그를 사용하여 서버 렌더링 시점에 다국어 레이블을 확정함.
- **다국어 입력 팝업 (`DictionaryPopup`)**:
    - 사용자가 직접 명칭을 입력해야 하는 경우(예: 결재선 이름), `Common.openDicLayerPopup`을 실행하여 4개 국어(또는 설정된 전체 언어)를 동시 입력받아 사전에 저장함.

---

## 3. 전자결재 프로세스별 적용 로직 (Process Logic)

| 프로세스 | 적용 로직 상세 | 관련 기술 표준 문서 |
| :--- | :--- | :--- |
| **로그인/인증** | 세션 생성 시 `LanguageCode`를 세션 저장소에 할당 | `sys_object_user_schema.csv` |
| **결재 목록 조회** | 결재 제목, 양식명 조회 시 `getLocaleColumn()`으로 언어별 컬럼 선택 | `COMMON_DEPENDENCY_SUMMARY.md` |
| **결재 상신/승인** | 버튼(`btn_MultiLanguage`) 및 알림 메시지(`Common.Inform`) 다국어 처리 | `POPUP_CALL_INVENTORY.csv` |
| **결재선 지정** | 조직도 팝업 내 부서명/직급명 로케일 기반 렌더링 | `manageorganization.js` |
| **관리자 설정** | 직무, 결재양식 관리 시 다국어 사전 키 생성 및 맵핑 | `AggregationFieldManage.jsp` |

---

## 4. [중요] GAP 분석 결과 기반 특이사항

### 4.1 동적 입력 컨텐츠의 한계
- **문제**: 사용자가 결재 문서 작성 시 직접 입력하는 '의견', '제목', '본문(HTML)'은 시스템 사전에 등록되는 데이터가 아님.
- **로직 분석**: 현재 기술 표준은 `sys_base_dictionary`에 등록된 **정적 키(Static Key)** 변환에 특화되어 있으며, 사용자 입력 **동적 데이터(Dynamic Data)**에 대한 실시간 번역 로직은 구현되어 있지 않음.
- **대안**: 해당 요구사항 충족을 위해 별도의 AI 번역 API(Google/Azure 등) 연동 설계가 필요하나, 현재 `smarts4j.docs.260324` 표준 범위 밖임.

### 4.2 캐시 동기화 이슈
- **로직**: 다국어 데이터는 성능을 위해 메모리 캐시를 사용함. 관리자가 다국어 사전을 수정한 후에는 반드시 `DicHelper`의 캐시 갱신 기능을 실행해야만 사용자 화면에 즉시 반영됨.

---

## 5. 산출 근거 및 참조 문서 (Citations)
본 명세서는 `\smarts4j.docs.260324` 하위의 다음 문서를 정밀 분석하여 작성되었습니다.
1.  **아키텍처**: `02_Architecture\COMMON_DEPENDENCY_SUMMARY.md` (로케일 처리 핵심 로직)
2.  **UI 표준**: `01_Common_UI_Standard\COMMON_CONTROLS_SPEC.md` (다국어 컨트롤 동작 방식)
3.  **데이터 구조**: `02_Architecture\assets\data\` 내 CSV 파일군 (사용자, 메뉴, 폴더, 사전 데이터 구조)
4.  **검증 자산**: `04_Verification_Assets\smarts4j.covicore\TEST_CASES_CORE.md` (다국어 리소스 현행화 테스트 로직)

---
**작성일**: 2026-03-24
**작성자**: 개발팀 (Senior Software Engineer)
**검토자**: PM (ghkim2)
