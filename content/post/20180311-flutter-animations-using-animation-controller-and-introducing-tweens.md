+++
date = "2018-13-05"
draft = false
title = "Flutter animations: Using AnimationController and introducing Tweens"
tags = ["Android", "iOS", "Flutter", "Animations", "Open Source"]
categories = ["Flutter"]
thumbnailImage="img/animations-basic-animationcontroller.png"
+++

In this article we will see how to properly use `AnimationController` and we will make an introduction the use of Tweens.

<!--more-->

## Recap 

In the last article we introduced the `AnimationController` and we analysed how it works. We will now use it properly in an example to animate an object.

Let's start with the basics:

{{< highlight dart "linenos=true">}}
class UsingAnimationControllerBody extends StatefulWidget {
  @override
  _UsingAnimationControllerBodyState createState() =>
    new _UsingAnimationControllerBodyState();
}

class _UsingAnimationControllerBodyState
  extends State<UsingAnimationControllerBody>
  with SingleTickerProviderStateMixin {

  AnimationController _controller;

  @override
  void initState() {
    super.initState();


    _controller = new AnimationController(
      duration: new Duration(seconds: 2),
      vsync: this
    )
      ..addListener(() {
        this.setState(() {});
      });


    _controller.forward();
  }

  @override
  Widget build(BuildContext context) {
    ..
  }
{{< / highlight>}}

This is what we learned in the last article. We have a `StatefulWidget` with its `State` that implements a `TickerProvider`. We created an `AnimationController` that longs for 2 secons and then we update the state on each tick. 

If you do not understand this code, check again previous article, because, as we said, it is very important to understand this base code to move on. Go, go. I'll wait here.

## Fly me to the moooon...

Ok, what we want is to put a plane on screen moving from one side to another. We will use a simple `Icon` to get the plane, and we will align it using a `FractionallySizedBox`. Let's see the code:

{{< highlight dart "linenos=true">}}
  @override
  Widget build(BuildContext context) {
    return new Stack(
      fit: StackFit.expand,
      children: <Widget>[
        new FractionallySizedBox(
          heightFactor: 0.2,
          widthFactor: 0.2,
          alignment: new Alignment(_controller.value, 0.0),
          child: new Icon(
            Icons.flight,
            size: 80.0,
          )
        ),
      ],
    );
  }

{{< / highlight>}}

The first thing we create is an `Stack` with an `expand` value for the `fit` property. This will make the `Stack` to take as much space as possible.

The only child is a `FractionallySizedBox`. This widget allows us to position an object in the parent based on fractions of the parent size.

For the size we use `heightFactor` and `widthFactor` to set the percentage of the parent (the `Stack`) that this box will take. I've added 0.2 as an approx value, sure we can calculate the appropiate one but that's an exercise for the reader.

The important param of `FractionallySizedBox` is alignment. This receives an `Alignment` object with two params, x and y, that aligns the child horizontally and vertically. For x, a value of -1.0 will mean completely aligned to the left of the parent, 1.0 will mean completely aligned to the right and 0.0 will mean centered on the parent. Same for y but applied to vertical alignment.

I really recommend playing around with those to understand how they work.

As child we just put an `Icon` with a plane and a size of 80.0.

Realize that for the x alignment we are using the value returned by the `_controller`.

If you run this example, you'll notice that the plane only flights from the center to the right of the screen. Why?

Well, as we haven't said anything about the bounds of the `AnimationController`, it goes from 0.0 (center) to 1.0 (right).

If we just modify the constructor of the `AnimationController`:

{{< highlight dart "linenos=true">}}
    _controller = new AnimationController(
      lowerBound: -1.0,
      upperBound: 1.0,
      duration: new Duration(seconds: 2),
      vsync: this
    )
{{< / highlight>}}

And modify the forward command:

{{< highlight dart "linenos=true">}}
    _controller.forward(from: -1.0);
{{< / highlight>}}

Now, it will work perfect, as the plane goes from left to right!

## Boing Boing

But this is boring. The plane just goes from left to right. Not quite a good business for a flying company. Let's make it fly from side to side all the time.

We just need to make the animation run forward or reverse every time it arrives to one of the sides.

{{< highlight dart "linenos=true">}}
    _controller = new AnimationController(
      lowerBound: -1.0,
      upperBound: 1.0,
      duration: new Duration(seconds: 2),
      vsync: this
    )
    ..addListener(() {
      this.setState(() {});
    })
    ..addStatusListener((status){
      if (status == AnimationStatus.completed) {
        _controller.reverse();
      } else if (status == AnimationStatus.dismissed) {
        _controller.forward();
      }
    });
{{< / highlight>}}

What we are doing is add an State listener. This listener will execute the animation reverse when completed, and run the animation forward when dismissed. 

And now we have a plane going back and forth. But wait, when coming back, the plane flights backwards! We are going to fix this, but first, let's improve our code.

## Introducing Animatables

We are going to see a new object from the animation framework, the `Animatable`. 

This class (which again, is empty, and only intended for inheritance), transforms the output of an animation into another value, and can generate another animation with the output result.

Animatables can be chained and composed, giving us an incredible flexibility to generate different animations.

There are two types of animatables: `Tween` and `CurvedTween`. We will talk about Curves in the future, let's stay with the Tweens.

A `Tween` is a class that gives us a linear interpolation between two values of any type given a moment. This moment is interpolated keeping in mind that 0.0 means begin and 1.0 means start (despite this, we can interpolate values ouside this range, again, we will see this talking about curves). Remember I told you animations are usually in the range between 0.0 and 1.0? That's why.

So, what we have is an `AnimationController` that will produce values between 0.0 and 1.0 and a `Tween` that will interpolate these values between another range.

The current example is easy to see, we need our value for alignment between -1.0 and 1.0. So, we have to make our controller to produce values from 0.0 to 1.0 and we have to interpolate them between -1.0 and 1.0. 

This can be achieved just using the function lerpDouble, that given a range start and end, interpolates a number between 0.0 and 1.0 to this range.

Buuut, we can use a `Tween` for this, in this case, `AlignmentTween` which can interpolate against two different `Alignment`. In this case, we will use `Alignment(-1.0, 0.0)` as beginning, and `Alignment(1.0,0.0)` as ending, to move it horizontally.

First, apart from the controller, we create a class field to store the final animation (trust me, you'll understand this in a minute).

{{< highlight dart "linenos=true">}}
  AnimationController _controller;
  Animation _animation;
{{< / highlight>}}

Second, we put our controller to work from 0.0 to 1.0 again:

{{< highlight dart "linenos=true">}}
    _controller = new AnimationController(
      duration: new Duration(seconds: 2),
      vsync: this
    )
{{< / highlight>}}

Then, in the `initState` method, just after creating and configuring our controller, we define an `AlignmentTween`.

{{< highlight dart "linenos=true">}}
   Tween _tween = new AlignmentTween(
    begin: new Alignment(-1.0, 0.0),
    end: new Alignment(1.0, 0.0),
  );
{{< / highlight>}}

See? just a `Tween` with an intial value and a final value.

And now, the magic, just below, we add:

{{< highlight dart "linenos=true">}}
    _animation = _tween.animate(_controller);

    _controller.forward();
{{< / highlight>}}

Booom! What's going on!?! What we are doing is to execute the method `animate` of our tween to generate a NEW animation. This new animation wraps the tween and the controller, and will be pumped by the controller ticks, that's why we still execute `forward` on the controller.

Read the last paragraph again. This is a very powerful mechanism. I won't get crazy with this in this article, but I'll do it in the next, so play with this idea.

And now, the last change, we move from:

{{< highlight dart "linenos=true">}}
          alignment: new Alignment(_controller.value, 0.0),		
{{< / highlight>}}

to...

{{< highlight dart "linenos=true">}}
          alignment: _animation.value,
{{< / highlight>}}

Because the animation generated by `AlignmentTween` doesn't return doubles anymore, but `Alignment` objects.

Again. Now we have an animation that in every tick will generate a new `Alignment` object. 
These objects will change from (-1.0, 0.0) to (1.0, 0.0) gradually and back.

## This is just an introduction

Ẃe just scratched the surface of the tweens, in the next article we will see other crazy things you can do with other tweens (color! opacity! rounded corners!). But first digest this. 

Try to move the plane different, or from another range. Try to have two things moving around with two tweens going in opposite directions. Get crazy.

Oh! almost forgot!

## The plane should return

Now we want our airplane to face the right direction in each case (planes do now fly backwards, as far as I know). First, we create an '_angle' variable and two constants with the different rotation:

{{< highlight dart "linenos=true">}}
  static const FACE_LEFT_ANGLE = -PI / 2;
  static const FACE_RIGHT_ANGLE = PI / 2;

  double _angle =  FACE_RIGHT_ANGLE;

{{< / highlight>}}

Then, we need to swap between the two constant values depending on the direction of the animation.

{{< highlight dart "linenos=true">}}
      if (status == AnimationStatus.completed) {
        _controller.reverse();
        _angle = FACE_LEFT_ANGLE;
      } else if (status == AnimationStatus.dismissed) {
        _controller.forward();
        _angle = FACE_RIGHT_ANGLE;
      }
{{< / highlight>}}

And simply rotate the Icon depending on the direction of the animation with a simple `Transform.rotate` widget.

{{< highlight dart "linenos=true">}}
    return new Stack(
      fit: StackFit.expand,
      children: <Widget>[
        new FractionallySizedBox(
          heightFactor: 0.2,
          widthFactor: 0.2,
          alignment: _animation.value,
          child: new Transform.rotate(
            angle: _angle,
            child: new Icon(
              Icons.flight,
              size: 80.0,
            ),
          )
        ),
      ],
    );
{{< / highlight>}}

## Enough for today

We introduced how to use the Animation controller to move an object around and used the `AlignmentTween` to map the value produced by the `AnimationController` to meaningful values for the `FractionallySizedBox.alignment` property.

In the next part we will use other Tweens and introduce Curves, but I really recommend you to play around with the concepts introduced in this article as they are important to master animations in Flutter.

You can find the code of this article on Github in the [flutter_animations](https://github.com/sergiandreplace/flutter_animations) repository on the [using_animation_controller.dart](https://github.com/sergiandreplace/flutter_animations/blob/master/lib/pages/using_animation_controller.dart) file. 

Stay tuned for more tutorials and articles!