# 작업 인수인계 (2026-09-17)

내일 다른 컴퓨터에서 이어서 작업하기 위한 메모.

---

## 1. 이 프로젝트가 뭔가

참조 사진에서 **자세(관절 구조)만** 뽑아내고, 인물·의상·배경·화풍은 프롬프트대로 새로 그리는 도구.

```
참조 사진  →  스켈레톤(뼈대)  →  프롬프트대로 그린 새 그림
              OpenPose 추출      Stable Diffusion + ControlNet
```

참조 사진의 **얼굴·피부색·옷은 전혀 전달되지 않는다.** 관절 좌표만 넘어간다.

전부 `pose_tool.ipynb` 노트북 한 개에 들어 있다. 86셀이고, 각 단계마다 설명 셀이 붙어 있다.

---

## 2. 현재 상태 — 동작 확인됨

2026-09-17 기준 **전체 파이프라인이 끝까지 정상 동작**한다. 샘플 이미지 2장 모두 통과.

- `samples/pose_01.png` — 등 스트레칭 기구에 누운 실사 사진 (550x367)
- `samples/pose_02.png` — 마릴린 먼로 전신 사진 (520x706)
- `outputs/result_seed125_cn1.2_g9.png` — 첫 생성 결과

## 3. 남은 일

| 할 일 | 내용 |
| --- | --- |
| **`README.md` 작성** | 템플릿 상태. 실사용 후 직접 채울 것 |
| **`prompts.md` 갱신** | 지금 내용은 **예전 샘플 이미지 기준**이라 실제와 안 맞음. 쓴 프롬프트·seed 기록용 |
| 설정 실험 | `SEED` 여러 개 → `POSE_STRENGTH` 0.8/1.0/1.2 비교 |

---

## 4. 실행 환경 — 중요

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

## 5. 내일 시작하는 순서

```
1. git clone https://github.com/wilstein91/pose-image-tool.git
2. VS Code로 폴더 열기
3. pose_tool.ipynb 열고 Colab 런타임(T4) 연결
4. STEP 0-2 부터 순서대로 실행
```

**STEP 1~2는 런타임 새로 연결할 때마다 다시 해야 한다.** Colab은 연결이 끊기면 설치 내용을 지운다.

한 바퀴 돌린 뒤에는 **STEP 7-3 → 8-1 → 9-1 세 칸만** `Ctrl`+`Enter` 로 반복하면 된다.
값을 고치기만 하고 그 셀을 실행하지 않으면 반영되지 않는다.

---

## 6. 시간을 많이 날린 함정들 — 다시 겪지 말 것

| 증상 | 원인 | 대응 |
| --- | --- | --- |
| `ERROR: pip's dependency resolver ... gradio ... huggingface-hub` | **정상 경고.** `gradio`(안 씀)와 버전 충돌 | 무시. STEP 1-3 결과만 확인 |
| 모델 로딩 실패 | `runwayml/stable-diffusion-v1-5` 주소가 2024년 삭제됨 | `stable-diffusion-v1-5/stable-diffusion-v1-5` 사용 중 |
| 3-A 업로드 셀이 안 끝남 | VS Code에서 `files.upload()` 미동작 | STEP 3-1(드라이브)로 |
| 결과 인물이 뭉개짐 | 16:9 이미지가 512x288로 축소됨 | `MIN_SIDE=384`, `ABS_MAX=704` 로 해결됨 |
| 스켈레톤이 엉망 | **애니메이션 캡처는 인식률 낮음** (실사로 학습된 모델) | 실사 사진 쓰거나 STEP 5-4 자동 크롭 |

---

## 7. 노트북 구조

