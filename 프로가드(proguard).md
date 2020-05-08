2020-05-08
===
## 나중에 하자
```
<!-- app build.gradle  proguard setting basic -->
buildTypes {
        release {
            minifyEnabled true

            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
        debug {
            minifyEnabled true

            proguardFile getDefaultProguardFile("proguard-android.txt")

            proguardFile 'proguard-rules.pro'
            proguardFile 'proguard-debug.pro'
        }
    }
<!-- in proguard-rules.pro -->
```


in proguard-rules.pro
```
-dontobfuscate                              #난독화를 수행하지 않도록 함
-keepattributes SoureFile,LineNumberTable   #소스파일, 라인 정보 유지
-dontwarn okio.**
-dontwarn retrofit2.**
```

