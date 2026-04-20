# Claude Code 작업 명세서 — AI 전략 탭 교체

> 작업 대상: `C:\Users\LJY\Desktop\클로드코드\forcool-dashboard\index.html`
> 작업 범위: **AI 전략 탭 내부만** 교체 (다른 탭/전역 구조 건드리지 말 것)
> 참고 레퍼런스: `weekly_ad_report_v6.html`, `monthly_ad_report_v2.html`

---

## 1. 작업 원칙

1. **기존 사이트 전역 디자인/레이아웃을 건드리지 말 것**
   - 사이드바, 탭바, 헤더, 푸터 그대로
   - 다른 탭(대시보드, 키워드, 캠페인 등) 그대로
2. **AI 전략 탭 내부 DOM만 교체**
   - 기존 `❶종합진단~❻예상효과` 6섹션 삭제
   - 신규 "주간/월간 보고서" 구조로 대체
3. **기존 함수 재활용**
   - `getKpiData()`, `getCampData()`, `getKwData()` 그대로 사용
   - `/api/ai` Worker 엔드포인트 재활용 (포비서 의견용)

---

## 2. 교체 대상

### 2-1. HTML — AI 전략 탭 섹션

**삭제**: 기존 `runAi()` 함수가 렌더링하던 6개 섹션 전체

**신규 구조 (주간/월간 탭 전환)**:
```html
<div id="aiTab">
  <!-- 상단 컨트롤 -->
  <div class="ai-controls">
    <div class="ai-toggle">
      <button class="active" onclick="switchReport('weekly')">📊 주간</button>
      <button onclick="switchReport('monthly')">📅 월간</button>
    </div>
    <div class="ai-actions">
      <select id="periodSelect"><!-- 주차/월 선택 --></select>
      <button onclick="window.print()">📄 PDF 저장</button>
    </div>
  </div>

  <!-- 보고서 렌더링 영역 -->
  <div id="reportContainer"></div>
</div>
```

### 2-2. JS — runAi 함수 교체

기존 `runAi()` 함수를 아래 구조로 재작성:

```js
async function renderWeeklyReport() {
  // 1) 기존 데이터 함수에서 값 가져오기
  const kpi = getKpiData();
  const camp = getCampData();

  // 2) 주간 data 객체 구성 (weekly_ad_report_v6.html 참고)
  const data = {
    week: getCurrentWeekLabel(),
    period: getCurrentPeriod(),
    manager: '김선명',
    monthlyGoal: getMonthlyGoal(),       // 신규: 월 목표값 설정 필요
    currentMonthAccum: kpi.monthAccum,
    daysElapsed: getDaysElapsed(),
    daysInMonth: 30,
    media: [
      { name: '파워링크', type: '검색광고', campaigns: camp.powerLink.count,
        cost: camp.powerLink.cost, revenue: camp.powerLink.rev, ... },
      // ADVoost, 카탈로그, 쇼핑커넥트 동일 패턴
    ],
    trend: get8WeeksTrend(),
    checks: autoGenerateChecks(kpi, camp),  // 임계치 기반 자동 판정
    marketContext: '',  // 수동 입력 (옵션)
    focus: [],          // 수동 입력 또는 AI 생성
    poviserText: await fetchPoviser('weekly', data)  // /api/ai 호출
  };

  // 3) HTML 렌더링 (v6의 render() 함수 구조 그대로)
  document.getElementById('reportContainer').innerHTML = buildWeeklyHTML(data);
}

async function renderMonthlyReport() {
  // 월간도 동일 패턴 (monthly_ad_report_v2.html 참고)
  // MoM, YoY, 연간 누적, Top5/Bottom3, 이슈 회고, 다음 달 계획
}
```

---

## 3. 필수 기능 요구사항

### ✅ 주간 보고서 섹션
- [ ] 4 KPI 카드 (광고비/매출/ROAS/CPA) + WoW 자동 계산
- [ ] 월 목표 달성률 프로그레스바 + 진도율 계산 (자동)
- [ ] HIGHLIGHT 3줄 (WHAT/WHY/SO WHAT) — 일평균 필요 매출 **자동 계산** 필수
  - 계산식: `(monthlyGoal - currentAccum) / (daysInMonth - daysElapsed)`
- [ ] 매체별 표 (캠페인 수 열 **필수 포함**) + WoW 자동 계산
- [ ] 8주 추이 차트 (Chart.js, 단색 계열)
- [ ] 이슈 체크리스트 (임계치 기반 자동 판정)
  - 전환 트래킹 이상치: ROAS > 10,000% 또는 ±3σ
  - 소재 피로도: CTR 2주 연속 하락 + -15% 이상
  - 전환율 변동성: CVR 편차 ±20% 초과
  - 예산-매출 효율: 예산 소진% > 매출 진도%
- [ ] 경쟁/시장 맥락 박스 (수동 입력 필드)
- [ ] 다음 주 FOCUS 3장 카드
- [ ] 포비서 1줄 의견 (`/api/ai` 호출)

### ✅ 월간 보고서 섹션
- [ ] 4 KPI 카드 + MoM/YoY **동시 표시**
- [ ] 연간 누적 달성률 프로그레스바
- [ ] EXECUTIVE SUMMARY 3줄 (달성/변화/방향)
- [ ] MoM/YoY 상세 비교 그리드
- [ ] 매체별 월간 성과 표 (캠페인 수 포함)
- [ ] 6개월 추이 차트 (이중축 bar+line)
- [ ] Top5 / Bottom3 캠페인 + **미들 비중 %** 자동 계산
  - "상위 5개 = 전체 매출의 N%, 중간 = M%, 하위 3개 = K%" 표시
