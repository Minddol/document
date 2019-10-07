# 시작하기

## NUGU SDK 사용하기

### Step 1: 최소 요구사항 확인하기

* Xcode 11.0 or later
* Swift 5.1
* iOS 10.0 or later

### Step 2: NUGU SDK 설치하기

{% tabs %}
{% tab title="Cocoapods" %}
{% code-tabs %}
{% code-tabs-item title="Podfile" %}
```ruby
target 'Your_Application' do
  pod 'NuguDefaultClient'
  pod 'NuguLoginKit'
end
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="Manually" %}
You can also download the entire iOS SDK.

See "" on Github
{% endtab %}
{% endtabs %}

### Step 3: NUGU 디바이스 생성하기

{% hint style="info" %}
NUGU 디바이스를 생성하기 위해서는 NUGU Developers를 통해 제휴가 필요합니다.
{% endhint %}

https://nugu.developers.co.kr에서 ClientID, ClientSecret, RedirectURI 정보를 발급받으세요.

### Step 4: NUGU에 로그인하기

{% hint style="info" %}
NUGU 서비스를 이용하기 위해서는 OAuth 인증이 필요합니다. 
{% endhint %}

```swift
import NuguLoginKit
```

#### Type1

> info.plist 파일에 URL Scheme 추가

info.plist 파일에 다음과 같이 URL Scheme을 추가합니다. \(또는 XCode에서 NUGU를 추가할 Target의 Info 탭을 눌러 URL Types를 추가 후 URL Schemes에 "nugu.user.{pocID}"를 입력니다.\)

{% code-tabs %}
{% code-tabs-item title="info.plist" %}
```markup
<dict>
  <key>CFBundleURLTypes</key>
  <array>
    <dict>
    <key>CFBundleURLSchemes</key>
    <array>
      <string>nugu.user.{pocId}</string>
    </array>
    </dict>
  </array>
</dict>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> 앱 델리게이트 연결

인 앱 브라우저를 통한 인증결과를 NuguLoginKit에서 처리하기 위해 다음과 같이 AppDelegate 클래스에 추가해야 합니다.

{% code-tabs %}
{% code-tabs-item title="AppDelegate.swift" %}
```swift
func application(_ app: UIApplication, open url: URL, options: [UIApplication.OpenURLOptionsKey : Any] = [:]) -> Bool {
    let handled = OAuthManager<Type1>.shared.handle(open: url, options: options)
    return handled
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> 인 앱 브라우를 통해 로그인

Step 3에서 생성한 디바이스의 정보를 이용하여 다음과 같이 OAuthManager를 통해 값을 설정한 후에 인 앱 브라우저\(SFSafariViewController\)를 이용한 T-ID 로그인을 시도합니다. 인증절차가 모두 완료되면 결과를 Closure를 통해 받을 수 있습니다.

{% code-tabs %}
{% code-tabs-item title="ViewController.swift" %}
```swift
func login() {
    OAuthManager<Type1>.shared.provider = Type1(
        clientId: "{client-id}",
        clientSecret: "{client-secret}",
        redirectUri: "{redirect-uri}",
        deviceUniqueId: "{device-unique-id}"
    )
    
    OAuthManager<Type1>.shared.loginBySafariViewController(from: self) { (result) in
        switch result {
        case .success(let authInfo):
            // Save authInfo
        case .failure(let error):
            // Occured error
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> 로그인 정보 갱신

인 앱 브라우저를 통한 T-ID 로그인이 정상적으로 완료된 후 얻은 Refresh-token이 있다면, 이 후에는 인 앱 브라우저 없이 로그인 정보를 갱신할 수 있습니다.

{% code-tabs %}
{% code-tabs-item title="ViewController.swift" %}
```swift
func refresh() {
    OAuthManager<Type1>.shared.loginSilently(by: "{refresh-token}") { (result) in
        switch result {
        case .success(let authInfo):
            // Save authInfo
        case .failure(let error):
            // Occured error
        }
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

### Step 5: NUGU 서비스 사용하기

> Info.plist 파일에 마이크 권한 문구 추가

NUGU 서비스는 음성인식을 위하여 마이크 권한이 필요합니다. 마이크 권한 문구를 Info.plist 파일에 추가합니다.

{% code-tabs %}
{% code-tabs-item title="info.plist" %}
```markup
<key>NSMicrophoneUsageDescription</key>
<string>For speech recognition</string>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> AVAudioSession Category 설정

NUGU 서비스를 이용하기 위해서는 AVAudioSession의 Category를 .playAndRecord로 설정이 필요합니다.

{% code-tabs %}
{% code-tabs-item title="ViewController.swift" %}
```swift
func setAudioSession() throws {
    try AVAudioSession.sharedInstance().setCategory(
        .playAndRecord,
        mode: .default,
        options: [.defaultToSpeaker, .allowBluetoothA2DP]
    )
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

> NUGU 음성인식 요청

음성인식을 요청하기 위해서는 다음과 같은 Flow를 통해 진행합니다.

1. NuguClient 인스턴스를 생성합니다.

{% code-tabs %}
{% code-tabs-item title="ViewController.swift" %}
```swift
let client = NuguClient.Builder.build()
```
{% endcode-tabs-item %}
{% endcode-tabs %}

2. 로그인 결과로 받은 Access-token을 NuguClient 인스턴스에 설정합니다.

{% code-tabs %}
{% code-tabs-item title="ViewController.swift" %}
```swift
client.accessToken = "{access-token}" 
```
{% endcode-tabs-item %}
{% endcode-tabs %}

3. NUGU 서버와 연결합니다.

{% code-tabs %}
{% code-tabs-item title="ViewController.swift" %}
```swift
client.networkManager.connect() 
```
{% endcode-tabs-item %}
{% endcode-tabs %}

4. 음성인식을 요청합니다. 이 때, 음성인식 상태를 알기 위해서는 delegate를 설정합니다.

{% code-tabs %}
{% code-tabs-item title="VIewController.swift" %}
```swift
class ViewController: UIViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        client.asrAgent.delegate = self
    }

    func recognize() {
        client.asrAgent.startRecognition()
    }
}

// MARK: - ASRAgentDelegate

extension ViewController: ASRAgentDelegate {
    func asrAgentDidChange(state: ASRState) {
        // Observe state
    }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## 더 알아보기

NUGU SDK의 Github Repository를 통해 NUGU Components의 주요 기능들을 확인하실 수 있습니다.   
구성요소 소개 페이지에서 필요한 구성요소를 확인하고, 해당 구성요소의 Repository에서 Readme를 통해 더 자세한 정보를 얻을 수 있습니다.   
또한, NuguClientKit Repository에 있는 샘플 앱을 통해서도 NUGU SDK의 주요 사용 방법을 확인하실 수 있습니다.

