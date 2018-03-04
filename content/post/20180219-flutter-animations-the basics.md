+++
date = "2018-03-04"
draft = false
title = "Flutter animations: The basics"
tags = ["Android", "iOS", "Flutter", "Animations", "Open Source"]
categories = ["Flutter"]
thumbnailImage="img/animations-basic-animationcontroller.png"
+++

Animations in Flutter are awesome. Starting from a simple idea, the framework evolves this idea into objects that can be chained and composited to make jaw dropping animations. But, there is a lot of information to grasp, so, I want to start a new series of articles centered on the use of the Flutter animations. We will start with the simplest concepts and move to awesomeness step by step.

<!--more-->

## The Animation class

The foundation of the Flutter animations is the `Animation` class, a simple abstract class with almost no content. In fact, only contains two values:

* `status`: the current status of the animation (we will talk about later)
* `value`: whatever value we want to control with our animation, type is based in the generic used to declare the class, so we can animate any kind of object.

That's it. Let me repeat it. That's it. Oh, and even the half of it is inherited.

No more fields in this class. Of course, this has no use but to create inherited classes.

Let's analyze this class in more detail. Here is a copy of the source code without comments and without the two fancy `toString` implementations:

{{< highlight dart "linenos=true">}}
abstract class Animation<T> extends Listenable implements ValueListenable<T> {
  const Animation();

  @override
  void addListener(VoidCallback listener);

  @override
  void removeListener(VoidCallback listener);

  void addStatusListener(AnimationStatusListener listener);

  void removeStatusListener(AnimationStatusListener listener);

  AnimationStatus get status;

  @override
  T get value;

  bool get isDismissed => status == AnimationStatus.dismissed;

  bool get isCompleted => status == AnimationStatus.completed;
}
{{< / highlight>}}

We are going to review it line by line. Maybe I'm being exhaustive, but fully understanding this class and its purpose is the key to understanding animations:

* Line 1: we declare the class as abstract, based on a generic `T`. It extends `Listenable` and implements `ValueListenable<T>` (we will see the use of these on the next lines)

* Line 5: `addListener` is inherited from Listenable. This is an abstract method, so we are forced to implement it in any child. The purpose of this method is to add listeners that will listen to changes in the value.

* Line 6: `removeListener` is the oposite of the previous, it remove a listener from our list of listeners. Also inherited from `Listenable`

