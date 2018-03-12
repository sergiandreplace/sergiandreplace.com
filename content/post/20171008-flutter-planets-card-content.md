+++
date = "2017-10-08"
draft = false
title = "Planets-Flutter: adding content to the card"
tags = ["Android", "iOS", "Flutter", "Planets", "Open Source"]
categories = ["Flutter"]
thumbnailImage="/img/planet-card-full-text.png"
aliases = [
  "/2017/10/planets-flutter-adding-content-to-the-card/"
]
+++

In the previous article, we learned how to create a card to show some information about a planet. Now it is time to put that information in the card.

<!--more--> 

## Job description

We want to be able to provide the planet info to `PlanetRow` widget and see it painted on screen.

## Creating a data source

As the list of planets is static and the data won't change, creating this information from code will be enough.

First, we create a class to hold information for a single planet:

{{< highlight dart "linenos=true">}}
class Planet {
  final String id;
  final String name;
  final String location;
  final String distance;
  final String gravity;
  final String description;
  final String image;

  const Planet({this.id, this.name, this.location, this.distance, this.gravity,
    this.description, this.image});
}
{{< / highlight>}}

Fine, just a simple object (PODO? Plain Old Dart Object?).

Now, we create the list of planets:

{{< highlight dart "linenos=true">}}
List<Planet> planets = [
  const Planet(
    id: "1",
    name: "Mars",
    location: "Milkyway Galaxy",
    distance: "227.9m Km",
    gravity: "3.711 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/mars.png",
  ),
  const Planet(
    id: "2",
    name: "Neptune",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "11.15 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/neptune.png",
  ),
  const Planet(
    id: "3",
    name: "Moon",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "1.622 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/moon.png",
  ),
  const Planet(
    id: "4",
    name: "Earth",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "9.807 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/earth.png",
  ),
  const Planet(
    id: "5",
    name: "Mercury",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "3.7 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/mercury.png",
  ),
];
{{< / highlight>}}

As you see, all the info is just mocked data. I invite you to use the real information if you want to.

Also, we place the images on the img folder. Find all of them in the repository.

Ok, now we are ready to modify the PlanetRow class.

## A PlanetRow for all planets

We need to make the PlanetCard to be able to receive one Planet object and paint it:

{{< highlight dart "linenos=true,hl_lines=3 5">}}
class PlanetRow extends StatelessWidget {

  final Planet planet;

  PlanetRow(this.planet);
  
  ...

  @override
  Widget build(BuildContext context) {
    return new Container(
      height: 120.0,
      margin: const EdgeInsets.symmetric(
        vertical: 16.0,
        horizontal: 24.0,
      ),
      child: new Stack(
        children: <Widget>[
          planetCard
          planetThumbnail,
        ],
      )
    );
  }
}
{{< / highlight >}}

First, we add a parameter to the constructor that receives a `Planet` object and stores it in a final field.

Next step, we will modify the planetThumbnail:

{{< highlight dart "linenos=true, hl_lines=7">}}
  final planetThumbnail = new Container(
    margin: new EdgeInsets.symmetric(
      vertical: 16.0
    ),
    alignment: FractionalOffset.centerLeft,
    child: new Image(
      image: new AssetImage(planet.image),
      height: 92.0,
      width: 92.0,
    ),
  );
{{< / highlight >}}

The only change we have applied is to modify the thumbnail path from a constant to the field image in the Planet object we received.

Also, in the previous article, we declared planetThumbnail and planetCard as final class members, in order to be able to reference the planet field, this declaration should be moved into the build function.

{{< highlight dart "linenos=true">}}

class HomePageBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new PlanetRow(planets[0]);
  }
}
{{< / highlight >}}

And finally, we modify the creation of PlanetRow in HomePageBody to give a planet of the array to it. You can try to change the 0 to another number to see how the thumbnail works.

Let's add the rest of fields to the card.

## A cute card

As we will be using three different text styles, we will create them as constants and reuse them later.

### Text styling

We add the Poppins-regular.ttf file to assets/fonts and add an entry to the pubspec.yml, so the fonts entry finally looks like this:

{{< highlight yaml>}}
  fonts:
    - family: Poppins
      fonts:
        - asset: assets/fonts/Poppins-SemiBold.ttf
          weight: 600
        - asset: assets/fonts/Poppins-Regular.ttf
          weight: 400
{{< / highlight >}}

If you don't know the weight of a font, a trick is to check the class `FontWeight`, as each constant (w100, w200, etc..) indicates the usual preffix to the font name associated to each weight (thin, extra-light, light, bold, regular, etc).

As all our text styles use the same font, we can create a base style indicating the font:

{{< highlight dart >}}
    final baseTextStyle = const TextStyle(
      fontFamily: 'Poppins'
    );
{{< / highlight >}}

Now we can create the header style copying the base style:

{{< highlight dart >}}
    final headerTextStyle = baseTextStyle.copyWith(
      color: const Colors.white,
      fontSize: 18.0,
      fontWeight: FontWeight.w600
    );
{{< / highlight >}}

The `copyWith` allows us to generate a new `TextStyle` object from another one modifying some of the attributes.

Same for the regular text style:

{{< highlight dart >}}
    final regularTextStyle = baseTextStyle.copyWith(
      color: const Color(0xffb6b2df),
      fontSize: 9.0,
      fontWeight: FontWeight.w400
    );
{{< / highlight >}}

