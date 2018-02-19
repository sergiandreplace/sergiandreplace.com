+++
date = "2018-02-19"
draft = false
title = "Flutter animations: The basics"
tags = ["Android", "iOS", "Flutter", "Animations", "Open Source"]
categories = ["Flutter"]
thumbnailImage="/img/planets-list.png"
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

* Line 1: we declare the class as abstract, based on a generic `T`. It extends `Listenable` and implements `ValueListenable<T>` (we will the use of these on the next lines)

* Line 5: `addListener` is inherited from Listenable. This is an abstract method, so we are forced to implement it in any child. The purpose of this method is to add listeners that will listen to changess in the value.

* Line 6: `removeListener` is the oposite of the previous, it remove a listener from our list of listeners. Also inherited from `Listenable`

Keep in mind that these two methods are expected to add and remove listeners for the value, but this is a de facto contract, as nothing forces us to follow this contract. This kind of "this is what you are supposed to do, but, hey, up to you" contracts are very usual in the animation framework, and they are what allows us to bend the rules to create new exciting stuff (don't worry, we will not do it, for the moment.

* Line 14: the `status` property. Notice that this is read only, as we have get, but not set. The status is defined by an enum with four states:

  * `dismissed`: the animation is stopped and the current frame is the first one.

  * `forward`: the animation is playing from the starting point to the ending point (it could not be the same than the start and the beginning).

  * `reverse`: the opposite of `forward`, basically, the animation is playing backwards

  * `completed`: opposite of dismissed, animation is stopped at the last frame.

* Line 17: the `value` stored in the animation. Its type is defined by the generic `T`. This one is also ready only, and is an override from the interface `ValueListenable<T>`.




Stay tuned for more tutorials and articles!