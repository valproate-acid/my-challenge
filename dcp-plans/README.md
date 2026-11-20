# Yukon DCP (`mooselumph/dcp/main`) 개선 플랜 모음

- **작성일**: 2026-11-20
- **기준 데이터**: 공개 스냅샷([balanced-public-final.json](file:///Users/a/개발/my-challenge/dcp/.work/balanced-public-final.json), 전체 1,790건 / 유효 `dcp-bounds-v4` 1,631건)
- **기준 기존 제출 (`valproate-acid`)**:
  - `dd8f2f3b` (`RWT-S16`, Fixed-set): $T=2^{n/2} n^2 L^2, S_C=n, S_Q=n^2, Q=n, \epsilon=1/16$ (동 모델 상위권에 지배됨)
  - `a8f6efa7` (`RQ-D64`, `ModelM`): $T=2^{n/130} n^{66} L^2, S_C=n^2, S_Q=n, Q=n^{65}, \epsilon=\frac{1}{4 n^{64}}$ (`cf6cb036`에 지배됨)
  - `cf6cb036` (`RQ-D64-Balanced`, `ModelM`): $T=2^{n/130} n^{2243/65} L^2, S_C=n^2, S_Q=n, Q=n^{65}, \epsilon=\frac{3333}{10000 n^{64}}$ (**`ModelM` 시간 1위**)

---

## 1. 공개 리더보드 모델별 상위권 요약

| 모델 (유효 건수 / 프론티어) | 대표 상위 제출 | $T$ | $S_C$ | $S_Q$ | $Q$ | $\epsilon(n)$ |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **CLEAN** (`1,199` / `756`) | `53c84f6f` (`kaankolcu`) | $2^{\frac{2}{\sqrt{3}}\sqrt{n}-\dots}$ (`0.5`) | $n$ | $2^{1.1547\sqrt{n}}$ | $2^{1.1547\sqrt{n}}$ | $0$ |
| **CLEAN (`poly-S`)** | `cda63ed3` (`mpjunior92`) | $2^{\frac{1}{\sqrt{2}}\sqrt{n\log n}}$ (`0.5`) | $n^2$ | $n$ | $2^{\frac{1}{\sqrt{2}}\sqrt{n\log n}}$ | $0$ |
| **CLEAN (`poly-Q`)** | `59f849cd` (`mpjunior92`) | $2^{\frac{1}{16}\frac{n}{\log n}-\dots}$ (`1`) | $2^{\frac{1}{16}\frac{n}{\log n}}$ | $n$ | $n^{18}/\log n$ | $0$ |
| **Heralded / Model X** (`34` / `22`) | `10fb49fd`, `083fe31f` (`kaankolcu`) | $2^{\frac{2}{\sqrt{3}}\sqrt{n}-\dots}$ (`0.5`) | $n$ | $2^{1.1547\sqrt{n}}$ | $2^{1.1547\sqrt{n}}$ | $\approx 1$ |
| **Adaptive malformed-copy** (`273` / `138`) | `e78ad51d` (`akki-rock05`) | $2^{\frac{1}{128}\frac{n}{\log n}} n^{133} L^3$ | $2^{\frac{1}{128}\frac{n}{\log n}} n L^2$ | $n^{131}$ | $n^{130}$ | $2.87\times 10^{-40}\frac{L}{n^{129}}$ |
| 〃 | `edca4431` (`kshitij-hash`) | $2^{\frac{1}{100}\frac{n}{\log n}} n^{105} L^3$ | $2^{\frac{1}{100}\frac{n}{\log n}} n L^2$ | $n^{103}$ | $n^{102}$ | $\frac{25 L}{64 n^{101}}$ |
| 〃 (`poly-SC`) | `a8e4fa5a` (`mpjunior92`) | $2^{\frac{1}{4}\frac{n}{\log n}-\dots}$ | $n$ | $2^{\frac{1}{4}\frac{n}{\log n}}$ | $2^{\frac{1}{4}\frac{n}{\log n}}$ | $\frac{1}{2^{22} n^4}$ |
| 〃 (`poly-S` 최고) | `75db891b` (`GordoAR`) | $2^{n/8} n^{9/2} L$ | $n^2$ | $n$ | $n^5$ | $\frac{1}{6.01\times 10^{10} n^4}$ |
| 〃 (`max-eps`) | `96a945cd` (`0xplemonade`) | $2^{n/2} n L^3$ | $n^2$ | $n$ | $n$ | $1/3$ |
| **Fixed-set (`B`)** (`87` / `18`) | `6d08c829` (`mpjunior92`) | $2^{3n/8} n^{9/2}$ | $n$ | $2^{n/8} n^{9/8}$ | $n$ | $\approx \frac{2}{3n}$ |
| **Online unflagged (`ModelM`)** (`8` / `6`) | **`cf6cb036` (`valproate-acid`)** | **$2^{n/130} n^{2243/65} L^2$** | **$n^2$** | **$n$** | **$n^{65}$** | **$\frac{3333}{10000 n^{64}}$** |

---

## 2. 개선 플랜 인덱스 (우선순위 순)

| 우선순위 | 파일 | 플랜명 | 타깃 모델 | 핵심 목표 | 난이도 |
| :---: | :--- | :--- | :--- | :--- | :---: |
| **1순위** | [plan-n1-adaptive-rq-tree.md](./plan-n1-adaptive-rq-tree.md) | `RQ-D4-Adaptive` & `RQ-D64-Adaptive` | `Adaptive malformed-copy` | `GordoAR`(`75db891b`) 완전 엄격 지배 + `Adaptive poly-S` 역대 최속($2^{n/130}$) 달성 | **낮음** |
| **2순위** | [plan-n2-bank-c128.md](./plan-n2-bank-c128.md) | `BANK-C128` (`ceil(128 log2 n)`) | `Adaptive` & `ModelM` | `Adaptive` 전체 1위 `e78ad51d`를 $\epsilon$ 약 $10^{38}$배 개선으로 직접 지배 + `ModelM` 최초 $2^{cn/\log n}$ | **중간** |
| **3순위** | [plan-n3-qubit-virtualization.md](./plan-n3-qubit-virtualization.md) | `RQ-D64-VirtSC` (Qubit Virtualization) | `ModelM` (및 `Adaptive`) | 고전 레코드를 basis-state qubit으로 가상화해 `ModelM` 최초 $S_C = O(n)$ 프론티어 개척 | **중상** |
| **4순위** | [plan-n4-modelm-tightening.md](./plan-n4-modelm-tightening.md) | `RQ-D32-Tight` & `cf6cb036` 마무리 | `ModelM` | `mpjunior92`(`6a56e70e`) 직접 엄격 지배, `L` 이중과금 제거 및 레드오션 회피 가이드 | **낮음** |

---

## 3. 공통 검증 파이프라인 (모든 플랜 공통)

모든 신규 제출물은 업로드 전 로컬 체크아웃(`/Users/a/개발/my-challenge/dcp`)에서 아래 순서로 검증한다:

1. **수학/부등식 정확 유리수 검증**: `fractions.Fraction` 기반 스크립트로 유한 구간($n=1$, $2 \le n < n_0$), 블록폭 합산, `Qcap`/`Bbad` 부등식 검증.
2. **문서 크기 및 가독성 점검**: `submission/proof.md` 등 증거 파일이 크기 제한(65,536 bytes 등) 대비 충분한 여유를 갖는지 확인.
3. **로컬 Admission Preflight**:
   ```bash
   .venv/bin/llm-verifier check --root .
   .venv/bin/python dcp_admission.py preflight --root . --receipt .work/admission.json --score .work/admission-score.json
   ```
4. **공개 스냅샷 파레토 지배 검증**: `.work/compare_balanced.py`와 동일한 정확 비교기로 `dominators == []` 및 목표 타깃 제출이 `candidate_dominates`에 포함되는지 확인.
5. **의존성 선언**: 이전 제출(본인 `cf6cb036`, 또는 `edca4431` 등)을 재사용한 경우 `yukon attribution prepare`로 해시된 `--dependencies-file` 생성 후 `yukon submit --dry-run` 통과 확인.
