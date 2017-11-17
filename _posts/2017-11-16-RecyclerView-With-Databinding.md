Databinding in RecyclerView + Kotlin

![alt text](/_posts/device-2017-11-17-184448.png)

## Why use it?
Get rid of boilerplate code. 
How many times setText is invoked in your onBindViewHolder? 
For complex resimilartems it can be really annoying. 
You have several lines to set texts, images etc. 
With databinding we will try to shorten these stuff.

## Setup
Frist, enable databinding in module-level build.gradle:
```groovy
android{
    ....
    dataBinding {
            enabled = true
        }
}
```

Also add databinding annotation processor
```groovy
dependencies{
    compile 'com.android.databinding:compiler:${android_plugin_version}'
}

```

## RecyclerView item ViewModel

It won't be real MVVM pattern at the time. 
Consider ViewModel as structure with all data you display. 
It will be very similar to your API pojos or database entities.
So what's the difference? 

In our API we have users first and last name in separate fields. 
Maybe it's convenient for backend database, 
maybe <any other reason why backend guys structured responses that way>. 
But look - we always display first and last name i one textview! 
So our ViewModel should provide already concatenated strings to put in that field. 
Another example - we display user position in list. API serves us it as int. For our local database it should also be Integer. 
But what if we accidentally set int value to textview? Android framework will try to find it in R.strings class. And probably won't find it and / or crashes. It would be nice to have position a String in our ViewModel. 

At this point we should have ItemViewModel class created. Now let's do real databinding. 

## Using data in XML
```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:tools="http://schemas.android.com/tools">

    <data>
        <variable
            name="item"
            type="com.rozkmin.recyclerdatabindingshowcase.ItemViewModel"/>
    </data>

    <FrameLayout xmlns:android="http://schemas.android.com/apk/res/android"
        android:orientation="horizontal" android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@{item.me ? @drawable/item_background_highlight : @drawable/item_background_normal}"
        tools:background="@drawable/item_background_highlight"
        android:layout_margin="12dp"
        android:padding="8dp">

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/white"
            android:layout_gravity="start|center"
            android:text="@{item.position}"
            android:textAppearance="@style/TextAppearance.AppCompat.Body2"
            tools:text="1."/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/white"
            android:layout_gravity="center"
            android:text="@{item.name}"
            android:textAppearance="@style/TextAppearance.AppCompat.Body1"
            tools:text="Edward Dolański"/>

        <TextView
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:textColor="@color/white"
            android:layout_gravity="end|center"
            android:text="@{item.team}"
            android:textAppearance="@style/TextAppearance.AppCompat.Body1"
            tools:text="Super Team"/>

    </FrameLayout>
</layout>
```

As you see each ViewModel property was assigned in xml with @{item.property}. We also provided <data/> and <variable>. 

Interesting thing is:

```xml
android:background="@{item.me ? @drawable/item_background_highlight : @drawable/item_background_normal}"
```

It is possible to create simple logic in layouts. Way more readable than writing several switches in RecyclerView.Adapter class

## Bind data
```kotlin
class ItemAdapter : RecyclerView.Adapter<ItemAdapter.ItemViewHolder>() {

    var items = listOf<ItemViewModel>()
        set(value) {
            field = value
            notifyDataSetChanged()
        }

    override fun onBindViewHolder(holder: ItemViewHolder, position: Int) =
     holder.bind(items[position])

    override fun onCreateViewHolder(parent: ViewGroup?, viewType: Int): ItemViewHolder =
            ItemViewHolder(ItemBinding.inflate(LayoutInflater.from(parent?.context), parent, false))

    override fun getItemCount(): Int = items.size

    inner class ItemViewHolder(private val itemBinding: ItemBinding) : RecyclerView.ViewHolder(itemBinding.root) {
        fun bind(itemViewModel: ItemViewModel) {
            itemBinding.item = itemViewModel
            itemBinding.executePendingBindings()
        }
    }
}
```

We create ItemViewHolder with ItemBinding - class generated from item.xml. If you don't see binding class in your classpath, try to rebuild project.
Interesting thing is happening in method bind():

```kotlin
fun bind(itemViewModel: ItemViewModel) {
            itemBinding.item = itemViewModel //assign our item instance to generated class
            itemBinding.executePendingBindings() //similar to notifyDataSetChanged()
        }
```

## Summary
