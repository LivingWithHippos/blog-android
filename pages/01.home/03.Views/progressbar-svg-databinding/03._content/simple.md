---
title: content
media_order: 'progress_determinate_01.webm,progressbar_determinate_secondary.webm,progressbar_determinate_shape.webm,progressbar_alternative_01.webm'
menu: Content
---

<div id="intro"/>
## Why Vectors?

Vectors can scale perfectly at any size and are perfect for simple images like logos and symbols around your app, and data binding makes adding functionalities to Views really easy.
We will be creating a vertical determinate progress bar easily adaptable to our needs.

!!! [[fa=fa-android /] Android Official Page on Vector Drawable](https://developer.android.com/guide/topics/graphics/vector-drawable-resources)

!!! [[fa=fa-android /] Android Official Page on Progress Bars](https://developer.android.com/reference/android/widget/ProgressBar)

<div id="setup"/>
#### The Setup

Add Data Binding to your app's `build.gradle` (see [here](https://developer.android.com/topic/libraries/data-binding/start)). It should be like this:

```gradle
android {
    ...
    buildFeatures {
        dataBinding true
    }
}

```

I set minSDK to 24 because it adds some vectors stuff, I think it can be set as low as 21.

Add the vector to your `res/drawawble` folder. Right click on it -> New -> Vector Asset -> choose "local file" and pick your file (`icon_arrows` in this example).

I personally recommend [Inkscape](https://inkscape.org/) to create and edit vectors.

<div id="build"/>
### ProgressBar XML file

- A progress bar has different Drawables set for every state: progress, secondary progress, "unprogressed". The determinate and Indeterminate Drawables are also separated. 

They can be defined in a single file: create a new Drawable (`download-progressbar.xml` here) and copy-paste this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@android:id/background"
        android:drawable="@drawable/icon_arrows">

    </item>
    <item android:id="@android:id/secondaryProgress">
        <clip
            android:clipOrientation="vertical"
            android:drawable="@drawable/icon_arrows"
            android:gravity="bottom" />
    </item>
    <item android:id="@android:id/progress">
        <clip
            android:clipOrientation="vertical"
            android:drawable="@drawable/icon_arrows"
            android:gravity="bottom" />
    </item>
</layer-list>
```
From the [documentation](https://developer.android.com/guide/topics/resources/drawable-resource#LayerList)

> A LayerDrawable is a drawable object that manages an array of other drawables. Each drawable in the list is drawn in the order of the list—the last drawable in the list is drawn on top.
> 
> Each drawable is represented by an <item> element inside a single <layer-list> element.

Let's take a look at the code

```xml
 android:drawable="@drawable/icon_arrows"
```

My vector, replace it with yours. The same one is used on all of the code because the bottom one ("unprogressed" or background) will always be visible. If you use another image it won't get hidden correctly by the progress, unless they're exactly the same shape/ progressively bigger/ you're showing both on purpose etc.

```xml
<item android:id="@android:id/background"
     ...
    android:id="@android:id/secondaryProgress"
     ...
    android:id="@android:id/progress" >
```

We need to set the correct id in the correct order for every item, to get it recognized by the system

```xml
<clip
      android:clipOrientation="vertical"
      android:drawable="@drawable/icon_arrows"
      android:gravity="bottom" />
```

> A drawable defined in XML that clips another drawable based on this Drawable's current level.

[clip](https://developer.android.com/guide/topics/resources/drawable-resource#Clip) will show our partial progress on top of the background image.

! [fa=fa-android /] Tip: vertical progress bar are evil and Android does not support them, but using these parameters you can easily create one

### Manage colors with Kotlin's Extensions and Databinding

Kotlin Extensions are ***cool*** and you can do a lot of stuff with them. In this case we can create a new xml parameter for our progress bar to set its progression color. Create a new `Extension.kt` file if you don't have one already and copy-paste this:

```kotlin
@BindingAdapter("progressColor")
fun ProgressBar.setProgressColor(color: Int) {
    // get our layer list
    val progressBarLayers = progressDrawable as LayerDrawable
    // find the progress drawable
    val progressDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.progress).mutate()
    // set our color
    progressDrawable.setTint(color)
}
```


`@BindingAdapter("progressColor")`: this annotation will process `progressColor` (you can change it) when found in a progress bar xml and execute the code
    
`mutate()` will avoid editing the color of every instance of the drawable, it's needed because in this case we're using the same one three times


### Add the progress bar to the Layout

```xml
<?xml version="1.0" encoding="utf-8"?>
<layout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools">

    <data>
    </data>

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/rootLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent" >

        <ProgressBar
            android:id="@+id/progressBar"
            style="@style/Widget.AppCompat.ProgressBar.Horizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:indeterminate="false"
            android:max="100"
            android:minWidth="100dp"
            android:minHeight="100dp"
            android:progress="30"
            android:progressDrawable="@drawable/download_progressbar"
            app:progressColor="@{@color/streaming_background}"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintStart_toStartOf="parent" />

    </androidx.constraintlayout.widget.ConstraintLayout>
</layout>
```

`<layout ...`: remember to add data-binding to your layout!

`style="@style/Widget.AppCompat.ProgressBar.Horizontal"`: just use any `ProgressBar.*` style, we will be overriding most of it anyway.

`android:progress="50"`: you will probably use a custom value passed with data binding such as `android:progress="@{data.progress}"`

`app:progressColor="@{@color/green_pasture}"`: set you progress vector color here

**important:** parameters passed to the `BindingAdapter` needs to be written data-binding style, such as `@{@color/green_pasture}` or `@{@true}`

[center] ![determinate progress bar result](progress_determinate_01.webm?resize=400) [/center]

<div id="addproperties"/>
### Adding other properties

We can add new extensions easily:
    
* Secondary progress bar color

```kotlin
@BindingAdapter("secondaryProgressColor")
fun ProgressBar.setSecondaryProgressColor(color: Int) {
    val progressBarLayers = progressDrawable as LayerDrawable
    val progressDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.secondaryProgress).mutate()
    progressDrawable.setTint(color)
}
```
    
```xml
    <ProgressBar...
            app:primaryProgressDrawable="@{@drawable/icon_hexagon}" />
```
  
[center] ![determinate progress bar result](progressbar_determinate_secondary.webm?resize=400) [/center]
    
```kotlin
@BindingAdapter("primaryProgressDrawable")
fun ProgressBar.setPrimaryProgressDrawable(drawable: Drawable) {
    val progressBarLayers = this.progressDrawable as LayerDrawable
    val oldDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.progress)
    if (oldDrawable is ClipDrawable)
        oldDrawable.drawable = drawable
    else
        if (oldDrawable is ScaleDrawable)
            oldDrawable.drawable = drawable
}
 ```
```xml
    <ProgressBar...
	app:primaryProgressDrawable="@{@drawable/icon_hexagon}" />
