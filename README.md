# 안드로이드 앱 Root Beer 루팅탐지 우회

## 개요

안드로이드 앱 루팅 탐지 우회 기법 역량 강화를 위한 프로젝트이다.
환경은 구성된 상태라고 가정하에 진행할 것이며, 동적 분석을 통해 앱 후킹을 진행하여 루팅 탐지를 우회한다.

### 앱 특징
이 앱은 루팅(탈옥)이 된 상태인지 검사를 한다. 루팅이 되어있음을 가정하고 동적 앱 후킹을 이용해 루팅 탐지를 우회해 볼 것이다.

## 수행 방안

### 점검 도구

1. frida
2. JADX
3. Nox (adb)

### 수행 환경

OS: Android (NoX)

Application: RootBeer Sample

## 수행 및 수행 결과

### 수행


#### 1. nox_adb shell 접속

```shell
nox_adb shell
```

![image](https://github.com/user-attachments/assets/d27b4d06-0b0c-46b4-88d5-39778431b08b)


#### 2. 추출 할 앱 찾기 (nox_adb)

```shell
pm list packages -l | grep beer
```

결과: /data/app/com.scottyab.rootbeer.sample-2/base.apk=com.scottyab.rootbeer.sample

복사할 부분: /data/app/com.scottyab.rootbeer.sample-2/base.apk

![image](https://github.com/user-attachments/assets/9f6cba10-f292-4c35-8634-65948d76b524)




#### 3. 앱 추출 (Desktop cmd)

```shell
nox_adb pull /data/app/com.scottyab.rootbeer.sample-2/base.apk

ls -al
```

![image](https://github.com/user-attachments/assets/164a6383-5137-41df-b8e6-f247debba5b0)


#### 앱 실행 및 분석 (nox)

![image](https://github.com/user-attachments/assets/263c5716-3d56-48f3-ba87-f082ee8eb5cb)

![image](https://github.com/user-attachments/assets/bbf2e854-bab5-435a-bb6b-ef0089d9475c)

실행 결과: BusyBox Binary, SU Binary, 2nd SU Binary check가 루팅 탐지에 걸렸다.

#### frida-server 백그라운드 실행 (nox_adb)

```shell
cd /data/local/tmp

./frida-server &
```

![image](https://github.com/user-attachments/assets/06e58f6d-b5b6-4e00-a352-1b6851d13be8)

#### 후킹 할 앱 코드 분석 (Desktop)

JADX를 실행하고 추출한 앱 apk파일을 불러와 AndroidManifest.xml 파일을 열어본다.

![image](https://github.com/user-attachments/assets/3bd92423-be56-4559-9cc6-397f96c9eaf7)

코드를 보면 

```
package="com.scottyab.rootbeer.sample" // Line 7

android:name="com.scottyab.rootbeer.sample.MainActivity" // Line 19

```

주요 소스코드 파일 위치를 유추 할 수 있다.

![image](https://github.com/user-attachments/assets/14ca5e45-c849-49ef-b595-b76c537a1b05)

그 중 우리는 앱 이름과 일치하는 RootBeer 파일의 소스코드를 분석해 볼 것이다.

![image](https://github.com/user-attachments/assets/8b7f0353-fc6a-42d8-b725-f059edea5f1b)

위 그림과 같이 루팅을 탐지하는 함수로 유추되는 소스코드가 존재한다.

앱 실행 했을 때 BusyBox Binary, SU Binary, 2nd SU Binary check가 탐지 되었다.

Ctrl + f 를 이용해 각 내용들을 검색해 보겠다.

![image](https://github.com/user-attachments/assets/31c55b38-3998-49f0-9a76-f3f731fe5b42)

BusyBox Binary, SU Binary에 대한 함수를 찾았다.

추가로 2nd SU Binary를 찾기 위해 su를 검색해 보니 su에대한 함수를 하나 더 발견 할 수 있었다.

![image](https://github.com/user-attachments/assets/07206906-1357-46c6-8ee2-6334a80877e8)

이 3개의 함수를 동적으로 후킹 해보겠다.

#### 함수 후킹 소스코드 작성성

```
// 자바 코드를 후킹할 것이기에 그것을 명시해주기 위한 소스코드 템플릿이며 js파일이다. beer.js로 저장

Java.perform(function(){


});

```

![image](https://github.com/user-attachments/assets/7c1fe1ed-e1cb-4ded-bdc0-6daa84aff1a9)

복사한 코드를 후킹하기위해 복사해 소스코드 템플릿에 붙혀넣어준다.

```

// beer.js
Java.perform(function(){
    let RootBeer1 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkForSuBinary"].implementation = function () {
    console.log(`RootBeer.checkForSuBinary is called`);
    let result = this["checkForSuBinary"]();
    console.log(`RootBeer.checkForSuBinary result=${result}`);
    return result;
    }

});

```

![image](https://github.com/user-attachments/assets/c442b54a-ff05-4f25-9503-7b194fffbb15)


```

// beer.js
Java.perform(function(){
    let RootBeer1 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkForSuBinary"].implementation = function () {
    console.log(`RootBeer.checkForSuBinary is called`);
    let result = this["checkForSuBinary"]();
    console.log(`RootBeer.checkForSuBinary result=${result}`);
    return result;
    }


    let RootBeer2 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkForBusyBoxBinary"].implementation = function () {
    console.log(`RootBeer.checkForBusyBoxBinary is called`);
    let result = this["checkForBusyBoxBinary"]();
    console.log(`RootBeer.checkForBusyBoxBinary result=${result}`);
    return result;
    }

});


```

![image](https://github.com/user-attachments/assets/7beb3773-23cd-46de-8064-9670cdfed738)


```

// beer.js
Java.perform(function(){
    let RootBeer1 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkForSuBinary"].implementation = function () {
    console.log(`RootBeer.checkForSuBinary is called`);
    let result = this["checkForSuBinary"]();
    console.log(`RootBeer.checkForSuBinary result=${result}`);
    return result;
    }


    let RootBeer2 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkForBusyBoxBinary"].implementation = function () {
    console.log(`RootBeer.checkForBusyBoxBinary is called`);
    let result = this["checkForBusyBoxBinary"]();
    console.log(`RootBeer.checkForBusyBoxBinary result=${result}`);
    return result;
    }


    let RootBeer3 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkSuExists"].implementation = function () {
    console.log(`RootBeer.checkSuExists is called`);
    let result = this["checkSuExists"]();
    console.log(`RootBeer.checkSuExists result=${result}`);
    return result;
    }
});

```

각 함수를 frida 스니펫으로 복사를 완료 했다.
소스코드를 살펴 보면 return 값으로 result를 반환 하는 것을 볼 수있고, 각 함수는 boolean형 함수이다.
그렇기에 return 값은 true, false 이다.
여기서 result에 값을 받아올 때 앞에 '!' 를 붙혀 not연산을 하여 반대로 바꿔주면 될것이다.

```

// beer.js
Java.perform(function(){
    let RootBeer1 = Java.use("com.scottyab.rootbeer.RootBeer");
    RootBeer["checkForSuBinary"].implementation = function () {
    console.log(`RootBeer.checkForSuBinary is called`);
    let result = !this["checkForSuBinary"]();
    console.log(`RootBeer.checkForSuBinary result=${result}`);
    return result;
    }


    let RootBeer2 = Java.use("com.scottyab.rootbeer.과(Desktop cmd)

![image](https://github.com/user-attachments/assets/0490dc2e-957b-4cf3-a547-f559538ba20b)


![image](https://github.com/user-attachments/assets/3e5d9a3e-79ab-4649-bb4f-a5cc4a74a16c)

![image](https://github.com/user-attachments/assets/221e088c-30d3-4b43-bab1-42e8f410fa48)

![image](https://github.com/user-attachments/assets/931fe8ca-e7d8-4148-a329-3ac129dae738)

cmd 콘솔의 로그를 보면 알 수 있듯이 루팅 했냐? false를 반환하면서 루팅 탐지에 우회 할 수 있었다.
