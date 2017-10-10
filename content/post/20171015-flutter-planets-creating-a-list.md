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

At this point we have a data source (a simple static array, but good enough) containing a list of planets and a custom `Widget` able to show information from a given planet. Now is the time to generate a dynamic list to show a list of the planets using the `PlanetCard` object.

## Making a list

Flutter uses the ListView widget to create lists and scrollable items. It's easy to think that it is like Android ListView/RecyclerView component, but in fact, it's most similar to the ScrollView. You can create a list with static items like this:

{{< highlight dart "linenos=true">}}
new ListView(
  children: Widget[] {
    new Item1(),
    new Item2(),
    new Item3()
  }
)
{{< / highlight>}}

And you will have a scrollable section containing three items.

For the case of dynamic lists, we will use the constructor `ListView.builder()`. It takes several extra parameters other than the SimpleListView:

* `itemCount`: the number of items to be shown on the list
* `itemBuilder`: a function of type `IndexedWidgetBuilder` responsible of creating each row. It receives the current context and the item index and must return the `Widget` to be shown.
* `itemExtent`: we can set the size for the rows. If all rows have the same size, this will speed up the painting of the items.

So, we can do this:

{{< highlight dart >}}
class HomePageBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new ListView.builder(
      itemBuilder: (context, index) => new PlanetRow(planets[index]),
      itemCount: planets.length,
      itemExtent: 152.0,
    );
  }
}
{{< / highlight >}}

Buuuut, this won't work.

The console will give us a message like this:

```
The following assertion was thrown during performResize():
Vertical viewport was given unbounded height.
Viewports expand in the scrolling direction to fill their container.In this case, a vertical
viewport was given an unlimited amount of vertical space in which to expand. This situation
typically happens when a scrollable widget is nested inside another scrollable widget.
If this widget is always nested in a scrollable widget there is no need to use a viewport because
there will always be enough vertical space for the children. In this case, consider using a Column
instead. Otherwise, consider using the "shrinkWrap" propery (or a ShrinkWrappingViewport) to size
the height of the viewport to the sum of the heights of its children.
```

The what of the what?

The problem is that the ListView is not able to calculate its own height (mainly because the `HomePagebody` Widget is inside a `Column`). A solution will be to put the ListView inside a Container with a specified height, but, this height could be different on each device. We can calculate it, but there is a better solution. Using a `Expanded` Widget:

{{< highlight dart >}}
class HomePageBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Expanded(
      child: new ListView.builder(
        itemBuilder: (context, index) => new PlanetRow(planets[index]),
        itemCount: planets.length,
      ),
    );
  }
}
{{< / highlight >}}

The Expanded Widget ocuppies all the remaining space after calculating the size of those widgets with specific size, so, it has the ability to give a proper size to the ListView.

