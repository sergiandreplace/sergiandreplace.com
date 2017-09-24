+++
date = "2017-09-24"
draft = false
title = "Planets-Flutter: from design to app"
tags = ["Android", "iOS", "Flutter", "Planets", "Open Source"]
categories = ["Flutter"]
+++

Flutter is the new, shiny and cool way to write apps for Android and iOS. 

I felt in love with Flutter during the Android Developer Days in Krakow, and I've decided to start a set of samples on how to create Flutter apps from a design. 

I've spent several days playing with it, aaaand, this is the result.

<!--more--> 

After browsing for a while on [material.uplabs.com](http://material.uplabs.com), I've picked up a nice sample on a Space Travel book app from Vijay Verma: 
https://www.uplabs.com/posts/space-travel-ui. Checkout his other designs, they are really cool.

I've contact him, and he send me the original assets. (Thanks Vijay!)

Here a sample of what we are going to achieve:

![sample](/img/planets-preview.png)

## Initial steps

Let's start by creating the minimum application on a main.dart file:

```dart
import 'package:flutter/material.dart';
import 'package:planets/ui/home/HomePage.dart';

void main() {
  Routes.initRoutes();
  runApp(new MaterialApp(
    title: "Planets",
    home: new HomePage(),
  ));
}
```
Simple and nice. We just create a Material App that uses a class named ```HomePage```. Now we create the ```HomePage``` class with simple content:

```dart
import 'package:flutter/material.dart';
class HomePage extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Text("hello"),
    );
  }
}
```

First we'll focus on the toolbar. Because the shape this toolbar takes, I decided to not use the Material AppBar but to create it manually, so the ```HomePage``` class will be as this:

```Dart
import 'package:flutter/material.dart';
import 'package:planets/ui/home/GradientAppBar.dart';

class HomePage extends StatelessWidget {

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Column(
        children: <Widget>[
          new GradientAppBar("treva"),
        ],
      ),
    );
  }
}
```

As you can see, we now use a new clas, ```GradienAppBar``` that will be responsible for holding our self-made app bar;

Let's start the class by creating a simple container:

```dart
import 'package:flutter/material.dart';

class GradientAppBar extends StatelessWidget {
  final String title;
  final double barHeight = 66.0;


  GradientAppBar(this.title);
  
  @override
  Widget build(BuildContext context) {
  
    return new Container(
      height: barHeight,
      decoration: new BoxDecoration(color: Colors.blue),
      child: new Center(
        child: new Text(
          title,
        ),
      ),
    );
  }
}
```

As you can see, the result is less than exciting:

![A really ugly header](/img/planets-first-appbar.png)

Several things are going on, the background is plain, the text is black, it's not using the right font, etc.

We are going to fix them one by one:

### Font face, size and color

First we need to use the proper font. Download the font Poppins from Google fonts. For the time being, we are only using Poppins-Semibold.ttf (You can find it [here](https://fonts.google.com/specimen/Poppins)).

Place the ttf file in your assets folder, I'm using a subfolder font to store it.

Add the following line to your pubspec.yaml:

```yaml
  fonts:
    - family: Poppins
      fonts:
        - asset: assets/fonts/Poppins-SemiBold.ttf
          weight: 600
```

Now that we added the font, we can create a TextStyle using it. Change the title ```Text``` object like this:

```dart
        ...
        child: new Text(
          title,
          style: const TextStyle(
            color: Colors.white,
            fontFamily: 'Poppins',
            fontWeight: FontWeight.w600,
            fontSize: 36.0
          )
        ),
        ...
```
If you execute it, the text will look much better.

### Bar size

If you check the previous screenshot, you'll realize that the text is completely centered, buuuut, the screen begin behind the status bar of the phone, so we don't have to count this space for centering or sizing. This is quite easy to achieve in Flutter. Add the following line in your class declarations:

```dart
final double statusBarHeight = MediaQuery
      .of(context)
      .padding
      .top;
```

And modify your container to add the status bar height to the desired height and padding it:

```dart
    ...
    return new Container(
      padding: new EdgeInsets.only(top: statusbarHeight),
      height: statusBarHeight + barHeight,
    ...
```

Much better now!

### Gradients!

The last part is to achieve the gradient background. We only need to change the BoxDecoration by removing the color property and setting up the gradient property like this:

```dart
    ...
    return new Container(
      ...
      decoration: new BoxDecoration(
        gradient: new LinearGradient(
          colors: [
            const Color(0xFF3366FF), 
            const Color(0xFF00CCFF),
          ]
          begin: const FractionalOffset(0.0, 0.0),
          end: const FractionalOffset(1.0, 0.0),
          stops: [0.0, 1.0],
          tileMode: TileMode.clamp
        ),
      ),
    ),
```

```LinearGradient``` parameters are quite easy to understand:

* colors: an array of the colors to be used in the gradient, in this case, two shades of blue.
* begin, end: position of the first color and the last color, in this case, ```FractionalOffset``` allows us to treat the coordinates as a range from 0 to 1 both for x and y. As we want an horizontal gradient, we use same Y for both measures, and the x changes from 0.0 (left) to 1.0 (right).
* stops: this array should have the same size than colors. It defines how the colors will distribute. [0.0, 1.0] will fill it from left to right. [0.0, 0.5] will fill the colors from left to half bar, etc.
* tileMode: what to do if the stops do not fill the whole area. In this case, we added clamp (it will reuse the last color used), but as our gradient goes from 0.0 to 1.0, it's not really necessary.

I encourage you to play with these effects to see how they interact, specially stops and begin/end.

### And now, a cool AppBar

So, now we have everything, the resulting code should look like this:

```dart
import 'package:flutter/material.dart';

class GradientAppBar extends StatelessWidget {

  final String title;
  final double barHeight = 66.0;

  GradientAppBar(this.title);

  @override
  Widget build(BuildContext context) {
    final double statusBarHeight = MediaQuery
      .of(context)
      .padding
      .top;

    return new Container(
      padding: new EdgeInsets.only(top: statusBarHeight),
      height: statusBarHeight + barHeight,
      child: new Center(
        child: new Text(
          title,
          style: const TextStyle(
            color: Colors.white,
            fontFamily: 'Poppins',
            fontWeight: FontWeight.w600,
            fontSize: 36.0
          )
      ),
      decoration: new BoxDecoration(
        gradient: new LinearGradient(
          colors: [
            const Color(0xFF3366FF),
            const Color(0xFF00CCFF)
          ],
          begin: const FractionalOffset(0.0, 0.0),
          end: const FractionalOffset(0.5, 0.0),
          stops: [0.0, 0.5],
          tileMode: TileMode.clamp
        ),
      ),
    );
  }
}
```

Resulting on a nice App bar:

![A really cool header](/img/planets-final-appbar.png)

## To be continued...

Enough for today, and we learned a lot of things:

* Flutter is flexible, having an App bar does not mean you have to use the AppBar class, remember, everything is a Widget.

* We added a font and formatted it, in next articles, we'll see how to add diferent weights for the same family

* We created a cool gradient, play with it, you can do a lot of things.

Enjoy Flutter!