```

We check for ScaleDrawable because that's what is used by the vanilla horizontal progressbar. If you don't set an `android:progressDrawable` your image will be stretched instead of clipped, and it will (probably) look strange.
    
 **Important:** change the color **after** changing the drawable, otherwise the color will get overwritten.
    
[center] ![determinate progress bar result](progressbar_determinate_shape.webm?resize=400) [/center]
    
As you can see from the video, we need to be careful of the vectors' shape, here we had an extra margin.
    
[details="Complete extension list"]
    
```kotlin
    
@BindingAdapter("backgroundProgressColor")
fun ProgressBar.setBackgroundProgressColor(color: Int) {
    val progressBarLayers = progressDrawable as LayerDrawable
    val progressDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.background).mutate()
    progressDrawable.setTint(color)
}

@BindingAdapter("progressColor")
fun ProgressBar.setProgressColor(color: Int) {
    val progressBarLayers = progressDrawable as LayerDrawable
    val progressDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.progress).mutate()
    progressDrawable.setTint(color)
}

@BindingAdapter("secondaryProgressColor")
fun ProgressBar.setSecondaryProgressColor(color: Int) {
    val progressBarLayers = progressDrawable as LayerDrawable
    val progressDrawable =
        progressBarLayers.findDrawableByLayerId(android.R.id.secondaryProgress).mutate()
    progressDrawable.setTint(color)
}

