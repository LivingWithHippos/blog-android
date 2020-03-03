---
title: content
menu: Content
---

<div id="intro"/>
## Why RecyclerViews?

RecyclerViews are a powerful and relatively easy way to create lists in Android.

!!! [[fa=fa-android /] Android Official Page on RecyclerView](https://developer.android.com/guide/topics/ui/layout/recyclerview)

#### Why use different items layout?

There can be various reasons to do this, usually showing different states for the same kind of items or showing different items.

<div id="setup"/>
#### The Setup

Add ViewBinding to your app's `build.gradle` (see [here](../../introduction) for an introduction). For completeness these are my Gradle files:

[details="build.gradle (app)"]

```gradle
apply plugin: 'com.android.application'
apply plugin: 'kotlin-android'
apply plugin: 'kotlin-android-extensions'

android {
    compileSdkVersion 29
    buildToolsVersion "29.0.3"

    viewBinding {
        enabled = true
    }

    defaultConfig {
        applicationId "dev.biasin.playground.RecyclerListViewBinding"
        minSdkVersion 22
        targetSdkVersion 29
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

}

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])

    // UI components
    implementation "com.google.android.material:material:$rootProject.materialVersion"
    implementation "androidx.appcompat:appcompat:$rootProject.appcompatVersion"
    implementation "androidx.constraintlayout:constraintlayout:$rootProject.constraintlayoutVersion"
    implementation "androidx.cardview:cardview:$rootProject.cardviewVersion"

    implementation "org.jetbrains.kotlin:kotlin-stdlib-jdk7:$rootProject.kotlin_version"
    implementation "androidx.core:core-ktx:$rootProject.core_version"
}

```

[/details]


[details="build.gradle (module)"]

```gradle
// Top-level build file where you can add configuration options common to all sub-projects/modules.

buildscript {
    ext.kotlin_version = '1.3.61'
    repositories {
        google()
        jcenter()
        
    }
    dependencies {
        classpath 'com.android.tools.build:gradle:3.6.0-rc03'
        classpath "org.jetbrains.kotlin:kotlin-gradle-plugin:$kotlin_version"

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
}

allprojects {
    repositories {
        google()
        jcenter()
        
    }
}

task clean(type: Delete) {
    delete rootProject.buildDir
}

ext {
    appcompatVersion = '1.1.0'
    materialVersion = '1.2.0-alpha04'
    constraintlayoutVersion = '1.1.3'
    cardviewVersion = '1.0.0'
    core_version='1.2.0'
}

```

[/details]

<div id="build"/>
### Building the List

- The list passed to the adapter is a generic one since we want to use different items, let's create an interface for that. A variable is used to identify each item type:

```kotlin
interface MultiListItem {
    val type: Int
}
```
In this example a Pair<String, String> and a String are used as items:

```kotlin
data class PoetAndQuote(val nameAndQuote: Pair<String,String>, override val type: Int = TYPE_PAIR): MultiListItem
data class Poet(val name: String, override val type: Int = TYPE_SINGLE): MultiListItem
```

- Write the xml layout of each RecyclerView's item. See [here](#layouts) for the one used in the project (or visit the [repo](https://github.com/LivingWithHippos/android-playground/tree/master/ViewBindList))

- Our ViewHolders must receive the correct ViewBinding corresponding to the just-created xml layout (layout_name.xml -> LayoutNameBinding) and initialize it. Re-build the project if you do not see the bindings in the auto-complete menu. _clicklistener is just a function used as callback, see [here](https://kotlinlang.org/docs/reference/lambdas.html#higher-order-functions-and-lambdas) to learn a little about higher-order function. It's optional, but you almost always need some feedback from taps on a list.


```kotlin
class FirstHolder(fBinding: FirstListItemBinding) :
    RecyclerView.ViewHolder(fBinding.root) {

    private lateinit var content: Pair<String, String>

    private val binding: FirstListItemBinding = fBinding

    fun bind(item: MultiListItem, _clickListener: (String) -> Unit) {

        with(item as PoetAndQuote) {
            content = this.nameAndQuote
        }.also {
            binding.tvName.text = content.first
            binding.tvDescription.text = content.second
            binding.cvFirstItem.setOnClickListener { _clickListener(content.first) }
        }
    }
}
```

Here the first element of the 'Pair<String, String>' is returned when tapping on the CardView.
'with' and 'also' are [scope functions](https://kotlinlang.org/docs/reference/scope-functions.html) and are also optional, they can be avoided with a more classic:


```kotlin
if(item is PoetAndQuote){
    content = item.nameAndQuote
    binding.tvName.text = content.first
    binding.tvDescription.text = content.second
    binding.cvFirstItem.setOnClickListener { _clickListener(content.first)
    }
```

- The adapter is the most crucial part of the list, and its constructor takes a list of items and the callback function.

```kotlin
class MultiViewBindingAdapter(
    items: List<MultiListItem>,
    private val clickListener: (String) -> Unit
) :
    RecyclerView.Adapter<RecyclerView.ViewHolder>() {

```
The item list with two accessory functions to get the number of items and their type:

```kotlin

    private var itemList: List<MultiListItem> = items

    override fun getItemCount(): Int = itemList.size

    override fun getItemViewType(position: Int): Int = itemList[position].type 

```
Function 'onCreateViewHolder' checks the item's type thanks to 'getItemViewType' and the interface 'MultiListItem' and returns the right ViewHolder implementation initialized with the correct binding.

```kotlin

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): RecyclerView.ViewHolder {
        val layoutInflater = LayoutInflater.from(parent.context)

        when (viewType) {
            TYPE_PAIR -> {
                val pairBinding =
                    FirstListItemBinding.inflate(layoutInflater, parent, false)
                return FirstHolder(pairBinding)
            }
            TYPE_SINGLE -> {
                val singleBinding =
                    SecondListItemBinding.inflate(layoutInflater, parent, false)
                return SecondHolder(singleBinding)
            }
            else -> throw NoSuchListItemType("No correspondent binding found for viewType $viewType")
        }
    }
```
Lastly the items and the click listener must be passed to their ViewHolder in `onBindViewHolder`

```kotlin
    override fun onBindViewHolder(holder: RecyclerView.ViewHolder, position: Int) {
        val item = itemList[position]
        if (holder is FirstHolder && item is PoetAndQuote)
            holder.bind(item, clickListener)
        if (holder is SecondHolder && item is Poet)
            holder.bind(item, clickListener)
    }

}
```

And here are the last loose variables/classes:

```kotlin
// we can use the layouts resources
const val TYPE_PAIR = R.layout.first_list_item
const val TYPE_SINGLE = R.layout.second_list_item

class NoSuchListItemType(message: String) : RuntimeException(message)
```

<div id="connectall"/>
### On the Activity/Fragment Side

A layout manager and an instance of the adapter are needed to create our list

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var layoutManager: LinearLayoutManager
    private lateinit var adapter: MultiViewBindingAdapter

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        // this is just the activity layout
        _binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupList()
    }
```
'setupList' makes all the operations needed to initialize our work.

```kotlin
fun setupList(){
        // creating our list of MultiListItem
        val items: MutableList<MultiListItem> = mutableListOf()
        // adding some items
        items.add(PoetAndQuote(Pair("Dante Alighieri",
            "Fatti non foste a viver come bruti ma per seguir virtute e canoscenza")))
        items.add(PoetAndQuote(Pair("Cecco Angiolieri",
            "S’i’ fosse foco, arderei ’l mondo; s’i’ fosse vento, lo tempesterei; ")))
        items.add(PoetAndQuote(Pair("Giovanni Boccaccio",
            "Viva amore, e muoia soldo.")))
        items.add(Poet("Alessandro da Sant'Elpidio"))
        items.add(Poet("Bosone da Gubbio"))
        items.add(Poet("Aimaro Monaco dei Corbizzi"))
        // shuffling so their position is different every time we start the app
        items.shuffle()

        // a layout manager for lists made of rows
        layoutManager = LinearLayoutManager(this)
        // this is how the function is passed as a parameter 
        adapter = MultiViewBindingAdapter(items){ name: String -> poetClicked(name) }

        // assigning them to the RecyclerView with id rvList, defined in the activity layout 
        binding.rvList.layoutManager = layoutManager
        binding.rvList.adapter = adapter

    }

    // this will be called on every click on a CardView
    private fun poetClicked(name: String) {
        Log.d("MainActivity", "Clicked poet $name")
    }
```

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