| STEP | 내용 | 재실행 필요 시점 |
| --- | --- | --- |
| 0 | GPU 확인 | 런타임 연결할 때마다 |
| 1 | 라이브러리 설치 (버전 고정) | 런타임 연결할 때마다 |
| 2 | 임포트 · 장치 결정 (fp16) | 런타임 연결할 때마다 |
| 3 | 참조 사진 (3-1 드라이브 / 3-2 URL) | 사진 바꿀 때 |
| 4 | 사진 열기 · 크기 자동 계산 | 사진 바꿀 때 |
| 5 | OpenPose 스켈레톤 추출 (5-4 자동 크롭) | 사진 바꿀 때 |
| 6 | SD 1.5 + ControlNet v1.1 로드 | 모델 바꿀 때 |
| **7** | **프롬프트 · 설정** | **실험할 때마다** |
| **8** | **생성** | **실험할 때마다** |
| **9** | **저장 · 비교** | **실험할 때마다** |
| 10 | 여러 장 배치 생성 | 선택 |
| 부록 | 문제 해결 · 용어 사전 | — |

### 주요 설정값 (STEP 7-3)

| 변수 | 기본값 | 뜻 |
| --- | --- | --- |
| `STEPS` | 30 | 노이즈 제거 횟수 |
| `GUIDANCE` | 7.5 | 프롬프트 충실도 (높이면 딱딱) |
| `POSE_STRENGTH` | 1.0 | 자세 엄격도 (높이면 정확, 낮추면 자연스러움) |
| `SEED` | 12345 | 같은 값 = 같은 그림 |

결과는 `outputs/result_seed{SEED}_cn{POSE_STRENGTH}_g{GUIDANCE}.png` 로 저장된다.
같은 이름이 있으면 `-1`, `-2` 가 붙어 덮어쓰지 않는다.

---

## 8. 폴더 운용 방식

| 위치 | 역할 |
| --- | --- |
| `C:\Aiffel_Work\pose-image-tool` | **git 작업 폴더.** 깃허브에 푸시하는 곳 |
| `G:\내 드라이브\Aiffel_Work\pose-image-tool` | 구글 드라이브 사본. Colab이 여기서 사진을 읽고 결과를 쓴다 |

구글 드라이브 동기화 폴더 안에 `.git` 을 두면 충돌이 나므로 **git은 C에만** 둔다.

노트북 STEP 3-1은 내 드라이브 아래에서 `pose-image-tool` 폴더를 **3단계 깊이까지 자동으로 찾는다.**
폴더를 옮겨도 경로를 고칠 필요 없다.

> **주의**: Colab이 결과를 쓰는 곳은 **G(드라이브)** 다. 생성 결과를 깃허브에 올리려면
> G의 `outputs/` 를 C로 복사한 뒤 커밋해야 한다.

---

## 9. 기술 스택

| 항목 | 값 |
| --- | --- |
| 본체 모델 | `stable-diffusion-v1-5/stable-diffusion-v1-5` |
| ControlNet | `lllyasviel/control_v11p_sd15_openpose` (v1.1) |
| 포즈 추출 | `controlnet_aux` OpenposeDetector + `lllyasviel/Annotators` |
| 스케줄러 | `UniPCMultistepScheduler` (적은 단계로 같은 품질) |
| 정밀도 | `float16` (T4는 Turing이라 bfloat16 미지원) |

버전 고정: `diffusers==0.31.0`, `transformers==4.46.3`, `accelerate==1.1.1`,
`controlnet_aux==0.0.9`, `huggingface_hub<1.0`

`huggingface_hub` 은 **1.0 미만이어야 한다.** 1.0부터 함수가 삭제되어 `diffusers 0.31`이 깨진다.

---

## 10. 더 해볼 것

- STEP 6-1의 `BASE_MODEL` 교체 → `Lykon/dreamshaper-8`(일러스트), `SG161222/Realistic_Vision_V5.1_noVAE`(실사)
- STEP 5-2에서 `include_face=True` → 표정까지 따라하기
- 애니 캡처를 꼭 쓰고 싶다면 OpenPose 대신 lineart/canny ControlNet 검토 (단, 윤곽선 전체를 따라감)
