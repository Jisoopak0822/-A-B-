# 이커머스 구매 전환율 최적화 및 A/B 테스트 설계
**E-commerce Conversion Optimization & A/B Testing**

## 1. 프로젝트 개요

이커머스 사용자 행동 로그를 기반으로 구매 퍼널의 병목 구간을 식별하고, 전환율 개선을 위한 제품 가설과 A/B 테스트 프레임워크를 설계한 프로젝트입니다.

단순한 EDA에 그치지 않고 **Funnel Analysis → Experiment Design → Power Analysis → Hypothesis Testing → Retention Analysis → Business Decision**까지 연결하여 실제 Product Data Scientist의 분석 흐름을 구현했습니다.

- **Data Source:** Kaggle E-commerce Behavior Data from Multi-Category Store
- **분석 단위:** User / User-Product
- **Primary KPI:** Purchase Conversion Rate
- **주요 방법론:** Funnel Analysis, A/B Testing, Power Analysis, Two-Proportion Z-Test, Confidence Interval, Retention Analysis

> 원본 데이터는 실제 A/B 테스트 데이터가 아닌 관찰 로그 데이터이므로, Treatment 효과는 실험 설계 방법론을 구현하기 위한 시뮬레이션으로 구성했습니다.

---

## 2. 비즈니스 문제

사용자는 상품을 조회한 뒤 장바구니 추가 및 구매 단계를 거치지만, 모든 조회가 실제 구매로 이어지지는 않습니다.

본 프로젝트에서는 다음 질문에 답하고자 했습니다.

- 구매 여정에서 가장 큰 이탈은 어디에서 발생하는가?
- 어떤 단계가 전환율 개선의 우선순위가 되어야 하는가?
- 개선안의 효과를 검증하려면 얼마나 많은 사용자가 필요한가?
- 통계적으로 유의한 결과가 실제 비즈니스 의사결정에도 충분한가?
- 단기 구매 전환뿐 아니라 재구매 행동은 어떻게 나타나는가?

---

## 3. 데이터 처리 전략

대용량 행동 로그에서 이벤트 행을 무작위로 추출할 경우 한 사용자의 행동 순서가 훼손될 수 있으므로 **User-level Random Sampling**을 적용했습니다.

선택된 사용자의 전체 행동 로그를 유지한 뒤 다음 전처리를 수행했습니다.

- 중복 로그 확인 및 제거
- `category_code`, `brand` 결측값 별도 범주 처리
- `event_time` datetime 변환
- 날짜, 시간, 요일 파생변수 생성
- 사용자 단위 행동 테이블 구축
- 동일 사용자·동일 상품 기준 행동 순서 검증

---

## 4. Funnel Analysis

동일 사용자와 동일 상품을 기준으로 **View → Cart → Purchase** 순서를 추적했습니다.

| Funnel | Conversion Rate |
|---|---:|
| View → Cart | **4.69%** |
| Cart → Purchase | **36.98%** |
| View → Purchase | **1.74%** |

### 핵심 인사이트

가장 큰 이탈은 **View → Cart 단계**에서 발생했습니다.

장바구니에 도달한 사용자 중 상당수는 실제 구매까지 이어진 반면, 상품 조회 이후 장바구니 추가로 전환되는 비율은 상대적으로 낮았습니다.

따라서 구매 직전 단계보다 **상품 상세 페이지 및 상품 탐색 경험을 우선적인 개선 후보**로 설정했습니다.

다만 관찰 로그만으로 낮은 전환율의 원인을 특정 UI나 기능 때문이라고 단정하지 않았습니다.

---

## 5. A/B Test 설계

### Business Hypothesis

상품 상세 페이지에서 배송비 및 예상 배송일과 같은 구매 의사결정 정보를 보다 명확하게 제공하면 사용자의 불확실성을 줄이고 최종 구매 전환율을 높일 수 있을 것이라고 가정했습니다.

### Experiment

**Control**
- 기존 상품 상세 페이지

**Treatment**
- 배송비 및 예상 배송일 정보 강조
- 장바구니 CTA 인근에 핵심 구매 정보 제공

### Metrics

- **Primary Metric:** Purchase Conversion Rate
- **Secondary Metric:** View-to-Cart Conversion Rate
- **Randomization Unit:** User ID
- **Significance Level:** 5%
- **Statistical Power:** 80%

---

## 6. Power Analysis

Baseline Purchase Conversion Rate는 Funnel Analysis에서 확인한 **1.74%**를 사용했습니다.

여러 Minimum Detectable Effect(MDE) 시나리오를 비교한 후, 실험 설계 시나리오로 **15% Relative Uplift**를 설정했습니다.

