# flutter_backservice

### 사용할 패키지
local_notification : 푸쉬 알림 표시 패키지
https://pub.dev/packages/flutter_local_notifications
isolate : 백그라운드 프로세스 구동 패키지
https://dart-ko.dev/language/concurrency

1. local notification 으로 알림 1초마다 실행
2. local notification 으로 앱을 잠시 닫은 상태에서도 푸쉬 알림 받기
3. background_service 패키지로 앱 종료 시에도 푸쉬 알림 받기
순으로 알아본다.

sample.dart : 공식 local notification 샘플 코드
main.dart : background service + local notification

### isolate상태의 process
event queue에 순서대로 비동기작업 도착 후 하나씩 이벤트 핸들링
짧은 이벤트들만 처리해야 함.

### 서순
1. android/app/build.gradle에 종속성 추가
android {
    ...
    compileSdk flutter.compileSdkVersion  //doctor로 확인해서 34이하면 34로 명시
    ...
    compileOptions {
        coreLibraryDesugaringEnabled true
        ...
    }
    defaultConfig {
        ...
        multiDexEnabled true
    }
    ...
}
dependencies {
    implementation 'androidx.window:window:1.0.0'
    implementation 'androidx.window:window-java:1.0.0'
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.2.2'
}

2. android/build.gradle에 종속성 추가
코드 최상단에 추가
buildscript {
    repositories {
        google()
        mavenCentral()
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:7.3.1'
    }
}

3. android/app/src/main/AndroidManifest.xml 세팅
application 태그 내의 하단에 추가
<receiver 
  android:exported="false" 
  android:name="com.dexterous.flutterlocalnotifications.ActionBroadcastReceiver" />

4. permission 받기??

4. main함수에서 initialize
FlutterLocalNotificationsPlugin 객체 생성 후 initialize
알림을 탭하여 어플을 열 때, 앱이 백그라운드에서 실행중일 때 알림을 띄우는 함수 정의 및 지정
플랫폼별 세팅 지정 -> android/app/src/main/res/drawable 폴더에 이미지 추가(주의 ! 이거안하면 동작이안됨;)
이후 이 객체를 컨트롤러로써 사용

5. 알림 생성
플랫폼별 NotificationDetail 정의 후, 객체.show에 파라미터로써 입력

### 발생한 오류
1. E/AndroidRuntime(10621): java.lang.NoSuchMethodError: No interface method addWindowLayoutInfoListener(Landroid/app/Activity;Lj$/util/function/Consumer;)V in class Landroidx/window/extensions/layout/WindowLayoutComponent; or its super classes (declaration of 'androidx.window.extensions.layout.WindowLayoutComponent' appears in /system_ext/framework/androidx.window.extensions.jar)
-> 안드로이드 디슈가 문제. android/app/build.gradle에 다음 implementation 추가
dependencies {
    implementation 'androidx.window:window:1.0.0'
    implementation 'androidx.window:window-java:1.0.0'
    coreLibraryDesugaring 'com.android.tools:desugar_jdk_libs:1.2.2'
}

### rlxk
Push Notification (푸시 알림):
푸시 알림은 서버에서 클라이언트 앱으로 전송되는 메시지입니다. 사용자가 앱을 사용하지 않는 상태에서도 알림을 받을 수 있습니다. 이것은 Firebase Cloud Messaging(FCM), OneSignal, 또는 해당 플랫폼의 기타 푸시 알림 서비스를 통해 전송됩니다.
Local Notification (로컬 알림):
로컬 알림은 사용자의 디바이스에서 앱 자체에서 생성되고 예약된 알림입니다. 푸시 알림과 달리, 로컬 알림은 앱이 사용자의 디바이스에 직접 예약합니다. 따라서 네트워크 연결 없이도 작동합니다. 특정 시간, 날짜 또는 사용자의 특정 동작에 응답하여 표시될 수 있습니다.
In-App Notification (인앱 알림):
인앱 알림은 사용자가 앱 내부에서 볼 수 있는 알림입니다. 이것은 주로 앱의 일부로 포함되어 있으며, 일반적으로 사용자에게 메시지를 전달하거나 액션을 유도하기 위해 사용됩니다. 푸시 알림과 달리, 인앱 알림은 사용자가 앱을 열어야만 볼 수 있습니다.
Terminated Notification (종료된 알림):
앱이 종료된 상태에서 사용자에게 알림을 표시하는 것을 말합니다. 이러한 종류의 알림은 보통 푸시 알림 또는 로컬 알림을 통해 구현됩니다. 사용자가 앱을 완전히 종료한 경우에도 알림을 받을 수 있습니다.


