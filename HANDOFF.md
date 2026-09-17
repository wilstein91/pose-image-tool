# 작업 인수인계 (vFLUX2 브랜치)

다른 컴퓨터에서 이어서 작업하기 위한 메모.
원본(SD 1.5 + ControlNet) 기준 메모는 `main` 브랜치의 `HANDOFF.md` 를 보세요.

---

## 1. 이 프로젝트가 뭔가

참조 사진에서 **자세(관절 구조)만** 뽑아내고, 인물·의상·배경·화풍은 프롬프트대로 새로 그리는 도구.

```
참조 사진  →  스켈레톤(뼈대)  →  프롬프트대로 그린 새 그림
              OpenPose 추출      FLUX.2 [klein] 4B (스켈레톤을 참조 이미지로)
```

참조 사진의 **얼굴·피부색·옷은 전혀 전달되지 않는다.** 모델에 넘기는 것은 스켈레톤 그림뿐이다.

전부 `pose_tool.ipynb` 노트북 한 개에 들어 있다. 88셀이고, 각 단계마다 설명 셀이 붙어 있다.

---

## 2. 이 브랜치에서 바뀐 것 (2026-09-17)

생성 모델을 **Stable Diffusion 1.5 + ControlNet → FLUX.2 [klein] 4B** 로 교체했다.
자세 추출(STEP 0~5)은 `main` 과 동일하다.

| 부분 | main | vFLUX2 |
| --- | --- | --- |
| STEP 1 | `diffusers` 설치 | **ComfyUI + ComfyUI-GGUF + `gguf`** 설치 (1-1), `controlnet_aux` (1-2) |
| STEP 4-3 | 8의 배수, 긴 변 512 | **16의 배수**, 긴 변 768 |
| STEP 6 | `from_pretrained` 로 파이프라인 로드 | **모델 3종 내려받기(11GB) + ComfyUI 백그라운드 기동 + 생성 함수 정의** |
| STEP 7 | `POSE_STRENGTH` 숫자 | **`POSE_STRICTNESS`** (`strict`/`normal`/`loose`) |
| STEP 8 | `pipe(...)` 호출 | **`flux2_generate(...)`** — ComfyUI HTTP API 호출 |
| 단계 수 | 30 | **4** |
| CFG | 7.5 | **1.0** (네거티브 무시됨) |

### 왜 ControlNet을 안 쓰나

FLUX.2 klein 용 OpenPose ControlNet이 2026년 9월 기준 아직 없다.
대신 FLUX.2 의 **참조 이미지(image reference)** 기능을 쓴다 —
스켈레톤을 `VAEEncode` → `ReferenceLatent` 로 프롬프트에 매달고,
"이 뼈대와 같은 자세로 그려라, 뼈대 선은 그리지 마라"를 영어 지시 문장으로 앞에 붙인다.
지시 문장 3종은 노트북 STEP 6-4의 `POSE_INSTRUCTION` 에 있다.

**결과: 자세 정확도는 main 보다 낮고, 그림 품질은 확실히 높다.**

---

## 3. 현재 상태

| 항목 | 상태 |
| --- | --- |
| 노트북 구조 · 워크플로우 조립 | **검증 완료** (노드 연결, 도달성, 크기 계산, 네거티브 분기, 공식 ComfyUI 템플릿과 구조 대조) |
| 모델 다운로드 URL 5종 | **HTTP 200 확인 완료** |
| **T4 실기기 생성 테스트** | **아직 안 함 — 이게 다음 할 일** |

## 4. 남은 일

| 할 일 | 내용 |
| --- | --- |
| **T4에서 한 바퀴 돌리기** | STEP 0 → 10. 특히 6-3(ComfyUI 기동)과 8-1(첫 생성)을 확인 |
| **`README.md` 테스트 결과 채우기** | 지금은 "미테스트" 로 비어 있음 |
| **`prompts.md` 갱신** | FLUX.2는 문장형 프롬프트라 기존 내용(단어 나열)이 안 맞음 |
| 설정 실험 | `POSE_STRICTNESS` 3단계 비교, `STEPS` 4 vs 8 |
| main 과 결과 비교 | 같은 참조 사진·같은 장면으로 두 브랜치 결과를 나란히 |

---

## 5. 실행 환경 — 중요

**VS Code + Google Colab 확장**으로 원격 Colab T4에서 실행한다. Colab 웹 브라우저가 아니다.

```
[내 PC / VS Code]              [구글 서버 / Colab VM]
 화면·편집기만                   실제 파이썬 실행 (Linux, T4 GPU)
 C:\  G:\                        /content  /content/drive
```

**여기서 비롯되는 제약** (직접 부딪혀 확인한 것):

| 기능 | VS Code에서 |
| --- | --- |
| `files.upload()` | **무한 대기. 절대 쓰지 말 것** (14분 날림) |
| `drive.mount()` | 동작함. ipywidgets "다운로드 활성화" 한 번 승인 필요 |
| URL 다운로드 | 항상 동작 |

**내 PC의 파일은 Colab 서버에서 보이지 않는다.** 폴더를 로컬로 옮겨도 해결 안 됨.
파일 전달은 구글 드라이브 마운트로 한다.

---

## 6. 시작하는 순서

```
1. git clone https://github.com/wilstein91/pose-image-tool.git
2. git checkout vFLUX2
3. VS Code로 폴더 열기
4. pose_tool.ipynb 열고 Colab 런타임(T4) 연결
5. STEP 0-2 부터 순서대로 실행
```

