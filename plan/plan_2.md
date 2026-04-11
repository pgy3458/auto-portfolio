# City Making — 탭별 개선 플랜

## Context
현재 index_2.html(단일 파일 SPA) + Code.gs(Apps Script) 구조.
사용자가 탭별로 기능 추가·디자인 수정을 요청. 탭 하나씩 수정 → 확인 → 다음 탭 순서.

---

## 수정 대상 파일 요약

| 탭 | HTML | Code.gs |
|---|---|---|
| 거래 입력 | ✅ 수정 | ❌ 불필요 |
| 포트폴리오 | ✅ 수정 | ✅ 수정 |
| AI 분석 | ✅ 수정 | ❌ (현재 저장 데이터 구조 유지) |

---

## Phase A — 거래 입력 탭 (Tab1)
**파일:** `go/index_2.html` 만 수정

### 1. 진행바 버그 수정
- **버그 원인:** `Tab1.setStep()` (line 1394)이 `#tab1-progress` 컨테이너 자체의 `style.width`를 변경 → 전체 길이가 늘어나는 현상
- **수정:** `Tab1.setStep()` 내 width 라인 제거, `.seg1~seg4`에 `.done` 클래스 토글로 교체
  ```js
  // 기존 (잘못됨)
  if (bar) bar.style.width = `${n === 4 ? 100 : ((n - 1) / 3) * 100}%`;
  // 수정
  for (let i = 1; i <= 4; i++) {
    document.getElementById('seg' + i)?.classList.toggle('done', i <= n);
  }
  ```
- **CSS 보강:** 진행바 height 3px → 6px, 색상 대비 강화

### 2. 거시경제 스냅샷 항목 삭제
- `Tab1._renderConfirmCard()` 내 `macro-row` div 블록 제거
- `_loadMacroSnapshot()` 호출 라인도 삭제

### 3. 이중클릭 방지 + 로딩창
- `submitInput()`: 버튼 즉시 `disabled = true` (setStep(2) 전에)
- `save()`: 버튼 ID 오참조 수정 → `tab1-save-btn` → `step3-btn`
- Step2 로딩 패널은 이미 존재, 스피너 CSS 정상

### 4. 단가·총금액 천 단위 표시
- `cf-price`, `cf-total` input → `type="text"`로 변경
- input 이벤트에 콤마 포맷터 적용
- `save()` 페이로드에서 콤마 제거 후 Number 변환

### 5. 메모 textarea 확대
- `rows="4"` → `rows="6"` + `min-height:120px`

### 6. UNKNOWN 저장 방지
- 종목명·티커·수량·단가·총금액 중 하나라도 0 또는 빈값(unknown)이면 저장 중단
- `save()` 실행 전 유효성 검사: 실패 시 별도 경고 패널 표시 ("입력값을 확인해주세요")
- 패널 내 [다시 입력하기] 버튼 클릭 → Step 1(초기 입력 화면)으로 복귀
- 목적: 0원·더미 데이터가 시트에 저장되는 것 방지

---

## Phase B — 포트폴리오 탭 (Tab2)
**파일:** `go/index_2.html` + `phase 6/Code.gs`

### 1. 데이터 로딩 표시
- HTML에 `id="tab2-loading"` 스피너 div 추가
- `Tab2._showLoading()` 로직은 이미 구현됨 (엘리먼트만 없었음)

### 2. 새로고침 강제 갱신
- `loadTab2()` 함수에 `Cache.invalidate('portfolio')` 추가

### 3. 화폐별 자산 탭 (₩KRW / $USD / ¥JPY)
- **엑셀 시트 변경 불필요** — `RAW_DATA` market 컬럼(r[3]: KR/US/JP)으로 화폐 구분 가능
- **Code.gs `getPortfolio()`:** `costByMarket: { KR, US, JP }` 집계 추가 (return에 포함)
- **HTML:** 총자산 카드 아래 탭 버튼 3개 추가
- **JS:** 탭 클릭 시 해당 화폐 종목 필터링

