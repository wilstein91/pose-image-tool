# 원하는 포즈로 이미지 만드는 도구

## 도구 설명
- 참조 사진으로부터 OpenPose를 통해 인물의 **자세(관절 구조)만** 뽑아내고, 인물·의상·배경·화풍은 프롬프트대로 새로 그려주는 Colab 노트북임.
- OpenPose로 자세를 뽑고 FLUX.2 [klein] 4B로 그림. 무료 Colab T4에서 한 장에 30~90초 걸림.

## 사용법
1. 구글 드라이브에서 `pose_tool.ipynb` 를 우클릭 → 연결 프로그램 → Google Colaboratory 로 열기. 연 뒤 **런타임 → 런타임 유형 변경 → T4 GPU** 로 설정.

2. STEP 0부터 순서대로 실행. STEP 3에서 참조 사진을 고르고, STEP 5에서 뽑힌 스켈레톤이 원본 자세와 닮았는지 확인한 뒤, STEP 6에서 모델을 받고(첫 실행 15~25분), STEP 7에서 프롬프트를 쓰고 STEP 8에서 생성. 한 바퀴 돈 뒤, 수정사항 발생 시 **STEP 7-3 → 8-1 → 9-1** 세 칸만 반복.

3. 결과는 구글 드라이브 쪽 `outputs/` 폴더에 `output_01.png` 처럼 사진 번호로 저장됨. 번호는 STEP 3-1의 `PICK` 을 따라가고, 같은 이름이 있으면 `-1`, `-2` 가 붙음.

## 테스트 결과
- 포즈 1: 기구에 등을 대고 누운 자세 

    → 프롬프트: `An athlete jumping over the moon, belly facing up, dramatic pose, dreamy lighting, bright moonlight, fantasy concept art style` 

    → 결과: 정상 출력. 초기 설정값때문에 목이 반대쪽으로 돌아간다거나 팔이 반대 방향으로 출력되는 이슈가 있었으나, prompt 지키는 수준을 strict로 올리니 보정 됨

- 포즈 2: 서서 양손을 앞으로 모은 전신 자세 

    → 프롬프트: `a young male waiting in front of a public restroom, holding his crotch, sweating, tense and anxious, comical and exaggerated, cartoon style` 

    → 결과: 정상 출력. 인물은 만화풍, 배경은 현실 배경으로 작성되어서 추후에 prompt에 Cartoonstyle 추가하여 해결.
- 포즈 3: 한 팔을 위로 들고 한쪽 무릎을 올린 자세 

    → 프롬프트: `a young good looking female standing on top of the skyscraper's tip, wind blowing her hair, finger pointing, cinematic lighting, photographic` 

    → 결과: 초도 이미지부터 정상 출력하였으나, seed 기능 확인을 위해 seed 변경만 한차례 진행.

## 한계
- FLUX.2용 OpenPose ControlNet이 아직 없어 스켈레톤을 Flux2에 참고 이미지로 제공하므로 자세가 100% 그대로 옮겨지지는 않음.
- 네거티브 프롬프트가 기본 설정에서 작동하지 않음. distilled 모델이라 `GUIDANCE = 1.0` 에서는 무시됨.
- Colab 런타임이 끊기면 모델 11GB를 다시 받아야 함. 용량은 큰데 다운속도는 제법 빠른 편임. 그래도 가급적 받은 자리에서 한번에 처리를 추천.
- 애니메이션·일러스트 참조는 관절 인식률이 낮음. 드래곤볼 손오공 포즈 인식 못함. OpenPose가 실사 사진으로 학습된 모델이라 아무래도 실사쪽이 안정적.