Keep in mind that these two methods are expected to add and remove listeners for the value, but this is a de facto contract, as nothing forces us to follow this contract. This kind of "this is what you are supposed to do, but, hey, up to you" contracts are very usual in the animation framework, and they are what allows us to bend the rules to create new exciting stuff (don't worry, we will not do it, for the moment).

* Line 14: the `status` property. Notice that this is read only, as we have get, but not set. The status is defined by an enum with four states:

  * `dismissed`: the animation is stopped and the current frame is the first one.

  * `forward`: the animation is playing from the starting point to the ending point (it could not be the same than the start and the beginning).

  * `reverse`: the opposite of `forward`, basically, the animation is playing backwards

  * `completed`: opposite of dismissed, animation is stopped at the last frame.

* Line 17: the `value` stored in the animation. Its type is defined by the generic `T`. This one is also ready only, and is an override from the interface `ValueListenable<T>`.

* Line 19 & 22: Just syntax sugar to check if the status of the animation is `dismissed` or `completed`. Same than checking the value of `status` but nicer to read when used.

Ok, that's nice, but not useful by itself. As we said before, this class is intended to be extended, and we will review the main child class, `AnimationController`

## The AnimationController class

`AnimationController` is one of the big actors on the Animation framework in Flutter. It extends the `Animation` class and adds the bare minimum elements to make it usable. Keep in mind that despite the name, this class is not used as Controller of other `Animation` object, but as a controller of an animation as concept, and it's still an `Animation` object.

What `AnimationController` basically does is to go from on double value to another in the time specified.

Let's review the basic constructor to check what it needs:

{{< highlight dart >}}
AnimationController({
    double value,
    this.duration,
    this.debugLabel,
    this.lowerBound: 0.0,
    this.upperBound: 1.0,
    @required TickerProvider vsync,
  })
{{< / highlight>}}

All parameters are optional, except for `TickerProvider`.

* `value`: the initial value for the animation, if not specified, it will set to the initial moment of the animation (`lowerBound`). It should be a double as `AnimationController` specifies double as the value for the `Animation` generic.

* `duration`: how long the the animation will take. This should be set at some moment before running the application or it will crash. You can do it here at the constructor or later with the setter.

* `debugLabel`: just an indentifier that is shown by the method `toStringDetails` to help us on debug to check what's going on if you have several animations running (believe me, really useful). This parameter doesn't affect the execution of the animation at all.

* `lowerBound`: the initial value for the animation, by default is 0, but you can set any value.

* `upperBound`: the final value for the animation, by default is 1, but, again, you can set any value.

* `vsync`: the `TickerProvider` that this animation will use. We will talk later about this.

### About lowerBound and upperBound

These two values are the key for all the animations, you can specify whatever values you want with just a rule: `lowerBound` value should be minor than the `upperBound` value. Quite obvious, isn't it?

Also, despite you can set any values, most of the time we will use the range 0.0..1.0. We will return to this on future articles, for the time being, let's use whatever values we want.

### Bomb ticking

We mentioned the class `TickerProvider` before. Obviously, this class provides `Ticker` objects, but, what is a `Ticker`? Is just a class wich is able to send a signal, like if it is saying "Now!, Now!, Now!, ...". Default implementations are used to mark the frames of an animation and to help us to reach a specific number of FPS (Frames Per Second).

The easiest way to get a `TickerProvider` is to add a Mixin to our class. We have to basic implementations: `SingleTickerProviderStateMixin` and `TickerProviderStateMixin`. The first one provides only one Ticker and the second one provides several ones. So, first one is more efficient if you have a single `AnimationController`

If you are not sure what a mixin is, check the [Dart documentation about mixins](https://www.dartlang.org/articles/language/mixins).

### Using AnimationController

These are the basic steps for using an `AnimationController`:

1. Instantiate an `AnimationController` with needed parameters, specially `duration` and `vsync`.
2. Add the needed listeners (probably just one).
3. Start the animation.
4. Perform the appropiate action in the listener callback (usually setState, but options are endless depending on your case).
5. Dispose the animation once the Widget is disposed.

This is like the minimum template for using an `AnimationController`. Of course there are many other things like stopping, playing reverse, playing from a specific point, using status of the animation, etc.

### The basic example

Let's see a basic usage for `AnimationController`:

{{< highlight dart "linenos=true">}}
class AnimationControllerOutputBody extends StatefulWidget {
  @override
  _AnimationControllerOutputBodyState createState() =>
      new _AnimationControllerOutputBodyState();
}

class _AnimationControllerOutputBodyState extends State<AnimationControllerOutputBody>
    with SingleTickerProviderStateMixin {

  AnimationController animation;

  @override
  void initState() {
    super.initState();
    animation = new AnimationController(
      vsync: this,
      duration: new Duration(seconds: 3),
    );
    animation.addListener(() {
      this.setState(() {});
    });
  }

  @override
  Widget build(BuildContext context) {
    return new GestureDetector(
      child: new Center(
        child: new Text(
          animation.isAnimating
              ? animation.value.toStringAsFixed(3)
              : "Tap me!",
          style: new TextStyle(
            fontSize: 50.0,
          ),
        ),
      ),
      onTap: () {
        animation.forward(from: 0.0);
      },
    );
  }

  @override
  void dispose() {
    animation.dispose();
    super.dispose();
  }
}

{{< / highlight>}}

* Lines 1-7 are the basic code to create a common `StatefulWidget`. We haven't discuss about `StatefulWidgets` in this blog yet. If you don't know anything about it, my suggestion is that you do the Google's codelabs on Flutter to understand it.

* Line 8: we add the mixin `SingleTickerProviderStateMixin` to our class so it can be a `TickerProvider`. 

* Line 10: declare our `AnimationController` as animation.

* Lines 15-18: instantiate our `AnimationController` in the `initState` method. We provide just two parameters, the `vsync`, using the class as `TickerProvider` because it has the `SingleTickerProviderStateMixin` mixin, and the animation duration to 3 seconds (an arbitrary number, use whatever you want). Check the class `Duration`, it allows us to define time durations in a variety of units.

* Line 19-21: we add a listener to our animation that will just run the `setState()` method.

Key concepts for this animation have already been declared. Once we start our animation, it will run `setState()` every time the ticker makes a "tick" during 3 seconds. Take your time to really understand where the parts of this phrase come from, as, again, these are the key concepts that create an animation.

* Line 25: just a build method for our widget. It will create a `Text` widget with large font centered in screen. This text is clickable as it's contained in a `GestureDetector`.

* Lines 29-31: if the animation is running, it will show the value of the animation (fixed to three decimals), otherwise, it will show the text "Tap me!".

* Lines 37-39: when the `GestureDectector` receives a click, we run the animation, using just the `forward` method. We also specify the from paramater so it starts from the beginning. This parameter is not needed the first time we run our animation, but once we run it, it will remain in the last frame unless other thing is said, so the next times we run `forward` without from parameter it won't do anything.

* Lines 44-47: we need to run the dispose method both in the widget and the animation. This will destroy the ticker and any other resources the animation has created. It's just a matter of being well-behaved to avoid resource cluttering.

So, what whill this widget do?

The first time it is built, it will show the text "Tap me!" as the animation is not running. Once we click the text, the animation will start, and will perform a `setState` every time the ticker makes "tick" during 3 seconds. So, for every tick, we will repaint the whole widget, but this time, instead of showing "Tap me!", it will show the current value for animation (remember, a number between 0.0 and 1.0 as we didn't say otherwise). The final tick will again request for a paint, but as the animation will be at 1.0 it will be stopped, coming back to the "Tap me!" text.

Again, review the last paragraph and the code lifecycle, as this is the basic behaviour of animations, and it's important to realy understand what's going on.

This is an example of the animation running:

![AnimationController going on](/img/animations-basic-animationcontroller.gif)

Note: the freezing before finishing the animation is due to the gif recording, the real app works like a charm.

Nothing really exciting, but trust me, understanding this basic example, will make much easier to make complex animations.

I think this is enough for today, but, I want to try something new. Allow me to suggest experiments you can do. Trying to expain EVERYTHING you can do with animations will take me a couple of lifes. An important part is you playing around with the stuff I explain, so, try this little experiment:

* If you tap the text while animating, the animation will start again. Do you know how to avoid this? so, the animation only starts on tap when the "Tap me!" is shown.
* Try to use the value of the animation not to show a number, but to set the font size of the text.
* Keep in mind that value goes from 0.0 to 1.0, obviously, if you animate fontSize from 0.0 to 1.0 you won't see anything. There are several ways to fix this, but I'm not going to tell you how, do your homework.
* If you achieve it, don't stop here, do the animation go and back, animate other properties, try to make the animation configurable, etc. The sky is the limit!

Next time we will see how to apply this to properties with easy examples and will introduce the concept of animatables.

The code for this article can be found at [https://github.com/sergiandreplace/flutter_animations](https://github.com/sergiandreplace/flutter_animations). This article code is committed in the branch `1_animation_controller_output ` in the file `animation_controller_output`. This will be an app containing a list of examples for animations. 

Stay tuned for more tutorials and articles!