- Baseline CVR: **1.74%**
- Target CVR: **약 2.00%**
- Relative MDE: **15%**
- Absolute Improvement: **약 +0.261%p**

Baseline conversion이 낮아 작은 효과를 검출하기 위해서는 비교적 큰 표본이 필요함을 확인했습니다.

---

## 7. A/B Test Simulation 결과

Power Analysis를 기반으로 사용자 단위 무작위 실험을 시뮬레이션했습니다.

| Metric | Result |
|---|---:|
| Control Users | 41,999 |
| Treatment Users | 42,495 |
| Control Conversions | 749 |
| Treatment Conversions | 852 |
| Absolute Lift | **+0.222%p** |
| Relative Lift | **+12.42%** |

Treatment 그룹에서 구매 전환율 개선이 관찰되었습니다.

---

## 8. Statistical Testing

Two-Proportion Z-Test와 95% Confidence Interval을 이용해 관찰된 전환율 차이의 불확실성을 평가했습니다.

- **Z-statistic:** 2.3618
- **P-value:** 0.0182
- **95% CI:** [+0.038%p, +0.405%p]
- **Relative Lift:** +12.42%
- **Target MDE:** +15%

`p-value < 0.05`이며 95% 신뢰구간이 0을 포함하지 않아 Treatment의 구매 전환율 증가에 대한 통계적 근거를 확인했습니다.

그러나 실제 관찰된 Relative Lift는 **12.42%**로 사전에 설정한 **15% MDE**에는 미달했습니다.

### Experiment Decision

**Statistically significant, but below business MDE**

통계적으로 유의하다는 이유만으로 바로 Full Rollout하지 않고, 비즈니스 효과 크기까지 함께 고려했습니다.

---

## 9. Retention Analysis

구매 사용자의 첫 구매 이후 재구매 행동을 분석했습니다.

분석 종료일에 가까운 사용자가 재구매하지 않은 것으로 잘못 분류되는 문제를 줄이기 위해 **Right Censoring**을 고려하고, 충분한 관찰 기간이 확보된 사용자만을 대상으로 계산했습니다.

- 전체 Repeat Purchase Rate: **37.83%**
- **7일 이내 재구매율: 15.25%**
- **14일 이내 재구매율: 26.32%**
- 30일 이내 재구매율: **관찰 기간 부족으로 제외**

이를 통해 단기 Purchase Conversion뿐 아니라 장기 고객 행동도 함께 고려했습니다.

---

## 10. Business Recommendation

### 1. View → Cart 구간 우선 개선

상품 조회 이후 장바구니 추가 단계에서 가장 큰 이탈이 나타났으므로 다음 요소를 우선적인 실험 후보로 제안했습니다.

- 배송 정보 가시성
- 가격 및 할인 정보 구조
- 상품 상세 정보
- CTA 위치 및 표현

### 2. 즉시 Full Rollout하지 않음

Treatment는 통계적으로 유의한 개선을 보였지만 사전에 설정한 15% MDE에는 미달했습니다.

따라서 즉시 전체 사용자에게 적용하기보다 **Treatment 개선 후 재실험**을 권고했습니다.

### 3. 세그먼트별 효과 분석

전체 평균 효과뿐 아니라 다음 그룹별 Treatment Effect를 추가 확인할 필요가 있습니다.

- 신규 / 기존 사용자
- 상품 카테고리
- 가격대
- 사용자 행동 수준

### 4. 장기 KPI 함께 모니터링

향후 실제 실험에서는 Purchase Conversion Rate뿐 아니라 7일·14일 재구매율도 함께 모니터링하여 단기 전환 개선이 장기 고객 가치로 이어지는지 검증해야 합니다.

---

## 11. 프로젝트 한계

- 원본 데이터는 실제 A/B 테스트 데이터가 아닌 관찰 데이터입니다.
- Treatment 효과는 실제 제품 변경의 결과가 아닌 시뮬레이션입니다.
- 배송비, UI 구성, 디바이스 정보 등 실제 제품 실험에 필요한 일부 변수가 존재하지 않습니다.
- 분석 기간이 약 1개월로 제한되어 장기 Retention 분석에 한계가 있습니다.
- MDE는 실제 기업의 비용 및 ROI 정보가 없는 상태에서 시나리오 분석 목적으로 설정했습니다.

---

## 12. 프로젝트를 통해 보여준 역량

- User-level / Product-level 행동 로그 처리
- Conversion Funnel 설계
- KPI 및 실험 지표 정의
- A/B Test Design
- Statistical Power & Sample Size Analysis
- Two-Proportion Z-Test
- Confidence Interval 및 Effect Size 해석
- Statistical Significance와 Business Significance 구분
- Right Censoring을 고려한 Retention Analysis
- 분석 결과를 비즈니스 의사결정으로 연결