### 4. 국가별 보유 종목 필터
- 보유 종목 테이블 상단 필터 버튼 (전체 / 한국 / 미국 / 일본)

### 5. 퀀트 지표 섹션 삭제
- HTML 퀀트 지표 섹션 제거 (line 524~545)
- `Tab2._renderQuant()` 호출 제거

### 6. 화폐 기호 표시 (사이트 전체)
- 보유 종목·거래 이력: market 기준 ₩/$/¥ 기호 적용

---

## Phase C — AI 분석 탭 (Tab4)
**파일:** `go/index_2.html` 만 수정

### 1. 현재 오류 원인
- HTML 요소 ID ↔ `Tab4` JS 내 참조 ID 불일치
- Code.gs 응답 필드(`quant`, `behavior`, `aiComment`) ↔ `Tab4._render()` 기대 필드(`quantSummary`, `behavioral`, `aiAnalysis`) 불일치
- **수정:** Tab4 HTML 재작성 + `Tab4._render()` 필드명 매핑 수정 (Code.gs 변경 없음)

### 2. 삭제 항목
- "퀀트 심화" 테이블 섹션
- "AI 종합 분석" 카드 섹션
- `loadTab4()` 전역 함수 (Tab4 객체로 통합)
- 탭 진입 시 자동 분석 제거 (`TabManager.switchTo()` loaders에서 `tab4` 삭제)

### 3. 개별 분석 버튼 구조
아래 4개 섹션 + 전체 분석 버튼 1개:

| 섹션 | 버튼 | 계산 방식 |
|---|---|---|
| VaR (Value at Risk) | [VaR 계산] | 포트폴리오 변동성 기반 JS 계산 |
| Fat-Tail Risk | [Fat-Tail 분석] | 거래 이력 첨도(Kurtosis) JS 계산 |
| 도파민 과부하 점수 | [분석] | 최근 30일 매매횟수 vs 수익률 상관관계 |
| 자산 상관관계 매트릭스 | [분석] | 보유 종목간 상관계수 JS 계산 |
| 전체 분석 버튼 | [전체 분석] | 위 4개 순차 실행 |

> **API 비용 없음:** 4개 분석 모두 보유 데이터 기반 JS 계산. Gemini 호출 없음.

### 4. `Tab4._render()` 필드명 수정
```js
// 수정: Code.gs 실제 응답 필드명에 맞춤
this._renderBehavioralWarnings(data.behavior);  // behavioral → behavior
// quantSummary, aiAnalysis 섹션 삭제이므로 해당 호출 제거
```

---

## 작업 순서

```
Phase A (거래 입력) → 브라우저 확인
Phase B (포트폴리오) → 브라우저 확인
Phase C (AI 분석) → 브라우저 확인
```

---

## 검증 체크리스트

### Phase A
- [ ] 진행바 4단계 1/4→2/4→3/4→4/4 정확히 표시
- [ ] 버튼 클릭 → 즉시 비활성화 + 로딩 진입
- [ ] 단가·총금액 콤마 구분 표시
- [ ] 거시경제 스냅샷 행 없음
- [ ] 메모 입력창 충분히 넓음

### Phase B
- [ ] 탭/새로고침 클릭 → 로딩 스피너 표시
- [ ] 화폐 탭 클릭 → 해당 화폐 종목만 표시
- [ ] 국가 필터 버튼 동작
- [ ] 퀀트 지표 섹션 없음
- [ ] 보유 종목 금액에 ₩/$¥ 기호

### Phase C
- [ ] AI 분석 탭 진입 시 자동 API 호출 없음
- [ ] 각 버튼 클릭 시 해당 분석 실행·결과 표시
- [ ] 전체 분석 버튼 동작
- [ ] 퀀트 심화/AI 종합 분석 섹션 없음
