# ledger

GitHub Pages를 프론트엔드, GitHub API를 백엔드로 활용하는 개인 도구.  
Personal Access Token을 프라이버시 레이어로 사용 — 퍼블릭 저장소지만 토큰 없이는 데이터 접근 불가.

---

## 구성 파일

| 파일 | 설명 |
|------|------|
| `index.html` | 가계부 앱 (메인) |
| `memo.html` | 메모 앱 (부가) |
| `data.nfo` | 가계부 데이터 (JSON) |
| `memo.nfo` | 메모 데이터 (JSON) |
| `raw/` | 메모 첨부 파일 (이미지·오디오·PDF) |
| `backups/` | 가계부 txt 백업 보관 폴더 |
| `.gitignore` | 로컬 유틸리티 동기화 제외 설정 |

---

## 가계부 (index.html) — 메인 기능

### 잔고 구조
- 슈잔고 / 바잔고 / 카잔고 / 지잔고 4개 계정으로 구성
- 카잔고는 세부항목(카뱅·농협·세이프·mmf·카Pay)으로 분리 관리
- 총잔고 및 계정별 잔고 실시간 표시

### 거래 입력 (입력 탭)
- **수입**: 카테고리 태그 선택 → 총수입 + 수수료 입력 → 순수입 자동 계산
- **지출**: 카테고리 + 차감 계정 선택 → 금액 입력
- **잔고 이전**: 계정 간 자금 이동, 수수료 처리 포함. 카잔고 관련 시 세부항목 선택
- **잔액 보정**: 실제 잔액 입력 → 계산값과 차이를 자동 수입/지출 처리
- **바잔고 월정산**: 고정잔액 변경 시 감소분을 수수료(지출)로 자동 처리
- 오늘 내역 하단 실시간 표시

### 리뷰 탭
- 기간 필터로 거래 내역 조회
- GitHub 저장/불러오기 (☁)
- 로컬 txt 백업 다운로드 / backups/ 폴더에서 txt 가져오기 (병합)

### 설정 탭
- 수입·지출 카테고리 커스터마이징
- 잔고 계정 추가/삭제
- 카잔고 세부항목 관리
- GitHub 토큰 관리

### 출입비번 탭
- 장소별 출입비번 저장·조회
- 장소명 검색 지원

### GitHub 동기화
- 자동 저장: 거래 입력 시 디바운스 후 GitHub에 자동 반영
- 탭 재활성화 시 자동 불러오기
- 페이지 이탈 전 미저장 데이터 강제 flush
- 데이터 형식: qcode 기반 JSON (`data.nfo`)

---

## 메모 (memo.html) — 부가 기능

### 기본 구조
- qcode 기반 식별자 (`연도w주차요일v시간10분블록`, 예: `26w271v194`)
- 탭: 작성 / 타임라인 / 주차 / 검색
- 주차별 타임라인 뷰, 키워드 검색 + 하이라이트
- 인라인 텍스트 편집
- 특수문자 삽입 (✶ ✦ ✧ ✬ ⛧)

### 첨부 파일
- **이미지**: 인라인 표시, 클릭 시 라이트박스. PNG → JPEG 자동 변환 (Canvas API, 91% 품질). 파일명 qcode 자동 부여
- **오디오**: 인라인 플레이어. 원본 파일명 표시. 파일명 더블클릭 수정 가능
- **PDF**: 아이콘 + 파일명 링크. 클릭 시 크롬 PDF 뷰어로 새 탭 오픈. 파일명 더블클릭 수정 가능
- 첨부 파일 개별 삭제 (✕)
- 저장 경로: `raw/{qcode}_{번호}.{확장자}`

---

## 2026-06-29 개선 (메모 기능)

- **PDF 첨부 추가**: 이미지·오디오에 이어 PDF 첨부 지원. 아이콘+파일명 표시, 클릭 시 크롬 PDF 뷰어로 오픈
- **오디오 로드 방식 전환**: `raw.githubusercontent.com` 직접 src → GitHub API fetch → Blob URL 주입 방식으로 변경. 브라우저의 1MB 부분 로드 문제 해결
- **대용량 파일 처리**: 1MB 이하는 Contents API, 초과 시 Git Blobs API로 자동 fallback (오디오·PDF 공통)
- **파일명 표시**: 오디오·PDF 모두 업로드 원본 파일명 표시 (`origName` 필드 저장)
- **파일명 인라인 편집**: 오디오·PDF 파일명 더블클릭으로 수정 가능, 즉시 저장
- **첨부 개별 삭제**: 이미지·오디오·PDF 각각 ✕ 버튼으로 개별 삭제 가능

---

## 로컬 유틸리티

### cleanup_raw.py
`memo.nfo`와 `raw/` 폴더를 대조하여 미참조 파일을 `deleted/` 폴더로 격리.

```
python cleanup_raw.py
```

빌드:
```
pyinstaller --onefile --console cleanup_raw.py
```

### .gitignore 제외 항목
```
cleanup_raw.py
cleanup_raw.exe
deleted/
```

---

## 기술 메모

- `raw.githubusercontent.com`은 `Content-Type: application/octet-stream` 강제 → fetch 시 CORS 차단. API 경유 필수
- GitHub Contents API는 1MB 초과 파일에서 `content` 필드 비어 반환 → Git Blobs API로 대응
- `window.open('', '_blank')`를 클릭 동기 컨텍스트에서 먼저 열고, fetch 완료 후 `tab.location.href` 교체해야 팝업 차단 없이 새 탭 오픈
- 카잔고 총액은 항상 세부항목 합으로 재계산 (단일 진실 소스)

---

## 로컬 동기화

- 저장소 경로: `E:\github\wiza0ard\ledger`
- 도구: GitHub Desktop
- Pull → 로컬 작업 → Commit → Push
