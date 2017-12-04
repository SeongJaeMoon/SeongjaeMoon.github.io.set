---
title: 안드로이드 Activity의 생명주기
date: 2017-12-01 15:10:00
---

안드로이드의 Acitivity는 각 상황에 맞는 독특한 생명주기를 갖는다. 사용자의 사용으로 어플리케이션의 UI가 계속해서 화면전환이 일어남에 따라 그때그때 필요한 메소드가 호출 되며, 오버라이딩해서 사용하게 된다. 현재는 Activity와 약간 다른 생명주기를 갖는 Fragment를 많이 사용하기 위해 공부 중이지만(너무 늦은감이 있다... 다음 안드로이드 정리는 Fragment를 정리하는 걸로~) Task관리와 더불어 화면전환이 일어날 때 리스너 해제나 상태 값 저장 등이 꼭 필요한 경우가 있다. 이럴 경우 각 생성주기에 맞춰 메소드 호출이 꼭 필요하다.
       
## Acitivity는 크게 7가지 상태 변화를 가지며 Context 클래스를 상속받는다. 

### 1.onCreate

onCreate는 Acitivity가 처음 실행 되는 상태에 제일 먼저 호출되는 메소드로 여기에 실행시 필요한 각종 초기화 작업을 적어준다. 기본적으로 내가 메소드를 정의하지 않더라도 안스에서 Acitivity를 생성하면 자동으로 생성된다.

onCreate() 메소드의 코드는 일반적으로 아래와 같이 작성된다.

``` bash
    @Override   
    public void onCreate(){
        onSaveInstanceState(Bundle bundle)
        //요 Bundle 타입의 객체를 저장하는 메소드는 사용자의 Back 버튼 클릭이나 finish() 메소드를 통한 액티비티 인스턴스의 소멸이 아닌
        //시스템의 제약 조건(ANR과 같은 상황이 아닐까 생각된다.)으로 인해 액티비가 소멸될 경우 인스턴스 자체는 소멸되지만 상태를 저장하고 있어야 하는 친구가 필요하다.
        //그 상태 값을 Bundle 클래스의 키-값 쌍으로 가지고 저장해주는 친구가 바로 요 메소드라고 할 수 있다.
        //자세한 내용은 밑에 url을 첨부한다.
        super.onCreate();
        //여기다가 Activity 생성시 필요한 코드들을 작성한다.
        /*
          각종 View 초기화 ex. button = (Button)findViewById(R.id.btn);
          사실 요새는 세상이 좋아져서 갓 ButterKnife를 이용해서 이렇게 사용하지 않고도 편하게 View를 초기화하고 각종 리스너 등록이 가능하다.
         */      
        //일반적으로 객체의 초기화나 변수 초기화 등의 모든 UI를 비롯한 실행하기 전 꼭 필요한 초기화 작업을 실행한다.
    }
```
### 2.onStart

onStart는 생명주기상 onCreate 다음에 호출된다. ~~사실 나는 자주 쓰진 않는다.~~ 보통 회원가입 등이 필요한 기능에서 리스너 객체 등을 onCreate에서 선언하고 onStart에서 선언된 리스너를 등록해서 이미 로그인 된 사용자인지를 구분하고 로그인 화면으로 넘어가지 않고, 바로 메인으로 넘어가게 할 때 사용한다. onStop과 짝을 이루고 다루게 된다. 

onStart() 메소드의 코드는 일반적으로 아래와 같이 작성된다.

 
```bash
    @Override
    public void onStart(){
        super.onStart();
        //여기다가 필요한 코드들을 작성한다.
        /*
        
        */
    }
```

### 3.onResume

### 4.onPause

### 5.onStop

### 6.onRestart

### 7.onDestroy

안드로이드 Acitivity에 관한 자세한 내용은 : [안드로이드 Document](https://developer.android.com/reference/android/app/Activity.html#ActivityLifecycle)
onSaveInstanceState에 관한 자세한 내용은 : [안드로이드 Document](https://developer.android.com/training/basics/activity-lifecycle/recreating.html?hl=ko)