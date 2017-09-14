## Manage your dependencies like a boss
* defining variables (version codes)
* custom compile configurations
* dependencies.gradle in separate file

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

### Automate naming apks on different flavors and configs:

```groovy
applicationVariants.all { variant ->
   variant.outputs.each { output ->
       def outputFile = output.outputFile
       def appName = "app-" + "${variant.baseName.replace("-release", "")}" + "_v" + "${variant.versionName}.apk"
       output.outputFile = new File(outputFile.parent, appName)
   }
}

```

## Keep dependencies in separate file
Now let's do something more tricky. Create file dependencies.gradle in your project directory:

```groovy
ext {
    def RxJava2Version = '2.1.1'
    def RxAndroid2Version = '2.0.1'
    def Retrofit2Version = '2.3.0'
    def OkHttp3Version = '3.8.0'

    rxDependencies = [
        rxJava    : "io.reactivex.rxjava2:rxjava:${RxJava2Version}",
        rxAndroid : "io.reactivex.rxjava2:rxandroid:${RxAndroid2Version}",
    ]

    networkDependencies = [
        retrofit         : "com.squareup.retrofit2:retrofit:${Retrofit2Version}",
        okhttp           : "com.squareup.okhttp3:okhttp:${OkHttp3Version}",
        converterGson    : "com.squareup.retrofit2:converter-gson:${Retrofit2Version}",
        adapterRxJava2   : "com.squareup.retrofit2:adapter-rxjava2:${Retrofit2Version}"
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
    }
}

allprojects {
    repositories {
        jcenter()
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

apply from: 'dependencies.gradle'
```

Now you can use it in your module level build.gradle file:
```groovy
dependencies {
    //other dependencies
    compile networkDependencies.values()
    compile rxDependencies.values()
    //other dependencies
}
```

I highly recommend to write these blocks for all your grouped libraries.
Now you build.gradle is much cleaner and easier to maintain. 
Its also great because you can keep.your favourite libraries in one file, keep it on some gist and just copy-paste it when creating new project. 
Much better than commenting and uncommenting specific lines.

## Compile configurations
You don't need jUnit or Mockito in you production code. If you use it, something went **very wrong**.

Gradle gives us some predefined configurations:

```groovy
testCompile unitTestDependencies.values()
androidTestCompile testDependencies.values()
```
It is set by default in android starter projects. 

### Custom configurations
Let’s say you have to build app for debug, other for your QA and build another one to show to your customers, 
and you want to include in it different version of your module. 
Based on flavor and build type following compile configs will be generated.

```groovy
freeFlavorReleaseCompile project(path: ':mylibrary', configuration: 'release')
freeFlavorQaCompile project(path: ':mylibrary', configuration: 'debug')
freeFlavorDebugCompile project(path: ':mylibrary', configuration: 'debug')

```


## Use environmental variables
```groovy
signingConfigs {
   release {
       storeFile file("my_secret_key.jks")
       keyAlias 'demoapp'
       storePassword "${System.env.PW_DA_KEYSTORE}"
       keyPassword "${System.env.PW_DA_KEYSTORE}"
   }
}

```

Use it for keeping your developer keys secret. You don't have to (and probably should not) keep your google 
play secrets on repo and share across your team. The same applies if you have some other secrets that you want to use only during build.