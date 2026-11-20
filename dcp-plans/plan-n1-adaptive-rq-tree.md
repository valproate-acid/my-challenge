# Plan N1 (1순위 · 최고 ROI): `RQ-D4-Adaptive` & `RQ-D64-Adaptive`

## 1. 배경 및 전략적 근거

- 현재 `yukon.org/dcp`에서 비표지(unflagged) 노이즈 모델의 최대 경쟁 리그(주전장)는 **`Adaptive malformed-copy model`**(유효 273건, 프론티어 138건)이다.
- 우리 최신 제출 `cf6cb036`(`RQ-D64-Balanced`)이 속한 `ModelM`(`Online unflagged replacement model`)은 전체 8건뿐이지만, 두 모델의 수학적 적대자 능력은 사실상 동치다.
- **`Adaptive malformed-copy model`의 `poly-S`($S_C=O(n^2), S_Q=O(n)$) 현재 1위**는 `GordoAR`의 `75db891b`(`PSG-D4`)로, 스펙이 고작 아래 수준에 머물러 있다:
  - $T = O\!\left(2^{n/8} n^{9/2} \log_2 n\right)$, $S_C = O(n^2)$, $S_Q = O(n)$, $Q = O(n^5)$, $\epsilon(n) = \frac{1}{60129542144 n^4}$
- 우리 `RQ-D64-Balanced`의 증명 골격(불균등 블록폭 + 인과적 first-edit coupling + $n=1$ 마팅게일 디코더)을 `Adaptive malformed-copy model`로 이식하면, **가장 적은 공수로 주전장 273건 리그에서 압도적 프론티어 돌파**가 가능하다.

---

## 2. 목표 스펙 (2-포인트 동시 공략)

### 포인트 1-A: `RQ-D4-Adaptive` (깊이 4 — `75db891b` 직접 엄격 지배)
| 자원 | `75db891b` (`GordoAR`, 현 1위) | **`RQ-D4-Adaptive` (목표)** | 비교 결과 |
| :--- | :--- | :--- | :--- |
| $T$ | $O(2^{n/8} n^{9/2} \log_2 n)$ | **$O(2^{n/10} n^{33/10} \log_2 n)$** (또는 $n^{33/10} L^2$) | **엄격 우위** (`1/10 < 1/8`) |
| $S_C$ | $O(n^2)$ | **$O(n^2)$** | 동률 |
| $S_Q$ | $O(n)$ | **$O(n)$** | 동률 |
| $Q$ | $O(n^5)$ | **$O(n^5)$** | 동률 |
| $\epsilon(n)$ | $\frac{1}{60129542144 n^4}$ | **$\frac{3333}{10000 n^4}$** (옵션: $\frac{1}{3 n^4}$) | **약 $2 \times 10^{10}$배 엄격 우위** |

### 포인트 1-B: `RQ-D64-Adaptive` (깊이 64 — `Adaptive poly-S` 역대 최속 신기록)
- $T = O\!\left(2^{n/130} n^{2243/65} \log_2 n\right)$ (보수적으로 $L^2$ 유지해도 $2^{n/130} \ll 2^{n/8}$로 압도)
- $S_C = O(n^2)$, $S_Q = O(n)$, $Q = O(n^{65})$, $\epsilon(n) = \frac{3333}{10000 n^{64}}$

---

## 3. 핵심 구현 및 증명 매핑 체크리스트

