## We'll try to do the following thing:
* cwite an app that shows random image from lorempixel
* create two flavors
* use Picasso in first flavor, in second use Glide
* keep Glide and Picasso imports far away from main code
* provide additional layer of abstraction for image handling
* @Inject image provider
* keep it clean and simple

## First of all - create and setup project.

### Configure flavors

The simplest solution is to add productFlavor block to your build.gradle, but you can also generate it from gui.

I added to my build.gradle the following:

```groovy
android {
    /**/
    flavorDimensions "default"
    
        productFlavors {
            glide {
                versionName defaultConfig.versionName + ".glide." + defaultConfig.versionCode
                applicationId "com.rozkmin.flavorshowcaseglide"
                dimension "default"
    
            }
            picasso {
                versionName defaultConfig.versionName + ".picasso." + defaultConfig.versionCode
                applicationId "com.rozkmin.flavorshowcasepicasso"
                dimension "default"
            }
        }
}
```

### Add dependencies

### Setup Glide and Picasso in separate flavors.

```groovy
    picassoCompile 'com.squareup.picasso:picasso:2.5.2'
    glideCompile 'com.github.bumptech.glide:glide:4.1.1'

    compile 'com.google.dagger:dagger:2.2'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.2'
```

{flavorName}Compile is generated for all build variant. You can also use:
```groovy
    picassoDebugCompile 'com.squareup.picasso:picasso:2.5.2'
    picassoReleaseCompile 'com.squareup.picasso:picasso:2.5.2'
        
```



## Make our view with image and text

### MainActivity
```xml
<?xml version="1.0" encoding="utf-8"?>
<FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="com.rozkmin.flavorshowcase.MainActivity">

    <ImageView
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:id="@+id/activity_main_image"/>

    <TextView
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_gravity="center"
        android:background="#8fffffff"
        android:textColor="#aa595959"
        android:textSize="40sp"
        style="@style/Base.Theme.AppCompat"
        android:id="@+id/activity_main_flavor"
        tools:text="Loaded with Glide"/>

</FrameLayout>

```

## Write some interfaces

Lets create our interface and @Inject it to our MainActivity

### The callback way

### The reactive way

## Implementation time

