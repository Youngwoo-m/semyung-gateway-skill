---
name: semyung-gateway
description: 세명대 AI 게이트웨이(BAZE/Mindlogic, factchat-cloud.mindlogic.ai)로 Claude 대화·이미지·영상·음성을 학교 크레딧으로 쓰는 방법. 사용자가 "세명대 게이트웨이", "학교 게이트웨이", "학교 크레딧으로", "게이트웨이 키", "BAZE" 를 말하거나, 개인 API 비용 없이 Claude/이미지/영상 생성을 하려 할 때 사용한다.
---

# 세명대 AI 게이트웨이

학교가 매달 크레딧을 주는 API 게이트웨이. Claude(Anthropic Messages 형식), 이미지·영상·음성 모델을 한 키로 쓴다.

## 키와 설정 (키 값은 대화·코드·로그에 그대로 쓰지 않는다)
- 설정 파일: `/Volumes/USB256/devAPP/AutoSNS/youtube-shorts-auto/config.json` 의 `llm_gateway`
  - `api_key`(주 키), `extra_keys`(추가 계정 키들, 차례로 넘어감), `base_url`, `model_map`
- 파이썬에서 읽기: `sys.path.insert(0, "/Volumes/USB256/devAPP/AutoSNS/youtube-shorts-auto"); from common import load_config; g = load_config()["llm_gateway"]`
- USB(`/Volumes/USB256`)가 연결되어 있어야 한다.

## 공통 규칙
- 주소: `https://factchat-cloud.mindlogic.ai/v1/gateway`
- 헤더: `Authorization: Bearer <키>`, `x-api-key: <키>`, `Content-Type: application/json`, **`User-Agent` 필수**(없으면 Cloudflare 1010/403). 예: `smuds-autosns/1.0`
- 월 할당(`monthly_allocated`)과 구매 크레딧(`purchased`)이 따로 있다. **구매 크레딧은 쓰지 않는 것이 원칙** — 월 할당이 모자라면 다음 키로 넘어간다. 키마다 약 500은 남겨 둔다.

## 엔드포인트
| 용도 | 경로 | 비고 |
|---|---|---|
| Claude 대화 | `/claude/` (Anthropic SDK `base_url`) | 바로 쓰기: `from shared.llm import client` (`/Volumes/USB256/devAPP/AutoSNS` 를 sys.path 에) → `client(cfg).messages.create(...)`. 실패하면 개인 Anthropic 키로 대비. 응답 앞의 thinking 블록 주의 |
| 모델 목록 | `GET /models/` | `type`: llm·image·video·audio … |
| 남은 크레딧 | `GET /credits/` | 키마다 조회 |
| 가격 미리 보기 | `POST /estimate/` | `{"kind":"video","model":..., "parameters":{"duration_seconds":8,"aspect_ratio":"9:16"}}` |
| 이미지 | `POST /images/generate/` (+ `GET /images/generate/{op}/`) | 예: `gemini-2.5-flash-image` 9:16 약 39크레딧, 결과 base64 는 `data[0].url` |
| 영상 | `POST /video/generation/` → `GET /video/generation/{op}/?model=` → `GET .../{op}/download/?model=` | 본문 `{"model","prompt","parameters":{"duration_seconds","aspect_ratio"}}` |
| 명세 | `GET /openapi.json` | |

## 영상 모델 (2026-10-15 기준)
- **Veo 는 게이트웨이에서 빠짐**(403 "not enabled for your tenant").
- 기본: `google/gemini-omni-flash/v1.1` — 8초 9:16 = 240크레딧, 720×1280, 약 50초.
- 다른 선택(8초): seedance-1.0-pro/fast 174, hailuo-02 270, pixverse v5 300, kling v2.5-turbo/pro 560 …
- 이미 만든 호출 코드: `/Volumes/USB256/devAPP/AutoSNS/youtube-shorts-auto/veo_clips.py`(`_req`, `_submit`, `monthly_remaining`) — `V.MODEL` 을 바꿔 쓴다. 예: `/Volumes/USB256/shorts-disaster/tools/veo_scene.py`.

## 쓰기 전에
1. `/credits/` 로 남은 월 할당 확인 → 2. `/estimate/` 로 가격 확인 → 3. 생성. 비싼 작업은 사용자에게 크레딧을 먼저 알린다.
4. 설명은 한국어로.