**STEP 1 · 2 · 6 은 런타임을 새로 연결할 때마다 다시 해야 한다.**
`main` 과 달리 **STEP 6도 매번 다시 해야 하고, 여기서 11GB를 다시 받느라 20분쯤 걸린다.**
그래서 **한 번 연결했을 때 실험을 몰아서 하는 편이 좋다.**

한 바퀴 돌린 뒤에는 **STEP 7-3 → 8-1 → 9-1 세 칸만** `Ctrl`+`Enter` 로 반복하면 된다.
값을 고치기만 하고 그 셀을 실행하지 않으면 반영되지 않는다.

---

## 7. 시간을 많이 날린 함정들 — 다시 겪지 말 것

| 증상 | 원인 | 대응 |
| --- | --- | --- |
| `ERROR: pip's dependency resolver ... gradio ... huggingface-hub` | **정상 경고.** `gradio`(안 씀)와 버전 충돌 | 무시. STEP 1-4 결과만 확인 |
| 3-A 업로드 셀이 안 끝남 | VS Code에서 `files.upload()` 미동작 | STEP 3-1(드라이브)로 |
| 스켈레톤이 엉망 | **애니메이션 캡처는 인식률 낮음** (실사로 학습된 모델) | 실사 사진 쓰거나 STEP 5-4 자동 크롭 |
| `RuntimeError: OpenPose 모델이 메모리에 없습니다` | STEP 6-3이 RAM 확보를 위해 내림 (의도된 동작) | STEP 5-1 다시 실행 |
| `ComfyUI 서버가 뜨지 않았습니다` | 설치 실패 또는 메모리 부족 | 셀이 찍어 주는 로그를 읽을 것. 대개 STEP 1-1 재실행 |
| `없는 부품: ['UnetLoaderGGUF']` | GGUF 커스텀 노드 미설치 | STEP 1-1 → 6-3 순서로 재실행 |
| 결과에 알록달록 뼈대 선이 그대로 | 모델이 참조 이미지를 베낌 | `POSE_STRICTNESS` 낮추고 프롬프트 끝에 `Clean photographic scene, no diagram lines.` |
| 네거티브 프롬프트가 안 먹음 | **정상.** distilled 모델 + `GUIDANCE=1.0` | 프롬프트 안에 문장으로 쓰거나 `GUIDANCE` 를 1.5~3.0 으로 |

**ComfyUI 로그 보는 법** — 새 셀에 `print(open("/content/comfyui.log").read()[-3000:])`

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
| `STEPS` | 4 | 노이즈 제거 횟수. klein 4B는 4면 충분 |
| `GUIDANCE` | 1.0 | 프롬프트 충실도(CFG). 1.0 초과 시 네거티브 작동, 대신 2배 느림 |
| `POSE_STRICTNESS` | `"normal"` | 자세 엄격도. `strict` / `normal` / `loose` |
| `SEED` | 12345 | 같은 값 = 같은 그림 |

결과는 `outputs/output_{번호}_seed{SEED}_{POSE_STRICTNESS}_s{STEPS}_g{GUIDANCE}.png` 로 저장된다.
앞의 번호는 STEP 3-1의 `PICK` 을 따라간다 (`PICK = 0` -> `output_01`). 3-2(URL)로 넣으면 번호 없이 `output`.
같은 이름이 있으면 `-1`, `-2` 가 붙어 덮어쓰지 않는다.
9-1의 `WITH_SETTINGS = False` 로 바꾸면 `output_01.png` 처럼 번호만 남는다.

---

## 9. 폴더 운용 방식

| 위치 | 역할 |
| --- | --- |
| `C:\Aiffel_Work\pose-image-tool` | **git 작업 폴더.** 깃허브에 푸시하는 곳 |
| `G:\내 드라이브\Aiffel_Work\pose-image-tool` | 구글 드라이브 사본. Colab이 여기서 사진을 읽고 결과를 쓴다 |

구글 드라이브 동기화 폴더 안에 `.git` 을 두면 충돌이 나므로 **git은 C에만** 둔다.

노트북 STEP 3-1은 내 드라이브 아래에서 `pose-image-tool` 폴더를 **3단계 깊이까지 자동으로 찾는다.**

> **주의**: Colab이 결과를 쓰는 곳은 **G(드라이브)** 다. 생성 결과를 깃허브에 올리려면
> G의 `outputs/` 를 C로 복사한 뒤 커밋해야 한다.
>
> **모델 11GB는 `/content` (Colab 임시 디스크)에 받는다.** 드라이브에 캐시하지 않는다 —
> 무료 드라이브 15GB를 거의 다 쓰는 데다, 드라이브에서 읽는 속도가 느려 이득이 없다.

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

---

## 11. 더 해볼 것

- `USE_GGUF=False` (fp8 원본)로 품질 비교
- STEP 5-2에서 `include_face=True` → 표정까지 따라하기
- FLUX.2 용 ControlNet이 klein 4B를 지원하게 되면 (`alibaba-pai/FLUX.2-dev-Fun-Controlnet-Union`)
  ControlNet 방식으로 되돌려 자세 정확도와 화질을 동시에 잡기
- `main` 과 같은 참조 사진·같은 장면으로 A/B 비교 이미지 만들어 README에 싣기
