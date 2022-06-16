# 201916016_A반_강현우_모바일 프로그래밍 시험
## ViewController

```javascript
import UIKit

class ViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
    }
```

### 메인 화면에서 다른 화면으로 넘어가는 버튼함수
```javascript
    @IBAction func b1(_ sender: UIButton) {
        tabBarController?.selectedIndex = 1
    }

    @IBAction func b2(_ sender: UIButton) {
        tabBarController?.selectedIndex = 2
    }

    @IBAction func b3(_ sender: UIButton) {
        tabBarController?.selectedIndex = 3
    }
}
```

## MovieViewController (영상 재생 페이지)

```javascript
import UIKit
import AVKit

class MovieViewController: UIViewController {

    override func viewDidLoad() {
        super.viewDidLoad()

        // Do any additional setup after loading the view.
    }

    @IBAction func btnPlayInternalMovie(_ sender: UIButton) {
        // 내부 파일  mp4
        let filePath:String? = Bundle.main.path(forResource: "FastTyping", ofType: "mp4")
        let url = NSURL(fileURLWithPath: filePath!)

        playVideo(url: url)
    }

    

    @IBAction func btnPlayExternalMovie(_ sender: UIButton) {
        // 외부 파일 mp4
        let url = NSURL(string: "https://dl.dropboxusercontent.com/s/e38auz050w2mcud/Fireworks.mp4")!

        playVideo(url: url)
    }
    ```

### AVPlayerViewController의 조작
```javascript
    private func playVideo(url: NSURL) {
        let playerController = AVPlayerViewController()

        // 비디오 URL로 초기화된 AVPlayer의 인스턴스를 생성
        let player = AVPlayer(url: url as URL)
        
        // AVPlayerViewController의 player 속성에 위에서 생성한 AVPlayer 인스턴스를 할당
        playerController.player = player

        self.present(playerController, animated: true) {

            player.play()  // 비디오 재생
        }
    }
}
```


## MapViewController (맵 검색 페이지)

```javascript
import UIKit
import MapKit

class MapViewController: UIViewController, CLLocationManagerDelegate {

    @IBOutlet var myMap: MKMapView!
    @IBOutlet var lbl1: UILabel!
    @IBOutlet var lbl2: UILabel!

    let locationMnager = CLLocationManager()

    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        lbl1.text = ""
        lbl2.text = ""
        locationMnager.delegate = self
        // 정확도를 최고로 설정
        locationMnager.desiredAccuracy = kCLLocationAccuracyBest
        //위치 데이터를 추적하기 위해 사용자에게 승인 요구
        locationMnager.requestWhenInUseAuthorization()
        // 위치 업데이트 시작
        locationMnager.startUpdatingLocation()
        myMap.showsUserLocation = true
    }
    ```

### 특정 위도와 경도에 핀 설치하고 핀에 타이틀과 서브 타이틀의 문자열 표시/ 함수 호출

```javascript
    func goLocation(latitudeValue: CLLocationDegrees, longitudeValue : CLLocationDegrees, delta span :Double) -> CLLocationCoordinate2D {
        let pLocation = CLLocationCoordinate2DMake(latitudeValue, longitudeValue)
        let spanValue = MKCoordinateSpan(latitudeDelta: span, longitudeDelta: span)
        let pRegion = MKCoordinateRegion(center: pLocation, span: spanValue)
        myMap.setRegion(pRegion, animated: true)
        return pLocation
    }

    func setAnnotation(latitudeValue: CLLocationDegrees, longitudeValue : CLLocationDegrees, delta span :Double, title strTitle:String, subtitle strSubtitle:String) {
        let annotation = MKPointAnnotation()
        annotation.coordinate = goLocation(latitudeValue: latitudeValue, longitudeValue: longitudeValue, delta: span)
        annotation.title = strTitle
        annotation.subtitle = strSubtitle
        myMap.addAnnotation(annotation)
    }
    ```


## 위치 정보에서 국가, 지역, 도로를 추출하여 레이블에 표시

```javascript
    func locationManager(_ manager: CLLocationManager, didUpdateLocations locations: [CLLocation]) {
        let pLocation = locations.last
        _=goLocation(latitudeValue: (pLocation?.coordinate.latitude)!, longitudeValue: (pLocation?.coordinate.longitude)!, delta: 0.01)
        CLGeocoder().reverseGeocodeLocation(pLocation!, completionHandler: {
            (placemarks, error) -> Void in
            let pm = placemarks!.first
            let country = pm!.country
            var address:String = country!
            if pm!.locality != nil {
                address += " "
                address += pm!.locality!
            }

            if pm!.thoroughfare != nil {
                address += " "
                address += pm!.thoroughfare!
            }

            self.lbl1.text = "현재 위치"
            self.lbl2.text = address
        })
        locationMnager.stopUpdatingLocation()
    }
```

## 세그먼트 컨트롤을 선택하였을 때 호출

```javascript
    @IBAction func sgChange(_ sender: UISegmentedControl) {
        if sender.selectedSegmentIndex == 0 {
        // "현재 위치" 선택 - 현재 위치 표시
            self.lbl1.text = ""
            self.lbl2.text = ""
            ;locationMnager.startUpdatingLocation()
  
        } else if sender.selectedSegmentIndex == 1 {
        // "폴리텍대학" 선택 - 핀을 설치하고 위치 정보 표시
            setAnnotation(latitudeValue: 37.751843, longitudeValue: 128.87605740000004, delta: 1, title: "한국폴리텍대학 강릉캠퍼스", 
            subtitle: "강원도 강릉시 남산초교길 121")
            self.lbl1.text = "보고 계신 위치 "
            self.lbl2.text = "한국폴리텍대학 강릉캠퍼스"

        } else if sender.selectedSegmentIndex == 2 {
        // "이지스 퍼블리싱" 선택 - 핀을 설치하고 위치 정보 표시
            setAnnotation(latitudeValue: 37.556876, longitudeValue: 126.914066, delta: 0.1, title: "이지스퍼블리싱", subtitle: "서울시 마포구 잔다리로 109 이지스 빌딩")
            self.lbl1.text = "보고 계신 위치"
            self.lbl2.text = "이지스퍼블리싱 출판사"
        }
    }
}
```

















