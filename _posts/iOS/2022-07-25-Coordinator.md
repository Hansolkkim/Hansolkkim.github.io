---
layout: post
title: Coordinator Pattern
date: '2022-07-25 10:43:10 +0900'
categories: [iOS]
tags: [iOS, Design Pattern, Coordinator Pattern]
---

## 계기

토이 프로젝트를 진행하면서, 보다 가벼운 ViewController를 만들 필요성을 느껴  **Coordinator Pattern**을 적용해보고,

이를 정리하기 위해 포스팅을 작성합니다.



## Coordinator Pattern?

Coordinator Pattern은 Soroush Khanlou가 2015년에 <a href="https://khanlou.com/2015/01/the-coordinator/" target="_blank">The Coordinator</a>라는 글을 쓰면서 소개가 됐다고 합니다.

> One of the biggest problems with the big view controllers is that they entangle your flow logic, view logic, and business logic. - Khanlou
>
> *Big(Massive) View Controller의 가장 큰 문제 중 하나는 하나의 flow login, view login, 비즈니스 로직이 얽혀있다는 것이다.*

```swift
func tableView(_ tableView: UITableView, didSelecteRowAt indexPath: IndexPath) {
    let object = self.dataSource[indexPath]
    let detailViewController = DetailViewController(with: object)
    self.navigationController?.present(
        detailViewController,
        animated: true, 
        completion: nil
    )
}
```

위 코드는 TableView에서 Cell이 터치됐을 때 다른 ViewController를 present하는 메소드입니다. 만약 화면이 많은 상황에서 위와 같은 코드가 존재하게 된다면, 해당 뷰는 재사용할 수 없게 되고, View 객체가 flow logic을 갖게 되어 View 객체의 역할을 벗어나게 됩니다.

위 메소드를 가지고 있는 ViewController는 다음에 어떤 ViewController를 호출하고 보여줄 것인지에 대한 flow logic을 알고 있게 되고, 4번째 줄에서는 심지어 부모 ViewController에게 해야하는 일에 대한 <u>정확한</u> 메세지를 보내 지시하고 있습니다.

더 큰 문제는, 이런 다음 단계에 대한 flow logic만을 갖고있는 ViewController가 여러 개 있을 수 있다는 것입니다.



Khanlou는 이런 문제를 해결하기 위해 ViewController를 더 높은 수준의 객체가 관리하게 되면 많은 이점을 가질 수 있음을 알게 됐고, 이 객체를 **Coordinator**라고 부르기로 했습니다.



즉, Coordinator라는 객체로 ViewController를 소유, 관리하여, **ViewController들 간의 결합도를 낮춰**주도록 하는 것입니다.

ViewController들은 이전에 어떤 VC가 있었고, 다음에 어떤 VC가 올 지 알 필요가 없고, 이런 flow logic을 Coordinator 객체가 관리해줍니다. 

이렇게 되면 어떤 순서로도 ViewController 전환이 가능해져 ViewController의 재사용이 가능해집니다.



## Coordinator Pattern 사용해보기

Coordinator Pattern을 활용해 HomeViewController에서 포인트 충전 버튼을 눌렀을 때, 등록된 카드가 있다면 포인트 충전 화면으로, 등록된 카드가 없다면 카드 등록 화면으로 이동하는 앱을 간단하게 만들어보도록 하겠습니다.

- 만들고자 하는 앱의 구조는 아래와 같습니다.

<img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202210261729371.png" alt="image-20221026172952150" style="zoom:50%;" />

#### 1. Coordinator Protocol 만들기

먼저, 화면 전환을 위한 Coordinator 객체의 protocol을 만듭니다.

```swift
protocol Coordinator: AnyObject {

    var childrenCoordinator: [Coordinator] { get }
    var navigationController: UINavigationController { get }

    func start()
}
```

1. 부모 자식 관계를 가지는 Coordinator를 설계하기 떄문에 childrenCoordinator 프로퍼티를 추가합니다.
2. Coodinator는 화면 전환시 기준이 되는 NavigationController를 소유합니다.
3. 부모 Coordinator가 자식 Coordinator를 생성하고 자식 Coordinator의 첫 화면을 띄우기 위해 `start()` 메소드를 가집니다.



#### 2. AppCoordinator 만들기

위에서 만든 `Coordinator` 프로토콜을 채택하는 AppCoordinator를 만듭니다.

```swift
class AppCoordinator: Coordinator {

    var childrenCoordinator: [Coordinator] = []
	let navigationController: UINavigationController

    init(navigationController: UINavigationController) {
        self.navigationController = navigationController
    }

    func start() {
        showHomeViewController()
    }
}

private extension AppCoordinator {
    func showHomeViewController() {
        let homeCoordinator = HomeCoordinator(navigationController: self.navigationController)
        childrenCoordinator.append(coordinator)
        coordinator.start()
    }
}
```



### 3. HomeCoordinator 만들기

AppCoordinator는 앱의 전체적인 Coordination을 관장(*)하는 Coordinator라고 가정하고,

HomeCoordinator 또한 만들어 줍니다.

