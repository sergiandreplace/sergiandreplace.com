+++
date = "2018-02-11"
draft = false
title = "Planets-Flutter: planet detail page"
tags = ["Android", "iOS", "Flutter", "Planets", "Open Source"]
categories = ["Flutter"]
thumbnailImage="/img/planets-list.png"
+++

Afte a while struggling on how to continue and how to put all the things I wanted to explain together I have decided to follow the KISSS methodolgy (Keep It Simple, Stupid Sergi) and continue with a simpler example. As the Planets tutorial was focused on UI, I didn't want to include Stateful Widgets. Decided, we will continue with the planet detail screen.


<!--more-->

## Job description

We want to create a detail page to show expanded information about each planet (yes, I know, Moon is not a planet).


### Defining Routing at application level

This is what we want to achieve

![planet detail](/img/planet-detail.png)

We will have four major components on this page:

* The background image
* The gradient between the background image and the background color
* The content itself, which should be scrollable
* The back button on top

So the basic class is easy:

{{< highlight dart "linenos=true">}}
class DetailPage extends StatelessWidget {

  final Planet planet;

  DetailPage(this.planet);

  @override
  Widget build(BuildContext context) {
    return new Scaffold(
      body: new Container(
        constraints: new BoxConstraints.expand(),
        color: new Color(0xFF736AB7),
        child: new Stack (
          children: <Widget>[
            _getBackground(),
            _getGradient(),
            _getContent(),
            _getToolbar(context),
          ],
        ),
      ),
    );
  }

{{< / highlight>}}

We just define a new `Scaffold` having only a `body` with a Container that we use for two basic things:

* set `BoxConstrains.expand()` as constraint. Once we have all the components, we can delete this, as the scroll for the content will already expand the content, but, while creating the background and the gradient, using this constraint will make the Container to use all the screen, instead being cut to the image size.

* set background color as 0xFF736AB7. As exercise to you, maybe is a good idea to group all colors in a single file as we did with text styles.

Inside we have an Stack, as all these four parts will be put one on top of the other. For the time being, comment all the  functions but the first, so we can construct it step by step.

# Background image

We will recover these images from **The Internet**. In the `planets.dart` file (our simple model) we add a new `picture` field to each object with an url to the corresponding image. You can copy these url from the repository.

All the images used are copyrighted from Nasa.

Now we need to put the image in place. What we want is the image to fit 300dp height and the whole screen width.

Simply we create the following function:

{{< highlight dart "linenos=true">}}
  Container _getBackground () {
    return new Container(
            child: new Image.network(planet.picture,
              fit: BoxFit.cover,
              height: 300.0,
            ),
            constraints: new BoxConstraints.expand(height: 300.0),
          );
  }
{{< / highlight>}}

First, we create a `Container` with a constraint to expand as much as possible keeping a height of 300dp.

Second, we use `Image.network` object. This constructor is a quick way to load an image from a url. We also setup the height to 300dp, and the fit parameter as `BoxFit.cover`. This constraint ensures that the downloaded image will cover all the Widget with the minimum image possible, this will work whatever the orientation of the image. I suggest you play around with the different values of `BoxFit` to understand how they work.

The current detail should look like this:

![planet detail](/img/planet-detail-background-image.jpg)

## Background gradient

We want to add a nice gradient on top the image to transition from the picture to the background color. Uncomment the `_getGradient()` line and copy the following code:

{{< highlight dart "linenos=true">}}
  Container _getGradient() {
    return new Container(
      margin: new EdgeInsets.only(top: 190.0),
      height: 110.0,
      decoration: new BoxDecoration(
        gradient: new LinearGradient(
          colors: <Color>[
            new Color(0x00736AB7),
            new Color(0xFF736AB7)
          ],
          stops: [0.0, 0.9],
          begin: const FractionalOffset(0.0, 0.0),
          end: const FractionalOffset(0.0, 1.0),
        ),
      ),
    );
  }
{{< / highlight>}}

This is very similar to the gradient we made on the toolbar chapter. We added a top margin to make it fit to the image bottom, realize that the top margin and the height add up to 300dp, the exact height of the image. We also use 0.0 and 0.9 as stops to make the gradient achieve the solid part at the 90%, as otherwise the border of the image was still slightly noticeable.

Now we have a smooother image integration into the background.

![planet detail gradient](/img/planet-detail-gradient.jpg)

## The content

Now the big part, the content itself. It will be scrollable on top of the image and the gradient, so out initial code will be like this:

{{< highlight dart "linenos=true">}}
  Widget _getContent() {
    return new ListView(
      padding: new EdgeInsets.fromLTRB(0.0, 72.0, 0.0, 32.0),
      children: <Widget>[
      ],
    );
  }
{{< / highlight>}}

This is a simple `ListView` to make the content scrollable. We add a padding to 72dp on the top to left space for the image and 32dp at the bottom to add a nice list ending.

The content of the ListView has two main parts, the top card and the text itself. If we analyze the top card, we realize is veeery similar to the one we created for the main list. The biggest difference is that the planet thumbnail is at the top instead of being at the left, and some margins and alignments are different. 

Could we reuse the same code that we made for the row? Sure. I've renamed the PlanetRow class to PlanetSummary. This is the resultant code with the changed lines different:

{{< highlight dart "linenos=true,hl_lines=6 8 17 34 43 46 62 72 86 87 88 89 106-114">}}
class PlanetSummary extends StatelessWidget {

