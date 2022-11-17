# Flutter

## Flutter 프로젝트 생성 명령어

---

Flutter의 새로운 프로젝트를 생성할때 이름은 스네이크 케이스(Snake Case)를 사용해야 한다.

```shell
flutter create test_app
```

## 주요 폴더 및 파일

---

### pubspec.yaml

**pubspec.yaml** 파일은 Flutter 프로젝트의 메타 데이터를 정의, 관리 하는 파일이다.

Node의 **package.json** 과 비슷한 역할을 함.

### lib 폴더

Flutter 코드 베이스를 저장하는 폴더입니다.

Flutter는 하나의 코드베이스로 IOS, Android 앱을 동시에 개발할 수 있습니다.

## Naming 규칙

---

### 파일 이름

파일이름은 SnakeCase로 작성

예 : api_response.dart



### 기본 문법 및 사용법

---

#### Json_serializble

Model.FromJson 실시간 계속 생성

flutter pub run build_runner watch

#### StatelessWidget

상태변경이 되지않는 위젯이라는 뜻

#### StatefulWidget

상태를 가질 수 있는 위젯

Intellij 기준 단축어 : stful





## IOS & Android 설정

---

### 앱 이름 변경

IOS : info.plist

```plist
<key>CFBundleDisplayName</key>
<string>[App Name]</string>
```

Android : AndroidManifest.xml

```xml
<aaplication
    android:label="[App Name]"
    android:icon="@mipmap/launcher_icon"
>
</apllication>
```
