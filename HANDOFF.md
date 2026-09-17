# 작업 인수인계 (2026-09-17 저녁 기준)

집이나 다른 컴퓨터에서 **이어서 작업하기 위한 메모**입니다.
현재 버전은 **FLUX.2 [klein] 4B** 기반이며 `main` 과 `vFLUX2` 가 같은 내용입니다.
이전의 Stable Diffusion 1.5 + ControlNet 버전은 커밋 `e8534f1` 이전 이력에 남아 있습니다.

> **제일 먼저 볼 것** → [3. 집에서 이어서 시작하기](#3-집에서-이어서-시작하기)

---

## 1. 이 프로젝트가 뭔가

참조 사진에서 **자세(관절 구조)만** 뽑아내고, 인물·의상·배경·화풍은 프롬프트대로 새로 그리는 도구.

```
참조 사진  →  스켈레톤(뼈대)  →  프롬프트대로 그린 새 그림
              OpenPose 추출      FLUX.2 [klein] 4B (스켈레톤을 참조 이미지로)
```

참조 사진의 **얼굴·피부색·옷은 전혀 전달되지 않는다.** 모델에 넘기는 것은 스켈레톤 그림뿐이다.
전부 `pose_tool.ipynb` 노트북 한 개에 들어 있다. 88셀이고 각 단계마다 설명 셀이 붙어 있다.

---

## 2. 현재 상태 — **동작 확인됨**

2026-09-17 저녁, **Colab T4에서 끝까지 정상 동작하는 것을 확인했다.** 결과 3장 생성 성공.

| 확인한 것 | 상태 |
| --- | --- |
| STEP 0~10 전체 파이프라인 | **정상** |
| ComfyUI 백그라운드 기동 · 부품 16개 | **정상** |
| 결과 생성 | **정상** — `samples/output_01.png` ~ `output_03.png` 3장 |
| **결과에 뼈대 선이 섞이는 문제** | 두 장 모두 **깨끗함.** 막대기·점·검은 배경 없음 |
| 자세 전이 품질 | 좋음. 누운 자세(01)·서 있는 자세(02) 모두 원본 자세를 따라감 |
| 그림 품질 | `main`(SD 1.5)보다 확연히 좋음. 손·얼굴이 무너지지 않음 |

> **주의**: 이 두 장을 만들 때 쓴 seed·설정이 **기록되지 않았다.** 그래서 똑같이 재현할 수 없다.
> 이 일을 막으려고 STEP 9-1이 이제 재현용 값을 화면에 찍는다. 다음부터는 그 줄을
> `prompts.md` 에 옮길 것.

---

## 3. 집에서 이어서 시작하기

### 3-1. 코드 가져오기

집 PC에 이미 레포가 있으면:

```bash
cd <프로젝트 폴더>
git pull origin main
```

처음 받는 PC라면:

```bash
git clone git@github.com:wilstein91/pose-image-tool.git
cd pose-image-tool
```

> 확인: `git log --oneline -1` 이 `결과물을 samples/ 로 통합` 커밋이면 최신이다.

### 3-2. 구글 드라이브 확인

Colab은 **드라이브만 볼 수 있다.** 드라이브 사본은 구글이 알아서 동기화하므로 보통 그대로 있다.
아래를 PowerShell에 붙여넣어 지금 어느 문자로 잡혔는지 확인한다.

```powershell
Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Name -ne 'C' } |
  ForEach-Object { Join-Path $_.Root 'Aiffel_Work\pose-image-tool' } |
  Where-Object { Test-Path $_ }
```

> 확인: 경로 한 줄이 출력되면 성공. 아무것도 안 나오면 구글 드라이브 데스크톱 앱이 꺼진 것이다.

### 3-3. 실행

```
1. VS Code로 폴더 열기
2. pose_tool.ipynb 열고 Colab 런타임(T4) 연결
3. STEP 0-2 부터 순서대로 실행
```

**STEP 1 · 2 · 6 은 런타임을 새로 연결할 때마다 다시 해야 한다.**
`main` 과 달리 **STEP 6에서 모델 11GB를 매번 다시 받느라 15~25분 걸린다.**
그래서 **한 번 연결했을 때 실험을 몰아서 하는 편이 좋다.**

한 바퀴 돌린 뒤에는 **STEP 7-3 → 8-1 → 9-1 세 칸만** `Ctrl`+`Enter` 로 반복하면 된다.
값을 고치기만 하고 그 셀을 실행하지 않으면 반영되지 않는다.

---

## 4. 남은 일

| 우선순위 | 할 일 | 내용 |
| --- | --- | --- |
| **높음** | 실험할 때마다 `prompts.md` 에 기록 | 9-1이 찍어 주는 재현용 줄을 그때그때 옮길 것. 이걸 건너뛰어 `output_01` 의 seed를 영영 잃었다 |
| 중간 | 참조 사진 추가 | 지금 3장. 새 사진은 C가 아니라 **드라이브의 `samples/`** 에 넣어야 STEP 3-1 목록에 뜬다 |
| 중간 | 설정 실험 | `POSE_STRICTNESS` 3단계 비교, `STEPS` 4 vs 8 |
| 낮음 | `README.md` 테스트 결과 보강 | 2026-09-17 작성 완료. 포즈 3종 기록됨 |
| 낮음 | `main` 과 A/B 비교 | 같은 사진·같은 장면으로 두 브랜치 결과를 나란히 |
| 낮음 | `USE_GGUF = False` | fp8 원본(4.1GB)과 품질 비교 |

---

## 5. 이전 버전(SD 1.5)에서 바뀐 것

생성 모델을 **Stable Diffusion 1.5 + ControlNet → FLUX.2 [klein] 4B** 로 교체했다.
자세 추출(STEP 0~5)은 예전과 거의 같다.

| 부분 | 이전 (SD 1.5) | 현재 (FLUX.2) |
| --- | --- | --- |
| STEP 1 | `diffusers` 설치 | **ComfyUI + ComfyUI-GGUF + `gguf`** (1-1), `controlnet_aux` (1-2) |
| STEP 4-3 | 8의 배수, 긴 변 512 | **16의 배수**, 긴 변 768 |
| STEP 6 | `from_pretrained` 로 파이프라인 로드 | **모델 3종 내려받기(11GB) + ComfyUI 기동 + 생성 함수 정의** |
| STEP 7 | `POSE_STRENGTH` 숫자 (0.8~1.2) | **`POSE_STRICTNESS`** (`strict`/`normal`/`loose`) |
| STEP 8 | `pipe(...)` | **`flux2_generate(...)`** — ComfyUI HTTP API 호출 |
| 단계 수 | 30 | **4** |
| CFG | 7.5 | **1.0** (네거티브 무시됨) |
| 파일 이름 | `result_seed..._cn..._g....png` | **`output_01.png`** (사진 번호만) |

### 왜 ControlNet을 안 쓰나

FLUX.2 klein 용 OpenPose ControlNet이 2026년 9월 기준 아직 없다.
대신 FLUX.2 의 **참조 이미지(image reference)** 기능을 쓴다 —
스켈레톤을 `LoadImage` → `VAEEncode` → `ReferenceLatent` 로 프롬프트에 매달고,
영어 지시 문장을 뒤에 붙인다.

### 모델에게 실제로 보내는 문장 — **순서가 핵심**

```
 ① 내가 쓴 프롬프트     "A knight in silver armor on a misty battlefield..."
 ② 자세 지시            "Pose: 이 참조 도면에 있는 자세로 몸을 두어라"
 ③ 도면 금지            "참조 이미지는 그림의 일부가 아니다. 도면일 뿐이다.
                         뼈대·점·선·와이어프레임·검은 배경을 결과에 절대 그리지 마라"
```

**처음에는 ③②① 순서였고, 그래서 결과에 알록달록한 막대기가 얹혀 나왔다.**
모델이 가장 먼저 읽은 것이 '골격'이라 그것을 그릴 대상으로 오해한 것이다.
장면을 먼저 말하도록 뒤집은 뒤 해결됐다. 문장 3종은 STEP 6-4의
`POSE_INSTRUCTION` 과 `NO_SKELETON` 에 있다.

**`strict` 가 오히려 독이 될 수 있다.** 엄격하게 시킬수록 참조 이미지를 통째로 베끼려 한다.
막대기가 보이면 `strict` → `normal` → `loose` 로 **내릴 것.**

---

## 6. 실행 환경 — 중요

**VS Code + Google Colab 확장**으로 원격 Colab T4에서 실행한다. Colab 웹 브라우저가 아니다.

```
[내 PC / VS Code]              [구글 서버 / Colab VM]
 화면·편집기만                   실제 파이썬 실행 (Linux, T4 GPU)
 C:\  X:\                        /content  /content/drive
```

**여기서 비롯되는 제약** (직접 부딪혀 확인한 것):

| 기능 | VS Code에서 |
| --- | --- |
| `files.upload()` | **무한 대기. 절대 쓰지 말 것** (14분 날림) |
| `drive.mount()` | 동작함. ipywidgets "다운로드 활성화" 한 번 승인 필요 |
| URL 다운로드 | 항상 동작 |

**내 PC의 파일은 Colab 서버에서 보이지 않는다.** 폴더를 로컬로 옮겨도 해결 안 됨.
파일 전달은 구글 드라이브 마운트로 한다.

**노트북 파일을 고친 뒤에는 고친 셀을 다시 실행해야 반영된다.** 파일만 새로 여는 것으로는
메모리가 바뀌지 않는다. 헷갈리면 **런타임 → 세션 다시 시작 및 모두 실행** 이 확실하지만,
이 브랜치는 모델 11GB를 다시 받으므로 20분이 든다.

---

## 7. 폴더 운용 방식

| 위치 | 역할 |
| --- | --- |
| `C:\Aiffel_Work\pose-image-tool` | **git 작업 폴더.** 깃허브에 푸시하는 곳 |
| `<드라이브>:\Aiffel_Work\pose-image-tool` | 구글 드라이브 사본. Colab이 여기서 사진을 읽고 결과를 쓴다 |

> **드라이브 문자를 외우지 말 것.** 예전 메모에는 `G:` 로 적혀 있었지만
> 2026-09-17 기준 사무실 PC 에서는 **`X:`** 로 잡혔다. 마운트 모양도 다르다
> (`X:\Aiffel_Work` vs `G:\내 드라이브\Aiffel_Work`). 집 PC는 또 다를 수 있다.
> 그래서 노트북과 아래 명령은 문자를 적지 않고 **찾아내는** 방식을 쓴다.

구글 드라이브 동기화 폴더 안에 `.git` 을 두면 충돌이 나므로 **git은 C에만** 둔다.
노트북 STEP 3-1은 드라이브 아래에서 `pose-image-tool` 폴더를 3단계 깊이까지 자동으로 찾는다.

### C 와 드라이브 사이에 무엇을 옮겨야 하나

| 방향 | 무엇을 | 왜 |
| --- | --- | --- |
| **드라이브 → C** | `samples/` 의 `output_*.png` | Colab이 결과를 드라이브에 쓴다. 깃허브에 올리려면 C로 가져와야 함 |
| **C → 드라이브** | `samples/` | 새 참조 사진을 C에만 넣으면 **Colab에서 보이지 않는다** |
| (선택) **C → 드라이브** | `pose_tool.ipynb`, `*.md` | VS Code로 C에서 여는 한 불필요 |

**결과물 가져오기 (드라이브 → C)** — STEP 9-1이 저장할 때마다 이 명령을 같이 찍어 준다.

```powershell
$src = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Name -ne 'C' } |
       ForEach-Object { Join-Path $_.Root 'Aiffel_Work\pose-image-tool\samples' } |
       Where-Object { Test-Path $_ } | Select-Object -First 1
Copy-Item "$src\output_*.png" 'C:\Aiffel_Work\pose-image-tool\samples\' -Force
```

**새 사진 넣기 (C → 드라이브)**

```powershell
$dst = Get-PSDrive -PSProvider FileSystem | Where-Object { $_.Name -ne 'C' } |
       ForEach-Object { Join-Path $_.Root 'Aiffel_Work\pose-image-tool\samples' } |
       Where-Object { Test-Path $_ } | Select-Object -First 1
Copy-Item 'C:\Aiffel_Work\pose-image-tool\samples\*' "$dst\" -Force
```

> 2026-09-17 저녁 기준 **양쪽 동기화를 맞춰 두었다.** 집에서는 `git pull` 만 하면 된다.

---

## 8. 노트북 구조

| STEP | 내용 | 재실행 필요 시점 |
| --- | --- | --- |
| 0 | GPU 확인 | 런타임 연결할 때마다 |
| 1 | ComfyUI·GGUF·controlnet_aux 설치 | 런타임 연결할 때마다 |
| 2 | 임포트 · 장치 결정 | 런타임 연결할 때마다 |
| 3 | 참조 사진 (3-1 드라이브 / 3-2 URL) | 사진 바꿀 때 |
| 4 | 사진 열기 · 크기 자동 계산 (16의 배수) | 사진 바꿀 때 |
| 5 | OpenPose 스켈레톤 추출 (5-4 자동 크롭) | 사진 바꿀 때 |
| **6** | **모델 11GB 받기 · ComfyUI 기동 · 생성 함수 정의** | **런타임 연결할 때마다** |
| **7** | **프롬프트 · 설정** | **실험할 때마다** |
| **8** | **생성** | **실험할 때마다** |
| **9** | **저장 · 비교** | **실험할 때마다** |
| 10 | 여러 장 배치 생성 | 선택 |
| 부록 | 문제 해결 · 용어 사전 | — |

### 주요 설정값 (STEP 7-3)

| 변수 | 기본값 | 뜻 |
| --- | --- | --- |
| `STEPS` | 4 | 노이즈 제거 횟수. klein 4B는 4면 충분. 올릴수록 참조에 더 달라붙는다 |
| `GUIDANCE` | 1.0 | 프롬프트 충실도(CFG). 1.0 초과 시 네거티브 작동, 대신 2배 느림 |
| `POSE_STRICTNESS` | `"normal"` | 자세 엄격도. `strict` / `normal` / `loose` |
| `SEED` | 12345 | 같은 값 = 같은 그림 |

결과는 `samples/output_01.png` 처럼 **사진 번호만으로** 저장된다. 참조 사진과 같은 폴더에 쌓인다.
번호는 STEP 3-1의 `PICK` 을 따라간다 (`PICK = 0` → `output_01`).
같은 이름이 있으면 `-1`, `-2` 가 붙어 덮어쓰지 않는다.
이름에 설정이 없으므로 9-1이 재현용 값을 화면에 찍는다 — **그 줄을 `prompts.md` 로 옮길 것.**

---

## 9. 시간을 많이 날린 함정들 — 다시 겪지 말 것

| 증상 | 원인 | 대응 |
| --- | --- | --- |
| `ERROR: pip's dependency resolver ... gradio ... huggingface-hub` | **정상 경고.** 안 쓰는 `gradio` 와의 충돌 | 무시. STEP 1-4 결과만 확인 |
| 3-3 업로드 셀이 안 끝남 | VS Code에서 `files.upload()` 미동작 | STEP 3-1(드라이브)로 |
| 스켈레톤이 엉망 | **애니메이션 캡처는 인식률 낮음** (실사로 학습된 모델) | 실사 사진 쓰거나 STEP 5-4 자동 크롭 |
| 결과에 알록달록 뼈대 선 | 지시 문장이 '골격'으로 시작해 모델이 그릴 대상으로 오해 | **수정됨.** 그래도 나오면 `POSE_STRICTNESS` 를 **내리고**, 프롬프트를 `A photograph of ...` 로 시작, `STEPS` 를 4로 |
| 네거티브 프롬프트가 안 먹음 | **정상.** distilled 모델 + `GUIDANCE=1.0` | 프롬프트 안에 문장으로 쓰거나 `GUIDANCE` 를 1.5~3.0 으로 |
| `TypeError: 'Image' object is not subscriptable` (5-4) | `detect_poses` 는 numpy 배열을 받는다 | **수정됨** |
| `RuntimeError: OpenPose 모델이 메모리에 없습니다` | STEP 6-3이 RAM 확보를 위해 내림 (의도된 동작) | STEP 5-1 다시 실행 |
| `NameError: photo_tag / flux2_generate` | 해당 셀을 아직 안 돌림 | 9-1·10-2는 **자동 준비**되어 그냥 진행됨. `flux2_generate` 는 STEP 6-4 실행 |
| `ComfyUI 서버가 뜨지 않았습니다` | 설치 실패 또는 메모리 부족 | 셀이 찍어 주는 로그를 읽을 것. 대개 STEP 1-1 재실행 |
| `없는 부품: ['UnetLoaderGGUF']` | GGUF 커스텀 노드 미설치 | STEP 1-1 → 6-3 순서로 재실행 |
| 새 사진이 STEP 3-1 목록에 안 뜸 | C에만 넣고 드라이브에 안 넣음 | 위 7절의 `C → 드라이브` 명령 실행 |
| 드라이브 경로를 못 찾음 | **드라이브 문자가 바뀜** (`G:` → `X:`) | 문자를 적지 말고 7절의 찾기 명령을 쓸 것 |
| `NameError: name 'os' is not defined` (3-0 / 3-1) | **런타임이 끊겨 커널이 새로 떴다.** 모두 실행은 이전 세션 것 | STEP 1부터 다시 실행. 두 셀은 자체 import 하도록 **수정됨** |
| 결과물이 STEP 3-1 참조 사진 목록에 섞여 나옴 | 결과가 `samples/` 에 함께 쌓임 | `output_` 으로 시작하는 파일은 후보에서 제외하도록 **수정됨** |

**ComfyUI 로그 보는 법** — 새 셀에 `print(open("/content/comfyui.log").read()[-3000:])`

---

## 10. 기술 스택

| 항목 | 값 |
| --- | --- |
| 디퓨전 모델 | `unsloth/FLUX.2-klein-4B-GGUF` → `flux-2-klein-4b-Q4_K_M.gguf` (2.6GB) |
| 대안 | `black-forest-labs/FLUX.2-klein-4b-fp8` (4.1GB) — STEP 6-1의 `USE_GGUF=False` |
| 텍스트 인코더 | `qwen_3_4b.safetensors` (Qwen3-4B, 8.0GB) |
| VAE | `flux2-vae.safetensors` (0.34GB) |
| 생성 엔진 | ComfyUI (백그라운드, `http://127.0.0.1:8188`, 외부 미공개) |
| GGUF 로더 | `city96/ComfyUI-GGUF` 커스텀 노드 |
| 포즈 추출 | `controlnet_aux` OpenposeDetector + `lllyasviel/Annotators` |
| 샘플러 | `euler` + `Flux2Scheduler` (`SamplerCustomAdvanced` 경유) |
| 자세 전달 | `LoadImage` → `VAEEncode` → `ReferenceLatent` |

버전 고정: `controlnet_aux==0.0.9`, `huggingface_hub>=0.34,<1.0`, `transformers>=4.51.3,<5`

`huggingface_hub` 은 **1.0 미만이어야 한다.** 1.0부터 함수가 삭제되어 `controlnet_aux 0.0.9`가 깨진다.
`transformers` 를 5.0 미만으로 묶는 이유도 같다 — 5.0은 `huggingface_hub>=1.0`을 요구한다.
ComfyUI는 `huggingface_hub` 을 요구하지 않으므로 이 고정과 부딪히지 않는다.

워크플로우 연결 구조는 ComfyUI 공식 템플릿
`image_flux2_klein_image_edit_4b_distilled.json` 과 대조해 맞췄다.

---

## 11. 더 해볼 것

- STEP 5-2에서 `include_face=True` → 표정까지 따라하기
- STEP 5-2에서 `include_hand=True` → 손 모양까지 (원본에서 손이 선명할 때만)
- FLUX.2 용 ControlNet이 klein 4B를 지원하게 되면 (`alibaba-pai/FLUX.2-dev-Fun-Controlnet-Union`)
  ControlNet 방식으로 되돌려 자세 정확도와 화질을 동시에 잡기
- `main` 과 같은 참조 사진·같은 장면으로 A/B 비교 이미지 만들어 README에 싣기
