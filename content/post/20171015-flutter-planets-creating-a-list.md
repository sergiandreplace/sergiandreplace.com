+++
date = "2017-10-15"
draft = false
title = "Planets-Flutter: creating a list of planets"
tags = ["Android", "iOS", "Flutter", "Planets", "Open Source"]
categories = ["Flutter"]
thumbnailImage="/img/planets-list.png"
+++

In the previous article we learned how to create a card to show information for a planet. Now is time to create a list of these cards showing all the planets.

<!--more--> 

## Job description

At this point we have a data source (a simple static array, but good enough) containing a list of planets and a custom `Widget` able to show information from a given planet. Now is the time to generate a dynamic list to show a list of the planets using the `PlanetCard` object.

## Making a list

Flutter uses the ListView widget to create lists and scrollable items. It's easy to think that it is like Android ListView/RecyclerView component, but in fact, it's most similar to the ScrollView. You can create a list with static items like this:

{{< highlight dart "linenos=true">}}
new ListView(
  children: Widget<>[
    new Item1(),
    new Item2(),
    new Item3()
  ]
)
{{< / highlight>}}

And you will have a scrollable section containing three items. That's good enough when you have a small set of items to put on a Scroll.

In case of dynamic lists, we will use the constructor `ListView.builder()`. It takes several extra parameters other than the SimpleListView:

* `itemCount`: the number of items to be shown on the list
* `itemBuilder`: a function of type `IndexedWidgetBuilder` responsible of creating each row. It receives the current context and the item index and must return a `Widget` to be shown.
* `itemExtent`: we can set the size for the rows. If all rows have the same size, this will speed up the painting of the items.

As you can see, it has some similarities with the Android `Adapter` class.

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

Widgets not being able to calculate height or width (named unbounded height/width) are a common issue you'll face when starting with Flutter. We will keep it simple for the time being, as this is a topic that deserves an article by itself.

Now you have a nice scrollable list of the planets and everything looks fine, but there is one point missing.

Remember when we created the planet row? We said that all rows have a separation of 32dp, including the top margin of first row and the bottom margin of last row. We decided to set a margin of 16dp as top and bottom for each row. That makes 32 dp separation between rows, but the first item top margin and the last item bottom margin are only 16dp.

Let's fix this, we just need to add the padding property to the `ListView.builder` constructor:

{{< highlight dart >}}
class HomePageBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Expanded(
      child: new ListView.builder(
        itemBuilder: (context, index) => new PlanetRow(planets[index]),
        itemCount: planets.length,
        padding: new EdgeInsets.symmetric(vertical: 16.0)
      ),
    );
  }
}
{{< / highlight >}}

And done! now our list has an internal vertical padding of 16 dp.

![A nice list of planets](/img/planets-list.png)

## Enter the Slivers

Despite our list is now fully working as intended, I want to introduce you to the use of Slivers. 

Slivers are a very powerful tool, and  the scrollable Widgets are based on internal use of Slivers.

A Sliver is a piece of scrollable content. Slivers should be placed inside a ScrollView widget. The basic class to create a ScrollView is `CustomScrollView` which allows a list of Slivers as children. There are several ScrollView classes, and reviewing all them goes beyond this article.

There are also several Slivers, these are some of them:

* `SliverAppBar`: used to create a collapsable material AppBar.
* `SliverList`: a linear list of items.
* `SliverFixedExtentList`: similar to the previous one, but for items with fixed height.
* `SliverToBoxAdapter`: a sliver with a single child with a defined size.
* `SliverPadding`: a simple sliver that contains antoher Sliver and allows us to apply a padding.

There many others, but those are the basic ones to create a list.

Also, several of those Slivers use a ChildDelegate of some sort. A ChildDelegate is a class responsible to create each children for the Sliver.

Let's see how to create the previous list using Slivers instead of the `ListView.builder`.

{{< highlight dart >}}
class HomePageBody extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Expanded(
      child: new Container(
        color: new Color(0xFF736AB7),
        child: new CustomScrollView(
          scrollDirection: Axis.vertical,
          slivers: <Widget>[
            new SliverPadding(
              padding: const EdgeInsets.symmetric(vertical: 24.0),
              sliver: new SliverFixedExtentList(
                itemExtent: 152.0,
                delegate: new SliverChildBuilderDelegate(
                    (context, index) => new PlanetRow(planets[index]),
                  childCount: planets.length,

                ),
              ),
            ),
          ],
        ),
      ),
    );
  }
}
{{< / highlight >}}

What are we doing here?

* We add a Container to set the background color of our list.
* We use a `CustomScrollView` instead of a `ListViewBuilder`. 
* We put a `SliverPadding` as our main Sliver. As said before, this allows to add padding to the scrollable content.
* The list of items is created with a `SliverFixedExtentList`, that allows us to create a list of items with the same height.
* Finally, we provide a `SliverChildBuilderDelegate` as delegate. This works similar to the `ListView.builder`, by providing a function to create each item and the number of items.

There are other ways to implement this list using Slivers, but, as an introduction, this will be enough. Slivers are a huge topic, explore them, investigate them, check source code, and get fun. Mastering slivers is an important step to master Flutter.

Another good example could be found on the Flutter samples on how to create a scroll that contains a big image at the top and a gridview below for the Shrine sample ([shrine_home.dart](https://github.com/flutter/flutter/blob/e20bff4d794457b73cfb97bab7652e4e106f6a2d/examples/flutter_gallery/lib/demo/shrine/shrine_home.dart#L385)).

## To be continued

Wow! a lot of information in this chapter for a simple list, a review:

* We learned how to create a simple list using ListView.builder
* We learned the basics of ScrollView
* We learned what Slivers are and how to use some of them
* We learned how to mimic the ListView.builder behaviour using Slivers

A lot to process. You can find the code on the repository, in the [Lesson_4_Planets_Flutter_Making_a_list_of_cards](https://github.com/sergiandreplace/flutter_planets_tutorial/tree/Lesson_4_Planets_Flutter_Making_a_list_of_cards) branch.

In next chapter, we will discuss about routers, navigation and heroes!