<sub>(*: 여기서는 생략됐지만 로그인 기능이 있다면 LoginCoordinator/ HomeCoordinator로 분기하는 등의 기능을 하는 Coordinator로 사용한다는 뜻입니다.)</sub>

```swift
class HomeCoordinator: Coordinator {
    // 다른 구현 내용은 생략합니다.
    func start() {
        let homeViewController = HomeViewController(navigation: self)
        navigationController.pushViewController(viewController, animated: true)
    }
}
```



### 4. HomeViewController 만들기

이제 Top-up Button이 있는 **HomeViewController**를 만들어줄 차례입니다.

HomeViewController는 Top-up Button이 눌렸음을 HomeCoordinator에게 알리고,

HomeViewController는 해당 노티를 이용해 다음에 어떤 ViewController를 보여줄지 결정합니다.

```swift
protocol HomeNavigation: AnyObject {
    func topupButtonDidTap()
}

class HomeViewController: UIViewController {
    
    private lazy var topupButton: UIButton = {
        ...
        let button = UIButton(...)
        button.addTarget(self, action: #selector(topupButtonDidTap), for: .touchUpInside)
        return button
    }()

    private weak var coordinator: HomeNavigation?

    ...
	
    @objc
    private func topupButtonDidTap() {
        coordinator?.topupButtonDidTap()
    }
}
```

여기서 **HomeNavigation**이라는 프로토콜을 사용했습니다.

HomeViewController가 자신의 Coordinator에게 특정 노티를 보내주기 위해 Coordinator를 알고 있어야 하지만

Coordinator의 모든 부분을 알 필요는 없고, 노티를 위한 메소드 정도만 알고 있도록 하기 위해 해당 프로토콜을 사용했습니다.



이제 HomeNavigation 프로토콜을 HomeCoordinator가 채택하도록 하여 화면 전환 플로우를 완성합니다.

```swift
class HomeCoordinator: Coordinator {

    private var paymentMethodsList: [PaymentMethod] = []
    // 다른 구현 내용은 생략합니다.
}

extension HomeCoordinator: HomeNavigation {
    func topupButtonDidTap() {
        let isPaymentMethodsListEmpty = paymentMethodsList.isEmpty

        if isPaymentMethodsListEmpty {
            showAddPaymentMethodViewController()
        } else {
            showTopupViewController()
        }
    }
}

private extension HomeCoordinator {
    func showAddPaymentMethodViewController() {
    }

    func showTopupViewController() {
    }
}
```



그리고 앱이 시작될 때에 AppCoordinator를 실행시켜주고, AppCoordinator에 대한 참조를 유지하기 위해

SceneDelegate를 아래와 같이 설정합니다.

```swift
class SceneDelegate: UIResponder, UIWindowSceneDelegate {

    var window: UIWindow?
    private var appCoordinator: Coordinator?

    func scene(_ scene: UIScene, willConnectTo session: UISceneSession, options connectionOptions: UIScene.ConnectionOptions) {
        guard let windowScene = (scene as? UIWindowScene) else { return }

        let window = UIWindow(windowScene: windowScene)
        self.window = window

        let navigationController = UINavigationController()
        self.window?.rootViewController = navigationController

        self.appCoordinator = AppCoordinator(navigationController: navigationController)
        appCoordinator?.start()

        self.window?.makeKeyAndVisible()
    }
}
```



### 5. 동작 화면

|                   등록된 카드가 존재할 때                    |                등록된 카드가 존재하지 않을 때                |
| :----------------------------------------------------------: | :----------------------------------------------------------: |
| ![SS2022-10-26PM04.23.08](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202210261624159.gif) | ![SS2022-10-26PM04.24.38](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202210261625922.gif) |

<a href="https://gist.github.com/Hansolkkim/4391e39bd72e7edb696f0f2cbf9b42ef" target="_blank">[전체 코드 확인]</a>



## 더 생각해봐야할 점?

토이 프로젝트에 Coordinator 패턴을 적용하다보니 생각해볼 부분이 있었습니다.

가장 큰 부분은 **<u>Coordinator를 나누는 기준을 무엇으로 할 것인가?</u>** 에 대한 의문이었습니다.

같이 공부하는 지인들과 토론해본 결과, <u>"뒤로 가기"가 필요하고, "뒤로 가기"의미가 있는 범위의 ViewController 들끼리 하나의 Coordinator에 소속되어 있는게 적절하지 않을까</u> 하는 결론을 내렸습니다.

일례로, 구현하진 않았지만 앞서 말했던 내용처럼, 만약 **로그인 뷰**가 있고 **홈 뷰**가 있다면 홈 뷰에서 로그인 뷰로 "뒤로 가기"가 필요하지 않고 의미가 있지 않은 범위의 관계이기 때문에 같은 Coordinator에 속할 필요가 없는 뷰들이라고 생각되었습니다.





### 참고자료

1. [[Khanlau - The Coordinator]](https://khanlou.com/2015/01/the-coordinator/)

2. [[iOS: Coordinator Pattern in Swift]](https://saad-eloulladi.medium.com/ios-coordinator-pattern-in-swift-39a15aa3b01b)

