# CONTEXT
Phase 2 진행 중 — 시트 구조 확정 · 코드 생성 단계

## 완료
- Phase 2 설계 → excelsheets.md 확정
- Phase 3 설계 → DESIGN/ 13개 파일 완료

## 현재 위치
Phase 2 (시트 초기화 코드 생성)
→ phase2_codegen.md 기반으로 Apps Script 작성 예정
→ 완료 후: Phase 3 UI 제작 진입

## 2026-03-29 완료
Phase 2 Python 시트 초기화 완료 — RAW_DATA/PORTFOLIO/AI_ANALYSIS 3개 시트 생성, 서식 적용 (setup_sheets.py)
Phase 3 설계 파일 전체 숙지 완료 (site_01~11.md) — index.html 코드 생성 직전 중단
Phase 3 config.js · app.js 생성 완료 — index.html 제작 단계

## 2026-04-01 완료
Phase 3 (사이트 디자인 + UI — 더미 데이터) 완료
index.html 파일 생성 및 점검 완료

## 2026-04-04 현재
Phase 6 불일치 분석 완료 → 수정 시도 후 오류 발생 → 원본 복귀. go/index.html · Code.gs 수정 보류.

## 2026-04-05 Phase 4 완료
go/app.js NLP 파서 강화: resolveDate(상대날짜→절대), validateParsed(6개 필드), _callGemini 프롬프트 갱신(market/action 추가, KR/US/JP 규칙), _renderConfirmCard market 필드 추가, save() action/market 반영

## 2026-04-09 완료
go/index_2.html: POST fetch 두 곳에서 headers(Content-Type) 제거. Code.gs e.postData.contents 방식 유지.

## 2026-04-08 완료 (3)
go/index_2.html: _callGemini → GAS doPost("parseNLP") 중계 방식으로 교체. Gemini 직접 호출 제거.

## 2026-04-08 완료 (2)
go/index_2.html: apikey.js script 태그 삭제, CONFIG에서 GEMINI_API_KEY·GEMINI_MODEL 두 줄 제거 (보안 — 키 노출 방지)

## 2026-04-08 완료
phase4_nlp.md 보안 구조 변경: Gemini 직접 호출 → GAS doPost("parseNLP") 중계 방식으로 수정. API 키 PropertiesService 격리.
Code.gs doPost에 case 'parseNLP' 추가 + parseNLP() 함수 구현 (Gemini 2.5-flash-lite, PropertiesService 키 로드).

## 2026-04-06 완료
go/index_2.html Step3 확인카드 더미 데이터(날짜/종목/티커/수량/단가/총액/거시경제) 제거
go/index_2.html Step3 카드에 id="tab1-confirm-card" 추가, step3-btn → Tab1.save() 연결 (AI 파싱 결과 표시 + 포트폴리오 저장 연동)