And, for the subheader, we can copy the regular one just changing the size of the font:

{{< highlight dart >}}
    final subHeaderTextStyle = regularTextStyle.copyWith(
      fontSize: 12.0
    );
{{< / highlight >}}

This is a far from ideal way to organize text styles, but it's better than nothing. In a future article we will discuss how to organize and centralize all the styling to save a lot of boilerplate.

### And now the content

I've modified some of the margins on the design to get better dp sizes (basically, approach them to multiples of 3).

![card content measures](/img/planet-card-content-measures.png)

The result is the following code:

{{< highlight dart >}}
final planetCardContent = new Container(
      margin: new EdgeInsets.fromLTRB(76.0, 16.0, 16.0, 16.0),
      constraints: new BoxConstraints.expand(),
      child: new Column(
        crossAxisAlignment: CrossAxisAlignment.start,
        children: <Widget>[
          new Container(height: 4.0),
          new Text(planet.name,
            style: headerTextStyle,
          ),
          new Container(height: 10.0),
          new Text(planet.location,
            style: subHeaderTextStyle

          ),
          new Container(
            margin: new EdgeInsets.symmetric(vertical: 8.0),
            height: 2.0,
            width: 18.0,
            color: new Color(0xff00c6ff)
          ),
          new Row(
            children: <Widget>[
              new Image.asset("assets/img/ic_distance.png", height: 12.0),
              new Container(width: 8.0),
              new Text(planet.distance,
                style: regularTextStyle,
              ),
              new Container(width: 24.0),
              new Image.asset("assets/img/ic_gravity.png", height: 12.0),
              new Container(width: 8.0),
              new Text(planet.gravity,
                style: regularTextStyle,
              ),
            ],
          ),
        ],
      ),
    );
{{< / highlight >}}

What is hapenning here?

* We create a container that will be the base widget for the whole content.
* We define the margins as per design.
* We have to define a constraint (`BoxConstraints.expand()`), otherwise, the container will adjust to the minimum size required by its children, and we want it to get the whole row.
* As each text is below the other, we use a Column to dispose them.
* We use empty containers to create the separation between text elements.
* First element shows the name of the planet using `headerTextStyle`.
* Second element shows the location of the planet using `subHeaderTextStyle`.
* We use a single container to create the blue line, just specifying the margins and size of it, and giving the appropiate background color.
* The gravity and distance should be put in a Row, as they flow horizontally.
* Each <what?> consists on an icon from assets, a separation container, and a text.

There is a much more convenient way of doing the row of values (gravity and distance).

Imagine we want the second value to always start in the center, despite the width of the row. In order to achieve this, we have to use a Expanded widget:

{{< highlight dart >}}
          new Row(
            children: <Widget>[
              new Expanded(
                child: new Row(
                  children: <Widget>[
                    new Image.asset("assets/img/ic_distance.png", height: 12.0),
                    new Container(width: 8.0),
                    new Text(planet.distance, style: regularTextStyle),
                  ]
                ),
              ),
              new Expanded(
                child: new Row(
                  children: <Widget>[
                    new Image.asset("assets/img/ic_gravity.png", height: 12.0),
                    new Container(width: 8.0),
                    new Text(planet.gravity, style: regularTextStyle),
                  ]
                ),
              )
            ],
          ),
{{< / highlight >}}

Now, the row contains two Expanded widgets. And they will share the space to fill at 50%. As the content aligns to the left, the second one will always start at the center.

As both contents are very similar, we can extract them to a function:

{{< highlight dart >}}
    Widget _planetValue({String value, String image}) {
      return new Row(
        children: <Widget>[
          new Image.asset(image, height: 12.0),
          new Container(width: 8.0),
          new Text(planet.gravity, style: regularTextStyle),
        ]
      );
    }
{{< / highlight >}}

So, the final row looks like this:
{{< highlight dart>}}
          new Row(
            children: <Widget>[
              new Expanded(
                child: _planetValue(
                  value: planet.distance,
                  image: 'assets/img/ic_distance.png')

              ),
              new Expanded(
                child: _planetValue(
                  value: planet.gravity,
                  image: 'assets/img/ic_gravity.png')
              )
            ],
          )
{{< / highlight >}}

It looks cleaner to me.

The final result now looks like this:

![final card](/img/planet-card-full-text.png)

## To be continued

In this episode we have learned several things related to layout and styling:

* How to use different weights of the same font. If you come from Android native development, this looks like a small miracle ;).
* How to simplify text styling by overriding styles. We will see a lot more on how to make this even easier and cleaner.
* How to create and use a small data source so that we can paint any planet.
* Using constrains in a container to make it expand.
* Using the expanded widget to distribute items equally.

The whole code for this project is uploaded to the [flutter-planets-tutorial](https://github.com/sergiandreplace/flutter_planets_tutorial) repository, and this article is in the branch [Lesson_3_Planets-Flutter_adding_content_to_the_card](https://github.com/sergiandreplace/flutter_planets_tutorial/tree/Lesson_3_Planets-Flutter_adding_content_to_the_card). Also, you will find all the assets used in the project.

In the next article, we will finally create a list of elements to show all the planets related information.

Stay tuned!








