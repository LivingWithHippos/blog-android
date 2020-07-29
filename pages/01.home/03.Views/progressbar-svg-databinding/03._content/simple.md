---
title: content
media_order: progress_determinate_01.webm
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
<
    android:id="@android:id/background"
    android:id="@android:id/secondaryProgress"
    android:id="@android:id/progress"
>
```

We need to set the correct id for every item to get it recognized by the system

```xml
<clip
            android:clipOrientation="vertical"
            android:drawable="@drawable/icon_arrows"
            android:gravity="bottom" />
```

> A drawable defined in XML that clips another drawable based on this Drawable's current level.

[clip](https://developer.android.com/guide/topics/resources/drawable-resource#Clip) will show our partial progress on top of the background image.

!!! [fa=fa-android /] Tip: vertical progress bar are evil and Android does not support them, But using these parameters you can easily create one

### Manage colors with Kotlin's Extensions and Databinding

Kotlin Extensions are ***cool*** and you can do a lot of stuff with them. In this case we can create a new xml parameter for our progressbar to set the color. Create a new `Extension.kt` file and copy-paste this:

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

the `@BindingAdapter("progressColor")` annotation will process `progressColor` when found in a progress bar xml and execute the code
`.mutate()` will avoid editing the color of every instance of the drawable, it's needed because in this case we're using the same one three times


### Add the Progress Bar to Our Layout

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

`<layout ...`

remember to add data-binding to your layout!

`style="@style/Widget.AppCompat.ProgressBar.Horizontal"`

Just use any `ProgressBar.*` style, we will be overriding most of it anyway.

`android:progress="50"`

you will probably use a custom value passed with data binding such as `android:progress="@{data.progress}"`

`app:progressColor="@{@color/green_pasture}"`

**important:** parameters passed to the `BindingAdapter` needs to be written data-binding style, such as `@{@color/green_pasture}"` or `@{@true}`
Swap `green_pasture` with your own color.

![determinate progress bar result](progress_determinate_01.webm)

<div id="additems"/>
### Adding new items

We can add new, different layouts pretty easily:

1. create a new <i>list_item_layout.xml</i>
2. create a class implementing `MultiListItem`
3. create a new ViewHolder with the new layout's binding
4. Edit `onCreateViewHolder()` and `onBindViewHolder()` to link all of this


<div id="layouts"/>
#### XML Layouts

[details="first_item_layout.xml"]

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    xmlns:card_view="http://schemas.android.com/tools">

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/cvFirstItem"
        android:layout_width="320dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:layout_margin="10dp"
        card_view:cardElevation="5dp"
        card_view:cardCornerRadius="10dp"
        android:backgroundTint="@color/firstBackground"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_margin="10dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent">

            <ImageView
                android:id="@+id/ivItemPic"
                android:layout_width="50dp"
                android:layout_height="50dp"
                android:paddingEnd="10dp"
                android:tint="@color/firstPicTint"
                android:src="@drawable/comment"
                app:layout_constraintStart_toStartOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"/>


                <TextView
                    android:id="@+id/tvName"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:textColor="@color/firstTextColor"
                    android:textStyle="bold"
                    app:layout_constraintStart_toEndOf="@id/ivItemPic"
                    app:layout_constraintTop_toTopOf="@id/ivItemPic"
                    android:text="Name" />

                <ImageView
                    android:id="@+id/ivQuote"
                    android:layout_width="40dp"
                    android:layout_height="40dp"
                    android:scaleX="-1"
                    android:tint="@color/firstTextColor"
                    app:layout_constraintStart_toStartOf="@+id/tvName"
                    app:layout_constraintEnd_toEndOf="@id/tvName"
                    app:layout_constraintTop_toBottomOf="@+id/tvName"
                    android:src="@drawable/quote" />


            <TextView
                android:id="@+id/tvDescription"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_margin="10dp"
                app:layout_constraintStart_toEndOf="@+id/tvName"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintTop_toTopOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                android:textColor="@color/firstTextColor"
                android:inputType="textMultiLine"
                android:textAlignment="center"
                android:text="Quote" />

        </androidx.constraintlayout.widget.ConstraintLayout>
    </com.google.android.material.card.MaterialCardView>
</androidx.constraintlayout.widget.ConstraintLayout>

