---
layout: post
title: CollectionView 안에 CollectionView 넣기 - (2)
date: '2022-06-27 23:04:00 +0900'
categories: [iOS]
tags: [iOS, CollectionView, dynamic, roundedLabel]

---



(앞서 작성했던 [1편](https://hansolkkim.github.io/posts/CollectionViewInCollectionView-1)에 이어 작성되었습니다.)

## 4. 문제 해결

### 3. IssueCollectionCell의 dynamic height

하지만, Issue들 중에 Label을 가지지 않은 Issue들도 있을텐데, 이런 Issue들의 Cell의 높이는 작아져야 하지 않을까? 하는 생각에서 3번째 문제 해결을 해보고자 합니다!

현재 상황에서, Label을 가지지 않는 Cell은 다른 Cell들과 높이가 같을 뿐더러, Label이 화면 길이를 넘어가는 만큼 쌓일 경우에는 Label Cell들이 표시되지 않는 문제가 있었습니다.



첫 번째 Issue Cell은 5개의 Label Cell은 가진 경우의 모습이고, 네 번째 Issue Cell은 Label Cell을 하나도 가지지 않은 상태입니다.

<img src="../../assets/img/SS%202022-06-27%20PM%2004.51.24.jpg" alt="SS 2022-06-27 PM 04.51.24" width="30%;" />

CollectionView에서는 TableView의 [`UITableViewAutomaticDimension`](https://developer.apple.com/documentation/uikit/uitableview/1614961-automaticdimension)과 같이, cell 안에 들어있는 컨텐츠에 맞춰서 높이를 결정해주는 기능이 없습니다.

따라서 DummyCell을 만들어서 미리 사이즈를 결정해준 뒤에, CollectionView의 sizeForItem에서 호출하는 방식이 이용됩니다.

이를 이용하기 위해서는, UICollectionViewCell의 사이즈의 결정 시점을 알아야 합니다.

>  먼저, CollectionView가 그려지려고 할 때, **UICollectionViewDataSource**의 `collectionView(_:numberOfItemsInSection:)` 메소드가 호출되어 그려질 섹션에 몇 개의 Cell이 있는지 반환을 요청합니다.
>
> 이 때, 반환받은 값이 0이 아니라면, **UICollectionViewDelegateFlowLayout**에 정의되어 있는 `collectionView(_:layout:sizeForItemAt:)` 메소드가 호출되어 반환받은 값으로 CollectionViewCell의 사이즈를 결정합니다.
>
> 그리고 **UICollectionViewDataSource**의 `collectionView(_:cellForItemAt:)` 메소드가 호출되어 위치(indexPath)에 맞는 Cell을 반환받습니다.



<img src="../../assets/img/SS%202022-06-27%20PM%2004.54.37.jpg" alt="SS 2022-06-27 PM 04.54.37" width="60%;" />

1. 먼저, IssueCollectionCell의 Size를 정해주는 `collectionView(_:layout:sizeForItemAt:)` 메소드에서 Dummy Cell을 생성하여, 들어갈 컨텐츠의 최대 높이를 상정해 넉넉하게 Size를 잡아줍니다.

	```swift
	// IssueCollectionViewController.swift
	
	func collectionView(_ collectionView: UICollectionView,
	                    layout collectionViewLayout: UICollectionViewLayout,
	                    sizeForItemAt indexPath: IndexPath) -> CGSize {
	
	    let width = UIScreen.main.bounds.width
	    let estimatedHeight: CGFloat = 200 // <--- 넉넉한 높이를 잡아준다.
	
	    let dummyCell = IssueCollectionCell(frame: CGRect(x: 0, y: 0, width: width, height: estimatedHeight))
	    ...
	}
	```

	

2. 여기서, DummyCell에, 실제로 그 위치의 Cell에 들어갈 데이터를 집어넣어 높이를 구해야 한다. 이를 위해 UICollectionViewDataSource에 있는 실제 Cell에 들어갈 데이터를 받아와야 합니다. DataSource가 나누어져있을 경우 completionHandler를 받는 아래의 메소드를 구현해주도록 합니다. (저는 실습 파일에 작성하여, Delegate, DataSource를 하나의 VC가 모두 담당하게 되어있었습니다.)

	```swift
	// IssueCollectionDataSource.swift
	func referIssue(at indexPath: IndexPath, handler: (Issue) -> Void) {
	    let issue = data[indexPath.item]
	    handler(issue)
	}
	```

	이제 위에서 작성하던 `collectionView(_:layout:sizeForItemAt:)` 메소드에서 `referIssue(at:handler:)` 메소드를 사용하는 부분을 추가합니다.

	```swift
	// IssueCollectionViewController.swift
	
	func collectionView(_ collectionView: UICollectionView,
	                    layout collectionViewLayout: UICollectionViewLayout,
	                    sizeForItemAt indexPath: IndexPath) -> CGSize {
	    ...
	    dataSource.referIssue(at: indexPath) { (issue) in
	        dummyCell.configure(with: issue)
	    }
	    ...
	}
	```

	

3. `referIssue(at:handler:)` 메소드의 handler에는 IssueCollectionCell의 `configure(with:)` 메소드를 호출하게 되는데, IssueCollectionCell은 여기서 전달받은 issue를 이용해 정확한 높이를 결정해야 합니다. 

	이를 위해, IssueCollectionCell(dummyCell)에서는 UI에 필요한 데이터들을 집어넣고, `LabelCollectionView.view.reloadData()` 메소드를 호출한 후, 결정된 LabelCollectionView를 dummyCell 내의 StackView에 넣어 실제 높이를 계산합니다.

	또한, LabelCollectionView가 자기 크기만큼 ContentsStackView 내에 들어가도록 heightAnchor를 지정해줍니다.

	```swift
	// IssueCollectionCell.swift
	
	func configure(with issue: Issue) {
	    titleLabel.text = issue.title
	    data = issue.labels
	    labelCollectionViewController.updateLabels(issue.labels)
	    labelCollectionViewController.reloadCollectionView()
	    contentsStackView.addArrangedSubview(labelCollectionViewController.view)
	
	    layoutIfNeeded()
	    labelCollectionViewController.view.snp.makeConstraints { make in
	        make.height.equalTo(labelCollectionViewController.contentSize.height)
	    }
	}
	```

	여기서 `labelCollectionViewController.contentSize.height`는 아래와 같이 정의된 computed Property입니다.

	```swift
	// LabelCollectionViewController
	
	var contentSize: CGSize {
	    return labelCollectionView.contentSize
	}
	```

	

4. 이제 `collectionView(_:layout:sizeForItemAt:)` 메서드를 완성할 수 있습니다.

	위에서 결정된 값들을 이용해서 EstimatedSize를 결정해줍니다. 이 때, 실제 Issue 데이터를 이용해 실제 길이가 저장된 dummy Cell을 `layoutIfNeeded()` 메서드와 `systemLayoutSizeFitting(_:)` 메서드를 사용해 딱 맞는 **EstimatedSize**를 결정합니다.

	```swift
	// IssueCollectionViewController.swift
	
	func collectionView(_ collectionView: UICollectionView,
	                    layout collectionViewLayout: UICollectionViewLayout,
	                    sizeForItemAt indexPath: IndexPath) -> CGSize {
	    ...
	    dummyCell.layoutIfNeeded()
	
	    let estimatedSize = dummyCell.systemLayoutSizeFitting(CGSize(width: width, height: estimatedHeight))
	
	    return CGSize(width: width, height: estimatedSize.height)
	}
	```

위의 과정을 모두 거쳐 아래와 같은 화면을 **드디어** 볼 수 있었습니다!

<img src="../../assets/img/SS%202022-06-27%20PM%2010.37.13.jpg" alt="SS 2022-06-27 PM 10.37.13" width="40%;" />



### 4. UILabel의 rounded style

다른 프로젝트들을 하면서 궁금했던 점이었는데, 이번 포스팅을 작성하면서 해결한 방법이 있어 문제 해결로 작성합니다!

기존에는, Cell의 Height가 얼마나 될지 예상하여 `UILabel.layer.cornerRadius`를 결정해줬습니다. 

이번에는 LabelCollectionCell의 Height가 EstimatedSize로 인해 결정되기 때문에 UI가 그려질 때 그에 맞는 높이에 의해 `cornerRadius`가 결정되도록 하려 했습니다.

위의 방법을 구현하기 위해 작성한 코드는 아래와 같습니다.

```swift
// LabelCollectionCell.swift

final class LabelCollectionCell: UICollectionViewCell {
    ...

    override func layoutSubviews() {
        super.layoutSubviews()
        labelLabel.snp.makeConstraints { make in
            make.top.leading.equalTo(self)
        }
        labelLabel.layer.cornerRadius = labelLabel.frame.height/2
    }
}

```

 `layoutSubviews()` 메서드는 레이아웃 정보 변경 사항이 뷰에 반영될 때 호출되는데, 이 메서드는 재귀적으로 모든 자식 뷰의 `layoutSubviews()` 메서드를 호출합니다.(따라서 실행될 시에 부하가 매우 큰 메서드임.)

IssueCollectionCell에서 `layoutIfNeeds()` 메소드가 호출되면

`layoutSubviews()` 가 호출되고,

 이로 인해 하위 뷰인 LabelCollectionCell의 `layoutSubviews()` 또한 호출될 것입니다.

또한, 이 시점에는 LabelCollectionCell의 Layout들이 모두 잡혀있는 상황이기 때문에, labelLabel의 높이 또한 잡혀있게 됩니다.

이 시점에 해당 UILabel의 `layer.cornerRadius`를 설정해줌으로써 원하는 시점에서 둥근 모양의 Label을 만들 수 있었습니다.

---

- [코드 내용은 여기에](https://gist.github.com/Hansolkkim/e2d7f286d26d5c33ea4cca1951774914)



## 5. 참고 자료

- [[ColectionViewCell Dynamic Height]](https://corykim0829.github.io/ios/UICollectionViewCell-dynamic-height/#)

- [[StackOverflow-Set rounded corner on UIImage in UICollectionViewCell in swift]](https://stackoverflow.com/questions/39799520/set-rounded-corners-on-uiimage-in-uicollectionviewcell-in-swift)