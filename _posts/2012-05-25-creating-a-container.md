---
layout: post
title: How to Create a Container View Controller that Works with Storyboard
tags: ios
---
I started writing iOS apps years ago, but gave up the part-time work for a full-time job and school. Recently, I returned to the iOS SDK, which has come a long, long way in the past years. 

Mainly, the Storyboarding feature is a dream, especially when used with the provided Container View Controllers like the UINavigationViewController or the UITabBarViewController. All of this is awesome with one exception; there is no way to easily create your own Storyboard compatible [Container View Controller](http://developer.apple.com/library/ios/#featuredarticles/ViewControllerPGforiPhoneOS/AboutViewControllers/AboutViewControllers.html#//apple_ref/doc/uid/TP40007457-CH112-SW17).

This shortcoming is strange considering what lengths the SDK as gone through to emphasize good OO and MVC patterns. Despite the shortcoming, it is still possible to create Container View Controller that works with Storyboard -- thanks to the UIStoryboard's [instantiateViewControllerWithIdentifier](http://developer.apple.com/library/ios/documentation/UIKit/Reference/UIStoryboard_Class/Reference/Reference.html#//apple_ref/occ/instm/UIStoryboard/instantiateViewControllerWithIdentifier:).

In your storyboard, select a View Controller that will be a child of the Container ViewController. Open the Utilities pane (the pane that opens on the right side) and navigate to the Attributes inspector (the tab with the slider icon). There should be a field for a **Title** and an **Identifier**. The **Identifier** is the string you will use to instantiate this ViewController in the Container View Controller.

Once you have set the **Identifier** for all the child View Controller you need to add some code in the Container View Controller to add these View Controllers to the [childViewControllers]() collection. It is important to use the UIViewController's interface for managing child View Controllers so that rotation methods are forwarded the the appropriate controller as well as modal transitions. Apple has a [small section](http://developer.apple.com/library/ios/documentation/UIKit/Reference/UIViewController_Class/Reference/Reference.html#//apple_ref/doc/uid/TP40006926-CH3-SW81) describing these methods in their documentation.

In the end you should have some code that looks like this:

{% highlight objc %}
    UIViewController* child = [self.storyboard instantiateViewControllerWithIdentifier:identifier];
  
    if (child != nil) {
      [self addChildViewController:child];
    } else {
      NSLog(@"No UIVC found with that identifier.");
    }
{% endhighlight %}

Note that self.storyboard isn't set from an **init** call. At the moment, there isn't a great place to put this initialization code. One option is to override **setStoryboard:** and another is **viewDidLoad**. The problem with both options is that you should guard the **addChildViewController:** call to make sure that View Controllers aren't instantiated twice.

{% highlight objc %}
    - (void)viewDidLoad {
      if ([self.childViewControllers count] == 0) {
        // add child vcs
        [self addChildViewController:self.firstChild];
        [self addChildViewController:self.secondChild];
      
        // init the view or views
        [self.view addSubview:self.firstChild.view];
        self.firstChild.view.frame = self.view.bounds;
      }
    }
{% endhighlight %}

I use properties to wrap the instantiation of the view controllers and to store a named reference to View Controllers when they provide distinct functionality. For example, I have a view that displays a chat interface and a drop down tray that displays some UITableView information.

{% highlight objc %}
    - (ChatViewController*)chatVC {
      if (!self->_chatVC) {
        self->_chatVC = [self.storyboard instantiateViewControllerWithIdentifier:(NSString*)CHAT_SB_ID];
      }
    
      return self->_chatVC;
    }
{% endhighlight %}
  
In this case the chatVC property has to be a strong reference, so be sure to clean all of this up in the **viewDidUnload** method.

{% highlight objc %}
    - (void)viewDidUnload {
      [self.chatVC removeFromParentViewController];
      self->_chatVC = nil;
    }
{% endhighlight %}

I couldn't find any pattern that worked easily with the Storyboard. If anybody has an alternative or feedback, get my on [twitter](http://www.twitter.com/squinlan).