+++
date = "2017-10-10"
draft = true
title = "Planets-Flutter: creating a list of planets"
tags = ["Android", "iOS", "Flutter", "Planets", "Open Source"]
categories = ["Flutter"]
thumbnailImage="/img/planets-card-sample.png"
+++

In the previous article we learned how to create a card to show information for a planet. Now is time to create a list of these cards showing all the planets.

<!--more--> 

## Job description

In order to have a list of items in screen we need to have this list created in a data source. So, we have first to create the data source for the planets, then we will create a dynamic list based on this data. Lastly we will add the fields needed to show all the info on the card.

## Creating a data source

As the list of planets is static and the data won't change, creating this information from code will be enough.

First we create a class to hold information for a single planet:

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
    distance: "54.6m Km",
    gravity: "3.711 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/mars.png",
  ),
  const Planet(
    id: "2",
    name: "Neptune",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "3.711 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/neptune.png",
  ),
  const Planet(
    id: "3",
    name: "Moon",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "3.711 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/moon.png",
  ),
  const Planet(
    id: "4",
    name: "Earth",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "3.711 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/earth.png",
  ),
  const Planet(
    id: "5",
    name: "Mercury",
    location: "Milkyway Galaxy",
    distance: "54.6m Km",
    gravity: "3.711 m/s ",
    description: "Lorem ipsum...",
    image: "assets/img/mercury.png",
  ),
];
{{< / highlight>}}

As you see, all the info is just mocked data. I invite you to put the real information if you want to.

Also, we place the images on the img folder. Find all them in this list:

* [mars.png](https://raw.githubusercontent.com/sergiandreplace/planets-flutter/master/assets/img/mars.png)
* [neptune.png](https://raw.githubusercontent.com/sergiandreplace/planets-flutter/master/assets/img/neptune.png)
* [moon.png](https://raw.githubusercontent.com/sergiandreplace/planets-flutter/master/assets/img/moon.png)
* [earth.png](https://raw.githubusercontent.com/sergiandreplace/planets-flutter/master/assets/img/earth.png)
* [mercury.png](https://raw.githubusercontent.com/sergiandreplace/planets-flutter/master/assets/img/mercury.png)

Ok, now we are ready to modify the PlanetRow class.

## A PlanetRow for all planets

Now we need to make the PlanetCard to be able to receive one Planet object and paint it:

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

First we add a parameter to the constructor that receives a `Planet` object and stores it in a final field.

Next, let's modify the planetThumbnail:

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

The only change we did is to change the thumbnail path from a constant to the field image in the Planet object we received.

{{< highlight dart "linenos=true">}}

class HomePageBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new PlanetRow(planets[0]);
  }
}
{{< / highlight >}}

And last, modify the creation of PlanetRow in HomePageBody to give a planet of the array to it. You can try to change the 0 to another number to see how the thumbnail works.

Let's add the rest of fields to the card.

## A cute card




## Making a list

Flutter uses the ListView widget to create lists and scrollable items. It's easy to think this is like Android ListView/RecyclerView component, but in fact, it's most similar to the ScrollView. You can create a list with static items like this:

{{< highlight dart "linenos=true">}}
new ListView(
  children: Widget[] {
    new Item1(),
    new Item2(),
    new Item3()
  }
)
{{< / highlight>}}

For the case of dynamic lists, we will use the constructor `ListView.builder()`. It takes several extra parameters other than the SimpleListView:

* `itemCount`: the number of items to be shown on the list
* `itemBuilder`: a function of type `IndexedWidgetBuilder` responsible of creating each row. It receives the current context and the item index and must return the `Widget` to be shown.

