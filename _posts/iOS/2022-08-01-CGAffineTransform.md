---
layout: post
title: CGAffineTransform에 대해
date: '2022-08-01 22:43:10 +0900'
categories: [iOS]
tags: [iOS, CGAffineTransform]
math: true
---



## 계기

토이 프로젝트 진행하면서 Custom View의 이동 방법에 대해 공부하다가 `CGAffineTransform`에 대해 새로 알게 되었습니다.

View를 직접 이동시키기위해서 Constraints를 업데이트하는 방법도 있었고, CGAffineTransform을 이용해 View가 보이는 위치를 바꿔주는 방법도 있었는데, 이번 포스팅에서는 CGAffineTransform에 대해 정리해보고자 합니다.



## Affine Transform?

아핀 변환은 점, 직선, 평면을 보존하는 선형 매핑 방법입니다. (여기서 "보존"이라는 단어의 의미는, 아핀 변환 이후에도 평행한 선들은 평행한 상태로 유지되는 등의 상태를 말하는 것 같습니다.)

`CGAffineTransform`에서 다룰 아핀 변환은 아래와 같은 변환이 있습니다.

| 아핀 변환 |                              예                              | 변환 행렬                                                    | 설명                                                         |
| :-------: | :----------------------------------------------------------: | ------------------------------------------------------------ | ------------------------------------------------------------ |
| 평행이동  | <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211021351991.jpg" alt="SS2022-11-02PM01.51.37" width="70%;" /> | $$\begin{bmatrix}1&0&0\\0&1&0\\t_x&t_y&1\\ \end{bmatrix}$$   | t<sub>x</sub>는 x축 방향 변위를, <br />t<sub>y</sub>는 y축 방향 변위를 나타냄 |
| 스케일링  | <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211021350545.jpg" alt="SS2022-11-02PM01.50.12" width="70%;" /> | $$\begin{bmatrix}s_x&0&0\\0&s_y&0\\0&0&1\\ \end{bmatrix}$$   | s<sub>x</sub>는 x축에서의 스케일링 인자를, <br />s<sub>y</sub>는 y축에서의 스케일링 인자를 나타냄. |
|   회전    | <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211021350772.jpg" alt="SS2022-11-02PM01.50.39" width="70%;" /> | $$\begin{bmatrix}cos(a)&sin(a)&0\\-sin(a)&cos(a)&0\\0&0&1\\ \end{bmatrix}$$ | a는 회전 각도를 나타냄.                                      |

위에서 주어진 **<u>변환 행렬</u>**을 이용해 기존의 점, 직선, 평면을 계산하게 됩니다.

기존의 점을 $(x, y, 1)$, 아핀 변환 중 2배 스케일링 변환된 점을 $(x’, y’, 1)$이라고 할 때, $(x’, y’, 1)$은 아래의 수식으로 구할 수 있습니다.


$$
(x', y', 1) = (x, y, 1) \times \begin{bmatrix}2&0&0\\0&2&0\\0&0&1\end{bmatrix}
\\
x' = 2x, \;\;\;y' = 2y
$$




## CGAffineTransform?

Swift에서는 위의 아핀 변환을 `CGAffineTransform`이라는 Struct를 이용해 제공해줍니다.

위에서 알아봤던 변환 행렬을 알 필요 없이 CGAffineTransform 구조체는 초기화 생성자를 이용해 아핀 변환 후 매핑될 좌표를 모두 계산합니다.

![SS2022-11-04AM11.57.32](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211041157829.jpg)

CGAffineTransform 구조체의 초기화 생성자는 위와 같습니다.

초기화 생성자로 아핀 변환 중 rotate, scale, translate 변환을 제공해주는 것을 확인할 수 있습니다.

또한 맨 아래 2개의 초기화 생성자를 통해 직접 변환 행렬을 정의해주는 아핀 변환도 가능합니다. 해당 초기화 생성자들의 매개변수는 아래의 변환 행렬을 기준으로 한 매개변수 입니다.


$$
\begin{bmatrix}a&c&0\\c&d&0\\t_x&t_y&1\ \end{bmatrix}
$$

