---
layout: post
title: "[AI] 개발자가 알아야 할 Foundation Model 이야기"
date: 2024-09-29
tags: [AI, LLM, Foundation Model]
categories: AI
excerpt_image: https://blogs.nvidia.com/wp-content/uploads/2023/03/Transformer-apps.jpg
---
![banner](https://blogs.nvidia.com/wp-content/uploads/2023/03/Transformer-apps.jpg)

요즘 AI가 정말 빠르게 발전하고 있고, 산업 전반을 완전히 바꿔놓고 있는 느낌이다.

특히 최근 주목받는 기술 중 하나가 바로 LLM (Large Language Model)인데, 해외에서는 이미 AI가 대부분의 코드 작성 업무를 담당하고, 개발자는 설계, 테스트, 최적화, 검증 같은 더 본질적인 일에 집중하는 '바이브 코딩(Vibe Coding)'이라는 접근법도 유행 중이다.

이렇게 LLM을 활용하다 보면 자주 듣게 되는 개념이 있는데, 바로 'Foundation Model(기초 모델)'이다. 처음 들으면 "이게 도대체 뭐지?" 싶을 수도 있어서 이번에 간단히 정리해봤다.

## Foundation Model 그래서 뭔데?
Foundation Model 이라는 용어는 2021년에 발표된 "On the Opportunities and Risks of Foundation Models" 라는 제목의 논문에서 처음 정의가 되었다. 이 논문에서 말하는 바는 결국 Transformer 기반의 대규모 언어모델이나 기타 신경망 모델들이 Foundation Model 이라는 새로운 범주를 구성한다고 있다고 분석하였다.  
Foundation Model은 단순하게 얘기하면 방대한 양의 데이터를 미리 학습시켜 놓은 범용 AI 모델이다. 여기서 방대한 양의 '데이터' 라고 했을 때 이 데이터는 지도학습을 위한 레이블이 없는 데이터로, 이렇게 레이블도 없는 많은 양의 데이터로만 특정 형태의 신경망을 학습시켜둔 상태를 Foundation Model 이라 할 수 있다. 다시 말해, 빅데이터를 기반으로 pre-trained 된 인공 신경망 모델이 foundation model 에 해당하고, 이 모델을 이후 여러 특정 작업에 맞게 fine-tuning 작업을 거쳐서 현재 우리가 널리 알고있는 ChatGPT, Claude, LLAMA 등의 Text generation LLM 모델로써 서비스될 수 있다. 

## 왜 'Foundation'이라고 부를까?

건축에서 기초가 튼튼해야 건물을 잘 지을 수 있는 것처럼, Foundation Model은 AI 응용 프로그램을 만드는 데 있어서 튼튼한 기반 역할을 한다. 잘 만들어진 기초 모델 하나만 있으면, 약간의 추가 학습(파인튜닝)만으로 챗봇, 자동 번역기, 글쓰기 도구 등 다양한 애플리케이션을 쉽게 구현할 수 있기 때문이다.

## Foundation Model 의 특징
1. 범용성: 하나의 모델이 다양한 작업에 적용될 수 있도록 설계되어, 특정 작업에 맞춰 미세 조정(fine-tuning)하거나 프롬프트를 사용하여 다양한 작업에 활용될 수 있다.

2. 적응력: 기존의 모델을 약간의 추가 학습만으로도 새로운 작업에 빠르게 적용할 수 있어, 시간과 비용을 절감할 수 있다.

3. 데이터 다양성: 텍스트, 이미지, 오디오 등 다양한 형태의 데이터를 사용하여 훈련되며, 이를 통해 다양한 도메인에서 활용이 가능하다.

## Foundation Model과 LLM의 중요성

Foundation Model이 중요한 이유는 크게 효율성과 범용성 때문이다. 과거 LLM 등장 이전에는 특정 작업을 수행할 때마다 새롭게 AI 모델(특정 신경망 구조) 을 개발하고 학습해야 했지만, Foundation Model을 사용하면 이미 학습된 모델을 약간의 추가 학습으로 빠르게 새로운 작업에 적용할 수 있어 시간과 비용이 크게 절약된다.

## 그렇다면 앞으로의 전망은?

