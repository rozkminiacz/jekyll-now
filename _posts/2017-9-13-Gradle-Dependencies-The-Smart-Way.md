# Managing your dependencies the smart way.

* Standard build.gradle in Android projects. Overview where everything is.
* Defining variables (version codes)
* compile vs testCompile
* dependencies.gradle in separate file.
* compile something only in one flavor

## build.gradle
I bet your module level build.gradle file looks like this: 
```groovy
apply plugin: 'com.android.application'
apply plugin: 'me.tatarka.retrolambda'
apply plugin: 'realm-android'

android {
    compileSdkVersion 25
    buildToolsVersion "25.0.2"
    defaultConfig {
        applicationId "com.rozkmin.demoapp"
        minSdkVersion 19
        targetSdkVersion 25
        versionCode 1
        versionName "1.0"
        testInstrumentationRunner "android.support.test.runner.AndroidJUnitRunner"
    }
    buildTypes {
        debug {

        }
        release {
            minifyEnabled true
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
        }
    }
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    
    productFlavors {
               freeFlavor {
                       applicationId "com.rozkmin.demoappfree"
                       resValue("string", "app_name", "DemoAppFree")
                       versionName "1.20.2"
               }
               premiumFlavor{
                       applicationId "com.rozkmin.demoapppremium"
                       resValue("string", "app_name", "DemoAppPremium")
                       versionName "1.20.2" 
                   }
               }
           }

dependencies {
    dependencies {
        compile fileTree(dir: 'libs', include: ['*.jar'])
        androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
            exclude group: 'com.android.support', module: 'support-annotations'
        })
        testCompile 'junit:junit:4.12'
        compile 'com.android.support:appcompat-v7:24.2.1'
        compile 'com.android.support:design:24.2.1'
        compile 'com.google.code.gson:gson:2.6.2'
        compile 'com.squareup.retrofit2:retrofit:2.1.0'
        compile 'com.squareup.retrofit2:converter-gson:2.1.0'
        compile 'com.android.support:support-v4:24.2.1'
        testCompile 'junit:junit:4.12'
        apt 'com.jakewharton:butterknife-compiler:8.4.0'
    }


}

```

In short words - a lot of **ugly** dependencies. and sometimes comments what does specific library do. 

Forget it. In this post we will create human readable and developer friendly build.gradle for your next app.

## Defining variables

First that we will do to make this code clean will be introducing variables.

Write 
```groovy
    def googlePlayVersion = '1.20.2' 
```

Now we can use it in our build file like that:

```groovy
    productFlavors {
               freeFlavor {
                       applicationId "com.rozkmin.demoappfree"
                       resValue("string", "app_name", "DemoAppFree")
                       versionName "$googlePlayVersion"
                       }
                
               premiumFlavor {
                       applicationId "com.rozkmin.demoapppremium"
                       resValue("string", "app_name", "DemoAppPremium")
                       versionName "$googlePlayVersion"
                       }
               }
```

Now let's do something more tricky. Create file dependencies.gradle in your project directory:

```groovy
ext {
    def RxJava2Version = '2.1.1'
    def RxAndroid2Version = '2.0.1'
    def Retrofit2Version = '2.3.0'
    def OkHttp3Version = '3.8.0'

    rxDependencies = [
                   rxJava : "io.reactivex.rxjava2:rxjava:${RxJava2Version}",
                   rxAndroid : "io.reactivex.rxjava2:rxandroid:${RxAndroid2Version}",
    ]

    networkDependencies = [
                   retrofit: "com.squareup.retrofit2:retrofit:${Retrofit2Version}",
                   okhttp  : "com.squareup.okhttp3:okhttp:${OkHttp3Version}",
                   converterGson : "com.squareup.retrofit2:converter-gson:${Retrofit2Version}",
                   adapterRxJava2 : "com.squareup.retrofit2:adapter-rxjava2:${Retrofit2Version}"
    ]
}
```

Now in your build top-level build file write **apply from: 'dependencies.gradle'**

```groovy
buildscript {
    repositories {
        jcenter()
        google()

    }
    dependencies {
        classpath 'me.tatarka:gradle-retrolambda:3.2.5'
        classpath 'com.android.tools.build:gradle:2.3.3'
        classpath "io.realm:realm-gradle-plugin:2.2.0"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
        classpath 'com.google.gms:google-services:3.0.0'
    }
}

allprojects {
    repositories {
        jcenter()
        maven { url "https://jitpack.io" }
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

apply from: 'dependencies.gradle'
```

