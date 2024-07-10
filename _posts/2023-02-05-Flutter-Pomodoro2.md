---
title: Flutter POMODORO App 만들기 (2) - Flutter 앱 시작화면/스플래시와 튜토리얼/가이드 스크린 만들기
author: gksdygks2124
date: 2023-02-05 10:48:00 +0900
categories: [Flutter, POMODORO]
tags: [flutter, shared_preferences, smooth_page_indicator, carouse_indicator, flutter check first run, flutter 최초실행 확인하기, flutter 앱 가이드 스크린 / 설명 스크린 만들기, flutter local storage, flutter indicators]
lastmode: 2023-02-05 10:48:00
sitemap:
  changefreq: daily
  priority : 1.0
---
> UI Designed by Omar Sherif(https://www.behance.net/iomarsherif)  
> Project UIKIT: https://www.behance.net/gallery/98918603/POMO-UIKIT?tracking_source=search_projects%7Cpomo+uikit

<br>

## 대략적인 구조
1. 앱의 가이드 화면은 최초 실행시에만 메인 화면 나오기 전에 나와야하며, 따로 가이드가 필요할 때 다시 볼 수 있어야 한다.
2. 가이드 화면은 스킵이 가능하고, 다음버튼도 있어야한다. 가이드 화면을 스킵했거나, 다 읽었을 경우 다음 앱 실행시에는 가이드화면이 아닌 바로 메인화면이 보여야한다.

<br>

## <b>앱 최초 실행 확인하기</b>
앱을 사용하면 앱을 설치 후 처음 실행했을 때 간단한 앱 설명서가 있는 경우가 있다. 이러한 설명스크린은 앱의 최소 실행 시에 확인할 수 있고, 2번째부터는 앱 실행 시에는 볼 수 없다. 이걸 어떻게 구현할까 생각하다가 당장 내가 생각한 방법은 최초 실행인지를 판단하는 isFirstRun이라는 간단한 불리언 값을 담는 변수를 통해서 확인하는 것이다. 이런 간단한 값을 따로 DB에 저장하는 것은 오히려 낭비라고 생각하기 때문에 SharedPreferences를 사용할 예정이다. SharedPreferences는 사실 Android OS쪽이고, ios쪽에서는 NSUserDefault가 맞다. OS마다 다른데 더 자세한 설명은 <a href="https://pub.dev/packages/shared_preferences#storage-location-by-platform">공식사이트</a>에 나와있다.

<br>

### <b>SharedPreferences란?</b>
앱 최초 실행확인을 위한 불리언 값, 자동로그인 여부 같은 앱에 필요한 간단한 값이나 앱 초기 설정값을 저장하기 위해 사용한다. 앱에 파일 형태(XML)로 데이터를 저장하며, 앱을 삭제하거나 앱 캐시를 삭제하기 전까지 반영구적으로 보존된다. Flutter에서 사용하기 위해서는 <a href="https://pub.dev/packages/shared_preferences/install">공식사이트</a>의 설치 방법에 따라 설치하면 된다.

<br>

### <b>SharedPreferences 사용하기</b>

<br>

#### Write Data
```dart
// Obtain shared preferences.
final prefs = await SharedPreferences.getInstance();

// Save an integer value to 'counter' key.
await prefs.setInt('counter', 10);
// Save an boolean value to 'repeat' key.
await prefs.setBool('repeat', true);
// Save an double value to 'decimal' key.
await prefs.setDouble('decimal', 1.5);
// Save an String value to 'action' key.
await prefs.setString('action', 'Start');
// Save an list of strings to 'items' key.
await prefs.setStringList('items', <String>['Earth', 'Moon', 'Sun']);
```

<br>

#### Read Data
```dart
// Obtain shared preferences.
final prefs = await SharedPreferences.getInstance();

// Try reading data from the 'counter' key. If it doesn't exist, returns null.
final int? counter = prefs.getInt('counter');
// Try reading data from the 'repeat' key. If it doesn't exist, returns null.
final bool? repeat = prefs.getBool('repeat');
// Try reading data from the 'decimal' key. If it doesn't exist, returns null.
final double? decimal = prefs.getDouble('decimal');
// Try reading data from the 'action' key. If it doesn't exist, returns null.
final String? action = prefs.getString('action');
// Try reading data from the 'items' key. If it doesn't exist, returns null.
final List<String>? items = prefs.getStringList('items');
```

<br>

#### Remove Data
```dart
// Obtain shared preferences.
final prefs = await SharedPreferences.getInstance();

// Remove data for the 'counter' key.
final success = await prefs.remove('counter');
```

<br>

### <b>main.dart 수정하기</b>
POMODORO 1편에 이어서 코드를 코드를 작성했다(이번 편에 추가된 코드는 주석으로 `added`표시를 했다). 저번 포스팅에 스플래쉬 스크린을 작성했다. 이번 편에는 스플래쉬 스크린 종료 후 홈스크린이 나오기 전에 앱이 최초 실행인지 아닌지 여부에 따라서 가이드 스크린을 보여줄지 홈스크린을 보여줄지 정해야한다.
```dart
import 'package:flutter/material.dart';
import 'package:flutter_native_splash/flutter_native_splash.dart'; // added
import 'package:pomodoro_app/screen/guide_how_screen.dart'; // added
import 'package:pomodoro_app/screen/home_screen.dart'; // added
import 'package:shared_preferences/shared_preferences.dart';

void main() {
  WidgetsBinding widgetsBinding = WidgetsFlutterBinding.ensureInitialized();
  FlutterNativeSplash.preserve(widgetsBinding: widgetsBinding);
  runApp(const MyApp());
  FlutterNativeSplash.remove();
}

class MyApp extends StatefulWidget {
  const MyApp({super.key});

  @override
  State<MyApp> createState() => _MyAppState();
}

class _MyAppState extends State<MyApp> {
  late SharedPreferences prefs; // added
  bool firstRun = true; // added

  // added
  void initPrefs() async {
    prefs = await SharedPreferences.getInstance();
    final isFirstRun = prefs.getBool('isFirstRun');
    if (isFirstRun == false) {
      setState(
        () {
          firstRun = false;
        },
      );
    }
  }
  
  // added
  @override
  void initState() {
    // TODO: implement initState
    super.initState();
    initPrefs();
  }

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        scaffoldBackgroundColor: const Color(0xFFE7626C),
        textTheme: const TextTheme(
          displayLarge: TextStyle(
            color: Color(0xFF232B55),
          ),
        ),
        cardColor: const Color(0xFFF4EDDB),
      ),
      home: firstRun ? const GuideMainScreen() : const HomeScreen(), // added
    );
  }
}

```

<br>

1) <span style="color:Crimson">import</span>(line2~4): shared_preferences를 사용하기 위해 패키지 설치 후 import를 해준다. `GuideHowScreen()`과 `HomeScreen()`위젯 또한 import를 해주는데 우선 간단하게 scaffold()위젯까지만 작성하고 import해준다.

