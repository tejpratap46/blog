# How to run Multiple Transitions in Android at Once

Transitions in Android help users navigate from one screen to another with a smooth animation.

There are mainly 2 ways you can create and attach a transition.

1. Create a `res/anim` resource and attach it to fragment transaction via `setCustomAnimations`
    
2. Create a `res/transition` resource and attach it to fragment via `setEnterTransition` and `setExitTransition`
    

### via [`setCustomAnimations`](https://developer.android.com/reference/androidx/fragment/app/FragmentTransaction#setCustomAnimations(int,%20int))

Simplest way to set custom transition animation is via `setCustomAnimations` which accepts 4 animation files, one for each transition.

```kotlin
val fragment = FragmentB()
supportFragmentManager.commit {
    setCustomAnimations(
        enter = R.anim.slide_in,
        exit = R.anim.fade_out,
        popEnter = R.anim.fade_in,
        popExit = R.anim.slide_out
    )
    replace(R.id.fragment_container, fragment)
    addToBackStack(null)
}
```

All transitions will be part of your `res/anim` folder.

Pros:

* Simplest way to add a transition.
    
* Supports all build-in animations such as `scale`, `Fade`, `translate` etc.
    
* Handles most common scenarios.
    

Cons:

* Cannot fine-tune transitions based of `Views`. Transition will be applied to whole view.
    

To overcome the above limitation, android has another method to attach a transition.

### Via [`setEnterTransition`](https://developer.android.com/reference/android/app/Fragment#setEnterTransition(android.transition.Transition)) Or [`setExitTransition`](https://developer.android.com/reference/android/app/Fragment#setExitTransition(android.transition.Transition))

`setEnterTransition` and `setExitTransition` work differently than `setCustomAnimations` as they accept Transition Object instead of animation, we need to create transitions in `res/transition` folder.

Pros:

* Instead of running animation on complete view, a transition will run transition on all the child view's and their child views if they have any background.
    
* We can exclude a view from transition with `excludeTarget(`[`R.id`](http://R.id)`.view_to_exclude, true)` . Transition will run on all views except this view and its children.
    
* Similarly, you can add a view to transition using `addTarget(`[`R.id`](http://R.id)`.rv_bottom_widget_list)` . Transition will only run on this view and exclude all other views in the fragment.
    

Now we can come to the part where we run multiple transitions on same fragment.

### Running Multiple Transitions At Once

We can use `TransitionSet` to create a set of all the transitions that we want to run together. The only caveat is:

> **You can only run one type of transition only once per view**

Q. What does it mean?

A. For example, if you are running `Fade` transition on a `layout` then that it's child, a `imageView` cannot have another `Fade` transition which goes on for a longer duration. You have only one choice, remove `imageVIew` from 1st transition and add only `imageView` in the second Fade Transition.

```kotlin
// Remove imageView from parent transition, this will remove Fade transition only from imageView
val firstTransition = new Fade().excludeTarget(R.id.imageView, true)
// add second fade transition to imageView so second transition will only run on imageView
val secondTransition = new Fade().addTarge(R.id.imageVIew)
```

### Add delay to transition

* With `setStarteDelay` : You can add delay with `setStartDelay(long delay)`. It will ask the transition to run after a specified delay.
    
    ```kotlin
    new Fade().setStartDelay(1000)
    ```
    

* with `setOrdering` : You can also create a `TransitionSet` and make it execute sequentially so each transition will execute one after another.
    
    ```kotlin
    TransitionSet()
        .setOrdering(TransitionSet.ORDERING_SEQUENTIAL)
    ```
    
* With `Interpolator` : interpolator allows you to return different time value instead of a linear time. With this, you can return time value as `zero` until you want to start your animation.
    
    ```kotlin
    public class SlideInterpolator implements TimeInterpolator {
        public SlideInterpolator() {
        }
    
        @Override
        public float getInterpolation(float input) {
            float out;
            if (input < 0.5f) {
                out = 0;
            } else {
                out = input;
            }
            return out;
        }
    }
    ```