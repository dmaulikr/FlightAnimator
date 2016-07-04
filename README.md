#FlightAnimator

[![Cocoapods Compatible](https://img.shields.io/badge/pod-v0.7.1-blue.svg)]()
[![Carthage compatible](https://img.shields.io/badge/Carthage-compatible-4BC51D.svg?style=flat)]()
[![Platform](https://img.shields.io/badge/platform-ios-lightgrey.svg)]()
[![License](https://img.shields.io/badge/license-MIT-343434.svg)](/LICENSE.md)

![alt tag](/Documentation/FlightBanner.jpg?raw=true)


##Features

- [x] [46+ Parametric Curves, Decay, and Springs](/Documentation/parametric_easings.md) 
- [x] Blocks Syntax for Building Complex Animations
- [x] Define, Cache, and Reuse Animations
- [x] Apply Unique Easing per Property Animation
- [x] Chain Animations:
	* Synchronously 
	* Time Progress Relative
	* Value Progress Relative
- [x] Advanced Multi-Curve Group Synchronization

##Communication

- If you **found a bug**, or **have a feature request**, open an issue.
- If you **need help** or a **general question**, use [Stack Overflow](http://stackoverflow.com/questions/tagged/flight-animator). (tag 'flight-animator')
- If you **want to contribute**, review the [Contribution Guidelines](/Documentation/CONTRIBUTING.md), and submit a pull request. 

##Installation

* [Release Notes](/Documentation/release_notes.md)
* [Installation Documentation](/Documentation/installation.md)

##Introduction

FlightAnimator is a natural animation engine built on top of CoreAnimation. Implemented with a blocks based approach it is very easy to create, configure, cache, and reuse animations dynamically based on the current state. 

FlightAnimator uses CAKeyframeAnimation(s) and CoreAnimationGroup(s) under the hood. One can apply animations on a view directly, or cache animations to define states, and apply them at a later time. The animations are technically a custom CAAnimationGroup, once applied to the layer, will dynamically synchronize the remaining progress based on the current presentationLayer's values.

Before beginning the tutorial feel free to clone the repository, and checkout the demo app included with the project. In the project one can set different timing curves for bounds, position, alpha, and transform. Feel free to experiment by adjusting the timing curves to explore the resulting effects.


##Basic Use 

There are two ways  to use this framework, perform an animation on a specific view right away, or register an animation on a view to perform later. 

When creating or registering an animation, the frame work uses a blocks based syntax to build the animation. During the build process, for each property animation, one can apply a value, the timing curve, and a the primary flag, which will be discussed at a later point in the documentation.

###Simple Animation

To perform a simple animation  call the `animate(:)` method on the view to animate. Let's look at a simple example below.

```swift
view.animate { (animator) in
      animator.bounds(newBounds).duration(0.5).easing(.OutCubic)
      animator.position(newPositon).duration(0.5).easing(.InSine)
}
```
The closure returns an instance of an FAAnimationMaker, which can be used to build a complex animation to perform, one property at a time. Apply different durations, and timing curves for each individual property in the animation. And that's it, the animation kicks itself off, applies the final animation to the layer, and sets all the final layers values on the model layer.

In the case there is a need to animate a custom defined NSManaged animatable property, i.e progress to draw a circle. Use the `value(value:forKeyPath:)` method on the animator to animate that property.

```swift
view.animate { (animator) in
      animator.value(value, forKeyPath : "progress").duration(0.5).easing(.OutCubic)
}
```

##Sequence

Chaining animations together in FlightAnimator is very easy. You can nest animations using three different types triggers:

* Simultaneously
* Time Progress Based
* Value Progress Based
 
These can be applied to the view being animated, or any other view accessible in the view heirarchy. Let's look at how to nest some animations using triggers.

####Trigger Simultaneously

To trigger an animation right as the parent animation begins, attach a trigger on a parent animator by calling `animator.triggerOnStart(...)`. The trigger will perform the animation enclosed accordingly right as the parent begins animating. 

```swift
view.animate { (animator) in
	animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
    animator.position(newPositon).duration(0.5).easing(.EaseOutCubic)
    
    animatortriggerOnStart(onView: self.secondaryView, animator: { (animator) in
         animator.bounds(newSecondaryBounds).duration(0.5).easing(.OutCubic)
         animator.position(newSecondaryCenter).duration(0.5).easing(.OutCubic)
    })
```

####Trigger Relative to Time Progress

A time based trigger will apply the next animation based on the progressed time of the overall parent animation. The progress value is defined with a range from 0.0 - 1.0, if the over all time of an animation is 1.0 second, by setting the atProgress paramter to 0.5, will trigger the animation at the 0.5 seconds into the parent animation. 

Below is an examples that will trigger the second animation at the halfway point in time of the parent animation by calling `triggerAtTimeProgress(...)`

```swift
view.animate { (animator) in
	animator.bounds(newBounds).duration(0.5).easing(.OutCubic)
    animator.position(newPositon).duration(0.5).easing(.OutCubic)
    
    animator.triggerAtTimeProgress(atProgress: 0.5, onView: self.secondaryView, animator: { (animator) in
         animator.bounds(newSecondaryBounds).duration(0.5).easing(.OutCubic)
         animator.position(newSecondaryCenter).duration(0.5).easing(.OutCubic)
    })
}
```

####Trigger Relative to Value Progress

A value based progress trigger will apply the next animation based on the value progress of the overall parent animation. Below is an examples that will trigger the second animation at the halfway point of the value progress on the parent animation by calling `animator.triggerAtValueProgress(...)`

```swift
view.animate { (animator) in
	animator.bounds(newBounds).duration(0.5).easing(.EaseOutCubic)
    animator.position(newPositon).duration(0.5).easing(.EaseOutCubic)
    
    animator.triggerAtValueProgress(atProgress: 0.5, onView: self.secondaryView, animator: { (animator) in
         animator.bounds(newSecondaryBounds).duration(0.5).easing(.OutCubic)
         animator.position(newSecondaryCenter).duration(0.5).easing(.OutCubic)
    })
}
```

##Cache & Reuse Animations

FlighAnimator allows for defining animations (aka states) up front using keys, and triggers them at any time in the application flow. When the animation is applied, if the view is in mid flight, it will synchronize itself accordingly, and animate to its final destination. To register an animation, call a globally defined method, and create an animations just as defined earlier examples within the maker block.

####Register Animation

The following example shows how to register, and cache it for a key on a specified view. This animation is only cached, and is not performed until it is manually triggered at a later point.

```swift
struct AnimationKeys {
	static let CenterStateFrameAnimation  = "CenterStateFrameAnimation"
}
...

registerAnimation(onView : view, forKey : AnimationKeys.CenterStateFrameAnimation) { (animator) in
      animator.bounds(newBounds).duration(0.5).easing(.OutCubic)
      animator.position(newPositon).duration(0.5).easing(.OutCubic)
})
```

####Trigger Keyed Animation


To trigger the animation call the following 

```
view.applyAnimation(forKey: AnimationKeys.CenterStateFrameAnimation)
```

In the case there is a need to apply the final values without actually animating the view, override the default animated flag to false, and it will apply all the final values to the model layer of the associated view.


```swift
view.applyAnimation(forKey: AnimationKeys.CenterStateFrameAnimation, animated : false)
```


##Advanced Use

###Timing Adjustments

Due to the dynamic nature of the framework, it won't always perform the expect way at first, and may take a few tweaks to get it just right. FlightAnimator has a few settings that allow for customization of the animation duration.

The following timing options are available:

* Designating timing priority during synchronization for the overall animation
* Designating a primary driver on individual property animations within a group

####Timing Priority

First a little background, the framework basically does some magic so synchronize the time by prioritizing the maximum time remaining based on progress if redirected in mid flight.

Lets look at the following example of setting the timingPriority on a group animation to .MaxTime, which is the default value for FlightAnimator.

```swift
func animateView(toFrame : CGRect) {
	
	let newBounds = CGRectMake(0,0, toFrame.width, toFrame.height)
	let newPosition = CGPointMake(toFrame.midX, toFrame.midY)
	
	view.animate(.MaxTime) { (animator) in
      	animator.bounds(newBounds).duration(0.5).easing(.OutCubic)
      	animator.position(newPositon).duration(0.5).easing(.InSine)
	}
}
```
Just like the demo app, This method gets called by different buttons, and takes on the frame value of button that triggered the method. Let's the animation has been triggered, and is in mid flight. While in mid flight another button is tapped, a new animation is applied, and ehe position changes, but the bounds stay the same. 

Internally the framework will figure out the current progress in reference to the last animation, and will select the max duration value from the array of durations on the grouped property animations. 

Lets assume the bounds don't change, thus animation's duration is assumed to be 0.0 after synchronization. The new animation will synchronize to the duration of the position animation based on progress, and automatically becomes the max duration based on the **.MaxTime** timing priority.

The timing priority can also be applied on ``triggerAtTimeProgress()``  or ``triggerAtValueProgress()``. Now this leads into the next topic, and that is the primary flag.

The more property animations within a group, the more likely the need to adjust how the timing is applied. For this purpose there are 4 timing priorities to choose from:

* .MaxTime 
* .MinTime
* .Median
* .Average


####Primary Flag

As in the example prior, there is a mention that animations can get quite complex, and the more property animations within a group, the more likely the animation will have a hick-up in the timing, especially when synchronizing 4+ animations with different curves and durations.

For this purpose, set the primary flag on individual property animations, and designate them as primary duration drivers. By default, if no property animation is set to primary, during synchronization, FlightAnimator will use the timing priority setting to find the corresponding value from all the animations after progress synchronization.

If we need only some specific property animations to define the progress accordingly, and become the primary drivers, set the primary flag to true, which will exclude any other animation which is not marked as primary from consideration.

Let's look at an example below of a simple view that is being animated from its current position to a new frame using bounds and position.

```swift
view.animate(.MaxTime) { (animator) in
      animator.bounds(newBounds).duration(0.5).easing(.OutCubic).primary(true)
      animator.position(newPositon).duration(0.5).easing(.InSine).primary(true)
      animator.alpha(0.0).duration(0.5).easing(.OutCubic)
      animator.transform(newTransform).duration(0.5).easing(.InSine)
}
```

Simple as that, now when the view is redirected during an animation in mid flight, only the bounds and position animations will be considered as part of the timing synchronization.


###.SpringDecay w/ Initial Velocity

When using a UIPanGestureRecognizer to move a view around on the screen by adjusting its position, and say there is a need to smoothly animate the view to the final destination right as the user lets go of the gesture. This is where the .SpringDecay easing comes into play. The .SpringDecay easing will slow the view down easily into place, all that need to be configured is the initial velocity, and it will calculate its own time relative to the velocity en route to its destination.

Below is an example of how to handle the handoff and use ``.SpringDecay(velocity: velocity)`` easing to perform the animation.

```swift
func respondToPanRecognizer(recognizer : UIPanGestureRecognizer) {
    switch recognizer.state {
    ........
    
    case .Ended:
    	let currentVelocity = recognizer.velocityInView(view)
        
      	view.animate { (animator) in
         	animator.bounds(finalBounds).duration(0.5).easing(.OutCubic)
  			animator.position(finalPositon).duration(0.5).easing(.SpringDecay(velocity: velocity))
      	}
    default:
        break
    }
}
```


##Reference 

[Supported Parametric Curves](/Documentation/parametric_easings.md)

[CALayer's Supported Animatable Property](/Documentation/supported_animatable_properties.md)

[Current Release Notes](/Documentation/release_notes.md)

[Contribution Guidelines](/Documentation/CONTRIBUTING.md)

## License

FlightAnimator is released under the MIT license. See [License](/LICENSE.md) for details.
