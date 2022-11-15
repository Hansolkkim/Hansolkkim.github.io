---
layout: post
title: UICollectionView Guide
date: '2022-08-25 17:43:10 +0900'
categories: [iOS]
tags: [iOS, UICollectionView]
---



# UICollectionView

(\* [CollectionView Programming Guide for iOS](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40012334-CH1-SW1)의 내용을 정리 및 개인적으로 추가 내용입니다.)

### UICollectionView?

-   컬렉션 뷰는 커스텀이 가능한(flexible and changeable) 레이아웃을 사용해 정렬된 data item들을 보여주는 객체입니다.

-   컬렉션 뷰를 사용하면 시각적 요소의 정확한 레이아웃을 정의할 수 있고, 동적으로 변경할 수도 있습니다. 
    
    따라서 Grid, Stack, Circular Layout, Dynamically changing layout 등 생각할 수 있는 모든 형태의 정렬을 구현할 수 있습니다.
    
-   컬렉션 뷰는 <u>표시되는 데이터</u>와 <u>해당 데이터를 표시하는데 사용되는 데이터</u>를 엄격하게 구분합니다.

    여기서는 <sub>1</sub> <u>UICollectionViewCell 객체와 그에 필요한 데이터(DataSource, Delegate)</u> / <sub>2</sub> <u>Layout을 위한 객체</u>를 구분한다는 말인 것 같습니다.
    컬렉션 뷰는 이 두 데이터를 병합하여 최종 모양을 보여줍니다.

    >   You provides the data, the layout object provides the placement information, and the collection view merges the two pieces together to archieve the final appearance.

-   **A CollectionView Manages the Visual  Presentation of Data-Driven Views**

    컬렉션 뷰의 유일한 관심사는 뷰를 가져와 특정 방식으로 배치하는 것입니다. 즉, 컬렉션 뷰는 컨텐츠와 관련된 객체가 아니라, 뷰의 Presentation(표시) 및 Arrangement(배치)에 관련된 객체입니다.

    컬렉션 뷰를 잘 이용하기 위해서는 CollectionView, Data Source, Layout Object, Custom Object의 관계, 상호 작용을 이해하는 것이 중요합니다.

-   **The Flow Layout Supports Grids and Other Line-Oriented Presentations**

    일반적으로 Flow Layout 객체를 이용해 행, 열을 가지는 Grid를 구현하지만 이를 커스텀한다면 모든 유형의 Linear-Flow를 지원할 수 있습니다.

-   **Gesture Recognizers Can Be Used for Cell and Layout Manipulations**

    다른 여타 뷰들과 마찬가지로 Gesture Recognizer를 컬렉션 뷰에 연결하여 해당 뷰의 컨텐츠를 조작할 수 있습니다.



### CollectionView Basics

-   컨텐츠를 화면에 표시하기 위해 컬렉션 뷰는 다양한 객체들과 협력(cooperate)합니다.
    일부 객체는 앱에서 반드시 제공해주어야 합니다. 예를 들어, 컬렉션 뷰에 표시할 항목 수를 알려주는 데이터 원본 객체를 앱이 제공해주어야 합니다.
    반드시 제공하지 않아도 될 객체는 UIKit에서 제공하며, 기본적인 컬렉션 뷰 디자인과 관련된 객체입니다.