<br>

2) <span style="color:Cyan">prefs, firstRun</span>(line22~23): prefs, firstRun를 사용하기 위해 변수 선언

<br>

3) <span style="color:GreenYellow">initPrefs()</span>(line26~36): initPrefs()라는 함수를 선언한다. shared preferences에서 값을 얻는 것은 비동기로 처리된다. 따라서 반환하는 값이 없고, 비동기 처리를 하기위해 `void`와 `async`로 함수를 작성한다. 함수의 역할은 아래와 같다.  
위에서 선언한 prefs변수를 shared preferences객체를 얻어 초기화를 진행한다. 함수 내에 지역함수 isFirstRun은 `getBool()` shared preferences 객체에 있는 isFirstRun이라는 키의 값으로 초기화를 진행한다(이름은 같지만 첫번째는 iniPrefs()의 지역변수 isFirstRun이고, 두번째는 shared preferences에 저장된 isFirstRun키와 값이다). 마지막으로 조건에 따라 위젯의 state(firstRun)의 값을 변경하는 코드이다 (<b>setState</b>에 관한 정보는 <a href="https://api.flutter.dev/flutter/widgets/State/setState.html">여기</a>를 참고하자).

<br>

4) <span style="color:GreenYellow">initState()</span>(line39~44): initState()는 Flutter Framework에서 제공되는 함수이다. 간단하게 설명하자면 initState()는 Widget의 LifeCycle동안 단 1번, 가장 먼저 호출되는 함수이다. build()함수 같은 경우는 setState()로 재호출이 가능하지만 말이다. (<b>initState()</b>에 대한 더 자세한 설명은 <a href="https://api.flutter.dev/flutter/widgets/State/initState.html">여기</a>를 참고하자)