푸쉬 알림 백그라운드 프로세스는 서버에서 신호를 계속 수신하면서 대기합니다.

일반적으로 백그라운드 프로세스의 진입점은 앱에서 정의한 서비스 클래스입니다. 안드로이드의 경우, 백그라운드 서비스는 Service 클래스를 상속하여 작성되며, iOS의 경우 백그라운드 작업을 처리하기 위해 백그라운드 모드를 사용하거나 푸시 알림 수신을 위한 특정 메서드를 구현합니다.

알림 수신을 대기하는 백그라운드 프로세스의 종료 지점은 일반적으로 사용자가 해당 서비스를 명시적으로 중지하거나 시스템 리소스가 부족하여 시스템이 해당 프로세스를 종료시킬 때입니다. 또한, 특정 작업이 완료되었거나 특정 조건이 충족되었을 때 백그라운드 프로세스가 자체적으로 종료될 수도 있습니다.




용법 :

작업을 처리할 최상위 또는 정적 메서드를 구성해야 합니다 .

@pragma('vm:entry-point')
void notificationTapBackground(NotificationResponse notificationResponse) {
  // handle action
}
initialize이 플러그인의 메소드 에서 이 함수를 매개변수로 지정하십시오 .

await flutterLocalNotificationsPlugin.initialize(
    initializationSettings,
    onDidReceiveNotificationResponse: (NotificationResponse notificationResponse) async {
        // ...
    },
    onDidReceiveBackgroundNotificationResponse: notificationTapBackground,
);
이 기능은 별도의 격리에서 실행된다는 점을 기억하세요(Linux 제외)! 또한 이 함수에는 @pragma('vm:entry-point')네이티브 측에서 호출되므로 트리 쉐이킹으로 인해 코드가 제거되지 않도록 하기 위한 주석이 필요합니다. 주석에 대한 공식 문서는 여기를 참조하세요 .

또한 개발자는 플러그인에 액세스하는 동안 작동하지만 Android에서는 컨텍스트 에 액세스 할 수 없다는Activity 점에 유의해야 합니다 . 이는 일부 플러그인(예 url_launcher: )이 메인을 Activity다시 시작하려면 추가 플래그가 필요하다는 것을 의미합니다.

알림에 대한 작업 지정 :

알림 작업은 플랫폼마다 다르며 플랫폼마다 다르게 지정해야 합니다.

iOS/macOS에서 작업은 카테고리에 정의됩니다. 자세한 내용은 구성 섹션을 참조하세요.

Android 및 Linux에서는 작업이 알림에서 직접 구성됩니다.

Future<void> _showNotificationWithActions() async {
  const AndroidNotificationDetails androidNotificationDetails =
      AndroidNotificationDetails(
    '...',
    '...',
    '...',
    actions: <AndroidNotificationAction>[
      AndroidNotificationAction('id_1', 'Action 1'),
      AndroidNotificationAction('id_2', 'Action 2'),
      AndroidNotificationAction('id_3', 'Action 3'),
    ],
  );
  const NotificationDetails notificationDetails =
      NotificationDetails(android: androidNotificationDetails);
  await flutterLocalNotificationsPlugin.show(
      0, '...', '...', notificationDetails);
}
각 알림에는 내부 ID와 공개 작업 제목이 있습니다.

notification Id : 각 알람의 인스턴스 식별
notification Channel Id : 같은 유형의 알림, 또는 순서대로 알림이 떠야 할 때 같은 채널에 등록.
notification Stream : 알림에 대한 이벤트 핸들링