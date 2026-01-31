# 형상 관리 계획 (Configuration Management Plan)

**프로젝트**: STM32 Dam Control System  
**버전**: 1.0  
**작성일**: 2025-01-31

---

## 1. Git 브랜치 전략

### 1.1 브랜치 구조

```
main (배포용)
  │
  ├── develop (개발용)
  │     │
  │     ├── feature/dht11-driver
  │     ├── feature/lcd-driver
  │     ├── feature/keypad-driver
  │     └── feature/data-logger
  │
  └── hotfix/sensor-bug (긴급 수정)
```

### 1.2 브랜치 규칙

**main**:
- 항상 안정적인 릴리즈 버전만 유지
- 직접 커밋 금지
- develop 또는 hotfix에서만 병합

**develop**:
- 개발 중인 최신 코드
- 테스트 통과된 코드만 병합
- feature 브랜치에서 병합

**feature/***:
- 새 기능 개발용
- 네이밍: `feature/기능명`
- 예: `feature/alarm-function`, `feature/threshold-setting`

**hotfix/***:
- 긴급 버그 수정
- main에서 분기, main과 develop 모두에 병합

---

## 2. 커밋 메시지 규칙

### 2.1 커밋 메시지 형식

```
<타입>: <제목>

<본문 (선택)>

<관련 이슈 (선택)>
```

### 2.2 타입 종류

| 타입 | 설명 | 예시 |
|-----|------|------|
| feat | 새 기능 추가 | `feat: DHT11 드라이버 추가` |
| fix | 버그 수정 | `fix: LCD 초기화 오류 수정` |
| docs | 문서 수정 | `docs: README에 핀 배치 추가` |
| style | 코드 포맷팅 | `style: 들여쓰기 정리` |
| refactor | 리팩토링 | `refactor: ADC 읽기 함수 최적화` |
| test | 테스트 추가 | `test: 키패드 단위 테스트 작성` |
| chore | 빌드/설정 변경 | `chore: .gitignore 업데이트` |

### 2.3 커밋 메시지 예시

**좋은 예**:
```
feat: 임계값 초과 시 경보 기능 구현

- 수위 80% 초과 시 LCD 경고 표시
- UART로 알람 메시지 전송
- 테스트 완료 (TC-SYS-03)
```

**나쁜 예**:
```
update
수정
기능 추가함
```

---

## 3. 버전 관리 (Semantic Versioning)

### 3.1 버전 번호 체계

```
v[Major].[Minor].[Patch]

예: v1.2.3
```

| 버전 | 증가 조건 | 예시 |
|-----|----------|------|
| Major | 하위 호환 안 되는 변경 | v1.0.0 → v2.0.0 |
| Minor | 하위 호환되는 기능 추가 | v1.1.0 → v1.2.0 |
| Patch | 버그 수정 | v1.1.1 → v1.1.2 |

### 3.2 프로젝트 버전 예시

- **v0.1.0**: 초기 프로토타입 (DHT11만 동작)
- **v0.2.0**: LCD 표시 추가
- **v0.3.0**: 키패드 입력 추가
- **v1.0.0**: 첫 릴리즈 (모든 기능 구현)
- **v1.1.0**: 경보 기능 추가
- **v1.1.1**: DHT11 타임아웃 버그 수정

---

## 4. Git 태그 사용

### 4.1 태그 생성

**경량 태그** (간단한 표시용):
```bash
git tag v1.0.0
```

**주석 태그** (릴리즈 정보 포함 - 권장):
```bash
git tag -a v1.0.0 -m "Release v1.0.0

주요 기능:
- 수위 모니터링
- 온습도 측정
- LCD 표시
- UART 데이터 로깅

테스트 완료: TC-SYS-01~05"

git push origin v1.0.0
```

### 4.2 태그 조회

```bash
git tag                    # 태그 목록
git show v1.0.0            # 태그 상세 정보
git checkout v1.0.0        # 특정 버전으로 이동
```

---

## 5. 릴리즈 프로세스

### 5.1 릴리즈 체크리스트

릴리즈 전 확인 사항:

- [ ] 모든 테스트 통과 (test-checklist.md 참조)
- [ ] README 업데이트
- [ ] CHANGELOG 작성
- [ ] 버전 번호 결정
- [ ] develop → main 병합
- [ ] Git 태그 생성
- [ ] GitHub Release 생성

### 5.2 릴리즈 단계

**1. develop 브랜치에서 테스트**
```bash
git checkout develop
# 빌드 및 테스트
```

**2. main 브랜치로 병합**
```bash
git checkout main
git merge develop
```

**3. 버전 태그 생성**
```bash
git tag -a v1.0.0 -m "Release v1.0.0"
git push origin main --tags
```

**4. GitHub Release 생성**
- GitHub 웹사이트에서 Release 생성
- 빌드된 .bin 파일 첨부
- 릴리즈 노트 작성

---

## 6. 파일 및 디렉토리 관리

### 6.1 .gitignore 설정

```gitignore
# 빌드 결과물
*.o
*.d
*.elf
*.bin
*.hex
*.map
*.lst

# IDE 설정
.settings/
.cproject
.project
Debug/
Release/

# 임시 파일
*.tmp
*.bak
*~

# OS 관련
.DS_Store
Thumbs.db

# 예외: 중요 설정 파일은 포함
!*.ioc
```

### 6.2 추적 대상 파일

**포함**:
- 모든 소스 코드 (.c, .h)
- 프로젝트 설정 (.ioc, Makefile)
- 문서 (README.md, docs/)
- 테스트 코드

**제외**:
- 빌드 결과물 (.bin, .elf)
- IDE 개인 설정
- 임시 파일

---

## 7. 이슈 및 마일스톤 관리

### 7.1 GitHub Issues 사용

**이슈 템플릿**:

```markdown
## 버그 보고
**증상**: DHT11 센서가 간헐적으로 읽기 실패

**재현 방법**:
1. 시스템 부팅
2. 1분 대기
3. 온습도 확인

**예상 동작**: 정상적으로 값 읽기
**실제 동작**: "Sensor Error" 표시

**환경**:
- 버전: v1.0.0
- 보드: STM32F411CEU6
```

### 7.2 라벨 사용

| 라벨 | 색상 | 용도 |
|-----|------|------|
| bug | 빨강 | 버그 |
| enhancement | 파랑 | 기능 개선 |
| documentation | 노랑 | 문서 관련 |
| good first issue | 초록 | 초보자용 |
| help wanted | 보라 | 도움 필요 |

### 7.3 마일스톤 예시

- **v1.0.0 - First Release** (목표: 2025-02-15)
  - #1: DHT11 드라이버 구현
  - #2: LCD 표시 기능
  - #3: 키패드 입력
  - #4: 데이터 로깅

- **v1.1.0 - Alarm Feature** (목표: 2025-03-01)
  - #10: 경보 기능 구현
  - #11: 임계값 설정 UI

---

## 8. CHANGELOG 작성

### 8.1 형식

```markdown
# Changelog

## [Unreleased]
### Added
- 온습도 트렌드 그래프 (개발 중)

## [1.1.0] - 2025-02-20
### Added
- 수위 임계값 초과 시 경보 기능
- 사용자 임계값 설정 메뉴

### Fixed
- DHT11 센서 타임아웃 오류 수정

## [1.0.0] - 2025-02-10
### Added
- 초기 릴리즈
- 수위 모니터링 기능
- 온습도 측정 기능
- LCD 실시간 표시
- UART 데이터 로깅
```

---

## 9. 백업 및 복구 전략

### 9.1 GitHub 원격 저장소

**주 저장소**: https://github.com/kyoung-mo/STM32-Dam-Control-System

**백업 주기**:
- 작업 중: 커밋 후 즉시 push
- 일일 백업: 자동 (GitHub 자체 백업)

### 9.2 로컬 백업

```bash
# 전체 프로젝트 압축
git archive --format=zip --output=backup_20250131.zip HEAD

# 특정 태그 백업
git archive --format=zip --output=release_v1.0.0.zip v1.0.0
```

---

## 10. 협업 가이드 (다른 사람과 작업 시)

### 10.1 Pull Request 규칙

**PR 제목 형식**:
```
[feat] 경보 기능 구현
[fix] LCD 초기화 버그 수정
```

**PR 설명 템플릿**:
```markdown
## 변경 사항
- 수위 임계값 초과 시 경보 표시
- UART로 알람 메시지 전송

## 테스트 결과
- [x] TC-SYS-03 통과
- [x] 빌드 성공
- [x] 하드웨어 동작 확인

## 관련 이슈
Closes #15
```

### 10.2 코드 리뷰 체크리스트

- [ ] 코드 스타일 일관성
- [ ] 주석 충분한가?
- [ ] 테스트 추가되었나?
- [ ] 문서 업데이트되었나?
- [ ] 빌드 에러 없나?

---

## 11. 프로젝트별 형상 항목

### 11.1 형상 관리 대상

| 항목 | 위치 | 관리 방법 |
|-----|------|----------|
| 소스 코드 | App/Src/, Core/Src/ | Git |
| 헤더 파일 | App/Inc/, Core/Inc/ | Git |
| 프로젝트 설정 | .ioc, Makefile | Git |
| 문서 | README.md, docs/ | Git |
| 테스트 코드 | test/ | Git |
| 빌드 결과 | Debug/, Release/ | Git 제외 |

### 11.2 베이스라인 설정

| 베이스라인 | 버전 | 날짜 | 설명 |
|-----------|------|------|------|
| BL-01 | v0.1.0 | 2025-01-15 | 프로토타입 |
| BL-02 | v1.0.0 | 2025-02-10 | 첫 릴리즈 |
| BL-03 | v1.1.0 | 2025-02-20 | 경보 기능 추가 |

---

## 12. 형상 변경 프로세스

### 12.1 변경 요청 (Change Request)

**작은 변경** (버그 수정):
1. 이슈 생성
2. feature 브랜치 생성
3. 수정 후 커밋
4. develop에 병합
5. 테스트

**큰 변경** (새 기능):
1. 이슈 생성 및 논의
2. 설계 문서 작성/업데이트
3. feature 브랜치 생성
4. 구현 및 테스트
5. PR 생성
6. 코드 리뷰
7. develop에 병합

### 12.2 긴급 변경 (Hotfix)

```bash
# main에서 hotfix 브랜치 생성
git checkout main
git checkout -b hotfix/critical-bug

# 수정 후
git commit -m "fix: 치명적 버그 긴급 수정"

# main과 develop 모두에 병합
git checkout main
git merge hotfix/critical-bug
git tag -a v1.0.1 -m "Hotfix v1.0.1"

git checkout develop
git merge hotfix/critical-bug
```

---

## 13. 변경 이력 추적

### 13.1 Git Log 활용

```bash
# 최근 10개 커밋 보기
git log --oneline -10

# 특정 파일 변경 이력
git log --follow App/Src/dht11.c

# 특정 버전 간 차이
git diff v1.0.0..v1.1.0

# 누가 수정했는지 확인
git blame App/Src/ap.c
```

### 13.2 릴리즈 노트 자동 생성

```bash
# v1.0.0 이후 변경사항 추출
git log v1.0.0..HEAD --oneline --no-merges
```

---

## 14. 품질 관리

### 14.1 커밋 전 체크리스트

- [ ] 코드 컴파일 성공
- [ ] 관련 테스트 통과
- [ ] 주석 작성 완료
- [ ] 불필요한 디버그 코드 제거
- [ ] .gitignore 확인 (빌드 결과물 포함 안 됨)

### 14.2 정기 점검

**주간**:
- develop 브랜치 빌드 테스트
- 이슈 트리아지

**월간**:
- CHANGELOG 업데이트
- 문서 정합성 확인
- 사용하지 않는 브랜치 정리

---

## 15. 참고 자료

- [Git 공식 문서](https://git-scm.com/doc)
- [Semantic Versioning](https://semver.org/)
- [Conventional Commits](https://www.conventionalcommits.org/)
- [GitHub Flow](https://guides.github.com/introduction/flow/)