- [ ] 월간 이슈 회고 (일자별)
- [ ] 다음 달 계획 4장 + **ROI 예상치 필수 포함**
  - 예: "예산 +15% → 예상 매출 +X%, 예상 ROAS Y%"
- [ ] 포비서 3문단 전략 코멘트 (`/api/ai` 호출)

---

## 4. 스타일 요구사항

### 4-1. 색상 팔레트 (절제)
```css
--bg: #ffffff;        /* 흰 배경 */
--text: #1f2937;      /* 본문 */
--navy: #1e3a5f;      /* 포인트 1개 */
--green: #047857;     /* 상승 지표만 */
--red: #b91c1c;       /* 하락 지표만 */
--amber: #b45309;     /* 경고 최소 */
```

### 4-2. 인쇄/PDF 최적화
```css
@page { size: A4; margin: 12mm 10mm; }
@media print {
  .ai-controls { display: none !important; }
  .section-box { page-break-inside: avoid; }
  body { -webkit-print-color-adjust: exact; }
}
```

### 4-3. 주간 vs 월간 시각 차별화
- 주간: 얇은 테두리 (1px)
- 월간: 두꺼운 테두리 (2px) + 좌측 엑센트 바 (3~4px navy)
- 뱃지: 주간 `WEEKLY REPORT`, 월간 `▣ MONTHLY REPORT`

---

## 5. 포비서 API 호출 (`/api/ai`)

기존 Worker 엔드포인트 재활용. 시스템 프롬프트만 분기:

### 주간용
```
당신은 10년차 네이버 광고 전문가입니다. 포쿨(스타리온 냉장고 공식 판매채널)의
주간 광고 성과를 분석합니다. 다음 데이터를 바탕으로 한 문장(3줄 이내)으로
총평을 작성하세요. "구조적", "역배분" 같은 단어는 사용하지 마세요.
팀 선배 조언자의 톤으로 작성합니다.

데이터: {JSON}
```

### 월간용
```
당신은 10년차 네이버 광고 전문가입니다. 포쿨 월간 결산을 대표에게 보고합니다.
다음 데이터를 바탕으로 3문단을 작성하세요:
1) ▸ 4월 총평 (성과 요약)
2) ▸ 매체 전략 방향 (파워링크/ADVoost/카탈로그/쇼핑커넥트)
3) ▸ 다음 달 주목 포인트 (번호 매긴 4가지)

"구조적" 단어 금지. 전략 컨설턴트 톤.

데이터: {JSON}
```

---

## 6. 10년차 광고 마스터 평가 피드백 (반영 필수)

1. ✅ **계산 오류 수정** — 일평균 필요 매출은 JS로 자동 계산, 수동 입력 금지
2. ✅ **WoW 현실화** — 실제 데이터 반영하면 자연스럽게 해결
3. ✅ **체크리스트 임계치 명시** — 판정 기준 UI에 표시
4. ✅ **캠페인 수 열 추가** — 매체별 표 필수 컬럼
5. ✅ **CVR 역전 자동 해설** — ADVoost > 파워링크 감지 시 1줄 해설 자동 생성
6. ✅ **경쟁/시장 맥락 필드** — 수동 입력 박스 추가
7. ✅ **다음 달 ROI 예상치** — 월간 계획 카드에 "예상 매출/ROAS" 필수
8. ✅ **Top/Middle/Bottom 비중 계산** — 월간 캠페인 섹션에 표시
9. ✅ **"구조적" 단어 금지** — 포비서 시스템 프롬프트에 명시
10. ✅ **주간/월간 시각 차별화** — CSS로 테두리 굵기/엑센트 바

---

## 7. 배포 체크리스트

작업 완료 후:

- [ ] 로컬에서 index.html 직접 브라우저 열어 확인
- [ ] 주간 ↔ 월간 탭 전환 정상 작동
- [ ] PDF 저장 버튼 → 인쇄 대화상자 → PDF 깔끔하게 나오는지 확인
- [ ] 기존 대시보드 탭, 키워드 탭 영향 없는지 확인
- [ ] 모바일 반응형 점검 (900px 이하)
- [ ] `window.print()` 시 버튼/컨트롤 숨김 확인
- [ ] `git push` → Cloudflare Pages 자동 배포 확인

---

## 8. 참고 파일

| 파일 | 용도 |
|---|---|
| `weekly_ad_report_v6.html` | **주간 보고서 완성 레퍼런스** (데이터-표시 분리 구조, 자동 계산 로직 포함) |
| `monthly_ad_report_v2.html` | **월간 보고서 완성 레퍼런스** (MoM/YoY, Top/Bottom 캠페인 구조) |
| 본 명세서 | Claude Code 작업 가이드 |

v6의 `data` 객체 구조와 `render()` 함수를 그대로 참고하면 됩니다. 데이터 바인딩만 실제 `getKpiData()` 결과로 교체.

---

## 9. 권장 작업 순서

1. **백업 먼저**: `index.html` 백업 복사 (안전망)
2. **기존 runAi 영역 식별**: 주석으로 교체 범위 마킹
3. **신규 HTML 구조 삽입**: AI 전략 탭 컨테이너만
4. **CSS 추가**: 기존 스타일 충돌 방지 위해 `#aiTab` 스코프로 작성
5. **JS 함수 작성**: `renderWeeklyReport()`, `renderMonthlyReport()`, `switchReport()`
6. **데이터 바인딩**: 기존 `getKpiData()` 등 연결
7. **포비서 API 연동**: `/api/ai` 호출 로직
8. **인쇄 스타일**: `@media print` 추가
9. **테스트**: 주간/월간 전환, PDF 출력, 반응형
10. **배포**: git push

---

**핵심: 기존 사이트는 그대로, AI 전략 탭만 바뀐다.**
