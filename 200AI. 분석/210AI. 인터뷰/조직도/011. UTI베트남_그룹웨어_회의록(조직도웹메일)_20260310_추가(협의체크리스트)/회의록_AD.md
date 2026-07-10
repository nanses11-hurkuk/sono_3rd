# (AD)

| Unnamed: 0 | Unnamed: 1 | Unnamed: 2 | Unnamed: 3 | Unnamed: 4 | Unnamed: 5 |
| --- | --- | --- | --- | --- | --- |
|  | No. | 항목 | 예시답변 | 비고 | 고객사 답변 |
|  | 1 | AS-IS AD 사용여부 | □ Y    □ N<br><br>Y인 경우, 사용중인 정책이 있는지?    □ Y    □ N |  | AD사용중<br>*asis는 V1, V2 ad 분리되어있으나, tobe는 통합 예정 |
|  | 2 | AD 구조 변경 가능여부 | ■ Y<br><br>* 아래와 같이 구조가 변경됩니다. |  | Y<br>서비스계정(프린터 등 그룹웨어와 연계되지 않은 계정은 별도 OU에 유지) |
|  |  |  | 1. 배포단위<br>    - 회사 하위의 부서/직위/직급/직책 개체 (사용자 개체 없음)<br>    - '유니버셜-보안' 그룹으로 생성<br>    - 부서/직위/직급/직책 삭제 시 AD개체도 삭제됨<br><br>    (대표속성) <br>    cn=부서/직위/직급/직책 코드<br>    sAMAccountName=부서/직위/직급/직책 코드 |  | 배포단위는 부서, 직위만 구성<br>>UTI내부 검토 후 확정<br><br>*ASIS : 모든 부서를 1레벨 OU로 생성하여 부서별 사용자 매핑하여 사용중 |
|  |  |  | 2. 조직단위<br>    - 회사, 부서 OU 및 사용자 개체<br>    - 사용자 개체의 소속그룹 : 부서/직위/직급/직책 개체<br>    - 사용자 퇴사(계정 삭제) 시 회사OU의 퇴직부서로 이동 및 비활성화 (삭제 X)<br>    <br>    (대표속성)<br>    cn=사용자 코드/LogonID/EmpNo/DisplayName<br>    sAMAccountName=사용자 LogonID<br>    userPrincipalName=사용자 LogonID@AD도메인(or 대체 UPN 접미사)<br> |  |  |
|  | 3 | 퇴사시 AD계정 비활성화 | ■ Y<br><br>* 퇴사시 AD객체 삭제하지 않고 비활성화 처리됨 |  | Y |