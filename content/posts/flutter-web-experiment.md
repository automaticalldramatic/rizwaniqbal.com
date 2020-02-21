---
title: "Flutter Web Experiment"
date: 2020-02-21T14:51:04+01:00
draft: false
toc: true
images: 
  - /images/hummingbird-736890_1920.jpg
tags: 
  - flutter
  - experiment
---

A few months back I was really intrigued by Flutter and started taking a keen interest in building a cross platform app in Dart on the Flutter framework. This led to a small repository to consider using Flutter as the common codebase for Web and Mobile if I were to build a product that needed to leverage this.

## Context

For me the web has always been the strongest use case, owing to the products I have worked on and the businesses I have been involved in. So, the desktop and the browser was the primary candidate and mobile apps were something that I needed to support. I started this learning experiment from the point of view of building a SaaS product that is primarily available via a browser but also has apps available on the Apple App Store and Google Play Store. 

I wanted to see how Flutter stands against React Native and other frameworks and paradigms people use for cross platform development. I wasn't looking for a silver bullet but to understand the pains a developer faces maintaining such codebases and what trade-offs there are.

Also, a friend and ex-colleague [Salih](https://salih.dev) was instrumental in pulling me towards this direction.

> All the code shown in this post is available at https://github.com/automaticalldramatic/flutter-web-responsive

## Process

I started understanding this eco system and how Flutter came about. Its almost impossible to talk about Flutter web without mentioning Angular Dart.

I started reading about Dart and landed on a lot of great videos on Angular Dart. One guy in particular, Evgeny Gusev from Wrike and [his talk on Angular Dart](https://www.youtube.com/embed/0mHspoS5Zf8) inspired me a lot.

However, after reading about the various projects, pros and cons and spending a couple of days setting up Angular Dart, I moved on to Hummingbird, as it was called then and I believe still retains that project name. My very short, very biased verdict on Angular Dart was:

> It looks like a framework for bigger teams. It was a pain running it locally and I  did not find enough packages for firebase. In the end I thought it will be best to just use flutter. 

Maybe I will write another post about my experience with Angular Dart, cause it left me with a lot of mixed feelings. Most developers on Reddit and the Flutter slack channel spoke about moving angulardart code to flutter web and I moved on as well.


### Structure

The first task at hand was to structure my project such that I could have a different template for mobile web. 

I was building a landing page as part of the experiment to understand how layouts work and how I could use the BLOC pattern in the future.

There is one screen that is the home page, to keep the structure failrly flat.

`main.dart` right now creates a theme for the app and loads the home page from `home_page.dart`.

```dart
main() {
  runApp(MyApp());
}

class MyApp extends StatelessWidget {
  const MyApp({Key key}) : super(key: key);
  
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      debugShowCheckedModeBanner: false,
      initialRoute: '/',
      theme: ThemeData(
        fontFamily: 'Nunito',
        primaryColorDark: Colors.black,
      ),
      home: HomePage(),
      routes: {
        '/#': (context) => HomePage(),
        '/terms': (context) => TermsPage(),
        '/privacy': (context) => PrivacyPage(),
      },
    );
  }
}
```

If you notice I am using a `Stateless` widget for my App. These are useful when the part of the user interface you are describing does not depend on anything other than the configuration information in the object itself and the BuildContext in which the widget is inflated. For almost all real world use cases, I reckon you would use a `SatefulWidget`.

I also created a `Theme` in place. Usually, you could load a theme while intialising the app based on some flag. This code also maps routes for the `HomePage()` widget and other widgets.

```dart
class HomePage extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return ResponsiveWidget(
      largeScreen: LargeScreen(),
      smallScreen: SmallScreen(),
    );
  }
}
```

The home page loads a widget called `ResponsiveWidget`. This widget uses the `LayoutBuilder` widget to create breakpoints and return the appropriate screen layout. Any component (a component is a widget, which in my terminology is a higher level widget using bloc providers and services) using the responsive widget can use it to define how it renders itself on different screen layouts.

```dart
import "package:flutter/material.dart";

class ResponsiveWidget extends StatelessWidget {
	final Widget largeScreen;
	final Widget mediumScreen;
	final Widget smallScreen;
	
	const ResponsiveWidget(
		  {Key key,
			  @required this.largeScreen,
			  this.mediumScreen,
			  this.smallScreen})
		  : super(key: key);
	
	static bool isSmallScreen(BuildContext context) {
		return MediaQuery.of(context).size.width < 800;
	}
	
	static bool isLargeScreen(BuildContext context) {
		return MediaQuery.of(context).size.width > 800;
	}
	
	static bool isMediumScreen(BuildContext context) {
		return MediaQuery.of(context).size.width >= 800 &&
			  MediaQuery.of(context).size.width <= 1200;
	}
	
	@override
	Widget build(BuildContext context) {
		return LayoutBuilder(
			builder: (context, constraints) {
				if (constraints.maxWidth > 1200) {
					return largeScreen;
				} else if (constraints.maxWidth <= 1200 && constraints.maxWidth >= 800) {
					return mediumScreen ?? largeScreen;
				} else {
					return smallScreen ?? largeScreen;
				}
			},
		);
	}
}
```

As you can see that the changes would flow through the deision tree. This might not be the right way to do it, but for someone coming from the web wanting to make this work across breakpoints, this was the cleanest way I knew how to answer this use case.

This is how the code for LargeScreen looks like:

```dart
class LargeScreen extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
      body: Stack(
        alignment: AlignmentDirectional.topStart,
        children: <Widget>[
          //////////////////////////////////
          // separate scrolling widget for grayline
          //////////////////////////////////
          Container(
            margin:
                EdgeInsets.only(top: MediaQuery.of(context).size.height * 0.9),
            child: SingleChildScrollView(
                controller: ScrollController(initialScrollOffset: 0.0),
                scrollDirection: Axis.horizontal,
                child: Row(
                  children: <Widget>[
//                    GrayLine(),
                  ],
                )),
          ),
          //////////////////////////////////
          // Container for our main view port
          //////////////////////////////////
          Container(
            child: SingleChildScrollView(
              child: Column(
                mainAxisAlignment: MainAxisAlignment.start,
                crossAxisAlignment: CrossAxisAlignment.stretch,
                children: <Widget>[
                  Container(
                    padding: EdgeInsets.fromLTRB(
                      MediaQuery.of(context).size.width * 0.15,
                      MediaQuery.of(context).size.height * 0.1,
                      MediaQuery.of(context).size.width * 0.15,
                      MediaQuery.of(context).size.height * 0.1,
                    ),
                    child: Column(
                      children: <Widget>[
                        LargeNavheader(),
                        SizedBox(
                          height: MediaQuery.of(context).size.height * 0.05,
                        ),
                        LargeHero(),
                        SizedBox(
                          height: MediaQuery.of(context).size.height * 0.05,
                        ),
                        DesktopScreenshot(),
                        SizedBox(
                          height: MediaQuery.of(context).size.height * 0.05,
                        ),
                        VisionProductLarge(),
                      ],
                    ),
                  ),
                  Footer(color: colorFromHex("#6E67D0")),
                  Terms(),
                ],
              ),
            ),
          ),
        ],
      ),
    );
  }
}
```

If you run the project, you would notice that this is a custom design that I created to test out a few things am used to having problems on the web. Like, an element absolutely positioned with a different scroll speed or a CSS grid like setup to display elements.

To wrap the main view, we could use an `InheritedWidget`, but since we aren't looking at change management and building a fairly straight forward static website served with Flutter, I have used a `Scaffold` widget.

This `Scaffold` widget creates stacks using `Stack` widgets or just plain scrolling views using `SingleChildScrollView` widget and builds the home page structure for different screen sizes on it.

Right now I have divided the screens into `_page` files but they should be in a screens directory to give it some sort of a structure.

At the end I ended up deciding that the best structure to follow would look something like this - 

```shell

        |
        |--screens
        |  |--home.dart
        |
        |--components
        |  |--header.dart
        |  |--hero.dart
        |
        |--themes  
        |  |--light.dart
        |  |--dark.dart
        |
        |--bloc
        |  |--
        |
        |--bloc_providers
        |  |--
        |
        |--models
        |  |--
        |
        |--services
        |  |--
        |
        |--main.dart


```

### Conclusions

At the end of this exercise I had a responsive website built on Flutter that could be packaged into an app for iOS and Android. But this is sort of a shell with only static content.

I really enjoyed working on Flutter and if I were to build a mobile app that required a performant app that shares a common code base, I would choose flutter over the other frameworks I have tried in the past. However, while saying this, I also understand that a lot depends on the environment, product and use cases you are trying to answer when taking a technology decision. This might not be the best one and this is not the post to discuss the pros and cons of Flutter vs others. 

Flutter, at the end of the day, is not a native platform for iOS or Android. As per the website, 
> Flutter doesn’t run code directly on the underlying platform; rather, the Dart code that makes up a Flutter app is run natively on the device, “sidestepping” the SDK provided by the platform.

So, based on the app requirements, it may or may not be the best choice.

For me, Flutter is matured, albeit for the web I would not recommend this as its too verbose and does not remotely give you the advantages that you get building on Web native technologies. I was still left desiring:
* Render speeds / fps on the browser - dart code compiled to JS is not as performant
* Pakcage sizes are too big
* SEO would be a constant problem for people desiring it
* And many others - like building a CMS around flutter seemed like a really bad idea to me

To summarize, I chose a stupid use case to tryout flutter, that is to build a static website. But this is an area, I feel, Flutter is not meant to shine in.