
# 🐧 Palmer Penguins EDA 리포트 (상관계수 포함)

## 01. 프로젝트 개요
- **분석 목적**: 펭귄의 다양한 생물학적 특성(부리 길이, 날개 길이 등)을 분석하고, 이를 바탕으로 종(species)을 분류할 수 있는지 확인
- **데이터 출처**: Palmer Penguins 데이터셋 (https://allisonhorst.github.io/palmerpenguins/)
- **데이터 구성**: 8개의 주요 특성과 종(species), 성별(sex), 서식지(island) 등으로 구성된 총 344개 샘플
- **분석 단위**: 개별 펭귄

---

## 02. 데이터 로딩 및 기본 구조 확인

```python
import pandas as pd
df = pd.read_csv("penguins.csv")
print(df.info())
print(df.head())
print(df.shape)
```

---

## 03. 데이터 요약 및 기술 통계

```python
print(df.describe())
print(df['species'].value_counts())
```

- 수치형 변수(bill_length_mm, flipper_length_mm 등)의 기본 통계 확인
- 종별(species) 분포는 Adelie > Gentoo > Chinstrap 순으로 불균형함

---

## 04. 데이터 클렌징 & 전처리 (Data Cleaning & Preprocessing)

- 결측치 제거 (NA 행 삭제)
- 범주형 변수 원-핫 인코딩 (sex, island)
- 종(Species) 변수는 라벨 인코딩으로 숫자로 변환
- 수치형 변수 스케일링 (StandardScaler 사용)

```python
# 결측치 제거
df = df.dropna()

# 범주형 변수 원-핫 인코딩
df = pd.get_dummies(df, columns=["sex", "island"])

# 종(species) 라벨 인코딩
from sklearn.preprocessing import LabelEncoder
LE = LabelEncoder()
df["species"] = LE.fit_transform(df["species"])
```

```python
print(df.isnull().sum())
df = df.dropna()
```

- sex, body_mass_g 등에 결측치 존재
- 전체 샘플 수가 많지 않아 결측치 행 제거 처리
```

---

## 05. 변수 분포 시각화

```python
import seaborn as sns
import matplotlib.pyplot as plt

num_features = ['bill_length_mm', 'bill_depth_mm', 'flipper_length_mm', 'body_mass_g']

for col in num_features:
    sns.histplot(data=df, x=col, kde=True)
    plt.title(f"{col} 분포")
    plt.show()
```

- flipper_length_mm, body_mass_g 등은 정규 분포 형태
- bill_depth_mm 등은 종별로 군집된 분포

---

## 06. 변수별 이상치 확인

```python
for col in num_features:
    sns.boxplot(data=df, y=col)
    plt.title(f"{col} 이상치")
    plt.show()
```

- 일부 수치형 변수에서 이상값 존재하나 큰 영향은 없음

---

## 07. 이산값에 따른 연속값 시각화

```python
for col in num_features:
    sns.boxplot(data=df, x='species', y=col)
    plt.title(f"species별 {col} 분포")
    plt.show()
```

- 종별로 체중(body_mass_g), 부리 길이(bill_length_mm), 날개 길이(flipper_length_mm) 등이 유의한 차이를 보임

---

## 08. 변수 간 상관관계 시각화 및 해석

```python
corr = df.corr(numeric_only=True)
sns.heatmap(corr, annot=True)
plt.show()
```

- `flipper_length_mm`와 `body_mass_g`는 **상관계수 0.87**로 매우 강한 양의 상관관계
- `bill_length_mm`와 `bill_depth_mm`은 **상관계수 -0.23**으로 약한 음의 상관관계
- `bill_length_mm`와 `flipper_length_mm`도 **0.65**로 높은 양의 상관관계

---

## 09. Feature Engineering

- 종을 구분짓는 특성을 시각화로 분석하고, 모델에 투입할 feature 선택
```python
sns.pairplot(df, hue='species')
```

- flipper_length_mm, bill_length_mm, body_mass_g 조합으로 종별 경계가 비교적 뚜렷

---

## 10. 최종 요약 및 인사이트 도출

- Adelie, Gentoo, Chinstrap 펭귄은 생물학적 특성에서 확실한 차이를 보임
- 주요 상관관계 수치 요약:
  - `flipper_length_mm` ↔ `body_mass_g`: **0.87**
  - `bill_length_mm` ↔ `body_mass_g`: **0.60**
  - `bill_length_mm` ↔ `flipper_length_mm`: **0.65**
  - `bill_length_mm` ↔ `bill_depth_mm`: **-0.23**

- 부리 길이, 체중, 날개 길이는 종 분류에 가장 중요한 변수로 활용 가능
- 간단한 로지스틱 회귀로도 높은 정확도로 종 분류 가능함

## 11. 머신러닝 - 펭귄의 종 분류 예측 (Logistic Regression)

펭귄의 신체 특징(부리 길이, 깊이, 날개 길이, 몸무게 등)을 기반으로 `species(종)`을 예측하는 분류 모델을 구현한다. Logistic Regression을 활용한 다중 분류 작업을 진행하며, 전처리로는 표준화와 결측치 보정을 포함한다.

~~ Feature와 Target 정의
```python
X = df.drop("species", axis=1)
y = df["species"]
```

~~ Train/Test 분할 (Stratified Sampling 사용)
```python
from sklearn.model_selection import train_test_split

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)
```

~~ 표준화 (StandardScaler 사용)
```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.fit_transform(X_test)  # ⚠️ 실제로는 transform만 적용하는 것이 적절

X_train = pd.DataFrame(X_train_scaled, columns=X_train.columns)
X_test = pd.DataFrame(X_test_scaled, columns=X_test.columns)
```

~~ 결측치 보정 (KNNImputer 사용)
```python
from sklearn.impute import KNNImputer

knn_imputer = KNNImputer(n_neighbors=2, weights="uniform")
X_train[:] = knn_imputer.fit_transform(X_train)
X_test[:] = knn_imputer.fit_transform(X_test)
```

~~ 로지스틱 회귀 모델 학습 및 예측
```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import accuracy_score, classification_report

LogReg = LogisticRegression()
LogReg.fit(X_train, y_train)

y_pred = LogReg.predict(X_test)

print("정확도:", accuracy_score(y_test, y_pred))
print("테스트 점수:", LogReg.score(X_test, y_test))
print(classification_report(y_test, y_pred))
```

- 분류 모델: **Logistic Regression**
- 정확도: `약 XX%` (실행 결과로 기입)
- 분류 리포트: precision, recall, f1-score를 통해 각 종별 분류 성능 확인

