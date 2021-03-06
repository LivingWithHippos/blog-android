---
title: content
media_order: 'progress_determinate_01.webm,progressbar_determinate_secondary.webm,progressbar_determinate_shape.webm,progressbar_alternative_01.webm'
menu: Content
---

<div id="intro"/>
## Why Vectors?

Vectors can scale correctly at any size and are perfect for simple images like logos and symbols around your app, and data binding makes adding functionalities to Views easy.
We will be creating a vertical determinate progress bar easily adaptable to our needs.

!!! [[fa icon=fa-android extras=fab /] Android Official Page on Vector Drawable](https://developer.android.com/guide/topics/graphics/vector-drawable-resources)

!!! [[fa icon=fa-android extras=fab /] Android Official Page on Progress Bars](https://developer.android.com/reference/android/widget/ProgressBar)

<div id="setup"/>
### Setup

Add Data Binding to your app's `build.gradle` (see [here](https://developer.android.com/topic/libraries/data-binding/start)). It should be like this:

```gradle
android {
    ...
    buildFeatures {
        dataBinding true
    }
}

```

I set minSDK to 24 because it adds some vectors stuff, but I think it could be set as low as 21.

Add the vector to your `res/drawable` folder. Right click on it -> New -> Vector Asset -> choose "local file" and pick your file (`icon_arrows` in this example).

I recommend [Inkscape](https://inkscape.org/) to create and edit vectors.

<div id="build"/>
### ProgressBar XML file

- A progress bar has different Drawables set for every state: progress, secondary progress, background. The determinate and Indeterminate Drawables are also separated. 

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

Let's take a look at the code:

```xml
 android:drawable="@drawable/icon_arrows"
```

My drawable, replace it with yours. The same one is used on all of the code because the bottom one ("unprogressed" or background) will always be visible. If you use another image, it won't get hidden correctly by the progress unless they're the same shape/ progressively bigger/ you're showing both on purpose, etc. (there's an example down here).

```xml
<item android:id="@android:id/background"
     ...
    android:id="@android:id/secondaryProgress"
     ...
    android:id="@android:id/progress" >
```

We need to set the correct id in the right order for every item:

```xml
<clip
      android:clipOrientation="vertical"
      android:drawable="@drawable/icon_arrows"
      android:gravity="bottom" />
```

> A drawable defined in XML that clips another drawable based on this Drawable's current level.

[clip](https://developer.android.com/guide/topics/resources/drawable-resource#Clip) will show our partial progress on top of the background image.

! [fa icon=fa-bug extras=fas /] Tip: vertical progress bars are evil, and Android does not support them, but using these parameters you can easily create one

### Manage colors with Kotlin's Extensions and Databinding

Kotlin Extensions are ***cool*** and you can do a lot of stuff with them. In this case we can create a new xml parameter for our progress bar to set its progression color. Create a new `Extension.kt` file if you don't have one already and copy-paste this:

```kotlin
@BindingAdapter("progressColor")
fun ProgressBar.setProgressColor(color: Int) {
    tintDrawable(android.R.id.progress, color)
}

fun ProgressBar.getLayerDrawable(): LayerDrawable {
    return (if (isIndeterminate) indeterminateDrawable else progressDrawable)  as LayerDrawable
}

fun ProgressBar.getDrawableByLayerId(id: Int): Drawable {
    return getLayerDrawable().findDrawableByLayerId(id)
}

fun ProgressBar.tintDrawable(layerId: Int, color: Int) {
    val progressDrawable = getDrawableByLayerId(layerId).mutate()
    progressDrawable.setTint(color)
}
```


`@BindingAdapter("progressColor")`: this annotation will process `progressColor` (you can change it to another tag) when found in a progress bar XML and execute the code
    
`mutate()` will avoid editing the color of every instance of the drawable, it's needed because in this case we're using the same one three times.

`getLayerDrawable()` will return the correct determinate or indeterminate layer list

! [fa icon=clipboard-list extras=fas /] todo: check if this could also be a [StateList](https://developer.android.com/guide/topics/resources/drawable-resource#StateList) or [LevelList](https://developer.android.com/guide/topics/resources/drawable-resource#LevelList) or a [TransitionDrawable](https://developer.android.com/guide/topics/resources/drawable-resource#Transition)

`getDrawableByLayerId(id: Int)` will return a reference to a Drawable in the layer list

This extension is useful because there's no easy way to change the Drawables' colors in the layer list.

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

Using our progress bar extensions we can add new functionalities easily:
    
Secondary progress bar color:

```kotlin
@BindingAdapter("secondaryProgressColor")
fun ProgressBar.setSecondaryProgressColor(color: Int) {
    tintDrawable(android.R.id.secondaryProgress, color)
}
```
    
```xml
    <ProgressBar...
            app:secondaryProgressColor="@{@drawable/icon_hexagon}" />
```
  
[center] ![determinate progress bar result](progressbar_determinate_secondary.webm?resize=400) [/center]

#### Swap a drawable

We can use this function to swap a drawable with another. Since we're only swapping it, all the scaling/rotation settings will be kept.

```kotlin
fun ProgressBar.swapLayerDrawable(layerId: Int, drawable: Drawable) {
    when (val oldDrawable = getDrawableByLayerId(layerId)) {
        is ClipDrawable -> oldDrawable.drawable = drawable
        is ScaleDrawable -> oldDrawable.drawable = drawable
        is InsetDrawable -> oldDrawable.drawable = drawable
        is RotateDrawable -> oldDrawable.drawable = drawable
        is VectorDrawable -> getLayerDrawable().setDrawableByLayerId(layerId, drawable)
        // ShapeDrawable is a generic shape and does not have drawables
        // is ShapeDrawable ->
    }
}
```  
The power of [smart casting!](https://kotlinlang.org/docs/reference/typecasts.html)


!!! [fa icon=fa-bell extras=far /] swapping Drawable not working? Debug swapLayerDrawable and check which class oldDrawable belongs to

Let's try swapping our main progress Drawable: 

```kotlin
@BindingAdapter("primaryProgressDrawable")
fun ProgressBar.setPrimaryProgressDrawable(drawable: Drawable) {
    setLayerDrawable(android.R.id.progress, drawable)
}
```
 
```xml
    <ProgressBar...
	app:primaryProgressDrawable="@{@drawable/icon_hexagon}" />
```
    
 **Important:** change the color **after** changing the drawable; otherwise, it will get overwritten.
    
[center] ![determinate progress bar result](progressbar_determinate_shape.webm?resize=400) [/center]
    
As you can see from the video, we need to be careful of the vectors' shape, as the hexagon was taller than the triangles, and it looked like it started sooner (they had the same progress number).
    
[details="Complete extension list"]
    
```kotlin
    
@BindingAdapter("backgroundProgressColor")
fun ProgressBar.setBackgroundProgressColor(color: Int) {
    tintDrawable(android.R.id.background, color)
}

@BindingAdapter("progressColor")
fun ProgressBar.setProgressColor(color: Int) {
    tintDrawable(android.R.id.progress, color)
}

@BindingAdapter("secondaryProgressColor")
fun ProgressBar.setSecondaryProgressColor(color: Int) {
    tintDrawable(android.R.id.secondaryProgress, color)
}

@BindingAdapter("backgroundProgressDrawable")
fun ProgressBar.setBackgroundProgressDrawable(drawable: Drawable) {
    swapLayerDrawable(android.R.id.background, drawable)
}

@BindingAdapter("primaryProgressDrawable")
fun ProgressBar.setPrimaryProgressDrawable(drawable: Drawable) {
    swapLayerDrawable(android.R.id.progress, drawable)
}

@BindingAdapter("secondaryProgressDrawable")
fun ProgressBar.setSecondaryProgressDrawable(drawable: Drawable) {
    swapLayerDrawable(android.R.id.secondaryProgress, drawable)
}

fun ProgressBar.tintDrawable(layerId: Int, color: Int) {
    val progressDrawable = getDrawableByLayerId(layerId).mutate()
    progressDrawable.setTint(color)
}

fun ProgressBar.swapLayerDrawable(layerId: Int, drawable: Drawable) {
    when (val oldDrawable = getDrawableByLayerId(layerId)) {
        is ClipDrawable -> oldDrawable.drawable = drawable
        is ScaleDrawable -> oldDrawable.drawable = drawable
        is InsetDrawable -> oldDrawable.drawable = drawable
        is RotateDrawable -> oldDrawable.drawable = drawable
        is VectorDrawable -> getLayerDrawable().setDrawableByLayerId(layerId, drawable)
        // ShapeDrawable is a generic shape and does not have drawables
        // is ShapeDrawable ->
    }
}

fun ProgressBar.getLayerDrawable(): LayerDrawable {
    return (if (isIndeterminate) indeterminateDrawable else progressDrawable)  as LayerDrawable
}

fun ProgressBar.getDrawableByLayerId(id: Int): Drawable {
    return getLayerDrawable().findDrawableByLayerId(id)
}
    
```
    
[/details]
    

<br>
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

<div id="links"/>
#### Useful Links

Vectors for your app:

    
!!!! [fa icon=fa-images extras=fas /] [Icon Monstr](https://iconmonstr.com/) 
    
!!!! [fa icon=fa-images extras=far /] [SVG Repo](https://www.svgrepo.com/) 
    
!!!! [fa icon=fa-images extras=fas /] [Icons Repo](https://iconsrepo.com/) 
    
!!!! [fa icon=fa-images extras=far /] [Game Icons](https://game-icons.net/) 
