# 201916016_A반_강현우_모바일 프로그래밍 시험

# ViewController

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

# MovieViewController (영상 재생 페이지)

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


# MapViewController (맵 검색 페이지)

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

# WebViewController (앱페이지 검색 페이지)

```javascript
import UIKit
import WebKit

class WebViewController: UIViewController, WKNavigationDelegate {

    @IBOutlet var txtUrl: UITextField!
    @IBOutlet var myWebView: WKWebView!
    @IBOutlet var myActivity: UIActivityIndicatorView!
```

### url의 인수를 통해 웹 페이지의 주소를 전달받아 웹 페이지를 보여 줌
```javascript
    func loadWepPage(_ url: String) {
        let myUrl = URL(string: url)
        let myRequest = URLRequest(url: myUrl!)
        myWebView.load(myRequest)
    }
```

### 웹 페이지의 초기 페이지 설정
```javascript
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view.
        myWebView.navigationDelegate = self
        loadWepPage("http://www.naver.com")
    }
```

### 텍스트 필드에 적힌 주소로 웹 뷰 로딩
```javascript
    @IBAction func btnGotoUrl(_ sender: UIButton) {
        let myUrl = checkUrl(txtUrl.text!)
        txtUrl.text = ""
        loadWepPage(myUrl)
    }
```

### 웹 플레이어가 로딩 중 일때 인디케이터의 온/오프
```javascript
    func webView(_ webView: WKWebView, didCommit navigation: WKNavigation!) {
        myActivity.startAnimating()
        myActivity.isHidden = false
    }

    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        myActivity.stopAnimating()
        myActivity.isHidden = true
    }

    func webView(_ webView: WKWebView, didFailProvisionalNavigation navigation: WKNavigation!, withError error: Error) {
        myActivity.stopAnimating()
        myActivity.isHidden = true
    }
```

### "http//" 문자열이 없을 경우 자동으로 삽입
```javascript
    func checkUrl(_ url: String) -> String {
        var strUrl = url
        let flag = strUrl.hasPrefix("http://")
        if !flag {
            strUrl = "http://" + strUrl
        }
        return strUrl
    }
```

### [Site1] 버튼 클릭 시 구글 홈페이지로 이동
```javascript
    @IBAction func btnGoSite1(_ sender: UIButton) {
        loadWepPage("http://www.google.com")
    }
```

### [Site2] 버튼 클릭 시 유튜브 홈페이지로 이동
```javascript
    @IBAction func btnGoSite2(_ sender: UIButton) {
        loadWepPage("http://m.youtube.com")
    }
```

### HTML 코드를 변수에 저장하고 번튼 클릭 시 문법에 맞게 웹뷰에 나타냄
```javascript
    @IBAction func btnLoadHtmlString(_ sender: UIButton) {
        let htmlString = "<h1> HTML String </h1><p> String 변수를 이용한 웹 페이지 </p><p><a href=\"http://2sam.net\">2sam</a>으로 이동</p>"
        myWebView.loadHTMLString(htmlString, baseURL: nil)
    }

    @IBAction func btnLoadHtmlFile(_ sender: UIButton) {
        let filePath = Bundle.main.path(forResource: "htmlView", ofType: "html")
        let mtUrl = URL(fileURLWithPath: filePath!)
        let myRequest = URLRequest(url: mtUrl)
        myWebView.load(myRequest)
    }
```

### 웹 페이지의 조작
```javascript
    @IBAction func btnStop(_ sender: UIBarButtonItem) {
        myWebView.stopLoading() // 웹 페이지의 로딩을 중지
    }

    @IBAction func btnReload(_ sender: UIBarButtonItem) {
        myWebView.reload() // 웹 페이지를 재로딩
    }

    @IBAction func btnReWind(_ sender: UIBarButtonItem) {
        myWebView.goBack() // 이전 웹 페이지로 이동
    }

    @IBAction func btnGoForward(_ sender: UIBarButtonItem) {
        myWebView.goForward() // 다음 웹 페이지로 이동
    }
}
```

# 동작

## 초기 페이지의 동작
<img src="https://user-images.githubusercontent.com/106363908/174185741-084a840e-63bb-4439-882d-65822ea78135.png" width="400" height="700">

## 맵 검색 페이지의 동작
<img src="https://user-images.githubusercontent.com/106363908/174185844-680487a0-a3b0-45da-a820-8426af894518.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174185873-4044c3ea-949a-4716-b34d-5b9470e09047.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174185894-f86f3e63-255d-4082-90b7-6022130a188e.png" width="400" height="700">

## 영상 재생 페이지의 동작
<img src="https://user-images.githubusercontent.com/106363908/174186322-42269fc0-b267-4317-8843-57c274d4db0b.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174186346-c1424426-700a-4bcc-bd9a-e9aba7f6316a.png" width="400" height="700">

## 영상 검색 페이지의 동작
<img src="https://user-images.githubusercontent.com/106363908/174186623-42e7f3e7-6628-4273-8530-287805860cc7.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174186655-552062fb-0c57-4ccf-aed4-1cbf1877a66f.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174186681-eeddb6c6-0c68-48c2-89be-6a01249ff568.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174186726-1f6ce86a-1702-404e-8988-df5017467603.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174186813-cb4370a8-e8f4-4ecb-bd33-59572aa85eae.png" width="400" height="700">

<img src="https://user-images.githubusercontent.com/106363908/174186835-d9774a7d-7179-4d2b-a09b-46f153fd53eb.png" width="400" height="700">