@BindingAdapter("backgroundProgressDrawable")
fun ProgressBar.setBackgroundProgressDrawable(drawable: Drawable) {
    val progressBarLayers = this.progressDrawable as LayerDrawable
    val oldDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.background)
    if (oldDrawable is ClipDrawable)
        oldDrawable.drawable = drawable
    else
        if (oldDrawable is ScaleDrawable)
            oldDrawable.drawable = drawable
}

@BindingAdapter("primaryProgressDrawable")
fun ProgressBar.setPrimaryProgressDrawable(drawable: Drawable) {
    val progressBarLayers = this.progressDrawable as LayerDrawable
    val oldDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.progress)
    if (oldDrawable is ClipDrawable)
        oldDrawable.drawable = drawable
    else
        if (oldDrawable is ScaleDrawable)
            oldDrawable.drawable = drawable
}

@BindingAdapter("secondaryProgressDrawable")
fun ProgressBar.setSecondaryProgressDrawable(drawable: Drawable) {
    val progressBarLayers = this.progressDrawable as LayerDrawable
    val oldDrawable = progressBarLayers.findDrawableByLayerId(android.R.id.secondaryProgress)
    if (oldDrawable is ClipDrawable)
        oldDrawable.drawable = drawable
    else
        if (oldDrawable is ScaleDrawable)
            oldDrawable.drawable = drawable
}
    
 ```
    
[/details]
    


Another complete example:
    
[center] ![determinate progress bar result](progressbar_alternative_01.webm?resize=400) [/center]
    
    
[details="And its code"]
  
download_progressbar.xml
    
 ```xml
    <?xml version="1.0" encoding="utf-8"?>
<layer-list xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@android:id/background"
        android:drawable="@drawable/icon_mountains">

    </item>
    <item android:id="@android:id/secondaryProgress">
        <clip
            android:clipOrientation="horizontal"
            android:drawable="@drawable/icon_mountains"
            android:gravity="right" />
    </item>
    <item android:id="@android:id/progress">
        <clip
            android:clipOrientation="vertical"
            android:drawable="@drawable/icon_mountains"
            android:gravity="bottom" />
    </item>
</layer-list>
 ```
    
fragment_layout.xml
    
 ```xml
    <?xml version="1.0" encoding="utf-8"?>
<layout xmlns:tools="http://schemas.android.com/tools"
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/rootLayout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">


        <ProgressBar
            android:id="@+id/progressBar"
            style="@style/Widget.AppCompat.ProgressBar.Horizontal"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_margin="20dp"
            android:indeterminate="false"
            android:max="100"
            android:minWidth="300dp"
            android:minHeight="300dp"
            android:progress="30"
            android:progressDrawable="@drawable/download_progressbar"
            app:layout_constraintBottom_toBottomOf="parent"
            app:layout_constraintEnd_toEndOf="parent"
            app:layout_constraintHorizontal_bias="0.5"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent"
            app:progressColor="@{@color/streaming_background}"
            app:secondaryProgressColor="@{@color/free_red}" />

    </androidx.constraintlayout.widget.ConstraintLayout>

</layout>

 ```
[/details]


#### Useful Links


!!!! [fa=fa-android /] [ListAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/**ListAdapter**) is a new class that requires less code and makes DiffUtils easier
  
!!!! [fa=fa-android /] [Jetpack Compose](https://developer.android.com/jetpack/compose/tutorial) will probably replace every part of the Android UI creation

!!!! [fa=fa-book /] There are also various third-party libraries, such as Airbnb [epoxy](https://github.com/airbnb/epoxy)

