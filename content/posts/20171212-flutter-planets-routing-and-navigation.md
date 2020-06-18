+++
date = "2017-12-12"
draft = false
title = "Planets-Flutter: routing and navigation"
tags = ["Android", "iOS", "Flutter", "Planets", "Open Source"]
categories = ["Flutter"]
thumbnailImage="/img/planets-list.png"
aliases = [
  "/2017/12/planets-flutter-routing-and-navigation/"
]
+++

Once the DevFest season is closed, is time to get back to the Flutter Planets tutorial. This time, we will see how to make the list a bit more alive by navigating into the detail page of each planet.

<!--more-->

## Job description

Right now we have a fancy list of planets. But this list is not doing anything, let's see how to make the each item of the list show the appropiate detail page for each planet.

## Routing and navigation

Each screen of our application is called a Route in Flutter. And the responsible to go through these screens is a `Navigator`. Similar to Android, the Routes are set in a navigation stack, even the two main navigation operations are `push` and `pop`


### Defining Routing at application level

The easiest way to define navigation is doing it at the MaterialApp declaration. For example, if we want to declare a page for the planet detail, we will add the following to our MaterialApp object:

{{< highlight dart "linenos=true">}}
new MaterialApp(
  title: "Planets",
  home: new HomePage(),
  routes: <String, WidgetBuilder>{
    '/detail': (_) => new DetailPage(),
  },
),
{{< / highlight>}}

As you can see, the `routes` parameter defines a dictionary of `String` keys. Each key is the name of the path to the page in web-ish format. Each item contain a function of type `WidgetBuilder` that receives the current context and returns the Widget to show as a page, in this case, a DetailPage.

There is another page declared, in this case, trough the home parameter, and its route is always "/".

After declaring the route, we can do that our `PlanetRow` object navigates to this page on click. In order to do this, we should wrap our whole row widget in a `GestureDetector` Widget.

From this:

{{< highlight dart "linenos=true">}}
return new Container(
  height: 120.0,
  margin: const EdgeInsets.symmetric(
    vertical: 16.0,
    horizontal: 24.0,
  ),
  child: new Stack(
    children: <Widget>[
      planetCard,
      planetThumbnail,
    ],
  )
);
{{< / highlight>}}

To this:

{{< highlight dart "linenos=true">}}
return new GestureDetector(
  onTap: () => Navigator.pushNamed(context, "/detail"),
  child: new Container(
    height: 120.0,
    margin: const EdgeInsets.symmetric(
      vertical: 16.0,
      horizontal: 24.0,
    ),
    child: new Stack(
      children: <Widget>[
        planetCard,
        planetThumbnail,
      ],
    ),
  )
);
{{< / highlight>}}

The `GestureDetector` has several parameters like `onTap`, `onDoubleTap`,  `onLongPress`, etc, that allow us to handle all the physical interactions with the user. In this case, we only use the onTap parameter and we provide it with the function to execute in case of tap.

This function executes the method `pushNamed` of the class `Navigator` and needs two parameters, the current context and the path to navigate to, "/detail" for the example.

Now we can create a simple detail page:

{{< highlight dart "linenos=true">}}
class DetailPage extends StatelessWidget{
  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      appBar: new AppBar(
        title: new Text("Planet Detail"),
      ),
      body: new Center(
        child: new RaisedButton(
          onPressed: () => Navigator.pop(context),
          child: new Text("<<< Go back"))
      ),
    );
  }
}
{{< / highlight>}}

Several things happen here. First we created an Scaffold with an AppBar, if you execute this code, you'll see it automatically adds the back button as it detects we are in a secondary page (and it works!). Second, we created a button that executes the Navigator.pop() method, so it also gets back.

The user then has three ways to get back: via back button, via the button we put on the center of the screen, and third one is different for Android or iOS. For android the back button on the phone works, for iOS, the left swipe gesture works.

## Passing parameters

Unfortunately, the routing table method doesn't allow us to send parameters to the new screen (at least out of the box).

In order to do this, we need to generate our `Route` on the fly.

