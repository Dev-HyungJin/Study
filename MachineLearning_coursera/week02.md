# Week2
## 1. Multivariate Linear Regression
### 1.1. Multiple Features
**Multivariate Linear Regression(다변량 선형 회귀)**
다중 변수를 사용하는 선형회귀를 다변량 선형회귀라 한다

$$\begin{align*}x_j^{(i)} &= i\text{번째 학습 예제의 }  j \text{번째 피처 값} \newline x^{(x)} &=  i\text{번째 학습 예제의 피처 입력 값들} \newline m &= \text{학습 예제의 수} \newline n &= \text{피처의 수} \end{align*}$$

다중변수가 적용된 가설 함수는 아래와 같다
$$\begin{align*}h_{\theta}(x) &= \theta_0 + \theta_1x_1 + \theta_2x_2 + \theta_3x_3 + ... + \theta_nx_n  \end{align*}$$

이 함수의 각 인자는 같이 생각할 수 있다.

|||||
|-|-|-|-|
|$$\theta_0$$|기본 집 값| | |
|$$\theta_1$$|평당 가격|$$x_1$$|평수|
|$$\theta_2$$|층당 가격|$$x_2$$|층수|

행렬 곱의 정의에 따라 가설함수는 아래처럼 간결하게 표시할 수 있다.
$$\begin{align*}h_\theta(x) =\begin{bmatrix}\theta_0 \hspace{2em} \theta_1 \hspace{2em} ... \hspace{2em} \theta_n\end{bmatrix}\begin{bmatrix}x_0 \newline x_1 \newline \vdots \newline x_n\end{bmatrix}= \theta^T x \\ x_0 &= 0 \end{align*} $$


### Gradient Descent for Multiple Variables
 
 다변량 션형회귀의 기울기 하강 공식은 아래와 같다.
 (j번째 피처,  i번째 학습 예제) 

 $$\theta _{ j }\\ =\frac { \delta  }{ \delta \theta _{ i } } J(\theta _{ 0 },\theta _{ 1 })\\ =\alpha \frac { \delta  }{ \delta \theta _{ i } } \frac { 1 }{ 2m } \sum _{ i=1 }^{ m }{ \left\{ { h }_{ \theta  }(x^{ (i) })-y^{ (i) } \right\} ^{ 2 } } \\ =\alpha \frac { 1 }{ m } \sum _{ i=1 }^{ m }{ \left\{ { h }_{ \theta  }(x^{ (i) })-y^{ (i) } \right\} x_j^{ (i) } } $$
 
### Gradient Descent  in Practice I- Feature Scaling

 기울기 하강에서 사용하는 입력값을 동일한 범위로 한정시키면 속도를 올릴 수 있다.
 θ는 범위가 작을 수록 더 빨리 하강하고 변수 값이 고르지 않을 때 비 효율적으로 진동하기 때문이다.
 이 문제를 피하기 위해 입력 값을 −1 ≤ x~(i)~ ≤ 1 또는 −0.5 ≤ x~(i)~ ≤ 0.5 로 수정 할 수 있다.
입력 값을 정확하게 범위 안에 넣을 필요는 없고 너무 작거나 큰 범위안에 입력값이 들어가지 않도록만 하면된다.

$$x_{ i }:=\frac { x_{ i }-\mu _{ i } }{ s_{ i } }\\ s_i\text{는 max-min 이나 standard deviation} \\ \mu_i\text{는 피처 값들의 평균}$$

### Gradient Descent  in Practice II- Learning Rate

**기울기 하강 디버깅** : 기울기 하강 함수를 반복적으로 수행한 뒤 J(θ)의 값을 그래프로 그려 값이 계속 감소하는지 확인한다.

**자동 수렴 테스트** : J(θ) 값의 감소 폭이 지정된 E 값 보다 작은지 검사한다. 10^-3^ 처럼 작은 E값을 지정해야 하는데 이 값을 정하기가 쉽지 않다.

수학적으로 α가 충분히 작을 때   J(θ)는 항상 수렴하게 되어 있기 때문에 J(θ) 값이 수렴하지 않고 지속적으로 증가하거나 오르락 내리락 한다면 α값을 더 작게 할 필요가 있다. 하지만 너무 작은 α를 지정하면 수렴 속도가 늘지기 때문에 주의해야한다.

### Features and Polynomial Regression

가설함수의 피처값을 향상시키기 위한 두가지 방법이 있다.

하나는 여러가지 피처 값을 합쳐 하나의 값으로 만드는 것이다.
예를 들어 x~1~, x~2~값이 있다면 두 값의 곱으로 x~3~를 만들어 사용할 수 있다.

**다항 회귀**
만약 데이터와 더 일치하는 가설 함수를 만들 수 있다면 가설 함수의 결과가 항상 직선일 필요가 없고 x를 2차, 3차 혹은 제곱근으로 바꿔 다항식으로 사용할 수 있다.
예를 들어 h~θ~(x) = θ~0~+θ~1~x~1~^2^+θ~2~x~1~^3^ 라면 아래처럼 x~2~, x~3~를 치환해 사용할 수 있다.
x~2~ = x~1~^2^
x~3~ = x~1~^3^

이때 각 피처 값의 범위가 기하급수적으로 커질 수 있다는 점을 주의해야한다.

### NormalEquation

J(θ)의 도함수의 결과가 0이 되는 θ 값을 찾는 방법

θ = (X^T^X)^-1^X^T^y (θ는 벡터)

기울기하강과 정규방정식의 장단점
>|기울기 하강|정규방정식|
>|-|-|
>|α를 선택해야함|α를 선택할 필요 없음|
>|반복 수행해야함|반복수행 필요 없음|
>|O(kn^2^)|O(n^3^)|
>|n이 클때 잘 동작함|n이 크면 느려짐|
>
>*n이 약 10000개 이상이면 둘중 어느것을 쓸지 고민하기 시작함

### Normal Equation Noninvertibility

아래 상황에서 X는 역행렬을 가지지 못할 수 있다.
>X행렬의 두 피처가 중복되는 경우
>피처 값이 학습 예제보다 너무 많이 있는 경우
하지만 Octave와 같은 프로그램에서는 pseudo inverse (pinv)함수를 제공하기 때문에 이 상황에서도 역행렬를 가질 수 있다.


