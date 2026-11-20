# Plan N3 (3순위 · 신규 프론티어 축 개척): `RQ-D64-VirtSC` (Qubit-Virtualized $S_C = O(n)$)

## 1. 배경 및 전략적 근거

- 현재 `ModelM` 프론티어 6건(`cf6cb036`, `6a56e70e`, `8ecae89b`, `65487ca7`, `5821de8d`, `04e5fcfe`)은 **전부 $S_C = O(n^2)$, $S_Q = O(n)$**에 묶여 있다.
- 반면 `CLEAN` 1위 `kaankolcu`(`53c84f6f`)와 `Adaptive` 상위권 `mpjunior92`(`a8e4fa5a`, `ad80a9c1`)는 **고전 메모리 $S_C = O(n)$**을 달성하며 독자적인 파레토 축을 장악했다.
- 특히 `Adaptive`의 `ad80a9c1`(`mpjunior92`)은 $T=2^{n/6}, S_C=O(n), S_Q=O(n^2), Q=O(n^4)$로, **고전 메모리와 양자 메모리의 차수를 맞바꾸는 전략**이 호스티드 심사(`dcp-uniform-clifford-t-bit-operations-v1`)에서 완전히 유효함을 입증했다.

---

## 2. 핵심 기법: Basis-State Qubit Virtualization

`RQ-D64-Balanced`에서 고전 메모리 $S_C = O(n^2)$가 발생하는 지점은 정확히 두 곳이다:
1. **깊이 우선 스택**: 최대 64개 레벨 × 형제 노드 라벨($O(n)$ bits) $\to O(n^2)$ bits
2. **스테이지당 측정 레코드 저장**: $K = O(n)$개 레코드 $(W_i, \beta_i, Z_i)$ (각 $O(n)$ bits) $\to O(n^2)$ bits

이를 다음과 같이 변환한다:
- **기저 상태 큐빗 뱅크(Counted Basis-State Qubit Bank)**:
  - 상수 개의 $O(n)$-bit 스테이징 워드만 고전 메모리($S_C = O(n)$)에 유지한다.
  - 나머지 $O(n)$개의 $O(n)$-bit 고전 레코드(스택 프레임 및 측정 레코드)는 **계산 기저 상태 $|0\rangle, |1\rangle$로 인코딩된 $O(n^2)$개 큐빗 레지스터**에 보관한다.
- **읽기/쓰기/초기화 비용 회계**:
  - 고전 컨트롤러가 인덱싱된 큐빗 레지스터에 접근할 때 주소 비용 $\lceil \log_2(M+2) \rceil = O(\log n)$이 발생한다(규칙: `rules.md` L35-36).
  - 쓰기 전 기존 값을 측정하여 $|0\rangle$로 리셋하거나, `53c84f6f`처럼 lazy generation tag를 활용해 $O(n)$ 스테이징 워드와 교환한다.
  - 모든 게이트·측정·주소 비용이 $O(\text{poly}(n))$ 배수 이내이므로, 전체 시간의 지수부 $2^{n/130}$은 그대로 보존된다.

---

## 3. 목표 스펙

- **타깃 모델**: `ModelM` (또는 `Adaptive malformed-copy model`)
- **자원 상한**:
  - $T = O\!\left(2^{n/130} n^{c} (\log_2 n)^2\right)$ (스택/레코드 접근 오버헤드 반영 후 정확한 유리수 차수 $c$ 산출)
  - **$S_C = O(n)$ bits** (기존 $O(n^2)$ 대비 차수 1단계 절감 → **`ModelM` 최초!**)
  - $S_Q = O(n^2)$ qubits
  - $Q = O(n^{65})$ copies
  - $\epsilon(n) = \frac{3333}{10000 n^{64}}$

---

## 4. 기대 효과 및 리스크

- **기대 효과**: `ModelM`의 모든 기존 6건($S_C = O(n^2)$)과 비교 불가능한 **완전히 비어 있는 $S_C = O(n)$ 프론티어 영역**을 단독 선점한다. `Adaptive`에 적용하면 `ad80a9c1`($2^{n/6}, S_C=O(n), S_Q=O(n^2)$) 대비 지수 계수를 $1/6 \to 1/130$으로 대폭 개선한 신규 프론티어가 된다.
- **주의할 심사 포인트**:
  - "고전 컨트롤러가 큐빗에 저장된 값을 읽을 때"는 반드시 **명시적인 계산 기저 측정(computational-basis measurement) + 재준비(X 게이트)**로 구현해야 하며, 고전 메모리에 동시에 로드되는 워드 수가 상수 개($O(n)$ bits)임을 every-branch로 증명해야 한다.
  - 모든 주소 비용(`ceil(log2(S_C + S_Q + 2))`)을 누락 없이 계상해야 한다.
