# Android Hilt Under the hood

## Overview
Hilt는 코드를 생성하기 위해 Annotation processor를 사용한다. 소스 파일을 자바 바이트 코드로 변환할 때 컴파일러 안에서 Annotation processing이 이루어진다. Annotation processor는 소스 파일 내의 주석에 트리거된다. 일반적으로 Annotation processor는 주석과 유형을 검사하여 검증 또는 새로운 소스 코드 생성과 같은 작업을 수행한다.

<img src="/images/android-hilt-under-the-hood1.png">

Hilt에는 세 가지 주요 Annotation이 있다. @AndroidEntryPoint, @InstallIn, @HiltAndroidApp

## @AndroidEntryPoint
```
@AndroidEntryPoint
class PlayActivity : AppCompatActivity() {
  @Inject lateinit var player: MusicPlayer
  // ...
}
```
@AndroidEntryPoint를 사용하면 Activity, Fragment, View, Service와 같은 안드로이드 프레임워크 클래스에 필드 주입이 가능하다. 위 예제에서 볼 수 있듯이 @AndroidEntryPoint 주석을 선언한 클래스 안에서 @Inject를 통해서 필드를 주입한다.
  
위 예제를 컴파일하면 아래와 같은 코드가 된다.
```
@AndroidEntryPoint(AppCompatActivity::class)
class PlayActivity : Hilt_PlayActivity() {
  @Inject lateinit var player: MusicPlayer
  // ...
}
```

위 예제 코드에서 볼 수 있듯이 코틀린 코드로 구현한 PlayActivity는 Hilt에 의해 생성된 Hilt_PlayActivity를 확장하고 있다는 것을 알 수 있다. Hilt_PlayActivity는 Annotation processor에 의해 생성되며 주입을 수행하는 데 필요한 모든 로직을 포함한다.
```
@Generated("dagger.hilt.AndroidEntryPointProcessor")
class Hilt_PlayActivity : AppCompatActivity {
  override fun onCreate() {
    inject()
    super.onCreate()
  }
  private fun inject() {
    EntryPoints.get(this, PlayActivity_Injector::class).inject(this as PlayActivity);
  }
}
```
예제 코드에서는 AppCompatActivity를 확장하였지만, 일반적으로 AndroidEntryPoint 주석에 전달된 클래스를 확장한다. 이를 통해서 원하는 클래스를 함께 주입할 수 있다.  
생성된 클래스의 목적은 주입을 처리하는 것이다. 주입된 변수가 주입되기 전에 사용되는 것은 방지해야 하기 때문에 Hilt에 주입 작업은 onCreate 메서드에서 실행된다.  
<br>
코드를 조금 더 살펴보면, 예제의 inject() 함수는 Injector 인스턴스가 필요하다. PlayActivity_Injector 라는 Injector 인스턴스는 Hilt의 Annotation processor에 의해 생성된다. 생성된 Injector는 EntryPoint 유틸리티 클래스를 사용하여 Injector의 인스턴스를 얻을 수 있다. Injector 안에는 Activity의 인스턴스를 주입할 수 있는 단일 메서드가 포함되어 있다.
```
@Generated("dagger.hilt.AndroidEntryPointProcessor")
@EntryPoint
@InstallIn(ActivityComponent::class)
interface PlayActivity_Injector {
  fun inject(activity: PlayActivity)
}
```

## @InstallIn
```
@Module
@InstallIn(SingletonComponent::class)
object MusicDatabaseModule {
  // ...
}
```
InstallIn 주석은 어떤 Module(or EntryPoint)에 Install 될 것인지 알려주는 주석이다. InstallIn 주석을 사용하면 Component만 맞다면 애플리케이션 종속성 어디에서나 Inject 될 수 있다. Hilt는 내부적으로 InstallIn module이 더 쉽게 검색될 수 있도록 메타데이터 주석을 생성한다. Hilt의 processor는 메타데이터를 고정 패키지에 넣어 애플리케이션의 모든 종속성에서 생성된 메타데이터를 쉽게 찾을 수 있다.
```
package hilt_metadata
@Generated("dagger.hilt.InstallInProcessor")
@Metadata(my.database.MusicDatabaseModule::class)
class MusicDatabaseModule_Metadata {}
```

## @HiltAndroidApp
```
@HiltAndroidApp
class MusicApp : Application {
  @Inject lateinit var store: MusicStore
}
```
HiltAndroidApp 주석을 사용하면 안드로이드 프로그램 클래스에 주입을 할 수 있다. AndroidEntryPoint와 기능이 같지만, HiltAndroidApp은 또 다른 중요한 기능을 갖고 있다.  
Hilt Annotation processor가 HiltAndroidApp 주석을 만나면 ‘HiltComponents_’ prefix가 붙은 Application class와 이름이 똑같은 Wrapper class를 만든다. Wrapper class 안에 Components들이 정의되어 있다.

<img src="/images/android-hilt-under-the-hood2.png">

Components를 구성하기 위해서 Hilt는 메타데이터 패키지 안에 모든 InstallIn 주석이 달린 클래스를 찾는다. InstallIn 주석이 달린 모듈은 해당 Components 선언의 모듈 목록에 배치된다. 

<br><br>
참고 자료 : <br>
https://medium.com/androiddevelopers/mad-skills-series-hilt-under-the-hood-9d89ee227059