<br>

5) <span style="color:Khaki">home: firstRun ? const GuideMainScreen() : const HomeScreen()</span>(line60): 삼항 연산자를 사용해여서 위에서 선언한 변수 firstRun의 값에 따라 홈화면을 제어하는 것이다.

<br>

앱을 최초 실행하면, build()함수가 호출되서 화면을 그리기 전에 main.dart에서 initState()가 먼저 호출된다. initState() 내부적으로 initPrefs()함수를 호출한다. 이때 앱의 최초실행이기 때문에 SharedPreferences에는 어떠한 값도 없는 상태이다. 따라서 initPrefs()의 지역함수 isFirstRun의 값은 Null이 된다. if문의 조건을 만족하지 못해서, firstRun의 값은 그대로 true를 유지한다. 이렇게 initState()스택이 끝나고, build()함수가 호출되는데 이때 home:에서 firstRun이 true이기 때문에 삼항연산자에 따라 `GuideMainScreen()`이 그려진다.

<br>
<br>

## <b>가이드 스크린 만들기</b>
![smooth_page_indicators](https://user-images.githubusercontent.com/92556048/216868869-055b8725-065d-44e0-989c-837af4d2ca4d.gif)

<br>

### <b>guide_main_screen.dart</b>
- `SmoothPageIndicator()`의 `effect`는 다양한 옵션을 갖고 있다. 모든 옵션들은 <a href="https://pub.dev/packages/smooth_page_indicator#effects">여기</a>에서 확인할 수 있다.
- `skipTutorial()`은 듀토리얼을 스킵했을 때 SharedPreferences에 isFirstRun: false(KEY:VALUE 형태로 저장한다. 따라서 앱을 재실행했을 때 main.dart의 firstRun이 false의 값을 갖기 때문에 가이드 스크린이 아닌 홈 스크린이 출력된다. 마지막으로 이 함수는 SharedPreferences를 사용하기 때문에 비동기 처리를 한다.
- `MaterialApp()`과 `Scaffold()`를 굳이 분리해서 다른 위젯으로 작성한 이유는 <a href="https://stackoverflow.com/questions/44004451/navigator-operation-requested-with-a-context-that-does-not-include-a-navigator">여기</a>를 참조하면 된다. (에러 발생해서)  

<br>

```dart
import 'package:flutter/material.dart';
import 'package:pomodoro_app/screen/guide_break_screen.dart';
import 'package:pomodoro_app/screen/guide_focus_screen.dart';
import 'package:pomodoro_app/screen/guide_how_screen.dart';
import 'package:pomodoro_app/screen/home_screen.dart';
import 'package:pomodoro_app/widget/appbar_skip_widget.dart';
import 'package:shared_preferences/shared_preferences.dart';
import 'package:smooth_page_indicator/smooth_page_indicator.dart';

class GuideMainScreen extends StatelessWidget {
  const GuideMainScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      home: GuideScreen(),
    );
  }
}

class GuideScreen extends StatelessWidget {
  GuideScreen({super.key});

  final _controller = PageController();

  void skipTutorial() async {
    final prefs = await SharedPreferences.getInstance();
    prefs.setBool('isFirstRun', false);
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF4EDDB),
      appBar: const AppbarSkip(),
      body: Column(
        children: [
          SizedBox(
            height: 570,
            child: PageView(
              controller: _controller,
              children: const [
                GuideHowScreen(),
                GuideFocusScreen(),
                GuideBreakScreen(),
              ],
            ),
          ),
          Row(
            children: [
              const Padding(padding: EdgeInsets.only(left: 40)),
              SmoothPageIndicator(
                controller: _controller,
                count: 3,
                effect: const ExpandingDotsEffect(
                  activeDotColor: Color(0xFFE7626C),
                  dotColor: Color(
                    0xFF232B55,
                  ),
                ),
              ),
              const SizedBox()
            ],
          ),
          const Padding(
            padding: EdgeInsets.symmetric(
              vertical: 45,
            ),
          ),
          SizedBox(
            width: 181,
            height: 49,
            child: ElevatedButton(
              onPressed: () {
                skipTutorial();
                Navigator.push(
                  context,
                  MaterialPageRoute(
                    builder: (context) => const HomeScreen(),
                  ),
                );
              },
              style: ElevatedButton.styleFrom(
                backgroundColor: const Color(
                  0xFF232B55,
                ),
                shape: RoundedRectangleBorder(
                  borderRadius: BorderRadius.circular(12),
                ),
              ),
              child: const Text(
                'SKIP',
                style: TextStyle(color: Colors.white),
              ),
            ),
          ),
        ],
      ),
    );
  }
}

```

<br>
<br>

### <b>appbar_skip_widget.dart</b>
- 아래 line13과 같이 AppBar()를 따로 Widget()을 만들어서 사용할 경우, `preferredSized`를 통해 반드시 AppBar()의 heigth를 정해줘야한다. 안 그러면 AppBar()에 height에 대한 정보가 없어서 오류가 발생한다.

<br>

```dart
import 'package:flutter/material.dart';
import 'package:pomodoro_app/screen/home_screen.dart';
import 'package:shared_preferences/shared_preferences.dart';

class AppbarSkip extends StatelessWidget with PreferredSizeWidget {
  const AppbarSkip({super.key});

  void skipTutorial() async {
    final prefs = await SharedPreferences.getInstance();
    prefs.setBool('isFirstRun', false);
  }

  @override
  Size get preferredSize => const Size.fromHeight(kToolbarHeight);

  @override
  Widget build(BuildContext context) {
    return AppBar(
      elevation: 0,
      backgroundColor: const Color(0xFFF4EDDB),
      leading: InkWell(
        onTap: () {
          skipTutorial();
          Navigator.push(
            context,
            MaterialPageRoute(
              builder: (context) => const HomeScreen(),
            ),
          );
        },
        child: const Padding(
          padding: EdgeInsets.only(
            left: 20,
            top: 10,
          ),
          child: Text(
            "SKIP",
            style: TextStyle(
              color: Color(0xFF232B55),
              fontSize: 13,
              fontWeight: FontWeight.bold,
            ),
          ),
        ),
      ),
    );
  }
}
```

<br>
<br>

### <b>guide_how_screen.dart</b>
guide_how_screen.dart, guide_focus_screen.dart, guide_break_screen.dart파일들은 안에 사용된 텍스트와 사진만 다르고, 코드 구조는 다 똑같기 때문에 guide_how_screen.dart파일만 올렸다. 다른 코드들은 <a href="https://github.com/HANYOHAN0923/pomodoro">깃허브</a>에서 볼 수 있다.

<br>

```dart
import 'package:flutter/material.dart';

class GuideHowScreen extends StatelessWidget {
  const GuideHowScreen({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      backgroundColor: const Color(0xFFF4EDDB),
      body: SizedBox(
        height: 570,
        child: Column(
          children: <Widget>[
            const Padding(
              padding: EdgeInsets.only(
                right: 150,
              ),
              child: Text(
                "HOW TO",
                style: TextStyle(
                  color: Color(0xFFE7626C),
                  fontSize: 35,
                ),
              ),
            ),
            const SizedBox(
              height: 50,
            ),
            const Text(
              'Choose a task that you',
              style: TextStyle(
                color: Color(
                  0xFF232B55,
                ),
                fontSize: 25,
              ),
            ),
            const Text(
              'need to complete',
              style: TextStyle(
                color: Color(
                  0xFF232B55,
                ),
                fontSize: 25,
              ),
            ),
            const SizedBox(
              height: 90,
            ),
            Image.asset(
              'images/step1.png',
            ),
          ],
        ),
      ),
    );
  }
}
```

