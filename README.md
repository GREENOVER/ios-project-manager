# iOS Project Manager Application Project
### 업무를 관리해주는 트렐로의 기능을 구현한 프로젝트 매니저 앱 프로젝트 (클라이언트 파트)
[Ground Rule](https://github.com/GREENOVER/ios-project-manager/blob/main/GroundRule.md)
***
## **✍️ 키워드**

- Localization
- JSON
- Decode
- FileManager
- Notification
- Popover
- NWPathMonitor
- Drag & Drop
- UserDefaults
- NotificationCenter
- Undo & Redo
- DateFormatter

## **🧑🏻‍💻 구현사항**

- 지역화를 구현해 한국 / 프랑스 / 영어로 언어가 나타나도록 구현하였다.
- Todo/Doing/Done의 상태를 이용하여 각 해당 테이블 셀에 맞게 구현하였다.
- 전체적인 뷰를 컬렉션 뷰 안에 테이블 뷰 3개가 들어가도록 구현하였다.
- 파일 매니저를 학습하고 구현하여 로컬에서 해당 앱의 JSON 데이터 정보를 저장하도록 구현하였다.
- 노티피케이션을 학습하고 구현하여 각 시간에 맞게 푸시 알림이 가도록 구현하였다.
- 각 모드 및 상태를 Enum 타입으로 정의하고 폴더화 시켰다.
- 팝오버를 학습하고 이력을 팝오버 할 수 있도록 구현하였다.
- 로컬에서 네트워크 연결 상태에 대해 학습하고 각 상태에 대한 대응을 구현하였다.
    - 네트워크 상태에 따라 레이블 변화를 디스패치큐로 메인 스레드에서 바로 보이도록 비동기 처리를 하였다.
    - 네트워크 상태 감지는 NWPathMonitor 메서드를 가지고 백그라운드에서 모니터링 될 수 있도록 구현하였다.
- 뷰를 코드 및 스토리보드로 구현하고 xib 파일 구현을 통해 셀을 구현하였다.
- 영역 별 드래그 앤 드롭 기능을 학습하고 구현하였다.
- 노티피케이션 센터를 이용해 메서드 상태 변화를 관찰하고 기능 동작하도록 구현하였다.
- Undo & Redo를 통해 되돌리기 및 되돌리기 취소 기능을 구현하였다.
- 날짜 형식을 알맞게 포맷팅하여 변환하였다.

## 🌲 **UI 구성**

- UICollectionViewCell 내에 UITableView 배치 방식으로 UI구성
    <img width="697" alt="스크린샷 2021-05-17 오후 1 13 44" src="https://user-images.githubusercontent.com/72292617/118431627-bf7e8900-b711-11eb-9463-5845a4e37b4e.png">


- 새로운 메모 등록 시 SheetView 배치

    <img width="695" alt="스크린샷 2021-05-17 오후 1 14 13" src="https://user-images.githubusercontent.com/72292617/118431653-cdcca500-b711-11eb-82aa-4486f110443e.png">


- 히스토리 내역에 대해 팝오버 뷰 배치

    <img width="692" alt="스크린샷 2021-05-17 오후 1 14 19" src="https://user-images.githubusercontent.com/72292617/118431656-d02eff00-b711-11eb-8354-aae20119ac9f.png">


## **👨‍🔧 트러블 슈팅**

### "네트워크 상태 파악하기"

- 문제점
    - 네트워크 상태 연결을 앱 실행되고있는 중간에 끊으면 네트워크 상태 탐지를 못하는 문제점
- 원인
    - 네트워크가 단절되고 와이파이로 연결되든 안되든 변경시점이랑 차이가 나서 탐지를 정확하게 못하는것같다.
- 해결방안
    - 우선 네트워크를 백그라운드에서 모니터링할 수 있도록 구현하여 네트워크 비정상적일 시 화면에 레이블이 나타날 수 있도록 구현하였다.

    ```swift
    extension ProjectManagerViewController {
      func configureNetworkMonitor() {
          let monitor = NWPathMonitor()
          
          monitor.pathUpdateHandler = { path in
              
              if path.status != .satisfied {
                  print("네트워크에 연결되어 있지 않습니다.")
                  DispatchQueue.main.async {
                      self.networkLabel.isHidden = false
                  }
              } else {
                  print("네트워크에 연결되어 있습니다.")
                  DispatchQueue.main.async {
                      self.networkLabel.isHidden = true
                  }
              }
          }
          monitor.start(queue: DispatchQueue.global(qos: .background))
      }
    }
    ```

    ![https://user-images.githubusercontent.com/72292617/116358762-f3fcd480-a838-11eb-9ef6-644606011268.png](https://user-images.githubusercontent.com/72292617/116358762-f3fcd480-a838-11eb-9ef6-644606011268.png)

    여기서 드는 생각이 텍스트를 찍어주는것은 뷰가 업데이트되는 시점일텐데 그렇다면 해당 isHidden을 해주는것을 메인스레드로 보내 동작하게하면 실시간으로 변화가 나타날것 같다.

### "지역화"

- 문제점
    - 지역화가 부분적으로만 이루어진 문제가 발생
- 원인
    - 코드에서 구현한 텍스트가 있고 스토리보드에서 구현한 텍스트가 있다. 지역화를 진행할때는 두 부분을 각 다른 파일에서 지역화를 시켜주도록 해야하는데, 코드 지역화 파일에만 지역화 내용을 추가해줘서 나타나는 문제

    ```
    "Project Manager" = "Chef de projet";
    "Cancel" = "Annuler";
    "Edit" = "mise en forme";

    "TODO"  = "Plan";
    "DOING" = "En cours";
    "DONE"  =  "Terminé";

    "Added '%@'." = "'%@'Ajouter.";
    "Moved '%@' from %@ to %@." = "'%@'un %@dans %@to Nous avons bougé.";
    "Removed '%@' from %@." = "'%@'un supprimer.";

    ```

    ![https://user-images.githubusercontent.com/72292617/116360760-32938e80-a83b-11eb-830b-17f23baaf126.png](https://user-images.githubusercontent.com/72292617/116360760-32938e80-a83b-11eb-830b-17f23baaf126.png)

- 해결방안
    - 코드의 변수로 선언된 부분은 위와 같이 Localizable.strings 파일을 이용해 지역화를 시켜주고 스토리보드에서 얹은 요소에 대한 지역화는 아래와 같이 Main.string 스토리보드 지역화 파일에 별도 작성을 해주어야한다.

    ![https://user-images.githubusercontent.com/72292617/116360915-61116980-a83b-11eb-8854-709c865598c2.png](https://user-images.githubusercontent.com/72292617/116360915-61116980-a83b-11eb-8854-709c865598c2.png)

    ```
    /* Class = "UIBarButtonItem"; title = "Done"; ObjectID = "1Iz-ek-qSt"; */
    "1Iz-ek-qSt.title" = "Terminé";

    /* Class = "UINavigationItem"; title = "TODO"; ObjectID = "8tB-ho-ekQ"; */
    "8tB-ho-ekQ.title" = "Plan";

    /* Class = "UINavigationItem"; title = "Title"; ObjectID = "BsE-ch-KEa"; */
    "BsE-ch-KEa.title" = "Titre";

    /* Class = "UITextField"; placeholder = "Title"; ObjectID = "Ne9-CO-XbT"; */
    "Ne9-CO-XbT.placeholder" = "Titre";

    /* Class = "UILabel"; text = "Label"; ObjectID = "VLk-en-wR1"; */
    "VLk-en-wR1.text" = "étiquette";

    /* Class = "UILabel"; text = "Title"; ObjectID = "eqg-sO-J0R"; */
    "eqg-sO-J0R.text" = "Titre";

    /* Class = "UIBarButtonItem"; title = "Cancel"; ObjectID = "keU-l8-vF1"; */
    "keU-l8-vF1.title" = "annuler";
    ```

    해당 스토리보드 상 객체의 오브젝트 ID를 캐치하여 추가하는 방향으로 수정해보았다.

    ![https://user-images.githubusercontent.com/72292617/116361089-8aca9080-a83b-11eb-88b8-81d5683f71c9.png](https://user-images.githubusercontent.com/72292617/116361089-8aca9080-a83b-11eb-88b8-81d5683f71c9.png)

### "런치스크린 지역화"

- 문제점
    - 런치스크린에서도 텍스트를 넣고 지역화를 해주고 싶었는데 런치 스크린은 지역화가 되지 않는 문제
- 원인
    - HIG에서 답을 찾았는데 런치 스크린은 정적임으로 표시되는 텍스트를 현지화하지 않아 텍스트를 포함하지말라고 친절하게 나와있다. iOS 앱은 "로드 중 메시지"를 표시하지 않아야한다. 이걸 토대로 생각해보았을때 런치 스크린도 설정의 일부인데 앱의 지역화 텍스트를 빌드하고 설정을 통해 잡아와야하는데 런치스크린도 그런 데이터를 설정하는 단계라 언제 끝날지 몰라 지역화를 지원하지 않는다.
- 추가 고민점
    - 텍스트 지역화는 앱 실행하여 설정단계에서 진행됨으로 런치 스크린 텍스트 지역화는 이뤄지지 않지만, 이미지를 지역화하여 런치스크린에 나타날 수 있게는 해줄 수 있다. 런치 스크린도 지역화 대상에 묶으면 파일들이 나타나고 각 지역에 맞게 해당 이미지를 얹어줄 수 있다.
- 해결방안
    - 지원되지 않는 런치 스크린에 대한 지역화 및 텍스트를 제거하였다.

### "유저 노티피케이션 해제"

- 문제점
    - 메모가 삭제 및 완료 되었는데도 유저노피티케이션 알림이 오는 문제
- 원인
    - 노티피케이션을 생성하고 호출이되었다면 메모가 삭제 및 완료되는 시점에서는 이미 호출된 노티피케이션 알림을 해제하는 기능도 구현해야하는데 이 부분이 구현되지 않아 발생하였다.
- 해결방안
    - 메모가 삭제되면 해당 Notification의 Identifier를 가지고 해제해주도록 구현 수정하였다.

    ```swift
        mutating func updateProgressStatus(with progressStatus: String) {
          self.progressStatus = progressStatus
          if self.progressStatus == ProgressStatus.done.rawValue {
              notificationManager.removeNofitication(name: "\(self.dueDate)")
          } else {
              notificationManager.configureNotification(name: "\(self.dueDate)", date: self.dueDate)
          }
      }
    ```

    위와 같이 상태를 체크할때 Done 영역에 있어도 노티피케이션의 알림을 해제해준다.

### "JSON 데이터 인코딩"

- 문제점
    - 메모를 업데이트하고 다시 앱을 실행하면 정상적으로 추가된 메모가 읽혀지지 않는 문제
- 원인
    - 메모 JSON 데이터에 대하여 디코딩을 하여 UI에 보이게해주는데 업데이트가 될때 로컬 디스크에 인코딩을 거치지 않고 저장하여 발생하였다.
- 해결방안
    - 메모가 업데이트가 될때 FileManager에서 구현해준 아래 업데이트 시 인코딩하여 저장하는 메서드를 호출하도록 수정하였다.

    ```swift
    func updateFile() {
          let data = try! JSONEncoder().encode(allItems)
          try? data.write(to: fileURL)
      }
    ```

## **🤔 고민한 점**

### "칸반보드의 UI를 어떻게 효율적으로 구성할 수 있을까?"

- 고민에 대한 노력
    - 각 보더 즉 테이블 뷰마다 identifier를 설정하여 재사용으로 배치되도록 구성하였다.
    - 우선 컬렉션뷰를 배치하고 컬렉션뷰 셀을 3개로 나눠 각 셀에 Todo/Doing/Done 테이블 뷰를 넣는 구조로 기능을 분리하였다.

    ```swift
    extension SectionCollectionViewCell: UITableViewDataSource {
      func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
          guard let cell = tableView.dequeueReusableCell(withIdentifier: BoardTableViewCell.identifier) as? BoardTableViewCell, let item = board?.item(at: indexPath.row) else {
              return UITableViewCell()
          }
          
          cell.updateUI(with: item)
          return cell
      }
      
      func tableView(_ tableView: UITableView, numberOfRowsInSection section: Int) -> Int {
          return board?.itemsCount ?? 0
      }
      ...
    ```

### "Mode case 네이밍에 관해 더 좋은 네이밍은 없을까?"

```swift
enum Mode {
  case editable
  case uneditable
...
```

- 고민에 대한 노력
    - '수정 할 수 없는' 이라는 뜻으로 네이밍하였는데, 수정할 수없는것이 읽기전용이라는 네이밍이 더 적합하여 readOnly로 개선하였다.

### "dueDate와 date 의미가 모호한데 개선해볼 수 없을까?"

```swift
var dueDate: Int
var date: Date {
```

- 고민에 대한 노력
    - 처음 dueDate를 systemTime으로 변경하였는데 명확하지 않게 느껴졌다. systemTime으로 고치니 date의 의미가 불명확해졌다. 그래서 아래와 같이 수정을 거쳤다.
    - dueDate는 아이템에 대한 데이터를 받아오는 시스템 시간임으로 timeStamp로 date는 받아온 timeStamp에 대한 날짜 표시로 변환하는것임으로 dueDate로 개선하였다.

    ```swift
    var timeStamp: Int
    var dueDate: Date {
        return Date(timeIntervalSince1970: TimeInterval(timeStamp))
    }
    ```

### "convertDateToString 메소드가 extension으로 선언되었을 때와 직접 선언한 객체에서 선언되었을 때의 장단점은 무엇일까?"

```swift
extension DateFormatter {
  func convertDateToString(date: Date) -> String {
      let currentLocale = Locale.current.collatorIdentifier ?? "ko_KR"let formatter = DateFormatter()

       formatter.locale = Locale(identifier: currentLocale)
       formatter.dateFormat = "yyyy.MM.dd"return formatter.string(from: date)
  }
}
```

- 고민에 대한 노력
    - DateFormatter를 extension하여 해당 메서드를 구현하게되면 추후 다른 클래스나 객체에서 해당 메서드를 호출할때 Item 타입의 인스턴스를 만들어 접근하지 않아도 된다. 클래스 내부에 메서드로 만들게되면 해당 메서드를 사용할때 객체 인스턴스를 만들고 호출해야함으로 인스턴스를 만드는 메모리가 잡혀 extension으로 접근하는것이 메모리적 측면에서 더 효율적이다.

### "함수 안에 함수를 넣어 선언하는 상황과 이유는 무엇일까?"

```swift
func updateUI(with item: Item) {
      self.titleLabel.text = item.title
      self.descriptionLabel.text = item.description
      self.dueDateLabel.text = item.dateToString
      
      func configureDueDateLabelColor() {
          let currentTimeInterval = Date().timeIntervalSince1970
          let currentDateToInt = Int(currentTimeInterval)
          
          if currentDateToInt > item.timeStamp {
              self.dueDateLabel.textColor = .red
          } else {
              self.dueDateLabel.textColor = .black
          }
      }
      
      configureDueDateLabelColor()
  }
```

- 고민에 대한 노력
    - 해당 메서드는 updateUI 메서드가 호출되는 경우에만 호출된다. 그럼으로 항상 내부적으로 호출되어 dueDate의 상태(기간만료 되었는지)를 체크해준다는걸 보여줄 수 있다.

### "히스토리 내역을 굳이 로컬디스크에 저장해놓을 필요가 있을까?"

- 고민에 대한 노력
    - 앱의 할일 리스트들은 로컬디스크에 저장해놓아야하지만 인앱 상황에서 변경되는 히스토리들은 굳이 로컬디스크에 캐싱하여 저장할 필요가 없이 앱이 켜져있는 동안의 히스토리만 보여주면 된다고 생각한다. 만약 로컬로 저장해놓는다면 계속 데이터가 쌓일것이고 사용자 입장에서도 불편할것같다. 그래서 HistoryManager라는 싱글톤 타입을 통해 앱이 켜져있을때만 historyContainer로 구현한 String,Date 자료구조에 담고 테이블뷰에 데이터를 나타낼 수 있도록하였다.

    ```swift
    class HistoryManager {
      static let shared = HistoryManager()
      var historyContainer = [(HistoryLog, Date)]()
    }
    let historyManager = HistoryManager.shared
    ```

    - 그러다 또 생각해본점이 히스토리 내역을 코어데이터를 사용하여 구현을 하면 심플하지 않을까 생각했다.
    결국 코어데이터도 디바이스에 저장하는것으로 로컬디스크와 차이는 디바이스에서만 저장됨으로 만약 다른 디바이스로 작동을 시킨다면 히스토리 정보가 남아있지 않게 되었다. 굳이 둘중에 따지자면 코어데이터 보다는 로컬디스크에 저장하는 방식이 더 맞다고는 느껴졌다.

### "로그 객체를 출력할때 자동으로 description이 출력되게 할 순 없을까?"

```swift
enum HistoryLog {
   case add(String)
   case move(String, String, String)
   case delete(String)

   var description: String {
       switch self {
       case .add(let title):
           return "Added \(title)"
       case .move(let title, let before,  let after):
           return "Moved \(title) from \(before) to \(after)"
       case .delete(let title):
           return "Deleted \(title)"
       }
   }
 }
```

- 고민에 대한 노력
    - 매번 description 출력을 위해 한단계 거치는것이 불편해서 찾아보았는데 CustomConvertibleString 프로토콜에서 해당 기능을 해준다. 이 부분을 토대로 개선해보았다.

    ```swift
    enum HistoryLog: CustomStringConvertible {
      case add(String)
      case move(String, String, String)
      case delete(String, String)

      var description: String {
          switch self {
          case .add(let title):
              return String(format: NSLocalizedString("Added '%@'.", comment: ""), title)
          case .move(let title, let before,  let after):
              return String(format: NSLocalizedString("Moved '%@' from %@ to %@.", comment: ""), title, before.localized, after.localized)
          case .delete(let title, let before):
              return String(format: NSLocalizedString("Removed '%@' from %@.", comment: ""), title.localized, before.localized)
          }
      }
    }
    ```

### "TableView 3개로 UI를 구성할 수 있지 않을까?"

- 고민에 대한 노력
    - 테이블뷰 3개로 구성할 수 있겠지만 컬렉션뷰를 사용할때 섹션의 커스터마이징 측면에서 더 용이하다고 판단되었다. 단순히 리스트를 보여준다면 테이블뷰가 나쁘지 않겠지만 컬렉션뷰를 통해 각 섹션간 드래그앤드롭과 데이터 변경등 더 커스텀스러워 컬렉션뷰 셀에 테이블뷰를 얹어 구현하게 되었다.

### "UserNotification 설정 시 badge는 무슨 역할을 할까?"

- 고민에 대한 노력
    - 실제로 감이오지 않아 구현해보고 테스트를 해봤는데 뱃지는 그냥 앱 아이콘에 표시되는 아래와 같은 수이다. 1로 설정해놓은 1이 10으로 설정해놓으면 10이 뜬다. 앱에서 사용자에게 몇개의 알림이 있다고 보내주고 싶을때 설정하면 좋을것 같다.

    ![https://user-images.githubusercontent.com/72292617/116362745-59eb5b00-a83d-11eb-96b0-bed29202ef9d.png](https://user-images.githubusercontent.com/72292617/116362745-59eb5b00-a83d-11eb-96b0-bed29202ef9d.png)

### "App의 NotificationDelegate를 어디서 다뤄주면 좋을까?"

- 고민에 대한 노력
    - 처음 VC에서 해당 노티피를 구성하고 호출하니까 막연하게 Extension하여 구성하였다. 그런데 생각해보면 만약 App이 닫혀있고 푸시 알림을 통해 다시 앱의 뷰가 로드된다면 뷰가 정상적으로 로드되지 않거나 지연될 때 해당 기능의 역할을 제대로 수행하지 못한다. 그래서 App의 전반적인 부분의 역할을 해주는 AppDelegate에서 다뤄주는것이 더 적합하다고 생각한다.

    ```swift
    class AppDelegate: UIResponder, UIApplicationDelegate {
      func application(_ application: UIApplication, didFinishLaunchingWithOptions launchOptions: [UIApplication.LaunchOptionsKey: Any]?) -> Bool {
          // Override point for customization after application launch.
          notificationManager.requestNotificationAuthorization()
          return true
      }
    ```

    - 실제로 리팩토링 하기 전 NotificationDelegate를 메인 VC에서 설정해주었는데 앱이 실행되고 뷰가 올라오는 속도가 느릴때 간혹 알림 허용 인증이 나타나지 않았다.

## **📱 동작화면**

![https://user-images.githubusercontent.com/72292617/116354491-784c5900-a833-11eb-8bcd-99899bfeb05d.gif](https://user-images.githubusercontent.com/72292617/116354491-784c5900-a833-11eb-8bcd-99899bfeb05d.gif)

