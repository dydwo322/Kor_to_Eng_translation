# 딥러닝을 이용한 한글-영분 번역기<br>(RNN model : Seq2seq with attention)
<br>

## 0. 목차
<table>
    <thead>
        <tr align=center>
            <th>번호</th>
            <th>목차</th>   
        </tr>
    </thead>
    <tbody>
        <tr align=center>
            <td>1</td>
            <td><a href="#1">요 약</a></td>
        </tr>
        <tr align=center>
            <td>2</td>
            <td><a href="#2">성능개선을 위한 노력</a></td>
        </tr>
        <tr align=center>
            <td>3</td>
            <td><a href="#3">결 과</a></td>
        </tr>
        <tr align=center>
            <td>4</td>
            <td><a href="#4">결론 및 한계점</a></td>
        </tr>              
     </tbody>
</table>
<br>

## 1.<a name="1">요 약</a>

![스크린샷_20221210_044906](https://user-images.githubusercontent.com/113493695/206839388-0b94509a-f754-4655-bbb4-169de9b50b5c.png)

### - 모델 핸들
○ LSTM & GRU (+ ATTENTION, BI LSTM)<br> 
<br>
○ PARAMS (hidden units, dropout, embedding dim)<br> 
<br>
○ Going deeper<br> 
<br>
○ Uniform Distribution<br> 
<br>
○ Transformer<br> 
<br>
### - 데이터 핸들링
○ 형태소 분석기 (5가지)<br> 
<br>
○ 데이터 추가<br> 
<br>
○ Transformer<br> 
<br>
○ Reverse<br> 
<br>
○ Sentences max length<br>
<br>
○ 불용어 사전<br> 
<br> 

## 2.<a name="2">성능개선을 위한 노력</a>

![스크린샷_20221210_045726](https://user-images.githubusercontent.com/113493695/206839849-8b898050-c619-4c1b-a5d5-22040943248f.png)

### Dataset 
- 문장 70만개  
ㆍ구어체 : 자연스러운 구어체 문장 → 400,000 문장  
ㆍ대화체 : 상황/시나리오 기반 대화 세트 → 100,000 문장 (추가)  
ㆍ문어체(뉴스) : 뉴스 텍스트 → 200,000 문장 (추가)

### Best 형태소 분석기 : Mecab (with LSTM)

![스크린샷_20221210_045954](https://user-images.githubusercontent.com/113493695/206840030-6dce1b57-7b38-476b-bf6f-d89f8a5bb0d1.png)

### Seq2seq 모델 비교 : LSTM vs GRU (with Mecab)

![스크린샷_20221210_050133](https://user-images.githubusercontent.com/113493695/206840082-238812d5-91a1-4cfd-9d05-4fc80b15d8d2.png)

### 모델 성능 향상 : Hidden Units Control

![스크린샷_20221210_050252](https://user-images.githubusercontent.com/113493695/206840134-79fef1d8-d154-4977-8e3e-62144d66bdef.png)

### 모델 성능 향상 : Embedding Dim Control

![스크린샷_20221210_050334](https://user-images.githubusercontent.com/113493695/206840155-393a09fc-2c2d-4417-b236-34d9f1f5661e.png)

### 모델 성능 향상 : Going Deeper
- Depth

![스크린샷_20221210_050427](https://user-images.githubusercontent.com/113493695/206840195-0a72e1c1-e59e-42ba-8682-566e6e13b7dc.png)

- Dropout

![스크린샷_20221210_050623](https://user-images.githubusercontent.com/113493695/206840267-bb06ea11-5b42-4664-bbc7-d03cf1a85eb0.png)

- Hidden Units, Embedding Dim

![스크린샷_20221210_050700](https://user-images.githubusercontent.com/113493695/206840291-da83082a-59d3-43ba-bc97-1ad25d97feed.png)

- Data Preprocessing

![스크린샷_20221210_050943](https://user-images.githubusercontent.com/113493695/206840403-1864671c-d80d-40ea-be63-7409f497c737.png)

### 중간 점검 및 피드백

- 학습 후 Train Data로 예측한 결과

![스크린샷_20221210_051053](https://user-images.githubusercontent.com/113493695/206840437-4cb4c37e-8451-4712-801f-d97e75d43034.png)

- 높은 Val_Acc에도 불구하고 대부분의 문장 오역 및 낮은 BLEU Score   
ㆍ오역  
ㆍ같은 단어 반복   
ㆍ이전 문장의 번역 단어 재등장

- 위 문제를 해결하기 위해 추가적인 핸들링 필요 판단   
  1. 데이터 추가 학습 (40만개 → 70만개)
  
  ![스크린샷_20221210_051545](https://user-images.githubusercontent.com/113493695/206840727-d4da26a1-b36f-48f2-a60a-225fe2657e61.png)
  
      데이터 추가 시 기대효과
          1) Val_Acc 향상
          2) Overfit 일부 해소
          3) 데이터 다양화 (대화체, 문어체)

  2. 문장 별 토큰 수 기준 긴 문장 삭제
  
  ![스크린샷_20221210_051812](https://user-images.githubusercontent.com/113493695/206840813-9a2a8914-d31b-43f2-87ad-cc6c72f58f3f.png)
  
     대부분의 문장이 30개 이하의 토큰으로 이루어짐 → 30개 초과 샘플 삭제 (패딩 효율↑)
  
  3. 추가 모델 핸들링
  
  ![스크린샷_20221210_052122](https://user-images.githubusercontent.com/113493695/206840921-ca07f526-99f8-42d8-a3f6-163825e0bef3.png)   
   Test 결과 → 주요 단어는 어느정도 짚어내지만, 이전과 동일한 결과 지속 (오역, 같은 단어 반복)
   
   ![스크린샷_20221210_052304](https://user-images.githubusercontent.com/113493695/206840967-a90f714c-b8a6-4d88-a10f-0e1d92574423.png)  
   로 미흡한 수준의 번역 결과, 이전과 동일한 결과 지속 (오역, 같은 단어 반복)

## 3.<a name="3">결 과</a>
### 평가지표 : Val_ACC

![스크린샷_20221210_052516](https://user-images.githubusercontent.com/113493695/206841018-27c0c6d4-cfe7-4846-945b-86d2c0dd9fc8.png)  
ㆍVal_Acc가 높다고 만족스러운 번역 결과를 얻지 못 함.  
ㆍ따라서 평가 지표를 1) 사람에 의한 평가 및 2) BLEU score로 변경.

### 평가지표 변경 : 사람에 의한 평가 & BLEU score

![스크린샷_20221210_052649](https://user-images.githubusercontent.com/113493695/206841068-a8e76287-f9ed-4571-803a-f36baefa14ff.png)  
ㆍ지금까지 학습한 모델들로부터 사람에 의한 평가 및 BLEU score 확인   
ㆍ번역 결과가 괜찮다고 생각하는 2가지 모델 선정 (① LSTM depth1, ② LSTM depth1 with Attention)

### 추가 모델핸들링 : Transformer

Transformer  
    ㆍPositional Encoding 사용 (RNN 사용 X)  
    ㆍ잔여학습(Residual Learning) 사용 (for 성능향상)  
    ㆍAttention(어텐션)과 Normalization(정규화) 과정 반복  
    ㆍ문장 전체의 Attention 값 계산, 한 번의 계산에 병렬적으로 출력값 획득 → 계산 복잡도↓


- Seq2seq with attention 모델의 한계점 보완
- Transformer를 사용하면 더 좋은 성능을 획득 가능

![스크린샷_20221210_053100](https://user-images.githubusercontent.com/113493695/206841231-ca25a903-1534-4dca-9d4d-59058974f417.png)  
- ㆍTransformer을 이용해 학습한 모델로 사람에 의한 평가 확인  
-  사람에 의한 평가를 기준으로 최종 모델 선정 (Transformer model)

## 4.<a name="4">결론 및 한계점</a>

<한계점>  
- layer가 깊어짐에 따라 val acc는 증가하였으나, 결과값 및 예측값에서 특정 단어가 반복되어 나오는 현상 조치  
- OOV 문제 해결 필요  
- BLEU score 향상을 위한 데이터 추가 필요  
- 추가 데이터 전처리 과정 필요  

<결론>  
- 다양한 데이터 및 모델 핸들링 시도  
- 사람에 의한 평가 및 BLEU score를 통해 선택된 LSTM depth1 with Attention은 나쁘지 않으나 충분히 만족스러운 결과 획득 X  
- Transformer를 이용하여 한글-영문을 번역기 구축
