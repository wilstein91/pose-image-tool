# 테스트에 쓴 프롬프트 모음

`vFLUX2` 브랜치 기준. 공통 설정 (별도 표기 없으면 동일)

| 항목 | 값 |
| --- | --- |
| 디퓨전 모델 | `flux-2-klein-4b-Q4_K_M.gguf` (FLUX.2 [klein] 4B, GGUF Q4) |
| 텍스트 인코더 | `qwen_3_4b.safetensors` (Qwen3-4B) |
| VAE | `flux2-vae.safetensors` |
| 자세 추출 | OpenPose (`controlnet_aux` + `lllyasviel/Annotators`) |
| 자세 전달 방식 | `ReferenceLatent` — 스켈레톤을 참조 이미지로 제시 |
| `STEPS` | 7 |
| `GUIDANCE` (CFG) | 1.0 |
| `POSE_STRICTNESS` | `strict` |
| `SEED` | 결과마다 다름 — 아래 각 항목 참고 |

**`controlnet_conditioning_scale` 은 이 버전에 없습니다.** FLUX.2 klein 용 OpenPose ControlNet이
아직 공개되지 않아 ControlNet을 쓰지 않기 때문입니다. 대신 **`POSE_STRICTNESS`**(`strict` / `normal` / `loose`)가
자세를 얼마나 엄격히 따를지를 정합니다. `main` 브랜치의 `controlnet_conditioning_scale` 에 해당하는 값입니다.

공통 네거티브 프롬프트 (세 결과 모두 동일)

```
colored skeleton lines, stick figure, black background,
bad anatomy, extra limbs, missing limbs, deformed hands, deformed face,
watermark, text, signature, blurry, lowres, earth
```

**다만 이 네거티브는 실제로는 적용되지 않았습니다.** klein 4B는 distilled 모델이라
`GUIDANCE = 1.0` 에서 네거티브 회로를 쓰지 않습니다. 앞의 세 단어(`colored skeleton lines`,
`stick figure`, `black background`)는 결과에 뼈대 선이 섞이는 것을 막으려고 넣은 것인데,
실제로 그 역할을 한 것은 STEP 6-4의 `NO_SKELETON` 지시 문장이다.
네거티브를 실제로 쓰려면 `GUIDANCE` 를 1.5~3.0 으로 올려야 하고, 대신 약 2배 느려진다.

---

## output_01 — 달 앞에 수평으로 뜬 인물 (참조: `samples/pose_01.png`)

참조 포즈: 등 스트레칭 기구에 등을 대고 누운 자세. 몸이 수평이고 배가 위를 향함.

```
An athlete jumping over the moon,moon in the bottom half of the image,
Belly facing up, dramatic pose. dreamy lighting, dusky, ethereal, magical,
bright moonlight. Fantasy concept art style, highly detailed.
```

| 항목 | 값 |
| --- | --- |
| 결과 크기 | 1024 x 560 |
| seed | **기록 없음** |
| `POSE_STRICTNESS` | `strict` |
| `GUIDANCE` | 1.0 |
| `STEPS` | **기록 없음** |

- 이 그림을 만들 당시에는 STEP 9-1이 재현용 값을 찍어 주지 않아 **seed만 남지 않았다.** 자세 엄격도와 `GUIDANCE` 는 전 구간 동일했다.
- 같은 프롬프트를 나중에 다시 돌린 기록은 남아 있다 — `seed=12343 / strict / steps=7 / cfg=1.0 / 576x816`.
  다만 그때 결과는 **576 x 816** 이라 지금 `output_01.png`(1024 x 560)와 다른 그림이다.
  긴 변 1024는 현재 노트북 설정(16의 배수, 긴 변 768)으로 나오지 않는 크기이므로, 노트북을 고치기 전 버전에서 만든 것으로 보인다.

## output_02 — 화장실 앞에서 참는 남자 (참조: `samples/pose_02.png`)

참조 포즈: 무릎을 살짝 붙이고 서서 양손을 앞으로 모은 전신 자세.

```
a young male waiting in front of a public restroom, urgent expression, holding his crotch, looking around,
sweating, tense and very anxious. comical and exaggerated, cartoon style, funny, humorous,
simple cartoonish background, clear lines, simple colors
```

| 항목 | 값 |
| --- | --- |
| 결과 크기 | 576 x 816 |
| seed | `12343` |
| `POSE_STRICTNESS` | `strict` |
| `STEPS` | 7 |
| `GUIDANCE` | 1.0 |

- 근거: STEP 7-4와 8-2의 저장된 출력이 같은 값을 가리키고, 찍힌 크기(576 x 816)가 파일과 일치한다. **신뢰도 높음.**
- 이 프롬프트를 고정한 채 참조 사진만 pose_02 → pose_03 으로 바꿔서도 돌려 봤다. 배경·인물은 그대로이고 **자세만 갈아끼워지는 것**이 확인된다.

## output_03 — 마천루 위에서 손가락으로 가리키는 여성 (참조: `samples/pose_03.png`)

참조 포즈: 한 팔을 위로 들어 손가락으로 가리키고, 한쪽 무릎을 접어 올린 자세.

```
a young good looking female, standing on top of the skyscraper's tip, wind blowing her hair,confident, happy and confident expression
sunny day, bright and clear sky, cinematic lighting, ultra detailed, photographic
finger pointing, straight long hair, elegant, beautiful
```

| 항목 | 값 |
| --- | --- |
| 결과 크기 | 576 x 816 |
| seed | `123` |
| `POSE_STRICTNESS` | `strict` |
| `STEPS` | 7 |
| `GUIDANCE` | 1.0 |

- seed 기능이 실제로 동작하는지 확인하려고 `12343` 에서 `123` 으로 한 번 바꿔 돌린 결과다.
- **세 결과 중 자세 전이가 가장 정확하다.** 들어 올린 팔·굽힌 무릎·손가락 방향까지 참조와 맞고, 뼈대 선도 섞이지 않았다.

---

## 프롬프트 작성 요령 (FLUX.2 기준)

- **영어 문장으로 쓴다.** `main`(SD 1.5)의 단어 나열식과 달리 문장을 이해하고, 77토큰 제한도 없다.
- 자세는 스켈레톤이 담당하므로 **인물·의상·배경·화풍**을 쓴다. 다만 살리고 싶은 동작(`finger pointing`, `Belly facing up`)은 한 번 더 적어 주면 안정적이다.
- 참조 사진의 **소품은 전달되지 않는다.** 기구·의자가 필요하면 프롬프트에 직접 쓴다.
- 결과에 알록달록한 뼈대 선이 섞이면 `POSE_STRICTNESS` 를 **내리고**(`strict` → `normal` → `loose`) 프롬프트를 `A photograph of ...` 로 시작한다.
- 피하고 싶은 것은 네거티브 대신 **프롬프트 안에 문장으로** 쓴다 (`GUIDANCE = 1.0` 에서는 네거티브가 무시됨).

## 기록 남기는 법

STEP 9-1이 저장할 때마다 재현용 한 줄을 화면에 찍는다. **그 줄을 여기에 바로 옮겨 적을 것.**

```
output_03.png  |  seed=123  strict  steps=7  cfg=1.0  576x816
프롬프트: ...
```

output_01 의 seed가 사라진 것도, output_03 의 seed를 기록에서 찾지 못한 것도 이 단계를 건너뛰었기 때문이다.
