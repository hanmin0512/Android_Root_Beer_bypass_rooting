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
