# 논문 본문 초안

## 제목
- 한글: YAMNet 기반 한국 가정 환경음 분류 기술
- 영문: Korean Domestic Environmental Sound Classification Based on YAMNet

## 저자
장성준*, 정승민, 홍한희, 장원석, 오준혁
경북대학교 컴퓨터학부

Seongjun Jang*, Seungmin Jeong, Hanhee Hong, Wonseok Jang, Junhyeok Oh
School of Computer Science, Kyungpook National University

## 요 약

본 논문에서는 청각장애인의 가정 내 안전과 일상생활 지원을 위해 YAMNet 기반의 환경음 분류 기술을 제안한다. 제안하는 시스템은 Google의 사전 학습 모델인 YAMNet을 1차 필터로 활용하여 521개 카테고리 중 관련 소리만 선별하고, 한국 가정 환경음에 특화된 2차 분류기를 통해 최종 판별하는 2단계 구조를 채택하였다. 이를 통해 대화나 음악 등 무관한 소리에 의한 오탐을 방지하면서, 도어락, 인터폰, 사이렌 등 한국 가정에 특화된 소리를 효과적으로 인식할 수 있다. 실험 결과, 11개 카테고리에 대해 90.5%의 분류 정확도를 달성하였으며, Raspberry Pi 환경에서 약 1.5~2초의 실시간 추론이 가능함을 확인하였다.

## Abstract

This paper proposes an environmental sound classification system based on YAMNet to support the safety and daily life of hearing-impaired individuals at home. The proposed system adopts a two-stage architecture that uses Google's pre-trained YAMNet model as a primary filter to select relevant sounds from 521 categories, followed by a secondary classifier specialized for Korean domestic environmental sounds. This approach prevents false positives from irrelevant sounds such as speech and music while effectively recognizing sounds specific to Korean households, including door locks, intercoms, and sirens. Experimental results show a classification accuracy of 90.5% across 11 categories, with real-time inference of approximately 1.5 to 2 seconds on a Raspberry Pi platform.

## Key words

environmental sound classification, YAMNet, transfer learning, hearing impaired, IoT, Raspberry Pi

---

## Ⅰ. 서 론

국내 등록 청각장애인은 약 44만 명으로, 이 중 85%가 65세 이상 고령자이며, 장애인 가구의 26.6%가 1인 가구로 나타났다[1]. 청각장애인은 화재경보, 초인종, 아기 울음소리 등 가정 내 중요한 소리를 인지하지 못하여 안전사고 위험에 노출되어 있다. 실제로 2018년 화성 화재, 2020년 부산 화재 사고에서 청각장애인 거주자가 경보음을 듣지 못해 사망하는 사례가 발생하였다. NFPA(미국 소방협회) 2023년 보고서에 따르면, 경보 장치가 미작동할 경우 화재 사망률이 2.2배 높아지는 것으로 나타났다[2].

기존 솔루션으로는 Google의 Sound Notifications, 영국의 Earzz 등이 있으나, 스마트폰 단일 마이크에 의존하거나 서양 가정음 기반으로 학습되어 한국 가정 환경에 적합하지 않은 한계가 있다. 국내 솔루션인 '소리우산'은 음량 기반 감지만 지원하여 소리의 종류를 구분하지 못한다.

이에 본 논문에서는 YAMNet을 활용한 2단계 분류 구조를 제안한다. 1단계에서 YAMNet의 521개 카테고리를 필터로 활용하여 무관한 소리를 차단하고, 2단계에서 한국 가정 환경음에 특화된 분류기를 통해 도어락, 인터폰, 사이렌 등 국내 가정에 특화된 소리를 분류한다. 이를 Raspberry Pi 기반 IoT 시스템으로 구현하여 저비용, 실시간 환경음 인식이 가능하도록 하였다.

---

## Ⅱ. 시스템 설계

### 2.1 전체 구조

제안하는 시스템은 그림 1과 같이 INMP441 I2S MEMS 마이크로 1초 단위의 오디오를 48kHz로 녹음한 후 16kHz로 리샘플링하여 YAMNet 모델에 입력한다. YAMNet의 분류 결과에 따라 직접 매핑, 2차 분류기 전달, 무시의 세 가지 경로로 처리된다. 감지된 소리는 FastAPI 서버를 통해 사용자에게 시각(LED) 및 촉각(진동) 알림으로 전달된다.

시스템은 Hub-Node 구조를 채택하여 Raspberry Pi 1대가 AI 추론을 담당하고, ESP32 노드가 각 방에 설치되어 소리 수집과 알림 출력을 수행한다. 이를 통해 기존 방마다 Raspberry Pi를 설치하는 방식 대비 약 45%의 비용 절감이 가능하다.

### 2.2 YAMNet 기반 1차 필터링

YAMNet은 Google이 AudioSet 데이터셋으로 사전 학습한 521개 카테고리의 오디오 분류 모델로, MobileNet 기반 경량 아키텍처를 사용한다[3]. 본 시스템에서는 YAMNet을 두 가지 용도로 활용한다.