First, remove the routes parameter from our MaterialApp (or you can left it, but we will not use it).

And modify the `GestureDetector` like this:

{{< highlight dart "linenos=true">}}
return new GestureDetector(
  onTap: () => Navigator.of(context).push(new PageRouteBuilder(
    pageBuilder: (_, __, ___) => new DetailPage(planet),
  )),
  child: new Container(
    height: 120.0,
    margin: const EdgeInsets.symmetric(
      vertical: 16.0,
      horizontal: 24.0,
    ),
    child: new Stack(
      children: <Widget>[
        planetCard,
        planetThumbnail,
      ],
    ),
  )
);
{{< / highlight>}}

Now, instead of recovering the PageRoute from the table of routes with `pushNamed` we are creating a new one using the class `PageRoutBuilder`, and it has a parameter named `pageBuilder` that should return a new Widget to show as page, in this case, `DetailPage` receiving as a parameter the planet to show.

Another interesting parameter from PageRouteBuilder is `transitionBuilder`. I dare you to play with it to modify the transitions used during navigation.

Now, we need to modify the DetailPage to show something about the planet received.

{{< highlight dart "linenos=true">}}
class DetailPage extends StatelessWidget {

  final Planet planet;

  DetailPage(this.planet);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Container(
        constraints: new BoxConstraints.expand(),
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Text(planet.name),
            new Image.asset(planet.image, width: 96.0, height: 96.0,),
          ],
        ),
      ),
    );
  }
{{< / highlight>}}

A simple layout that just shows the planet name and planet image in the screen:

![planet card](/img/sad-planet-detail.jpg)

## The cool part, adding a Hero

We will finish this article with something simple and really cool. Heroes.

`Hero` is a Widget intended to perform easy animations between screens. To use it, just do these two little changes.

In the PlanetRow class, modify the `planetThumbnail` to this:

{{< highlight dart "linenos=true">}}
final planetThumbnail = new Container(
  margin: new EdgeInsets.symmetric(
    vertical: 16.0
  ),
  alignment: FractionalOffset.centerLeft,
  child: new Hero(
      tag: "planet-hero-${planet.id}",
      child: new Image(
      image: new AssetImage(planet.image),
      height: 92.0,
      width: 92.0,
    ),
  ),
);
{{< / highlight>}}

As you can see, we just wrapped the image in a `Hero` widget, we also added a tag, with the value "planet-hero-" plus the planet id.

And now we do the same on the planet detail page.

{{< highlight dart "linenos=true">}}
class DetailPage extends StatelessWidget {

  final Planet planet;

  DetailPage(this.planet);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Container(
        color: const Color(0xFF736AB7),
        constraints: new BoxConstraints.expand(),
        child: new Column(
          mainAxisAlignment: MainAxisAlignment.center,
          children: <Widget>[
            new Text(planet.name),
            new Hero(tag: "planet-hero-${planet.id}",
              child: new Image.asset(
                  planet.image,
                  width: 96.0,
                  height: 96.0,
              ),
            )
          ],
        ),
      ),
    );
  }
}
{{< / highlight>}}

See? exactly the same for the image on the detail. 

Now execute it, and you will see your planets flying around when moving from page to page (or back). Cool and easy, isn't it?

The only thing to keep in mind, is that we can not have two heroes in the Widget tree with the same tag, that's why we added the id of the planet to the tag, and, in order to animate, the tag should be the same in both pages.

## To be continued

Ok, now we know a lot of things about routing:

* How to do a simple routing using routing tables
* How to get back once we navigate to a page
* How to do more advancing routing creating our own Router
* How to add a bit of pizzazz to our transitions using the super cool heroes

If you don't want to write code, you can find this example on the branch [Lesson_5_Flutter_Planets_Navigation](https://github.com/sergiandreplace/flutter_planets_tutorial/tree/Lesson_5_Flutter_Planets_Navigation) of the repository.

For the next chapter, we will tidy up some things and start creating a georgeous detail learning some more things about layout.

Stay tuned!