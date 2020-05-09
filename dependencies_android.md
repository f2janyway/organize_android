
    dataBinding {
        enabled true
    }we
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    // For Kotlin projects
    kotlinOptions {
        jvmTarget = "1.8"
    }


    <!-- dagger -->
    implementation 'com.google.dagger:dagger-android:2.27'
    implementation 'com.google.dagger:dagger-android-support:2.27' // if you use the support libraries
    kapt 'com.google.dagger:dagger-android-processor:2.27'
    kapt 'com.google.dagger:dagger-compiler:2.27'

    implementation "androidx.preference:preference-ktx:1.1.0"


    <!-- retrofit -->
    implementation 'com.squareup.retrofit2:retrofit:2.7.1'
    implementation 'com.squareup.retrofit2:converter-gson:2.7.1'

    <!-- coroutines -->
    implementation "org.jetbrains.kotlinx:kotlinx-coroutines-core:1.3.5"
    
    <!-- dispatchers.Main -->
    implementation("org.jetbrains.kotlinx:kotlinx-coroutines-android:1.3.2")  

    <!-- material  -->
    implementation 'com.google.android.material:material:1.2.0-alpha04'

    <!-- recyclerview -->
    implementation "androidx.recyclerview:recyclerview:1.1.0"
    implementation "androidx.recyclerview:recyclerview-selection:1.1.0-rc01"

    <!-- workmanager  -->
    implementation "androidx.work:work-runtime-ktx:2.3.2"

    <!-- google -->
    implementation 'com.google.android.gms:play-services-maps:17.0.0'
    implementation 'com.google.android.gms:play-services-location:17.0.0'

    implementation 'com.google.maps.android:android-maps-utils:0.6.2'

    
    implementation "androidx.lifecycle:lifecycle-extensions:2.2.0"
    <!-- room -->
    implementation "androidx.room:room-runtime:$room_version"
    kapt "androidx.room:room-compiler:$room_version"

    // optional - Kotlin Extensions and Coroutines support for Room
    implementation "androidx.room:room-ktx:2.2.3"
    <!-- viewmodel -->
    implementation "androidx.lifecycle:lifecycle-viewmodel-ktx:2.2.0"


    <!-- glide -->
    implementation 'com.github.bumptech.glide:glide:4.11.0'
    kapt 'com.github.bumptech.glide:compiler:4.11.0'

    //firebase
    implementation 'com.google.firebase:firebase-analytics:17.2.2'

    <!-- //fcm -->
    implementation 'com.google.firebase:firebase-messaging:20.1.0'

    <!-- espresso recyclerview -->
    androidTestImplementation 'com.android.support.test.espresso:espresso-contrib:3.0.2'

    <!-- flexbox -->
        implementation 'com.google.android:flexbox:2.0.1'


    <!-- navigation -->
    // Java language implementation
    implementation "androidx.navigation:navigation-fragment:$nav_version"
    implementation "androidx.navigation:navigation-ui:$nav_version"

    // Kotlin
    implementation "androidx.navigation:navigation-fragment-ktx:$nav_version"
    implementation "androidx.navigation:navigation-ui-ktx:$nav_version"

    // Dynamic Feature Module Support
    implementation "androidx.navigation:navigation-dynamic-features-fragment:$nav_version"

    // Testing Navigation
    androidTestImplementation "androidx.navigation:navigation-testing:$nav_version"
    <!-- navigation -->

    core 의 버젼을 올리면 안됨 이게 최대
    implementation 'com.google.zxing:core:3.3.0'
    implementation('com.journeyapps:zxing-android-embedded:4.1.0') { transitive = false }


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
    -dontobfuscate                              #난독화를 수행하지 않도록 함
    -keepattributes SoureFile,LineNumberTable   #소스파일, 라인 정보 유지
    -dontwarn okio.**
    -dontwarn retrofit2.**


```

private fun getHashKey() {
        var packageInfo: PackageInfo? = null
        try {
            packageInfo =
                packageManager.getPackageInfo(packageName, PackageManager.GET_SIGNATURES)
        } catch (e: PackageManager.NameNotFoundException) {
            e.printStackTrace()
        }
        if (packageInfo == null) Log.e("KeyHash", "KeyHash:null")
        for (signature in packageInfo!!.signatures) {
            try {
                val md: MessageDigest = MessageDigest.getInstance("SHA")
                md.update(signature.toByteArray())
                Log.d("KeyHash", Base64.encodeToString(md.digest(), Base64.DEFAULT))
            } catch (e: NoSuchAlgorithmException) {
                Log.e("KeyHash", "Unable to get MessageDigest. signature=$signature", e)
            }
        }
    }