첫째, **소리 필터링**이다. 대화, 음악, 웃음소리, 차량 소리 등 가정 환경음과 무관한 소리를 차단한다. 기존의 단일 분류기 방식에서는 모든 소리가 11개 카테고리 중 하나로 강제 분류되어 대화가 "사이렌"으로, 음악이 "벨소리"로 오분류되는 문제가 있었다. YAMNet의 521개 카테고리를 1차 필터로 활용함으로써 이러한 오탐을 방지한다.

둘째, **직접 매핑**이다. YAMNet이 높은 정확도로 분류 가능한 5개 카테고리(개 짖음, 물 소리, 인터폰, 노크 소리, 창문 깨지는 소리)는 YAMNet의 결과를 직접 사용한다. 해당 카테고리에 대해 신뢰도 0.2 이상일 경우 즉시 알림을 전송한다.

셋째, YAMNet이 관련 소리로 판단하되 직접 분류가 어려운 67개 인덱스의 소리는 2차 분류기로 전달한다.

### 2.3 2차 분류기

2차 분류기는 YAMNet의 1024차원 임베딩 벡터를 입력으로 받아 11개 카테고리에 대한 확률을 출력하는 Dense 네트워크이다. 구조는 Dense(256) → Dropout → Dense(128) → Dropout → Dense(11, softmax)이며, YAMNet의 가중치는 고정하고 Dense 레이어만 학습한다.

학습 데이터는 팀원이 직접 한국 가정 환경에서 수집한 311개의 원본 소리이며, 가우시안 노이즈 추가, 볼륨 변경, 시간 이동 등 10가지 데이터 증강 기법을 적용하여 2,480개의 학습 데이터를 확보하였다. Data Leakage를 방지하기 위해 원본 소리 단위로 Train/Val을 먼저 80:20으로 분리한 후 Train 세트에만 증강을 적용하였다.

2차 분류기가 필요한 카테고리는 도어락, 인터폰, 가전제품, 사이렌, 아기 울음의 5개로, 한국 가정에 특화된 소리 패턴을 학습한다. 예를 들어, 한국의 사이렌은 서양과 패턴이 다르고, 도어락의 "삑삑" 소리는 YAMNet의 학습 데이터에 포함되지 않은 소리이다.

---

## Ⅲ. 실험 및 결과

### 3.1 실험 환경

실험은 Raspberry Pi 4 Model B에서 TensorFlow Lite 런타임을 사용하여 수행하였다. YAMNet 모델(16MB)과 Hearo 분류기(298KB)를 탑재하였으며, INMP441 I2S 마이크로 실시간 오디오를 수집하였다.

### 3.2 분류 성능

표 1은 11개 카테고리에 대한 분류 정확도를 나타낸다. 검증 데이터 63개(원본 소리만, 증강 미적용)에 대해 전체 정확도 90.5%를 달성하였다. 특히 인터폰(99%+), 물 소리 등 YAMNet 직접 매핑 카테고리에서 높은 정확도를 보였다.

[표 1. 카테고리별 분류 정확도 - confusion_matrix.png 참고하여 작성 필요]

### 3.3 추론 속도

1회 추론 사이클은 오디오 캡처(1.0초) + YAMNet 추론(0.3~0.5초) + 2차 분류(0.05~0.1초)로 총 약 1.5~2초가 소요되어 실시간 환경음 인식이 가능함을 확인하였다.

### 3.4 2단계 구조의 효과

2단계 구조 도입 전후를 비교한 결과, 대화/음악 등 무관한 소리에 의한 오탐이 완전히 차단되었다. 단일 분류기 사용 시 TV 소리가 "사이렌"(60%)으로 분류되던 문제가 YAMNet 필터를 통해 "Speech/Music"으로 정확히 분류되어 차단되었다.

---

## Ⅳ. 결 론

본 논문에서는 YAMNet 기반 2단계 분류 구조를 활용한 한국 가정 환경음 분류 기술을 제안하였다. 제안 방법은 YAMNet의 521개 카테고리를 1차 필터로 활용하여 무관한 소리를 차단하고, 한국 가정음에 특화된 2차 분류기를 통해 11개 카테고리에 대해 90.5%의 정확도를 달성하였다. Raspberry Pi 환경에서 약 1.5~2초의 실시간 추론이 가능하며, Hub-Node 구조를 통해 기존 대비 45%의 비용 절감을 실현하였다.

향후 연구에서는 수집 데이터를 확대하여 15개 이상의 카테고리로 확장하고, 실제 청각장애인 사용자를 대상으로 현장 테스트를 수행할 계획이다. 또한 ESP32 노드와의 MQTT 통신을 구현하여 다중 공간 환경에서의 실증을 진행할 예정이다.

---

## 참고문헌 (본문 내 인용 번호)

[1] 한국장애인고용공단, "2024 장애인 통계," 2024.
[2] NFPA, "Smoke Alarms in U.S. Home Fires," 2023.
[3] Google Research, "YAMNet: Yet Another Mobile Network for Audio Classification," TensorFlow Hub, 2020.
[4] 이하 references.md 참고
