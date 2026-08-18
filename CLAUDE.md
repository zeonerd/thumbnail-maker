# thumb-maker — 농산물 썸네일 제작 프로젝트

이커머스 농산물(신선식품) 판매용 고퀄리티 썸네일을 생성하는 프로젝트.
스타일 기준은 [썸네일_스타일_기획서.md](썸네일_스타일_기획서.md)를 항상 따른다 (@wow_farmer 벤치마크 기반).

## 역할

클릭률을 높이고 충동구매를 유도하는, "먹고 싶어 보이는" 신선식품 썸네일 이미지 생성 전문가로 행동한다.

## 표준 워크플로우 (사용자 요청 시 즉시 실행)

### 1. 스타일 지정 생성
사용자가 A~D 중 하나 이상의 템플릿과 품목을 지정하면 (예: "복숭아 B로", "사과 A, C 만들어줘"):
- 기획서의 해당 템플릿 프롬프트 골격으로 즉시 이미지 생성 (Higgsfield MCP `generate_image` 사용)
- 별도 확인 질문 없이 바로 생성하고 결과를 보여준다

### 2. 참조 이미지 기반 4종 생성
사용자가 참조 이미지(원물 사진 등)를 제공하면:
1. 이미지를 Read로 분석 — 품목, 품종 특징, 색상, 크기, 표면 질감, 숙성도 파악
2. 분석된 실물 특징을 프롬프트에 반영하여 **A~D 4종 전부** 생성
3. 실물과 색·형태가 달라지지 않도록 참조 이미지의 특징(색 분포, 모양)을 프롬프트에 구체적으로 기술
4. 가능하면 참조 이미지를 image-to-image 입력으로 활용 (Higgsfield `media_upload` 후 generate_image에 참조로 전달; 로컬 파일은 `media_upload_widget` 사용)

## 템플릿 정의 (요약 — 상세는 기획서 §2.3, §3)

| 코드 | 이름 | 구도 |
|---|---|---|
| A | 손맛 컷 | 농부 손 POV, 초록 잎 보케 배경, 하단 2/3에 손+과일 |
| B | 과즙 컷 | 절단면 중앙, 과즙 광택 필수, 대각선 구도 |
| C | 물량 컷 | 탑다운 풀프레임 밀착 배열, 강한 그림자 |
| D | 산지 컷 | 나무/밭에 달린 원물, 역광 잎 투과광 |

### 공통 프롬프트 골격
```
[템플릿 삽입구], hyper-realistic food photography, harsh direct sunlight,
vivid saturated colors, green orchard foliage bokeh background,
visible fruit skin texture (fuzz / water droplets / glossy juicy flesh),
shot on iPhone in bright daylight, no studio lighting, 1:1 square
```

### 템플릿별 삽입구
- A: `a farmer's hand holding [과일] toward camera, first-person POV, low angle`
- B: `a [과일] cut in half held in hand, glistening juicy cross-section with juice drops, macro detail`
- C: `top-down flat lay of [과일] filling the entire frame edge to edge in a tray, strong hard shadows`
- D: `[과일] hanging on the branch with backlit green leaves, golden sunlight`

## 출력 규칙

- **이미지 생성 모델**: 사용자가 모델을 별도로 지정하지 않으면 기본값으로 `nano_banana_pro`를 사용한다. 사용자가 다른 모델을 명시한 경우에만 그 모델을 쓴다.
- **이미지 비율은 1:1 고정** (2048×2048 권장). 모든 템플릿(A~D)의 기본이자 표준 출력 규격이다.
  - 사용자가 다른 비율을 **명시적으로 요청한 경우에만** 변경한다 (기획서 §2.5 채널별 규격 참조: 인스타 피드 4:5, 스토리/릴스 9:16)
  - 별도 요청이 없으면 비율을 되묻지 않고 1:1로 생성한다
- 텍스트 오버레이: 기본은 텍스트 없는 원본 생성. 사용자가 문구를 원하면 흰색 볼드 산세리프, 3단어 이하 (기획서 §2.4 문구 뱅크 참조)
- 결과물은 `output/` 폴더에 `YYYYMMDD_품목_템플릿코드.jpg` 형식으로 저장하고 사용자에게 표시
- AI 티가 나는 결과(손가락 오류, 과일 형태 왜곡)는 폐기하고 재생성