-   **A Collection View Is a Collaboration of Objects**

    컬렉션 뷰의 디자인은 데이터가 화면에 배열되고 표시되는 방식과 표시되는 데이터를 분리합니다.

    아래의 표는 컬렉션 뷰를 구현하기 위해 필요한 클래스, 프로토콜들입니다.

    ![SS2022-11-10PM03.44.54](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211101545160.jpg)

    <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211151527782.jpg" alt="SS2022-11-10PM03.49.38" width="50%;" />

    위의 그림은 컬렉션 뷰와 연결된 핵심 객체들 간의 관계를 보여줍니다.

    컬렉션 뷰는 DataSource에서 표시할 셀에 대한 데이터를 가져옵니다.
    **DataSource와 Delegate 객체**는 Cell Selection, Highlighting 등을 포함하는 컨텐츠를 관리하는 역할을 합니다. 앞서 두 번이나 강조한, <u>표시되는 데이터를 관리</u>하는 객체입니다.

    **Layout 객체**는 Cell들이 속하는 위치를 결정하고 그 정보를 Layout Attribute 객체 형태로 컬렉션 뷰에 알려줍니다.

    그런 다음 컬렉션 뷰는 레이아웃 정보를 실제 셀과 병합하여 최종 Visual Presentation을 결정합니다.

    >   해당 과정을 확인하기 위해 커스텀 컬렉션뷰를 작성해 어떤 순서로 컬렉션 뷰가 그려지는지 확인해봤습니다.
    >
    >   ![SS2022-11-10PM04.59.20](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211101659004.jpg)
    >
    >   1.   가장 먼저 Layout 객체의 `prepare()` 메소드가 호출됩니다.
    >        해당 메소드는 Layout 객체에게 현재 레이아웃을 업데이트하도록 하는 메소드입니다. (레이아웃 업데이트는 컬렉션 뷰가 컨텐츠를 처음 표시할 때와 뷰의 변경으로 인해 레이아웃이 무효화될 때 발생합니다.)
    >        레이아웃을 업데이트할 때마다 컬렉션 뷰는 `prepare()` 를 호출해 Layout 객체에 이후 레이아웃 작업을 준비하도록 합니다.
    >        이 메서드의 기본 구현은 아무 작업이 없고, 커스텀 레이아웃을 위해 overriding하여 데이터 구조를 설정하거나 레이아웃을 수행하는데 필요한 초기 계산을 수행하도록 하는 역할을 합니다. (컬렉션 뷰의 사이즈, item의 위치를 결정하기 위한 초기 계산 등을 수행한다고 합니다.)
    >
    >        [[UICollectionView를 이용한 LINE iOS 대화방 리팩토링]](https://engineering.linecorp.com/ko/blog/ios-refactoring-uicollectionview-1/) 포스팅에서 확인해보면, 아래와 같이 prepare 함수를 활용했음을 확인할 수 있었습니다.
    >
    >        >   UICollectionViewLayout의 prepare() 함수에서는 UICollectionView에 표시되는 전체 크기를 계산하고 각 셀의 레이아웃 속성을 미리 계산하여 메모리에 적재한 뒤 유지합니다.
    >        >   레이아웃의 데이터 구조를 설계할 때는, UICollectionView에서 특정 화면 영역이나 특정 IndexPath에 대한 레이아웃 속성을 요청할 때 빠르게 응답할 수 있도록 설계하는 것이 중요합니다.
    >        >   내부적으로 Dictionary나 Array 등을 활용하여 최대한 빠르게 응답할 수 있도록 설계했습니다.
    >
    >   2.   그 다음 Data Source 객체의 `numberOfItemsInSection` 관련 메소드가 호출됩니다.
    >        섹션마다 몇 개의 item이 필요한지 확인하기 위한 메소드인데, 이는 Layout의 ContentSize를 결정하여 `collectionViewContentSize` 프로퍼티를 결정하기 위해서 호출됩니다. (ContentSize는 ScrollView에서 스크롤하기 위한 contents의 전체 크기를 위해 필요한 값입니다.)
    >        collectionViewContentSize를 알기 위해 item 개수만이 필요한 것은 아니지만, 현재 collectionView를 정의하는 코드에서 itemSize를 결정해주었기 때문에 그 이후의 작업이 진행되었습니다.
    >        itemSize를 결정하는 방법에는 `Layout.itemSize = ...`과 같이 Layout 객체에 직접 정해주는 방법도 있지만 `UICollectionViewDelegateFlowLayout` 프로토콜의 `sizeForItem` 관련 메소드를 통해 설정해주는 방법도 있습니다. 
    >        하지만 이 방법은 Cell마다 Size가 다를 경우에 필요한 방법입니다. ContentSize를 구하는 과정에서 모든 Cell마다 `sizeForItem` 메소드를 호출하기 때문에 비효율적이기 때문입니다.
    >        따라서 Cell이 모두 같은 Size라면 앞서 말한 첫 번째 방법을 사용하는 것이 효율적이라고 합니다.
    >
    >   3.   그 다음 앞서 말했던 Layout 객체의 ContentSize 결정을 위한 `collectionViewContentSize` 프로퍼티의 결정이 이루어집니다.
    >
    >   4.   그 이후에 Layout 객체에서 `layoutAttributesForElements` 메소드를 이용해 각각의 Element 마다 필요한 Attribute를 생성해줍니다.
    >
    >   5.   그리고 Data Source 객체에서 Cell에 표시될 데이터 결정을 위해 `cellForItemAt` 메소드를 호출합니다.

-   **Reusable Views Improve Performance**

    컬렉션 뷰는 **효율성**를 개선하기 위해 뷰의 **Recycling Program**을 사용합니다.
    뷰가 화면에서 벗어나면 뷰에서 제거되고 (삭제되지는 않고) 재사용 큐에 배치됩니다.
    새 컨텐츠가 스크롤되어 화면에 나타나면 재사용 큐에 있던 뷰가 큐에서 빠져나와 새 컨텐츠가 되어 그 용도가 변경됩니다.

    이런 **Recycling**을 위해 컬렉션 뷰에 의해 표시되는 모든 뷰들은 `UICollectionReusableView`클래스의 자식 클래스여야 합니다.

    컬렉션 뷰는 Cell, Supplementary View, Decoration View 세 종류의 ReusableView를 제공합니다. (각 용도는 [해당 사이트](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/CollectionViewBasics/CollectionViewBasics.html#//apple_ref/doc/uid/TP40012334-CH2-SW1)에서 참고하면 될 것 같습니다!)

    컬렉션 뷰는 테이블 뷰와는 다르게 Data Source에서 제공하는 Cell, Supplementary Views의 특정 스타일을 사용하지 않습니다.
    대신 기본 Reusable View Class는 수정할 수 있는 빈 캔버스와 같습니다. 이를 사용하여 뷰 계층 구조를 만들든, 이미지를 표시하든, 컨텐츠를 동적으로 그릴 수 있습니다.

    DataSource 객체는 연결된 컬렉션 뷰에서 사용하는 Cell, Supplementary View를 제공하는 역할을 하지만 뷰를 직접 생성하지는 않고, 뷰를 요청하면, 데이터 소스는 컬렉션 뷰의 메서드를 사용하여 원하는 유형의 뷰를 큐에서 빼와서 구성하도록 합니다.

-   **The Layout Object Controls the Visual Presentation**

    Layout 객체는 컬렉션 뷰 내에서 item의 배치 및 시각적 스타일을 결정하는 단독 책임을 갖습니다.
    DataSource 객체가 뷰와 실제 컨텐츠를 제공하지만, Layout 객체는 해당 뷰의 크기, 위치 및 기타 모양 관련 속성을 결정합니다.
    이런 책임 분리 덕분에 데이터 객체를 변경하지 않고도 레이아웃을 동적으로 변경할 수 있습니다.

    컬렉션 뷰에서 사용하는 Layout 객체는 실제로 해당 뷰를 소유하지 않기 때문에 관리하는 뷰를 직접적으로(directly) 건드리지 않습니다.
    대신 컬렉션 뷰에서 Cell, Supplementary View, Decoration View의 위치, 크기 및 시각적 모양을 describe하는 attributes를 생성합니다.
    그런 다음 attributes를 실제 뷰 객체에 적용하는 것은 컬렉션 뷰의 책임입니다.
    <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211151527538.jpg" alt="SS2022-11-10PM04.21.36" style="zoom:50%;" />

    위의 그림은 수직 방향 스크롤의 컬렉션 뷰에서 Flow Layout 객체가 Cell, Supplementary View를 정렬하는 방법을 보여줍니다.
    수직 방향 스크롤 Flow Layout에서 컨텐츠 영역의 Width는 고정된 상태로 유지되고, Height는 컨텐츠를 수용할 수 있도록 커집니다.
    면적을 계산하기 위해 Layout 객체는 뷰와 Cell을 한 번에 하나씩 배치하고 각각에 가장 적합한 위치를 선택합니다. (참고로 Flow Layout에서는 Cell, Supplementary View의 크기는 Layout 객체나 Delegate를 통해 속성으로 지정됩니다.)

    Layout 객체는 뷰의 크기나 위치 그 이상을 제어할 수 있습니다. Transparency, 3D 공간에서의 변형, 다른 뷰들과 겹쳐 보이도록 하는 등의 여러 뷰 관련 속성을 지정할 수 있습니다



### Designing Your Data Source and Delegate

-   모든 컬렉션 뷰는 **Data Source** 객체를 가지고 있습니다.
    Data Source 객체는 앱이 보여주려고 하는 컨텐츠 그 자체입니다. 
    Data Source는 앱 데이터 모델로부터 만들어진 객체일 수도, 혹은 컬렉션 뷰 컨트롤러일 수도 있습니다. Data Source에 대한 유일한 Requirement는, 그것이 컬렉션 뷰에 필요한 정보를 제공할 수 있어야 한다는 것입니다.
    (여기서 정보란, 얼마나 많은 item이 있는지 또는 그 item들을 보여줄 때 어떤 뷰들을 사용하는지에 대한 정보 등을 의미합니다.)
-   **Delegate** 객체는 필수는 아니지만 존재하는 것이 권장되는 객체입니다.
    Delegate 객체는 컬렉션 뷰를 통해 컨텐츠를 보여주는 것과 상호 작용과 관련된 측면을 관리하는 객체입니다.

#### The Data Source Manages Your Content

-   Data Source 객체는 컬렉션 뷰를 사용해 표시하는 컨텐츠를 관리하는 객체입니다.
    Data Source  객체는 지원해야 하는 기본 behavior 및 methods를 정의하는  `UICollectionViewDataSource` 프로토콜을 준수해야 합니다.
    Data Source의 역할은 다음 질문에 대한 답변을 컬렉션 뷰에 제공하는 것입니다.

    1.   몇 개의 Section을 컬렉션 뷰가 포함하나?
    2.   주어진 Section에 대해서 Section은 몇 개의 item을 포함하나?
    3.   주어진 Section이나 Item에 대해서 대응하는 컨텐츠를 보여주기 위해 어떤 뷰를 사용하나?

-   Section과 Item은 컬렉션 뷰 컨텐츠의 기본 구성 원칙입니다.
    컬렉션 뷰는 일반적으로 하나 이상의 Section이 있으며, 각 Section에는 차례로 0개 이상의 Item이 포함됩니다.
    Item은 표시하려는 주요 컨텐츠를 보여주는 반면, Section은 해당 Item들을 논리적 그룹으로 구성합니다.

-   컬렉션 뷰는 NSIndexPath 객체를 사용해 포함된 데이터를 참조합니다.
    Item을 찾으려 할 때 컬렉션 뷰는 Layout 객체에서 제공한 Index Path 정보를 사용합니다.
    Item은 (Section Number, Item Number) 형태의 Index Path를 가지고, Supplementary View나 Decoration View는 (Section Number, Layout 객체가 제공한 어떠한 값) 형태의 Index Path를 가집니다.

-   Data Source 객체에서 Section과 Item을 어떻게 정렬하든지에 관계없이, 해당 Section과 Item의 시각적 Presentation은 Layout 객체에 의해 결정됩니다.
    ![SS2022-11-11PM02.49.46](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211151527087.jpg)
    위의 그림에서 처럼, 서로 다른 Layout 객체는 Section과 Item을 매우 다른 방식으로 보여줄 수 있습니다.
    Flow Layout 객체는 이전 Section 아래에 다음 Section이 오는 형태로, 세로로 정렬을 하는 반면, Custom Layout 객체는 Section을 비선형 배열로 배치하는 것을 보아, Data와 Layout의 역할이 분리되어 있음을 다시 한 번 확인할 수 있습니다.

-   **Designing Your Data Objects**

    효율적인 Data Source는 기본 데이터 객체를 구성하는데 이점을 갖기 위해 Section과 Item을 사용합니다. 
    Data를 Section과 Item으로 구성하면 나중에 Data Source methods를 손쉽게 구현할 수 있습니다.
    또한 Data Source의 Method들은 자주 호출되기 때문에 이런 method들이 데이터를 가능한 한 빨리 추출할 수 있도록 해야합니다.

    한 가지 간단한 솔루션은 아래 그림과 같이 data Source가 nested array 집합을 사용하는 것입니다.
    이 구성에서 상위 배열은 Data Source의 Seciton을 나타내는 하나 이상의 배열이고, 그러면 각 Section 배열에는 해당 Section에 필요한 데이터 Item이 포함됩니다.
    Section에서 Item을 찾기 위해서는, Section 배열을 검색한 다음 해당 배열에서 Item을 검색하면 됩니다.

    ![SS2022-11-11PM03.08.27](https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211151527458.jpg)

-   **Telling the Collection View About Your Content**

    컬렉션 뷰는 Data Source 객체에게 다음과 같은 상황에서 데이터를 요구합니다.

    1.   컬렉션 뷰가 처음  화면에 나타날 때
    2.   컬렉션 뷰에 다른 Data Source 객체를 할당할 때
    3.   컬렉션 뷰의 `reloadData()` 메소드를 직접 호출할 때
    4.   Delegate 객체가 `performBatchUpdate:completion` 관련 메소드를 실행할 때나 이동, 추가, 삭제 메소드를 실행할 때

    위와 같은 상황에서 `numberOfSectionInCollectionView:` 관련 메소드를 사용해 Section의 개수를 제공하고, `collectionView:numberOfItemInSection:` 관련 메소드를 사용하여 각 Section에 있는 Item의 개수를 제공합니다.



#### Inserting, Deleting, and Moving Sections and Items

하나의 Section이나 Item을 삽입, 삭제 또는 이동하기 위해서는 다음의 단계를 따라야 합니다.

1.   Data  Source 객체의 데이터를 업데이트
2.   컬렉션 뷰에서의 적절한 메소드 호출

**컬렉션 뷰에 변경 사항을 알리기 전에 Data Source를 업데이트**하는 것이 중요합니다. 
컬렉션 뷰의 Method는 Data Source에 현재 적절한 데이터가 있다고 가정하기 때문입니다.
그렇지 않으면 컬렉션 뷰가 Data Source에서 잘못된 Item 집합을 받거나, Data Source에 존재하지 않는 항목을 요청하여 앱이 충돌될 수 있습니다.

하나의 Item을 Programmitacally하게 추가, 삭제, 이동할 경우, 컬렉션 뷰는 자동으로 해당 변경을 반영하는 애니메이션을 만듭니다.
하지만 여러 변경 사항을 함께 애니메이션하려면 블록 내에서 모든 추가, 삭제, 이동 관련 메소드를 호출하고 해당 블록을 `performBatchUpdates:completion:` 관련 메소드에 전달해야 합니다.
이렇게 하면 batch update process가 모든 변경 사항을 동시에 애니메이션으로 만들어 줍니다.

>   위의 내용을 코드를 짜서 구현해보았습니다.
>
>   먼저, 제일 처음 말했던 “컬렉션 뷰에 변경 사항을 알리기 전에 Data Source를 업데이트”하는 부분을 확인해보기 위해 아래의 두 코드로 비교해보았습니다.
>
>   ```swift
>   @IBAction func tapInsertButton(_ sender: UIButton) {
>       let newData = data.randomElement() ?? UIColor.black
>       data.append(newData)
>       let insertingIndexPath = IndexPath(item: data.count - 1, section: 0)
>       collectionView.insertItems(at: [insertingIndexPath])
>   }
>   ```
>
>   ```swift
>   @IBAction func tapInsertButton(_ sender: UIButton) {
>       let insertingIndexPath = IndexPath(item: data.count, section: 0) // 오류 발생
>       // let insertingIndexPath = IndexPath(item: data.count - 1, section: 0)
>       collectionView.insertItems(at: [insertingIndexPath])
>       let newData = data.randomElement() ?? UIColor.black
>       data.append(newData)
>   }
>   ```
>
>   맨 위의 코드로 실행할 경우, 새로운 Cell이 추가되며 애니메이션까지 잘 나타남을 확인했습니다. 
>
>   하지만 두 번째 코드로 실행할 경우, IndexPath의 item 매개변수의 값이 `data.count` 일 경우, Data Source에 `data.count` 번째 데이터가 없음을 컬렉션 뷰가 알아채어 오류가 발생했고, `data.count - 1`일 경우 실제 추가된 indexPath와 입력받은 indexPath가 일치하지 않다는 경고가 발생했습니다(이 경우 애니메이션도 활성화되지 않았습니다.).
>
>   따라서 원하는 애니메이션을 하고 정상적으로 추가/제거 되도록 하는 로직을 만들기 위해서는 Data Source를 먼저 업데이트하고 그 변경 사항을 컬렉션 뷰에 알려야 함을 확인할 수 있었습니다.
>
>   <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211151527881.gif" alt="SS2022-11-15PM03.19.34" width="20%;" />
>
>   
>
>   그 다음, 여러 변경 사항을 `performBatchUpdate:completion:` 관련 메소드에 전달하는 방법을 이용하기 위해 아래와 같은 코드를 작성했습니다.
>
>   ```swift
>   @IBAction func tapUpdateButton(_ sender: UIButton) {
>       let newData = [data.randomElement()!, data.randomElement()!, data.randomElement()!]
>       collectionView.performBatchUpdates {
>           self.data.append(contentsOf: newData)
>   		let insertingIndexPaths = [
>               IndexPath(item: data.count, section: 0),
>               IndexPath(item: data.count + 1, section: 0),
>               IndexPath(item: data.count + 2, section: 0)
>           ]
>           self.collectionView.insertItems(at: insertingIndexPaths)
>           
>           self.data.removeFirst()
>           self.data.removeFirst()
>           let deletingIndexPaths = [
>               IndexPath(item: 0, section: 0),
>               IndexPath(item: 1, section: 0)
>           ]
>           self.collectionView.deleteItems(at: deletingIndexPaths)
>       }
>   }
>   ```
>
>   위처럼, 컬렉션 뷰의 맨 뒤에 3개의 새로운 Cell을 추가하고, 맨 앞의 2개의 Cell을 삭제하는 코드를 작성했는데 오류가 발생했습니다.
>
>   추가된 새로운 Cell의 IndexPath를 컬렉션 뷰가 확인하기 전에 (그 뒤에서 이루어질 거라 예상했던) Data Source에서 Cell 데이터의 삭제가 발생했고, 그로 인해 존재하지 않는 IndexPath의 Cell을 추가하려고 한다는, 최종적으로 변경된 IndexPath 정보를 컬렉션 뷰에 알려야 한다는 오류였고, 이를 해결하기 위해 아래처럼 코드를 수정했습니다.
>
>   ```swift
>   @IBAction func tapUpdateButton(_ sender: UIButton) {
>       let newData = [data.randomElement()!, data.randomElement()!, data.randomElement()!]
>       collectionView.performBatchUpdates {
>           self.data.append(contentsOf: newData)
>   		let insertingIndexPaths = [
>               IndexPath(item: data.count-2, section: 0),
>               IndexPath(item: data.count-1, section: 0),
>               IndexPath(item: data.count, section: 0)
>           ]
>           self.collectionView.insertItems(at: insertingIndexPaths)
>           
>           self.data.removeFirst()
>           self.data.removeFirst()
>           let deletingIndexPaths = [
>               IndexPath(item: 0, section: 0),
>               IndexPath(item: 1, section: 0)
>           ]
>           self.collectionView.deleteItems(at: deletingIndexPaths)
>       }
>   }
>   ```
>
>   따라서, 여러 변경 사항을 한꺼번에 적용하기 위해서는 컬렉션 뷰에 변경 사항 적용 후의 최종적인 Data Source에 대한 IndexPath 정보를 알려야함을 확인할 수 있었습니다.
>
>   <img src="https://raw.githubusercontent.com/Hansolkkim/Image-Upload/forUpload/img/202211151527183.gif" alt="SS2022-11-15PM03.25.04" width="20%;" />





### 참고 자료

[[Collection View Programming Guide for iOS]](https://developer.apple.com/library/archive/documentation/WindowsViews/Conceptual/CollectionViewPGforIOS/Introduction/Introduction.html#//apple_ref/doc/uid/TP40012334-CH1-SW1)

[[Apple Develper- UICollectionViewLayout]](https://developer.apple.com/documentation/uikit/uicollectionviewlayout)

[[UICollectionView를 이용한 LINE iOS 대화방 리팩토링 - 1]](https://engineering.linecorp.com/ko/blog/ios-refactoring-uicollectionview-1/)