1. **5번째 클레임 `error_model` 문구 정확 일치 (필수)**:
   - 파레토 비교는 `error_model` 문자열이 바이트 단위로 동일해야 같은 모델로 묶인다.
   - 아래 `Adaptive malformed-copy model` 표준 문구를 그대로 사용한다:
     > `Adaptive malformed-copy model with fraction epsilon(n). Parameters: N >= 2 is the modulus, s in Z/NZ the secret, n = ceil(log2 N), Q the total number of source copies the algorithm acquires, and epsilon(n) the tolerated malformed fraction. Copies are delivered one at a time in acquisition order j = 1, ..., Q. Before copy j is delivered, an adversary that knows N, s, the full description of the algorithm, and every measurement outcome the algorithm obtained before copy j is delivered, and that may hold, process and measure an arbitrary private quantum system, decides adaptively whether copy j is malformed; on every execution branch at most floor(epsilon(n) Q) of the Q acquired copies are malformed. A malformed copy is an arbitrary (n+1)-qubit state (the register shape of an ideal copy) supplied from the adversary's system: it may be pure or mixed, may be entangled with the adversary's remaining system and with other malformed copies, and may depend on all of the information above; it is not entangled with clean copies or with the algorithm's workspace, ancillas or internal random bits. Every copy that is not malformed is clean: at the moment of its delivery, a fresh ideal coset state rho_s = (1/N) sum_x |psi_{x,s}><psi_{x,s}| in tensor product with everything else (the adversary's system, all other copies and the algorithm's workspace), i.e. clean copies are independent ideal coset states. The adversary interacts with the algorithm only by supplying copies; after delivery it has no access to the copy or to the algorithm's quantum or classical systems. All measurements have countably many outcomes. The algorithm is not told which copies are malformed. Quantifiers: the guarantee must hold for every N >= 2 (no validity threshold), every s in Z/NZ and every such adversary, including the adversary that corrupts nothing (the clean problem): the algorithm outputs the entire s with probability at least 2/3, and the four declared resource bounds hold on every execution branch. The fraction bounds the number of malformed copies among acquired copies; it is not a per-copy corruption probability.`
2. **인과적 결합 호환성 검증**:
   - `cf6cb036`의 `first-edit coupling`은 (i) 교체 결정이 현재 fresh honest draw 이전에 이루어지고, (ii) clean copy가 이전 모든 시스템과 텐서 독립인 $\rho_s$이며, (iii) 모든 분기에서 실제 교체 수 $\le \lfloor \epsilon(n) Q \rfloor$라는 조건만 사용한다.
   - `Adaptive malformed-copy model`은 이 세 조건을 모두 만족하며, 오히려 적대자가 알고리즘 내부 랜덤 비트에 접근하지 못하므로 `ModelM`보다 조건이 더 깔끔하다.
3. **`L = ceil(log2 n)` 이중 과금 제거 (권장)**:
   - `cf6cb036`에서 보수적으로 남겨둔 `L^2`를 `L`로 축소한다(Grover 1회 반복 $O(n^2)$ 게이트에 주소 비용 $O(L)$을 한 번만 곱하고, `QSEARCH`의 $h$를 $2L$비트만큼 조정).
4. **깊이 $d=4$ 전용 파라미터 및 유한 구간 검증 (`RQ-D4-Adaptive`)**:
   - $d=4$에서는 $\sum_{\ell=1}^4 b_\ell + h \ge n$이므로 지수 계수가 $\frac{1}{2(4+1)} = \frac{1}{10}$이 된다.
   - 유한 구간 분기: $n=1$($Q_1=2^{31}$ 마팅게일), $2 \le n < n_{\text{exh}}$(소진 탐색: $\frac{3333}{10000 n^4} \cdot 2^n(n+8) < 1$이 성립하는 범위만 — $d=4$에서는 $n=2, \dots, 9$ 수준이므로 주의! **$n \ge 10$부터는 즉시 `TREE4`를 가동**하거나 중간 유한 구간용 고정 깊이 노이즈 허용 프로토콜을 명시해야 함).
   - **주의**: $d=64$와 달리 $d=4$에서는 $n^4$가 작으므로 `2^n (n+8) < n^4`가 성립하는 구간이 매우 좁다. 반드시 `fractions.Fraction` 스크립트로 모든 $n \ge 2$에서 $B_{\text{bad}} \le K/128$을 검증해야 한다.
5. **보너스 옵션 — 정확 $\epsilon = \frac{1}{3 n^d}$ 업그레이드 검토**:
   - `96a945cd`(`0xplemonade`) / `38d66dc0`(`mpjunior92`)의 `RQ-SPLIT`을 참고해 $n=1$에서 정확 `1/3`을 달성할 수 있는지 확인(불필요한 복잡도 증가 시 `3333/10000`으로 빠르게 먼저 제출).

---

## 4. 의존성 선언

- `cf6cb036-2b0b-4b21-8781-1965f3390693` (`RQ-D64-Balanced`) 및 `6a56e70e-a4f2-4ee2-b072-aa09b0313ddf` (`RQ-D32`)를 `--dependencies-file`에 정확히 선언한다.
