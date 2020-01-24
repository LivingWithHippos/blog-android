---
title: 'View Binding - Introduction'
highlight:
    theme: darkula
shortcode-core:
    load_fontawesome: true
---

## VIEW BINDING FOR ANDROID

!!! [[fa=fa-android /] Android Official Page on View Binding](https://developer.android.com/topic/libraries/view-binding)


#### What is View Binding?

View Binding is a new feature in Android development aimed at easing the use of views in your code. It provides compile-time type safety for code that interacts with views and is already being suggested on the official Android documentation as a replacement for findViewById. This page will be a super short introduction with the necessary instruction to get it ready.

#### Good Old findViewById
The old way to obtain a View from your XML layout in your code required a setup like this:

[fa icon=fa-file-code /] *our layout example: fragment_first.xml*
```XML
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".FirstFragment">

    <TextView
        android:id="@+id/textview_first"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/hello_first_fragment"
        app:layout_constraintBottom_toTopOf="@id/button_first"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <Button
        android:id="@+id/button_first"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="@string/next"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@id/textview_first" />
</androidx.constraintlayout.widget.ConstraintLayout>
```
[fa icon=fa-file-signature /] *FirstFragment.kt*

```kotlin
class FirstFragment : Fragment() {

    var button: Button? = null

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        //we directly inflate and return the view
        return inflater.inflate(R.layout.fragment_first, container, false)
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // we call findViewById from the passed view
        button = view.findViewById(R.id.button_first)
        button?.setOnClickListener {
            // do something
            ...
        }
    }
}
    
```

The code is pretty straightforward: when we need to use our button we call findViewById and reference it by its XML layout id.
We use a var button: Button? and button?.setOnClickListener to manage null values. Of course there are other (better) ways, but this is just for show.

#### Setting up our View Binding

! [fa icon=fa-exclamation-triangle] Note: Android Studio >= 3.6 is required for View Binding. At the moment 3.6 it's not stable, so you need to download it from [here](https://developer.android.com/studio/preview/). 

 First things first enable viewbinding in your gradle app module.
 
 ```
 android {
     ...
    viewBinding {
        enabled = true
    }
```
Now since a class for every XML layout is going to be generated, rebuild [fa=fa-hammer /] your project. 

!! [fa icon=fa-bug /] View Bind is not fully stable and re building the project can be useful to fix random issues such as losing references to Binding classes. You can also remove viewbinding from you app gradle file, Sync and re-add it or use File -> Invalidate cache / restart.

#### Using our View Binding
For every *layout_name.xml* in your res/layout folder a correspondent *LayoutNameBinding* class will be created. 
We can now use this to call every piece of our view.

```kotlin
class FirstFragment : Fragment() {

    // we use a nullable binding so we can remove it when the fragment is destroyed
    private var _binding: FragmentFirstBinding? = null
    private val binding get() = _binding!!

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // we initialize it once here, in the onCreateView
        _binding = FragmentFirstBinding.inflate(inflater, container, false)
        // binding.root is our root view
        val view = binding.root
        return view
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        // we can obtain what we need with binding.view_id
        binding.buttonFirst.setOnClickListener {
            ...
        }
    }
    
    override fun onDestroyView() {
        _binding = null
    }
}
```

The code to use View Binding in activities is pretty much the same:

```kotlin
class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)
        ...
```

And this is it for a short introduction. See the other links in [this section](https://sandnaut.com/blog/android/viewbinding) for more interesting applications of the View Binding.

#### Other resources
!!! [fa=fa-android /] [Generated binding classes.](https://developer.android.com/topic/libraries/data-binding/generated-binding) The page is about data binding, but there is some interesting info working for view binding, too.

