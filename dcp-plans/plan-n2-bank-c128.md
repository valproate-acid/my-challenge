# Plan N2 (2순위 · 최고 임팩트): `BANK-C128` (`ell = ceil(128 log2 n)`)

## 1. 배경 및 전략적 근거

- `Adaptive malformed-copy model`(273건) 전체에서 **시간 1위**를 차지하고 있는 것은 고정 깊이 $2^{cn}$ 계열이 아니라, **Bank-first log-depth collimation**으로 $T = 2^{c \frac{n}{\log_2 n}}$을 달성한 계열이다.
- 현재 1위와 2위의 결정적 차이를 비교하면 **명확한 공략 지점**이 보인다:

| 항목 | 1위 `e78ad51d` (`akki-rock05`) | 2위 `edca4431` (`kshitij-hash`) | **Plan N2: `BANK-C128` (목표)** |
| :--- | :--- | :--- | :--- |
| 깊이 규칙 $\ell$ | `128 * ceil(log2 n)` (구형) | **`ceil(100 * log2 n)` (신형)** | **`ceil(128 * log2 n)` (신형)** |
| $T$ | $O(2^{\frac{1}{128}\frac{n}{\log n}} n^{133} L^3)$ | $O(2^{\frac{1}{100}\frac{n}{\log n}} n^{105} L^3)$ | **$O(2^{\frac{1}{128}\frac{n}{\log n}} n^{133} L^3)$** |
| $S_C$ | $O(2^{\frac{1}{128}\frac{n}{\log n}} n L^2)$ | $O(2^{\frac{1}{100}\frac{n}{\log n}} n L^2)$ | **$O(2^{\frac{1}{128}\frac{n}{\log n}} n L^2)$** |
| $S_Q$ | $O(n^{131})$ | $O(n^{103})$ | **$O(n^{131})$** |
| $Q$ | $O(n^{130})$ | $O(n^{102})$ | **$O(n^{130})$** |
| $\epsilon(n)$ | $\frac{\log_2 n}{3.49 \times 10^{39} n^{129}}$ (붕괴) | **$\frac{25 \log_2 n}{64 n^{101}}$** | **$\Theta\!\left(\frac{\log_2 n}{n^{129}}\right)$ (예: $\frac{32 \log_2 n}{64 n^{129}} = \frac{\log_2 n}{2 n^{129}}$ 수준)** |

- **핵심 메커니즘**:
  - 구형 규칙 $\ell = g \lceil \log_2 n \rceil$은 최악의 경우 $2^\ell < 2^g n^g$가 되어 루트당 카피 수 $m^* = 2^{D^*} R_0$ 상한에 $2^{g-1}$의 불필요한 손실 인자가 붙는다($g=128$에서 $2^{127} \approx 1.7 \times 10^{38}$배 손실!).
  - 반면 신형 규칙 $\ell = \lceil g \log_2 n \rceil$(즉 $2^\ell \ge n^g$인 최소 정수, `bit_length(n^g - 1)`로 정확 계산)은 $2^\ell < 2 n^g$를 보장하므로, **동일한 $T, S_C, S_Q, Q$ 점근 상한을 유지하면서 $\epsilon(n)$을 약 $10^{38}$배 개선해 `e78ad51d`를 단번에 엄격 파레토 지배**할 수 있다.

---

## 2. 두 가지 전개 경로

### Route 2A: `Adaptive malformed-copy model`에서 `e78ad51d` 직접 격파
- `edca4431`(`BANK-C100`)의 아키텍처와 증명(Lemma F4', F6, Section 9)을 $g=128$(또는 $g=130$)로 확장한다.
- **`g=128`**: `e78ad51d`와 $T, S_C, S_Q, Q$가 완전히 같고 $\epsilon(n)$만 $\sim 10^{38}$배 크므로 **엄격 파레토 지배** 성립.
- **`g=129` 또는 `g=132`**: `e78ad51d`보다 시간 지수 계수 자체를 더 줄여 **`Adaptive` 전체 시간 단독 1위**를 새로 경신.

### Route 2B: `ModelM` (`Online unflagged replacement model`) 최초의 $2^{c n / \log_2 n}$ 개척
- 현재 `ModelM` 프론티어 6건은 모두 $2^{c n}$ 계열이다.
- `BANK-C`의 bank-first 구조(먼저 $Q_{\text{bank}}$ 카피를 모두 획득한 뒤 고정된 disjoint-leaf 이진/다중 콜리메이션 트리로 처리)는 온라인 교체 모델 `ModelM`에서도 그대로 성립한다(각 원본 카피가 최대 1개 루트에만 기여하므로 fan-out = 1, 따라서 $B_{\text{bad}} \le \epsilon(n) Q_{\text{bank}}$가 전체 루트 중 최대 $\epsilon(n) Q_{\text{bank}}$개만 오염시키며, $\epsilon m^* \le 1/64$이면 오염 루트 비율 $\le 1/64$).
- 적당한 깊이(예: $g=16$ 또는 $g=64$)로 `ModelM`에 적용하면 **`ModelM`의 모든 기존 $2^{cn}$ 제출물 대비 시간 점근 차원이 한 단계 낮은 $2^{\frac{1}{g}\frac{n}{\log_2 n}}$ 최초 기록**을 세울 수 있다.

---

## 3. 증명 시 필수 확인 항목

1. **Lemma F4' 상수 재유도 ($g=128$)**:
   - $n < 4096$ (depth 3 분기): $m^* \le 28$ ($n \le 7$), $m^* \le 14 n$ ($8 \le n < 4096$)이 $C \cdot n^{g+1} / \log_2 n$ 이하임을 검증.
   - $n \ge 4096$: $R_0 = \max(\lceil n/\ell \rceil + v, 2v + 2)$ 두 분기 각각에서 $\frac{L \cdot m^*}{n^{g+1}}$의 단조감소성과 $L=12$($n=4096$)에서의 최대값 $C$를 정확 유리수 연산으로 계산.
2. **Lemma F6 ($\epsilon m^* \le 1/64$) 연결**:
   - $\epsilon(n) = \frac{\log_2 n}{64 C n^{g+1}}$로 두면 모든 $n \ge 2$에서 $\epsilon(n) m^* \le 1/64$ 성립.
3. **$\ell = \lceil g \log_2 n \rceil$ 계산 비용 계상 (Section 9)**:
   - $n^g$ 계산을 반복 제곱(binary exponentiation) 또는 $g-1$회 곱셈으로 명시하고, 그 비트 연산량 및 메모리 비용을 $T, S_C$에 포함.
4. **의존성 선언**:
   - `edca4431` (`BANK-C100`, `kshitij-hash`)의 원본 파일(commit SHA 확인)을 `--dependencies-file`로 정직하게 선언하고, 모든 보조정리를 본 제출물 `submission/proof.md` 안에 완결적으로 재수록한다.
