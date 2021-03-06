---
layout: '[post]'
title: 안전모 프로젝트 리뷰(DB부분)
date: 2018-02-20 23:00:00
categories: Android
tag:
- 안드로이드
- 자전거앱
- Android
- Java
- Firebase
- Database
- JXL
- geofire
- Q-GIS
- Project
- Capstone Design
- 안전모
- 안전하게 자전거 타는 모임
- LBS
- 위치기반서비스
- OpenSource
thumbnail: /uploads/ajm/mainlogo1.jpg
---

나의 게으름 덕분에 안전모 프로젝트 리뷰가 이제야 두 번째를 맞이하였다. 후.. 언제쯤 코드 리팩토링이 완료되어 안전모를 플레이 스토어에 업로드 가능할지 감이 안 잡힌다. 올 상반기까지 꼭 마무리하여 업로드 해야겠다. 이번 포스팅에선 안전모 DB 부분에 대해서 코드 리뷰를 해보도록 하자.

먼저, 앞선 [프롤로그](https://seongjaemoon.github.io/2018/01/14/ajmIntro/)에서 봤듯이 위치 기반 음성 안내 서비스를 위해서 실시간으로 앱 사용자의 위치를 DB에 저장한 경위도 좌표와 비교할 필요가 있었다. 그 전에, 직각 교차로 DB는 어떻게 구축되었는지 간단하게 알아보자.

### Q-GIS를 통한 DB구축.
DB구축을 위해 어디에, 어떻게, 왜 DB를 구축할지 정의할 필요가 있었다. 일단, 기존의 논문자료를 통해 직각교차로에서 자전거 사고 발생과 높은 상관 계수를 나타내는 것을 알 수 있었다.(더 자세한 내용은 아래 논문명을 참고) 때문에, 직각 교차로 DB가 어느정도 되는지가 궁금해졌다.

우선, 위치 정보를 갖는 파일(예를 들어, shp, shx 등)을 보거나 분석하기 위해선 그에 맞는 분석 프로그램이 필요하다. 때문에, 우리는 위치 정보 분석을 위해 [Q-GIS](https://www.qgis.org/ko/site/forusers/download.html)를 분석 툴로 선정하였다.
#### 데이터 조인
![서울시 25개구 데이터 및 자전거 도로](/uploads/ajm/ajmQ1.png)
위 사진은 통계청에서 제공하는 서울시 25개구 행정경계 shape 파일과 서울시 열린데이터 광장에서 제공하는 서울시 자전거 도로 shape 파일을 Q-GIS를 통해 조인(Join) 연산한 사진이다.
##### 교차로 데이터 확인
![서울시 교차로 데이터](/uploads/ajm/ITS-Q.png)
위 사진은 [ITS표준노드링크](http://nodelink.its.go.kr/)에서 제공하는 평면 교차로 shape 파일을 Q-GIS를 통해 확인한 사진이다. 그렇다. 육안으로 보기에도 너무나 많다. 이렇게 많은 DB를 핸들링 하기도 실질적으로 힘들테고, 만약 핸들링할 수 있다고 가정해도 사용자 입장에서 자전거를 타는 내내 음성 안내만 들어야 하는 상황이 올 수 밖에 없다. 데이터를 적절하게 가공하는 부분이 필요한 것을 알 수 있다.

![Q-GIS를 통한 DB구축](/uploads/ajm/Q-GIS1.png)
위 사진은 ITS표준노드링크에서 제공하는 평면 교차로 데이터를 바탕으로 Q-GIS의 플러그인을 통해 국내 주요 지도 엔진의 위성 사진, 지적도, 로드뷰 등을 보고 직접 하나하나 관심지점(Point of interest)를 만들었다. 누락된 부분이 있을 수도 있고, 지형지물의 변화로 인해 데이터의 정확도가 떨어질 수 있음은.. ~~어쩔 수 없다.~~ 계속 DB만 구축하고 있을 순 없으니까 하하.. 그래도 지형지물은 비교적 쉽게 변하지 않으므로, 위안을 삼아본다. Q-GIS에서 지도 엔진 및 플러그인을 실행하는 더 자세한 방법은 [여기](http://blog.daum.net/geoscience/413)를 참고하자.

![인구수, 자전거 도로, 사고다발지역, 직각교차로](/uploads/ajm/ajmQ2.png)
위 사진은 DB 구축을 시작하기 전에 논문 자료를 통해 자전거 도로 근방에서 더 사고가 많이 발생하는 것을 인지하고 있었지만, 육안으로 데이터를 확인하기 위해 여러 데이터를 조인하여 단계구분도를 만든 사진이다. 지리적 특징을 적절하게 회귀 분석하려면 어렵겠지만, 시간이 되는데로 해봐야겠다. ~~또 미룬다~~

### 구축된 DB 저장
이렇게 약 한 달이 넘는 시간(고생고생생고생!) 만에 구축된 DB를 실제 서버에 물리적으로 저장하고, 이를 안드로이드 클라이언트에게 제공할 방법이 필요하다. 그래서 사용한 것이 바로바로 Firebase Realtime Database다. (나중엔 Cloud Firestore도 알아보자.) AWS와 고민했지만, 한 번 써본 적 있는 Firebase를 쓰기로 했다. AWS를 공부할 시간이 부족했데스네~ 아무튼 Firebase 코드를 살펴보자.

### Firebase 활용
![새로운 프로젝트 만들기1](/uploads/ajm/firebase1.png)
우선 파이어베이스를 사용하려면 [파이어베이스 콘솔](https://console.firebase.google.com/)에 접속해서 새로운 프로젝트를 만들어야 한다. 파이어베이스 콘솔에 접속해서 새로운 프로젝트 추가를 클릭해준다.
![새로운 프로젝트 만들기2](/uploads/ajm/firebase2.png)
처음 프로젝트를 만들면 위와 같은 화면이 우리를 반긴다. 안드로이드 앱에서 사용할 것이기 때문에 'Android 앱에 Firebase 추가'를 선택한다.
![새로운 프로젝트 만들기3](/uploads/ajm/firebase3.png)
패키지명은 안스에서 사용하는 패키지명과 동일하게 입력하고, 닉네임(앱 이름), 인증키(나중에 등록이 꼭 필요한 경우가 있다.)등은 말 그대로 선택 사항이므로 선택적으로 입력한다. 나머지 2개 단계는 사이트에서 요구 하는데로 안드로이드 패키지에 app 폴더 밑에 google-services.json 파일 추가 및 build.gradle에 의존성 추가를 진행하면 된다.
```java
compile 'com.google.firebase:firebase-database:11.8.0'
compile 'com.firebaseui:firebase-ui-database:2.3.0'
compile 'com.firebase:geofire-android:2.1.1'
```
파이어베이스 데이터베이스를 사용하기 위해 데이터베이스 라이브러리에 대한 의존성을 추가하고, geofire라는 라이브러리를 build.gradle 파일에 추가로 등록 해줘야 한다. 이 geofire 라이브러리가 위치 정보 핸들링을 전담 한다고 할 수 있다. 여기서 사용하는 버전 정보는 실시간으로 바뀔 수 있으므로, 최신 버전 확인을 위해 [여기](https://github.com/firebase/geofire-java)와 [여기](https://github.com/firebase/FirebaseUI-Android)를 참고하자.

gradle 파일에 라이브러리를 추가하고 Sync project가 정상적으로 이루어졌다면(왜 때문인지 한 방에 안 될 때가 더러 있다.), 이제 자바 코드 상에서 불러다가 사용할 수 있게되었다. 얄루!

이제 좀 전에 구축해놨던 DB를 파이어베이스에 저장해보자. 파이어베이스는 JSON 트리 형식으로 데이터를 저장하며, 일반적으로 우리가 아는 RDBMS에서 사용하는 쿼리문으로 데이터를 읽고 쓸 수가 없다. 때문에 열심히 [API 문서](https://firebase.google.com/docs/database/android/start/)를 읽어가면서 개발을 진행해야 한다. 이 JSON 트리를 어떻게 구성하느냐에 따라서 데이터를 읽고 쓰는 속도가 엄청나게 차이가 난다고 한다.. (참고로 무료로 쓸땐 동접자가 100명이 넘어가면 트래픽이 걸린다고 한다.) 그래서 남들은 어떻게 저장하는지가 궁금해져? [스택오버플로우](https://stackoverflow.com/search?q=geofire)를 열심히 찾아봤다.
```json
{
  "rules": {
    ".read": true,
    ".write": true,
      "$geofire":{
      ".indexOn": "g"
      }
    }
}
```
파이어베이스 데이터베이스에 데이터를 저장할 땐 저장 규칙이 필요하다. 그렇다. 아직 안스 코드도 못 건드렸는데 할게 참 많다는 것을 알 수 있다. 우선 파이어베이스 대쉬보드에서 데이터베이스 > 규칙을 들어가면 읽고(read), 쓰는(write) 규칙이 true로 되어있다. 쉽게 말해 누구나 데이터베이스를 읽고 쓰는게 자유롭다. 저렇게 쓰여진 코드를 스택오버플로우에 가져가면 엄청 욕 먹는 것을 심심찮게 볼 수 있다. 일단, 우리는 편의상 읽고 쓰는 것을 자유롭게 해놓도록 했다.

그 밑에 $geofire와 .indexOn : "g" 이 부분을 중점적으로 봐야한다. 결과적으로 말하면, 실제 데이터가 저장되는 최상위 트리가 geofire이며, 해당 해쉬 값을 찾는 인덱스가 g 값이라는 규칙이다. 이게 뭔 소리람. 우선, java 코드를 통해 실제로 파이어베이스에 저장된 데이터를 보며 다시 살펴보자.

#### java 코드 작성
- JXL & Excel

자바 코드상에서 위치 좌표를 핸들링 하기 위해선, 구축한 DB 데이터를 규칙적으로 자바 코드로 불러오는 작업이 필요하다. 안전모에선 엑셀 파일 형태로 데이터를 작성하고, 이를 'JXL 라이브러리'를 통해 읽어오도록 개발되었다.

JXL 라이브러리는 쉽게 말해 엑셀 파일로 저장된 파일을 자바 코드로 한 줄 한 줄 읽어 올 수 있게 해주는 라이브러리이다. 단, 엑셀 2003 이하 버전(xls)으로 저장된 파일만 JXL로 핸들링이 가능하다. (JXL 라이브러리에 대한 더 자세한 설명은 [여기](http://www.nextree.co.kr/p5229/)를 참고.) jxl.jar 파일을 다운로드 받고, app > libs 폴더 밑에 넣어준다.

다운로드를 받고 build.gradle에 아래와 같이 의존성을 추가한다.
```java
compile files('libs/jxl.jar')
```
![엑셀로 저장된 파일](/uploads/ajm/excelDB.png)
엑셀 데이터는 앞서 Q-GIS를 통해 통합된 데이터의 위치 정보와 속성 정보를 위와 같이 각각 다른 컬럼으로 구성한다. 이렇게 구성된 파일은 src > main > assets 폴더 밑에 넣어준다. 준비가 끝났으니, 안스 자바 코드를 살펴보자.

- 자바 코드

```JAVA
//Application을 상속 받는 클래스에 아래 코드 작성.
@Override
    public void onCreate() {
        super.onCreate();
        //파이어베이스를 사용할 수 있도록 준비.
        if (!FirebaseApp.getApps(this).isEmpty()) {
            FirebaseDatabase.getInstance().setPersistenceEnabled(true);
        }
    }
```
```JAVA
private void excelToFirebase() {
    Workbook workbook = null;
      try {
        //데이터 베이스 정보 가져오기
          DatabaseReference databaseReference = FirebaseDatabase.getInstance().getReference();
          //assets 폴더 밑에 있는 pois.xls라는 파일을 가져오기.
          InputStream is = getBaseContext().getResources().getAssets().open("pois.xls");
          workbook = Workbook.getWorkbook(is);
          if (workbook != null) {
             Sheet sheet = workbook.getSheet(0);
              if (sheet != null) {
                  int maxColumn = sheet.getColumns();
                  int rowStartIndex = 0;
                  int rowEndIndex = sheet.getColumn(maxColumn - 1).length - 1;
                    //첫 로우 값은 속성명이므로 +1부터 시작
                  for (int row = rowStartIndex + 1; row <= rowEndIndex; row++) {
                    //각 컬럼에 해당하는 값을 가지고 온다. 속성, 경,위도 좌표(숫자도 String으로 가져온다.)
                      String content_nm = sheet.getCell(0, nRow).getContents();
                      String coor_x = sheet.getCell(1, nRow).getContents();
                      String coor_y = sheet.getCell(2, nRow).getContents();
                    //geofire 노드에 생성할 key 값을 생성한다.
                      String pushId = databaseReference.child("geofire").push().getKey();
                      //GeoFire 객체를 생성한다. 쿼리를 생성할 수 있다.
                      GeoFire geoFire = new GeoFire(databaseReference.child("geofire"));
                      //GeoFire 객체의 setLocation 메소드를 통해 key 값에 해당하는 위치 값을 설정한다.
                      geoFire.setLocation(pushId, new GeoLocation(Double.parseDouble(coor_x), Double.parseDouble(coor_y)));
                      //GeoHash 객체를 생성한다. 이 Hash 값을 통해 해당 인덱스를 찾아 위치 비교가 가능해진다.
                      GeoHash geoHash = new GeoHash(Double.parseDouble(coor_x), Double.parseDouble(coor_y));

                      //맵 컬렉션을 통해 원하는 노드를 생성하고 해당 노드 밑에 자식을 생성하여 데이터를 저장할 수 있다.
                      Map<String, Object>updates = new HashMap<>();

                      updates.put("location/"+ pushId + "/geohash", geoHash.getGeoHashString());
                      updates.put("location/"+ pushId + "/name", content_nm);
                      updates.put("geofire/" + pushId + "/g", geoHash.getGeoHashString());
                      updates.put("geofire/"+ pushId + "/l", Arrays.asList(Double.parseDouble(coor_x), Double.parseDouble(coor_y)));

                      //데이터베이스에 저장한다.
                      databaseReference.updateChildren(updates);
                  }
                } else {
                  Log.w(TAG, "Sheet is null!!");
                }
          } else {
              Log.w(TAG, "WorkBook is null!!");
          }
      } catch (Exception e) {
          e.printStackTrace();
      } finally {
          if (workbook != null) {
              workbook.close();
          }
      }
  }
```
위 코드는 location 노드엔 속성 값과 해쉬 값을, geofire 노드엔 좌표 값과 해쉬 값을 저장하는 것을 확인할 수 있다. 사용자의 위치와 이 해쉬 값의 정보를 비교하여 내가 원하는 위치에 접근했을 경우에 목표하는 액션을 취할 수가 있게 된다.(반경도 설정 가능하다.) 위치 정보는 총 5가지의 상태를 갖는데, 아래 코드로 살펴보자.
```JAVA
private void getFirebaseList(){
        try {
           //데이터베이스 인스턴스 가져오기
           ref = FirebaseDatabase.getInstance().getReference();
           DatabaseReference refs = ref.child("geofire");
           final DatabaseReference refs2 = ref.child("name");

           geoFire = new GeoFire(refs);
                //geoquery 객체 생성 0.02 --> 20m 반경, GeoLocation의 매개변수는 사용자의 경위도 위치 정보  
                GeoQuery geoQuery = geoFire.queryAtLocation(new GeoLocation(userLattitude, userLongitude), 0.02);

                geoQuery.addGeoQueryEventListener(new GeoQueryEventListener() {
                //Firebase Key 값에서 Location을 찾았을 경우 발생
                @Override
                public void onKeyEntered(String key, GeoLocation location) {
                //Firebase location/key/name -> 위치에 해당하는 속성 정보 이름
                query = ref.child("location").child(key).child("name");
                query.addListenerForSingleValueEvent(new ValueEventListener() {
                  @Override
                  public void onDataChange(DataSnapshot dataSnapshot) {
                    if (dataSnapshot.exists()){
                        try {
                          //Firebase에서 속성 정보 가져오기
                            String content = dataSnapshot.getValue(String.class);
                            //이 곳에 특정 위치 반경에 접근했을 경우 원하는 액션을 작성하면 된다.
                            }catch (Exception e){
                                e.printStackTrace();
                            }
                        }
                    }
                  @Override
                  public void onCancelled(DatabaseError databaseError) {
                      //에러 발생시 실행된다.
                      }
                    });
                  }
                  @Override
                  public void onKeyExited(String key) {
                  //위치에서 벗어났을 때 실행된다.
                  }

                  @Override
                  public void onKeyMoved(String key, GeoLocation location){
                  //위치에서 움직였을 때 실행된다.
                  }

                  @Override
                  public void onGeoQueryReady() {
                  //쿼리가 사용 가능 상태가 되면 실행된다.
                  }

                  @Override
                  public void onGeoQueryError(DatabaseError error) {
                  //에러 발생시 실행된다.
                  }
                });
          }catch (Exception e){
            e.printStackTrace();
         }
     }
```
그렇다. GeoQueryEventListener()의 익명 구현 객체의 메소드들을 오버라이딩해서 원하는 상황에 맞게 액션을 취해주면 된다. ([여기](https://github.com/firebase/geofire-java)를 참고하면 위 코드에 내용들을 더 자세하게 알 수 있다.) 안전모에선 onKeyEntered 메소드가 호출되면 음성 안내를 제공하도록 구현되어있다. 다양하게 응용이 가능할 것으로 보인다.

아무튼, 어찌된 영문인지 위치를 벗어났을 경우에 실행하고 싶은 코드는 작동이 잘 되지 않아 추가 테스트가 필요하다. addListenerForSingleValueEvent에 대한 내용은 역시 [API 문서](https://firebase.google.com/docs/database/android/start/)를 참고.

위 코드가 정상 작동하는지 실내에서 안스의 로그캣을 통해 들어오는 데이터를 확인할 수도 있지만, 제대로 작동하는지 확인하기 위해선 apk 파일을 테스트 디바이스에 업로드하고 실제 야외에서 GPS 데이터를 수신 받으며 테스트를 진행해야 정신건강에 좋다고 할 수 있다.


#### 파이어베이스 데이터베이스 데이터 확인
![location](/uploads/ajm/firebaseDB1.png)![geofire](/uploads/ajm/firebaseDB0.png)
파이어베이스 콘솔의 Database 탭을 들어가면 데이터가 저장된 모습을 육안으로 확인할 수 있다.

위 사진에서 볼 수 있듯이, location과 geofire의 자식이 똑같은 geohash 값을 갖는다. 지속적인 g:의 geohash 값의 비교를 통해서 location에 저장된 hash 값에 해당하는 위치에 사용자가 접근하면 음성 안내가 이루어지게 되는 것이다. 하지만, 위처럼 저장하면 값이 중복되어 저장되는 일이 벌어지는데, 이를 수정하는 구조를 생각해봐야겠다.

실제 테스트는 많은 디바이스에서 동시에 진행되지 않아 동시에 많은 트래픽이 필요로 하면 어떤 현상이 일어나는지는 알고 싶진 않지만.. 알아보도록 해야겠다.

이렇게 안전모 프로젝트의 DB부분을 간단하게 구성했다. 생각보다 많은 내용을 담지 못한 것 같다. ~~역시 귀차니즘?~~ 고쳐야할 부분도 많고, 추가할 부분도 많다. 갈 길이 먼 것 같다. 하지만 계속 나아가야겠지? 다음은 아직 나오지 않은 UI 부분에 대해 전체적으로 살펴보는 걸로~

* 오타나 잘못된 부분을 지적 해주시면 감사히 생각하고 수정토록 하겠습니다 :)

참고 논문
* C-ITS환경의 자전거 및 이륜차 안전서비스 연구(김진태 외 3명)
* IPA분석을 통한 서울시 자전거 이용자 특성비교 연구(박기혁)
* 신호교차로의 자전거 좌회전 운영방안 평가에 관한 연구(이충민 외 3명)
* 4지 신호교차로에서 효율적 자전거 교통류 처리방안 연구(목승준 외 2명)
