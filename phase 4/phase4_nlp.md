# Phase 4 — NLP 파서 개발 설명서

## 보안 아키텍처 (변경)
Gemini API 키 노출 방지를 위해 **HTML/JS 직접 호출 금지**.
호출 경로: `app.js` → `Code.gs (Apps Script doPost)` → `Gemini API`
- API 키는 Apps Script 내 `PropertiesService`에만 저장
- `_callGemini()`는 GAS 웹앱 URL로 `fetch(POST)` 요청만 수행

## 수정 대상
- `go/app.js` → `Tab1._callGemini()` (line 291), `Tab1.submitInput()` (line 273)
- `go/Code.gs` → `doPost(e)` 핸들러에 `action: "parseNLP"` 분기 추가

## JSON 스키마 (확정)
```json
{
  "date": "YYYY-MM-DD",   // 필수
  "name": "종목명",        // 필수
  "ticker": "티커|UNKNOWN",
  "market": "KR|US|JP",   // 필수 (신규)
  "action": "매수|매도",   // 필수
  "qty": 수량,             // 필수, number
  "price": 단가,           // 필수, number
  "total": 총금액,         // number
  "asset": "주식|ETF|채권|현금|기타"
}
```

## 날짜 전처리 — `resolveDate(text)`
`submitInput()` 호출 전 실행. "오늘"→today, "어제"→today-1, "N일 전"→today-N 변환 후 Gemini 전달.

## 유효성 검증 — `validateParsed(obj)`
필수 필드(`date/name/market/action/qty/price`) 누락 시 `_showParseError("필드명 누락")` 호출 → Step 1 복귀.

## 오류 분기
| 케이스 | 처리 |
|--------|------|
| API 오류 | `ErrorLog.add` + Step 1 복귀 |
| 필드 누락 | 재입력 요청 메시지 표시 |
| JSON 파싱 실패 | 재시도 1회 후 오류 탭 기록 |

## 프롬프트 강화 포인트
- 한/영 혼용 예시 추가 (예: "Apple 10주 $180에 매수")
- 시장 구분 규칙 명시 (KR: 한국 종목, US: 미국, JP: 일본)
- 상대 날짜는 전처리 후 절대 날짜로 변환하여 전달

## Apps Script 연동 명세

### `app.js` — `_callGemini(text)`
```js
// GAS 웹앱 URL은 config.js의 GAS_URL 상수 사용
const res = await fetch(GAS_URL, {
  method: "POST",
  headers: { "Content-Type": "application/json" },
  body: JSON.stringify({ action: "parseNLP", text })
});
const json = await res.json();
// json.result → Gemini 파싱 결과 JSON 문자열
```

### `Code.gs` — `doPost(e)` 분기 추가
```js
case "parseNLP":
  return ContentService
    .createTextOutput(JSON.stringify({ result: _callGeminiInternal(payload.text) }))
    .setMimeType(ContentService.MimeType.JSON);
```
- `_callGeminiInternal(text)`: API 키를 `PropertiesService.getScriptProperties().getProperty("GEMINI_KEY")`로 로드 후 Gemini REST 호출
- 응답은 JSON 문자열 그대로 `result` 필드에 래핑하여 반환
