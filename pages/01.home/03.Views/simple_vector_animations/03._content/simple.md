---
title: content
media_order: 'paths_fade_01.webm,fade_01.webm,path_colors_01.webm'
menu: Content
---

<div id="intro"/>
## Vector Animations

We can use Vectors paired with AnimatedVectorDrawable to create beautiful animation easily.

!!! [[fa icon=fa-android extras=fab /] Android Official Page on AnimatedVectorDrawable](https://developer.android.com/reference/android/graphics/drawable/AnimatedVectorDrawable)

<div id="setup"/>
### Setup

Data binding is optional, but it makes adding XML attributes simlpe. Add Data Binding to your app's `build.gradle` (see [here](https://developer.android.com/topic/libraries/data-binding/start)). It should be like this:

```gradle
android {
    ...
    buildFeatures {
        dataBinding true
    }
}

```

Add the vector to your _res/drawable_ folder. Right click on it -> New -> Vector Asset -> choose "local file" and pick your file (`icon_crown` in this example).

[details="XML vector if anyone is interested"]

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="30dp"
    android:height="30dp"
    android:viewportWidth="30"
    android:viewportHeight="30">
  <path
      android:pathData="M28.1798,25.3804H1.82c-0.2436,0 -0.441,-0.1972 -0.441,-0.441 0,-0.2434 0.1973,-0.441 0.441,-0.441H27.772L28.9118,9.9312 20.6261,14.7322c-0.2113,0.1228 -0.4812,0.0504 -0.6029,-0.1605 -0.1221,-0.211 -0.0501,-0.4801 0.1605,-0.6029l9.0121,-5.2215c0.1422,-0.0819 0.3175,-0.0783 0.4561,0.008 0.1383,0.0875 0.2173,0.2445 0.2046,0.4078L28.6194,24.9732c-0.018,0.2297 -0.2095,0.4068 -0.4396,0.4068"
      android:strokeWidth="0.0352777"
      android:fillColor="#100f0d"
      android:strokeColor="#00000000"
      android:fillType="nonZero"/>
  <path
      android:pathData="M28.1798,25.3804H1.82c-0.2303,0 -0.4217,-0.1771 -0.4396,-0.4068L0.1435,9.1636c-0.0127,-0.1633 0.0661,-0.3203 0.2046,-0.4078 0.1382,-0.0875 0.3139,-0.091 0.4563,-0.008L18.0025,18.7124c0.2106,0.1228 0.2826,0.3923 0.1605,0.6029 -0.1224,0.211 -0.3926,0.2826 -0.6029,0.1605L1.0883,9.9317 2.2277,24.499H28.1798c0.2434,0 0.441,0.1976 0.441,0.441 0,0.2438 -0.1975,0.441 -0.441,0.441"
      android:strokeWidth="0.0352777"
      android:fillColor="#100f0d"
      android:strokeColor="#00000000"
      android:fillType="nonZero"/>
  <path
      android:pathData="m20.3026,14.7915c-0.1133,0 -0.2258,-0.0427 -0.3119,-0.1288l-4.9911,-4.9915 -4.9907,4.9915c-0.1722,0.1722 -0.4512,0.1722 -0.6237,0 -0.1722,-0.1725 -0.1722,-0.4516 0,-0.6237L14.6882,8.7364c0.1722,-0.1722 0.4512,-0.1722 0.6234,0l5.3026,5.3026c0.1725,0.1722 0.1725,0.4512 0,0.6237 -0.0861,0.0861 -0.1986,0.1288 -0.3115,0.1288"
      android:strokeWidth="0.0352777"
      android:fillColor="#100f0d"
      android:strokeColor="#00000000"
      android:fillType="nonZero"/>
</vector>
```

[/details]


<div id="build"/>
### Animation XML files

1. Create a file under _res/animator_

`fade_loop_vector.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android">
    <objectAnimator
        android:duration="1000"
        android:propertyName="alpha"
        android:valueFrom="1.0f"
        android:valueTo="0.0f"
        android:repeatMode="reverse"
        android:repeatCount="infinite"  />
</set>
```
The code is pretty self-explanatory: 

`android:duration="1000"` is the animation duration in milliseconds

`android:propertyName="alpha"` is the property that is going to be changed (important)

`android:valueFrom` and `android:valueTo` are what they sound like

`android:repeatMode="reverse"` and `android:repeatCount="infinite"` are used to loop our animation from start to finish and then from finish to start forever

You can find a list of animable attributes for VectorDrawable in a table under [here.](https://developer.android.com/reference/android/graphics/drawable/AnimatedVectorDrawable#xml-for-the-vectordrawable-containing-properties-to-be-animated)
<br>
2. Create a file under _res/drawable_

`download_loading.xml`

```xml
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/icon_crown" >
    <target
        android:name="mainVector"
        android:animation="@animator/fade_loop_vector" />
</animated-vector>
```


`<target` is the interesting part: 

- `android:name="mainVector"` is telling the animation to search in the drawable for the mainVector name
- `android:animation="@animator/fade_loop_vector"` will apply the properties to this target

<br>
3. Add the target name to the vector xml.

Since the vector is just an XML file, we can open it and add our name to the correct target. In this case

`icon_crown.xml`

```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:width="30dp"
    android:height="30dp"
    android:viewportWidth="30"
    android:viewportHeight="30"
    android:name="mainVector">
```
<br>
4. Add it to the layout and start the animation


```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/rootLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">


        <ImageView
            android:id="@+id/ivLoading"
            android:layout_width="200dp"
            android:layout_height="200dp"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:srcCompat="@drawable/download_loading"
            app:startAnimation="@{true}"/>

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>

```

`app:startAnimation="@{true}"` normally you have to get a reference in your code to the ImageView to start the animation, but here we'll be using data binding once again.

Create a new Kotlin extension:

```kotlin
@BindingAdapter("startAnimation")
fun ImageView.startAnimation(start: Boolean) {
    if (drawable is Animatable) {
        if (start)
            (drawable as Animatable).start()
        else
            (drawable as Animatable).stop()
    }
}
```

If you don't use Kotlin/extensions/data binding get a reference to your ImageView from your fragment/activity class and adapt the function above for ImageView.getDrawable().

[center] ![fade effect](fade_01.webm?resize=400) [/center]

<div id="addproperties"/>
### Adding Other Features

#### Partial path effects

Thanks to the `android:name` attribute, it's easy to operate on single paths. 

1. Go back to the vector xml and add names where needed.

`icon_crown.xml`

```xml
<path
      ...
        android:name="rightSide"
        /> 
<path
      ...
      android:name="leftSide"
      />
<path
      ...
      android:name="middle"
      />
```

2. Create a new _res/animator_ file (one per animation) and add your animator code.

`fade_loop_path_2.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    >
    <objectAnimator
        android:duration="2000"
        android:propertyName="fillAlpha"
        android:valueFrom="1"
        android:valueTo="0"
        android:repeatMode="reverse"
        android:repeatCount="infinite"  />
</set>
```

The only difference from before is the `android:propertyName="fillAlpha"` instead of `alpha`, which goes from 0 to 1 (no float).

3. Create a new Drawable to link the animator to the vector. `animated_crown.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<animated-vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:drawable="@drawable/icon_crown">
    <target
        android:name="leftSide"
        android:animation="@animator/fade_loop_path_1" />
    <target
        android:name="rightSide"
        android:animation="@animator/fade_loop_path_2" />
    <target
        android:name="middle"
        android:animation="@animator/fade_loop_path_3" />
</animated-vector>
```

Add it to your ImageView as seen before, and it's ready

[center] ![multiple fade effects](paths_fade_01.webm?resize=400) [/center]

#### Path colors

Add an objectAnimator to your animator and pick the fillColor property to get a gorgeous-looking effect.

`fade_loop_path_1.xml`:

```xml
<?xml version="1.0" encoding="utf-8"?>
<set xmlns:android="http://schemas.android.com/apk/res/android"
    >
    <objectAnimator
        android:duration="2000"
        android:propertyName="fillAlpha"
        android:valueFrom="1"
        android:valueTo="0"
        android:repeatMode="reverse"
        android:repeatCount="infinite"  />

    <objectAnimator
        android:duration="3000"
        android:propertyName="fillColor"
        android:valueFrom="#C7F464"
        android:valueTo="#FF6B6B"
        android:repeatMode="reverse"
        android:repeatCount="infinite"  />
</set>
```

[center] ![colorful paths!](path_colors_01.webm?resize=400) [/center] 

<div id="links"/>
#### Useful Links

!!!! [[fa icon=fa-palette extras=fas /] Palettes from ColoursLovers](https://www.colourlovers.com/palettes) 

!!!! [[fa icon=fa-android extras=fas /] Android documentation on object-animator](https://developer.android.com/guide/topics/graphics/prop-animation#object-animator) 

!!!! [[fa icon=fa-android extras=fas /] Motionlayout: The new animation library](https://developer.android.com/training/constraint-layout/motionlayout) 