```
[/details]

First layout result:

![item layout](first_item_layout.png?resize=380)

[details="second_item_layout.xml"]

```xml
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    xmlns:card_view="http://schemas.android.com/tools">

    <com.google.android.material.card.MaterialCardView
        android:id="@+id/cvFirstItem"
        android:layout_width="320dp"
        android:layout_height="100dp"
        android:layout_gravity="center"
        android:layout_margin="10dp"
        card_view:cardElevation="5dp"
        card_view:cardCornerRadius="10dp"
        android:backgroundTint="@color/secondBackground"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent">

        <androidx.constraintlayout.widget.ConstraintLayout
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_margin="10dp"
            app:layout_constraintStart_toStartOf="parent"
            app:layout_constraintTop_toTopOf="parent">

            <ImageView
                android:id="@+id/ivItemPic"
                android:layout_width="60dp"
                android:layout_height="60dp"
                android:src="@drawable/brush"
                android:tint="@color/secondPicTint"
                app:layout_constraintEnd_toEndOf="parent"
                app:layout_constraintBottom_toBottomOf="parent"
                app:layout_constraintTop_toTopOf="parent" />


                <TextView
                    android:id="@+id/tvName"
                    android:layout_width="wrap_content"
                    android:layout_height="wrap_content"
                    android:textColor="@color/secondTextColor"
                    android:textStyle="bold"
                    android:text="Name"
                    app:layout_constraintTop_toTopOf="parent"
                    app:layout_constraintBottom_toBottomOf="parent"
                    app:layout_constraintStart_toStartOf="parent"
                    app:layout_constraintEnd_toStartOf="@id/ivItemPic"/>


        </androidx.constraintlayout.widget.ConstraintLayout>
    </com.google.android.material.card.MaterialCardView>
</androidx.constraintlayout.widget.ConstraintLayout>
```

[/details]

<div id="results"/>
#### Results

This is the list shown

![list layout](result.jpg?resize=400)

And this is the log of the clicks on the list

```
2020-02-24 12:09:10.788 7939-7939/dev.biasin.playground.viewbindlist D/MainActivity: Clicked author Aimaro Monaco dei Corbizzi
2020-02-24 12:09:11.235 7939-7939/dev.biasin.playground.viewbindlist D/MainActivity: Clicked author Bosone da Gubbio
2020-02-24 12:09:13.012 7939-7939/dev.biasin.playground.viewbindlist D/MainActivity: Clicked author Dante Alighieri
2020-02-24 12:09:13.276 7939-7939/dev.biasin.playground.viewbindlist D/MainActivity: Clicked author Alessandro da Sant'Elpidio
2020-02-24 12:09:16.844 7939-7939/dev.biasin.playground.viewbindlist D/MainActivity: Clicked author Aimaro Monaco dei Corbizzi
2020-02-24 12:09:17.108 7939-7939/dev.biasin.playground.viewbindlist D/MainActivity: Clicked author Giovanni Boccaccio
```

<div id="future"/>
#### What now?

We can try and make this more generic. 
Instead of our clickListener returning a String we can use an interface

```kotlin
// this interface can be used to return every kind of object implementing it
interface  ListResponse
```

and another one to be implemented by the listener, be it an Activity or a Fragment

```kotlin
// this interface must be implemented in fragment/ listener listening to events on lists and passed to the adapter constructor
interface ListClickListener {
    fun onListClick(response: ListResponse)
}
```
A generic holder could be like this

```kotlin
abstract class GenericHolder(binding: ViewBinding) :
    RecyclerView.ViewHolder(binding.root) {

    abstract fun bind(item: MultiListItem, onListClick: (ListResponse) -> Unit)
}
```
Trying an implementation with our old Holder

```kotlin
class SecondGenericHolder(sBinding: SecondListItemBinding): GenericHolder(sBinding){

    private val binding: SecondListItemBinding = sBinding

    override fun bind(item: MultiListItem, clickListener: (ListResponse) -> Unit) {
        binding.cvFirstItem.setOnClickListener { clickListener(
            IntResponse(
                item.type
            )
        ) }
    }

    data class IntResponse(val response: Int):
        ListResponse
}
```

Inflate is also a problem because of the `ViewBinding` interface, which encapsulates only the `getRoot()` function. 

```java
package androidx.viewbinding;

import android.view.View;
import androidx.annotation.NonNull;

/** A type which binds the views in a layout XML to fields. */
public interface ViewBinding {
    /**
     * Returns the outermost {@link View} in the associated layout file. If this binding is for a
     * {@code <merge>} layout, this will return the first view inside of the merge tag.
     */
    @NonNull
    View getRoot();
}
```

We can't make for example `onCreateViewHolder()` more generic: the `inflate()` function is generated in each LayoutNameBinding implementation.

```kotlin
override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)

        when (viewType) {
            TYPE_PAIR -> {
                // ViewBinding has no inflate() method
                val pairBinding =
                    SecondListItemBinding.inflate(layoutInflater, parent, false)
                return ThirdHolder(pairBinding)
            }
            else -> throw NoSuchListItemType("No correspondent binding found for viewType $viewType")
        }
    }
```

A lot more could be written, but that would go better in another article.

#### Useful Links

What are some alternatives to RecyclerViews?

!!!! [fa=fa-android /] [ListAdapter](https://developer.android.com/reference/androidx/recyclerview/widget/**ListAdapter**) is a new class that requires less code and makes DiffUtils easier
  
!!!! [fa=fa-android /] [Jetpack Compose](https://developer.android.com/jetpack/compose/tutorial) will probably replace every part of the Android UI creation

!!!! [fa=fa-book /] There are also various third-party libraries, such as Airbnb [epoxy](https://github.com/airbnb/epoxy)

