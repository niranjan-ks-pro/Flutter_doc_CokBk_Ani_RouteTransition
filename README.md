# Flutter_doc_CokBk_Ani_RouteTransition
 https://docs.flutter.dev/cookbook/animation/page-route-animation

 Animate a page route transition
Cookbook
Animation
Animate a page route transition
A design language, such as Material, defines standard behaviors when transitioning between routes (or screens). Sometimes, though, a custom transition between screens can make an app more unique. To help, PageRouteBuilder provides an Animation object. This Animation can be used with Tween and Curve objects to customize the transition animation. This recipe shows how to transition between routes by animating the new route into view from the bottom of the screen.

To create a custom page route transition, this recipe uses the following steps:

Set up a PageRouteBuilder
Create a Tween
Add an AnimatedWidget
Use a CurveTween
Combine the two Tweens
1. Set up a PageRouteBuilder
To start, use a PageRouteBuilder to create a Route. PageRouteBuilder has two callbacks, one to build the content of the route (pageBuilder), and one to build the route’s transition (transitionsBuilder).

 Note: The child parameter in transitionsBuilder is the widget returned from pageBuilder. The pageBuilder function is only called the first time the route is built. The framework can avoid extra work because child stays the same throughout the transition.

The following example creates two routes: a home route with a “Go!” button, and a second route titled “Page 2”.

content_copy
import 'package:flutter/material.dart';

void main() {
  runApp(
    const MaterialApp(
      home: Page1(),
    ),
  );
}

class Page1 extends StatelessWidget {
  const Page1({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: Center(
        child: ElevatedButton(
          onPressed: () {
            Navigator.of(context).push(_createRoute());
          },
          child: const Text('Go!'),
        ),
      ),
    );
  }
}

Route _createRoute() {
  return PageRouteBuilder(
    pageBuilder: (context, animation, secondaryAnimation) => const Page2(),
    transitionsBuilder: (context, animation, secondaryAnimation, child) {
      return child;
    },
  );
}

class Page2 extends StatelessWidget {
  const Page2({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(),
      body: const Center(
        child: Text('Page 2'),
      ),
    );
  }
}
2. Create a Tween
To make the new page animate in from the bottom, it should animate from Offset(0,1) to Offset(0, 0) (usually defined using the Offset.zero constructor). In this case, the Offset is a 2D vector for the ‘FractionalTranslation’ widget. Setting the dy argument to 1 represents a vertical translation one full height of the page.

The transitionsBuilder callback has an animation parameter. It’s an Animation<double> that produces values between 0 and 1. Convert the Animation into an Animation using a Tween:

content_copy
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  const begin = Offset(0.0, 1.0);
  const end = Offset.zero;
  final tween = Tween(begin: begin, end: end);
  final offsetAnimation = animation.drive(tween);
  return child;
},
3. Use an AnimatedWidget
Flutter has a set of widgets extending AnimatedWidget that rebuild themselves when the value of the animation changes. For instance, SlideTransition takes an Animation<Offset> and translates its child (using a FractionalTranslation widget) whenever the value of the animation changes.

AnimatedWidget Return a SlideTransition with the Animation<Offset> and the child widget:

content_copy
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  const begin = Offset(0.0, 1.0);
  const end = Offset.zero;
  final tween = Tween(begin: begin, end: end);
  final offsetAnimation = animation.drive(tween);

  return SlideTransition(
    position: offsetAnimation,
    child: child,
  );
},
4. Use a CurveTween
Flutter provides a selection of easing curves that adjust the rate of the animation over time. The Curves class provides a predefined set of commonly used curves. For example, Curves.easeOut makes the animation start quickly and end slowly.

To use a Curve, create a new CurveTween and pass it a Curve:

content_copy
var curve = Curves.ease;
var curveTween = CurveTween(curve: curve);
This new Tween still produces values from 0 to 1. In the next step, it will be combined the Tween<Offset> from step 2.

5. Combine the two Tweens
To combine the tweens, use chain():

content_copy
const begin = Offset(0.0, 1.0);
const end = Offset.zero;
const curve = Curves.ease;

var tween = Tween(begin: begin, end: end).chain(CurveTween(curve: curve));
Then use this tween by passing it to animation.drive(). This creates a new Animation<Offset> that can be given to the SlideTransition widget:

content_copy
return SlideTransition(
  position: animation.drive(tween),
  child: child,
);
This new Tween (or Animatable) produces Offset values by first evaluating the CurveTween, then evaluating the Tween<Offset>. When the animation runs, the values are computed in this order:

The animation (provided to the transitionsBuilder callback) produces values from 0 to 1.
The CurveTween maps those values to new values between 0 and 1 based on its curve.
The Tween<Offset> maps the double values to Offset values.
Another way to create an Animation<Offset> with an easing curve is to use a CurvedAnimation:

content_copy
transitionsBuilder: (context, animation, secondaryAnimation, child) {
  const begin = Offset(0.0, 1.0);
  const end = Offset.zero;
  const curve = Curves.ease;

  final tween = Tween(begin: begin, end: end);
  final curvedAnimation = CurvedAnimation(
    parent: animation,
    curve: curve,
  );

  return SlideTransition(
    position: tween.animate(curvedAnimation),
    child: child,
  );
}
