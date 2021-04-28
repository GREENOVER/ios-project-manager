# iOS Project Manager Application Project
### 업무를 관리해주는 트렐로의 기능을 구현한 프로젝트 매니저 앱 프로젝트 (클라이언트 파트)
[Ground Rule](https://github.com/GREENOVER/ios-project-manager/blob/main/GroundRule.md)
***
#### What I learned✍️
- Localization
- JSON
- Decode
- FileManager
- Notification
- Alamofire
- Popover
- NWPathMonitor
- Handler
- Drag & Drop
- UserDefaults
- NotificationCenter
- Undo & Redo
- DateFormatter

#### What have I done🧑🏻‍💻
- 지역화를 구현해 한국 / 프랑스 / 영어로 언어가 나타나도록 구현하였다.
- Todo/Doing/Done의 상태를 이용하여 각 해당 테이블 셀에 맞게 구현하였다.
- 전체적인 뷰를 컬렉션 뷰 안에 테이블 뷰 3개가 들어가도록 구현하였다.
- 파일 매니저를 학습하고 구현하여 로컬에서 해당 앱의 JSON 데이터 정보를 저장하도록 구현하였다.
- 노티피케이션을 학습하고 구현하여 각 시간에 맞게 푸시 알림이 가도록 구현하였다.
- Alamofire를 학습하고 서버 파트의 주소를 이용하여 GET API 호출을 구현하였다.
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



#### Trouble Shooting 👨‍🔧



#### Thinking Point🤔
- 고민점 (1)
  - "칸반보드의 UI를 어떻게 효율적으로 구성할 수 있을까?"
- 원인 및 대책
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
  각 보더 즉 테이블 뷰마다 identifier를 설정하여 재사용으로 배치되도록 구성하였다.
- 고민점 (2)
  - "Mode case 네이밍에 관해 더 좋은 네이밍은 없을까?"
  ```swift
  enum Mode {
    case editable
    case uneditable
  ...
  ```
- 원인 및 대책
  - '수정 할 수 없는' 이라는 뜻으로 네이밍하였는데, 수정할 수없는것이 읽기전용이라는 네이밍이 더 적합하여 readOnly로 개선하였다.
- 고민점 (3)
  - "dueDate와 date 의미가 모호한데 개선해볼 수 없을까?"
  ```swift
  var dueDate: Int
  var date: Date {
  ```
- 원인 및 대책
  -ㅇ
  var date: Date  dueDate는 
  var date: Date {




#### InApp📱
![InApp_1](https://user-images.githubusercontent.com/72292617/116354491-784c5900-a833-11eb-8bcd-99899bfeb05d.gif)

