# 테스트에 쓴 프롬프트 모음

공통 설정 (별도 표기 없으면 동일)

| 항목 | 값 |
| --- | --- |
| 모델 | `runwayml/stable-diffusion-v1-5` |
| ControlNet | `lllyasviel/sd-controlnet-openpose` |
| steps | 30 |
| guidance_scale | 7.5 |
| controlnet_conditioning_scale | 1.0 |
| 해상도 | 512 x 512 (긴 변 기준 리사이즈) |

공통 네거티브 프롬프트

```
lowres, bad anatomy, extra limbs, missing limbs, fused fingers, deformed hands,
watermark, text, signature, jpeg artifacts, blurry
```

---

## output_01 — 전투 자세 (참조: `samples/pose_01.png`)

참조 포즈: 상체를 앞으로 숙이고 한 팔은 머리 위로, 다른 손은 앞을 가리키는 전투 자세.

```
a knight in ornate silver armor in a low battle stance, one arm raised overhead,
the other hand pointing forward, windswept cape, dramatic rim light,
fantasy concept art, highly detailed
```

- seed: `12345`
- controlnet_conditioning_scale: `1.0`
- 메모: 머리 위로 올린 팔은 스켈레톤이 잘 잡히지만, 가리키는 손의 **손가락**은 OpenPose가 잡지 못해 자주 뭉갠다. `hand`(손) 관련 네거티브를 넣거나 seed를 몇 번 바꿔 고르는 편이 빠르다.

## output_02 — 치맛단 잡은 자세 (참조: `samples/pose_02.png`)

참조 포즈: 무릎을 살짝 붙이고 서서 양손으로 치맛단을 앞으로 모아 잡은 자세.

```
a woman in a long flowing white summer dress holding the hem with both hands,
standing on a city sidewalk at night, warm shop lights behind,
cinematic photograph, 50mm, soft film grain
```

- seed: `777`
- controlnet_conditioning_scale: `1.1` (기본값 1.0에서는 다리 간격이 벌어져 조금 올렸다)
- 메모: 전신이 세로로 긴 구도라 512x512로 맞추면 다리가 잘린다. 가로 448 / 세로 640으로 생성했다.

---

## 프롬프트 작성 요령

- 자세는 스켈레톤이 담당하므로 프롬프트에서 다시 설명하지 않아도 된다. 대신 **인물·의상·배경·화풍**을 적는다.
- 단, 참조 포즈에서 특히 살리고 싶은 동작(예: `one arm raised overhead`)은 프롬프트에 한 번 더 써 주면 안정적이다.
- 결과가 참조를 너무 따라가면 `controlnet_conditioning_scale`을 0.7~0.8로 낮추고, 자세가 흐트러지면 1.1~1.3으로 올린다.
- 참조 이미지의 가로세로 비율을 생성 해상도에 맞춰야 스켈레톤이 왜곡되지 않는다.
