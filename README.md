# Sepsis_Antibiotics

### Object

패혈증(Sepsis)은 미생물에 감염되어 전신에 심각한 염증 반응이 나타나는 상태를 말한다.
패혈증의 치료는 적절한 항생제를 사용하여 원인이 되는 장기의 감염을 치료하는 것이 중요하다.
패혈증의 원인균을 알아내기 위해서는 환자의 혈액을 채취하여 균을 배양하는 검사가 필요하지만
이는 적어도 3~5일 정도의 시간이 필요하므로 만일 환자의 상태가 위독하다면 배양 검사 결과가 나오기 전에 경험적인 치료를 시행해야 한다.
따라서 균 배양검사 결과가 나오기 전, 보다 유효한 항생제를 예측하여 처방할 수 있다면 환자의 예후 향상에 큰 도움이 될 것이라 생각되어,
'패혈증 환자의 데이터분석에 기반한 항생제 추천 보조 ai모델'을 주제로 개발하였다.

### Data

이 프로젝트에서는 MIMIC-IV database에서 sepsis환자들의 vital sign, lab data, demographic 등을 사용하였다. 
MIMIC-IV는 미국 매사추세츠주 보스턴에 있는 3차 학술 의료기관에 입원한 환자의 실제 입원 기간을 포함하는 관계형 데이터베이스로
의료 분야의 다양한 연구를 지원하기 위해 만들어졌고, laboratory measurements, medications administered, vital signs 등 각 환자에 대한 포괄적인 데이터를 포함하고 있습니다.
https://physionet.org/content/mimiciv/2.1/

패혈증환자는 sepsis-3 정의에 따라 32365개의 내원번호(hadm_id)를 추려서 사용하였습니다.
    - That is, the earliest time at which a patient had SOFA >= 2 and suspicion of infection.
    - Implicitly this assumes baseline SOFA was 0 before ICU admission.

Input features

dynamic features = ['FiO2', 'HR', 'PaO2', 'SBP', 'Temperature','Creatinine','Glucose', 'Hb', 'Lactate', 'Plt','Sodium' ,'WBC', 'DBP', 'MAP', 'RESP', 'Chloride', 'Hct', 'PCO2', 'Potassium', 'PH', 'Anion Gap', 'Bilirubin']

    - input 1
    - 입실 시점으로부터 1시간 간격으로 추출
    - 1시간 간격으로 가장 마지막 값을 활용
    - 입실 이후 1) 최대 24시간 동안의 자료만 활용, 2) 48시간 동안의 자료만 활용
    - 입실과 퇴실의 간격이 1시간 이하인 경우 삭제
    - 소수점 둘째자리까지 사용
    - Missing value인 경우 carry forward하고, `확인필요` 보다 짧은 경우 Normal value로 대체
    - 변수별로 MIN, MAX의 Normal range 를 설정해서 정규화
    - 24시간보다 짧은 경우 zero padding함

static features = ['weight', 'age', 'gender', 'insurance', race(Asian, Black, Hispanic, White)]

    - input 2
    - min-max normalization

### Model

#### 전체 모델
![그림2](https://user-images.githubusercontent.com/96029849/204230104-bf1383b1-fd3e-47d9-b94a-bdf4ae8e1461.png)

    - 개별 모델에서 sepsis 환자의 dynamic features(vital sign, lab data), static features(demographics, weight etc)을 input으로 환자의 생존, 사망을 예측
    - 사용 항생제 별로 학습 data와 모델을 나누어 다수의 사망 예측모델을 구축하여 사망율을 예측 > 가장 사망율이 낮을 것이라고 예측하는 모델의 항생제를 채택 

#### 개별 모델
<img src="https://user-images.githubusercontent.com/96029849/204230316-2a35dd77-a1e3-427c-b503-e1540a005bbc.png" width="400" height="1000"/>

### 결과

#### 개별 모델 성능

<img src="https://user-images.githubusercontent.com/96029849/204468615-71d8c7b0-4e51-417a-9e51-00a946b0e1c0.png" width="600" height="400"/>
<img src="https://user-images.githubusercontent.com/96029849/204468800-8aa0170b-ebd6-4d55-9f4f-8a9cad899ef2.png" width="400" height="400"/>

- 