  final Planet planet;
  final bool horizontal;

  PlanetSummary(this.planet, {this.horizontal = true});

  PlanetSummary.vertical(this.planet): horizontal = false;

  @override
  Widget build(BuildContext context) {

    final planetThumbnail = new Container(
      margin: new EdgeInsets.symmetric(
        vertical: 16.0
      ),
      alignment: horizontal ? FractionalOffset.centerLeft : FractionalOffset.center,
      child: new Hero(
          tag: "planet-hero-${planet.id}",
          child: new Image(
          image: new AssetImage(planet.image),
          height: 92.0,
          width: 92.0,
        ),
      ),
    );

    Widget _planetValue({String value, String image}) {
      return new Container(
        child: new Row(
          mainAxisSize: MainAxisSize.min,
          children: <Widget>[
            new Image.asset(image, height: 12.0),
            new Container(width: 8.0),
            new Text(planet.gravity, style: Style.regularTextStyle),
          ]
        ),
      );
    }


    final planetCardContent = new Container(
      margin: new EdgeInsets.fromLTRB(horizontal ? 76.0 : 16.0, horizontal ? 16.0 : 42.0, 16.0, 16.0),
      constraints: new BoxConstraints.expand(),
      child: new Column(
        crossAxisAlignment: horizontal ? CrossAxisAlignment.start : CrossAxisAlignment.center,
        children: <Widget>[
          new Container(height: 4.0),
          new Text(planet.name, style: Style.headerTextStyle),
          new Container(height: 10.0),
          new Text(planet.location, style: Style.subHeaderTextStyle),
          new Container(
            margin: new EdgeInsets.symmetric(vertical: 8.0),
            height: 2.0,
            width: 18.0,
            color: new Color(0xff00c6ff)
          ),
          new Row(
            mainAxisAlignment: MainAxisAlignment.center,
            children: <Widget>[
              new Expanded(
                flex: horizontal ? 1 : 0,
                child: _planetValue(
                  value: planet.distance,
                  image: 'assets/img/ic_distance.png')

              ),
              new Container (
                width: 32.0,
              ),
              new Expanded(
                  flex: horizontal ? 1 : 0,
                  child: _planetValue(
                  value: planet.gravity,
                  image: 'assets/img/ic_gravity.png')
              )
            ],
          ),
        ],
      ),
    );


    final planetCard = new Container(
      child: planetCardContent,
      height: horizontal ? 124.0 : 154.0,
      margin: horizontal
        ? new EdgeInsets.only(left: 46.0)
        : new EdgeInsets.only(top: 72.0),
      decoration: new BoxDecoration(
        color: new Color(0xFF333366),
        shape: BoxShape.rectangle,
        borderRadius: new BorderRadius.circular(8.0),
        boxShadow: <BoxShadow>[
          new BoxShadow(
            color: Colors.black12,
            blurRadius: 10.0,
            offset: new Offset(0.0, 10.0),
          ),
        ],
      ),
    );


    return new GestureDetector(
      onTap: horizontal
          ? () => Navigator.of(context).push(
            new PageRouteBuilder(
              pageBuilder: (_, __, ___) => new DetailPage(planet),
              transitionsBuilder: (context, animation, secondaryAnimation, child) =>
                new FadeTransition(opacity: animation, child: child),
              ) ,
            )
          : null,
      child: new Container(
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
  }
}

{{< / highlight>}}

The changes applied to the class to make it reusable are highlighted. Let's review them:

* line 6-7: now we have two constructors. The usual one that sets `horizontal` as true, and another one named `Planet.vertical` that sets the value as false. We will use this boolean to know what to do in the whole class.

* line 17: simply align the image to the left or to the center depending on the value of horizontal.

* line 35: we added a simple container of 8dp width to make the separation a bit larger, otherwise the two values look too close when centered.

* line 43: depending on boolean, the card content will have a left margin of 76dp or a top margin of 42dp, and 16dp for all other values.

* line 46: another ternary operator to decide if items on the card should align left or center.

* lines 62 and 72: the flex parameter makes the items to expand as much as possible or to the content. In the case of horizontal, we want them to be in the left  half and in the right half, but for the vertical one, that looks ugly.

* lines 86 to 89: this is were half of the tricks are done. We setup different margins and height depending on the orientation. Try to play around with the numbers.

* lines 106 to 114: as we do not want the card in the detail to be clickable, we set the onClick callback to null if horizontal is false.

And now we can left the invocation from the list as it is (because the default constructor sets `horizontal` to true) and in we just need to add a `new PlannetSumary.vertical(planet)` to our ListView.

There is still room for improvement, like moving the navigation code out of the component, moving all the ternary operators to values defined in the constructors, etc. But I left this as an exercise.

Another small reuse we can do is to convert the blue separator to a Widget as we will reuse in the content. That's a simple one:

{{< highlight dart "linenos=true">}}
class Separator extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return new Container(
        margin: new EdgeInsets.symmetric(vertical: 8.0),
        height: 2.0,
        width: 18.0,
        color: new Color(0xff00c6ff)
    );
  }
}
{{< / highlight>}}

Finally, we can construct the whole list. It will look as this:

{{< highlight dart "linenos=true">}}
  Widget _getContent() {
    final _overviewTitle = "Overview".toUpperCase();
      return new ListView(
        padding: new EdgeInsets.fromLTRB(0.0, 72.0, 0.0, 32.0),
        children: <Widget>[
          new PlanetSummary(planet,
            horizontal: false,
          ),
          new Container(
            padding: new EdgeInsets.symmetric(horizontal: 32.0),
            child: new Column(
              crossAxisAlignment: CrossAxisAlignment.start,
              children: <Widget>[
                new Text(_overviewTitle, style: Style.headerTextStyle,),
                new Separator(),
                new Text( planet.description, style: Style.commonTextStyle),
              ],
            ),
          ),
        ],

    );
  }
{{< / highlight>}}

Basically, we declare a variable for the "Overview" title and we fill the `ListView` with two widgets: a `PlanetSummary` and a `Container`. We use the container to give a 32dp padding in both sides. We put a `Column` inside. We use a crossAxisAlignment of `start` to align it to the left (or right in RTL language) and put the two texts and the separator inside. Easy peasy.

Check the repository for the text styles as I did a bit of refactor of the TextStyles class.

The result right now is this:

![planet detail](/img/planet-detail-content.jpg)

## Back button

Last part is simple, add a back button, something we already did for our original toolbar.

{{< highlight dart "linenos=true">}}
  Container _getToolbar(BuildContext context) {
    return new Container(
            margin: new EdgeInsets.only(
                top: MediaQuery
                    .of(context)
                    .padding
                    .top),
            child: new BackButton(color: Colors.white),
          );
  }
{{< / highlight>}}

As you can see is just a `Container` with a `BackButton`. We force the white color to the button, and we add a margin from `MediaQuery` to put under the notification bar.

## That's all folks!

Now your detail should look as intended. Check the final result at the repository at branch [Lesson_6_planets_Flutter_planets_detail](https://github.com/sergiandreplace/flutter_planets_tutorial/tree/Lesson_6_Planets_Flutter_Planet_detail)

## To NOT be continued

So, this is the end!

I've made the Planets app to grow up to the original idea I had in mind. Of course there is still for improvement, but as this tutorial was intended to explain UI things, is good to finish it here and move to new examples more suitables to explain other things. Firebase integration? Backend access? MVP? Redux? who knows... but I have some ideas. Also, I want to write some short posts about specific Flutter things and mix the Flutter content with other things (mainly Google Assistant with DialogFlow, my other toy at this moment).

A big thanks to all of you who followed this tutorial up to this point. I've seen some of you create other apps using this tutorial as template, keep working and please, show me your results!

And again, a big thanks to Vijay Verma for his great design that inspired me.

Stay tuned for more tutorials and articles!