1.   `init(rotationAngle:)` 초기화 생성자 사용 예시

     

     <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211041350575.gif" alt="SS2022-11-04PM01.50.25" width="50%;" />

     `init(rotationAngle: CGFloat)` 초기화 생성자의 경우, 매개변수에 회전하고자 하는 <u>회전 각</u>을 넣어주면 됩니다. (제 경우에는 `.pi`, `.pi/4` 등과 같은 값을 넣으면 나중에 알아보기 편했습니다.)

     ```swift
     func rotateButtonDidTap() {
         if didRotate {
              UIView.animate(withDuration: 0.5) {
                 self.targetView.transform = CGAffineTransform(rotationAngle: .pi/4)
             } completion: { [weak self] _ in
                 self?.didRotate = false
             }
         } else {
             UIView.animate(withDuration: 0.5) {
                 self.targetView.transform = .identity
             } completion: { [weak self] _ in
                 self?.didRotate = true
             }
         }
     }
     ```
     
     
     
     

2.   `init(scaleX:y:)` 초기화 생성자 사용 예시

     <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211041354019.gif" alt="SS2022-11-04PM01.53.52" width="50%;" />

     `init(scaleX: CGFloat, y: CGFloat)` 초기화 생성자의 경우, 매개변수에 변환하고자 하는 배율을 넣어주면 됩니다.

     
     
     ```swift
func scaleButtonDidTap() {
         if didScale {
             UIView.animate(withDuration: 0.5) {
                 self.targetView.transform = CGAffineTransform(scaleX: 2.0, y: 2.0)
             } completion: { [weak self] _ in
                 self?.didScale = false
             }
         } else {
             UIView.animate(withDuration: 0.5) {
                 self.targetView.transform = .identity
             } completion: { [weak self] _ in
                 self?.didScale = true
             }
         }
     }
     ```
     
     

     

3.   `init(translationX:y:)` 초기화 생성자 사용 예시

     <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211041358357.gif" alt="SS2022-11-04PM01.56.53" width="50%;" />

     `init(translationX: CGFloat, y: CGFloat)` 초기화 생성자의 경우, 매개변수에 x, y 축 방향으로 이동하고자 하는 변위만큼의 값을 넣으면 됩니다.

     

     ```swift
     func translateButtonDidTap() {
         if didTranslate {
             UIView.animate(withDuration: 0.5) {
                 self.targetView.transform = CGAffineTransform(translationX: 40, y: 40)
             } completion: { [weak self] _ in
                 self?.didTranslate = false
             }
         } else {
             UIView.animate(withDuration: 0.5) {
                 self.targetView.transform = .identity
             } completion: { [weak self] _ in
                 self?.didTranslate = true
             }
         }
     }
     ```

     

     

![SS2022-11-04PM12.03.47](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211041203600.jpg)

또한 CGAffineTransform 구조체가 제공해주는 타입 프로퍼티에는 `identity`가 있습니다.

해당 프로퍼티는 아핀 변환 이전의 형태로 다시 돌아갈 때 사용하는 프로퍼티입니다. 아래와 같이 사용하면 아핀 변환된 View가 다시 이전의 상태로 돌아갑니다.

```swift
UIView.animte(withDuration: 0.5) {
    self.View.transform = .identity
}
```



![SS2022-11-04PM02.19.38](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211041419258.jpg)

CGAffineTransform 구조체가 제공해주는 인스턴스 메소드는 위와 같습니다.

`concatenating(_:)` 는 <u>여러 CGAffineTransform의 변환 행렬들을 곱</u>하여 새로운 변환 행렬을 가지는 CGAffineTransform을 만들어주는 메소드입니다.

**행렬의 곱셈**은 교환 법칙이 성립하지 않습니다. 따라서 `concatenating(_:)` 메소드를 이용하여 새로운 변환 행렬을 만들고자 할 때는 CGAffineTransform 객체들의 순서를 생각하여야 합니다.



### 참고 자료

[[Apple-Developer-CGAffineTransform]](https://developer.apple.com/documentation/corefoundation/cgaffinetransform)

[[MathWorks-Affine Transform]](https://kr.mathworks.com/discovery/affine-transformation.html)
