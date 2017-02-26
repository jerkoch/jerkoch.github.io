---
layout: post
title:  "Swiper, No Swiping..."
description: An adventure creating a swipeable UITableViewCell based on the stock Mail.app, implemented in Swift.
image: /assets/images/swipecellkit-hero.png
date:   2017-02-07
disqus: true
---

...right on a `UITableViewCell`.

Yes, I know I'm late to the cell swiping party. You know what they say: necessity is the mother of invention.

#### Backstory

My journey started innocently enough. I simply wanted to add a single swipe action to the left side of a `UITableViewCell`, but a harsh reality quickly set in.

If you have dreams of enabling *Mail.app*-like gesture features with only a few lines of code, you will quickly be disappointed to learn that the existing `UITableViewRowAction` class is very limited. You can change the text and background color for an action, but no icons, no swiping right, and no dragging to perform destructive or selection type behavior. 

No problem, I figured a quick search on Stack Overflow would help brighten my day. That led me to [MGSwipeTableCell](https://github.com/MortimerGoro/MGSwipeTableCell) and [SWTableViewCell](https://github.com/CEWendel/SWTableViewCell). Both have great APIs, are very popular, and provide many customizations to make most developers happy.  I decided to take *MGSwipeTableCell* for a spin.

One thing you need to know about me - I'm a stickler for the details. Visually, *MGSwipeTableCell* got me what I needed, but the physics just didn't feel right. It was only *after* I ported it to Swift (yes, you read that right) that I realized its animation timings were all fixed and did not take into account things such as gesture velocity. Looking back at the *SWTableViewCell* demo app also left me unsatisfied.

It was then that I took a step back and accepted the fact that I was about to roll my own swipeable `UITableViewCell`. And of course, I would write it in Swift. I spent a long time analyzing the iOS 10 Mail.app swiping behavior - slow motion screen recordings and all.  I also looked at the News.app, which has similar swipeable cell behavior, although it uses a `UICollectionView`. Either way, they are both very similar, with some slight differences in transition animations and the highlighting style when tapping action buttons.

I wanted to write something that would not only provide a great user experience, but also provide an equally as great developer experience. And the same principles applied: clarity, consistency, and delight. *Let's put the discoverability debate aside for a moment :)*

In order to support both the Mail.app and News.app cell swiping styles, my three main objectives were: 

1. Left and Right swiping support with configurable transitions as the actions are exposed
2. Draggable cells past a threshold triggering auto-expansion of the default action
3. Flexible action button styling (text attributes, support images, etc.)

My first implementation used `UIScrollView`. It seemed like the obvious choice.  But it wasn't long before the technical debt started to accrue. This came in the form of hacks trying to handle dragging past thresholds and handling the extents of the scroll view's `contentView`, among many other things. More importantly, however, the physics still didn't feel the same as my two benchmark apps - and that's what I cared about most.

I'm not sure what this says about my developer-chops, but I'm constantly trying to write as little code as possible. Usually, spending way more time searching for the *right* way rather than the *quickest* way.  I justify it as an investment in any future code that I write.  

I had never really gotten my hands dirty with `UIPanGestureRecognizer`.  I knew that throwing out `UIScrollView` meant giving up all its cozy abstractions, and that would put me one more layer out of my comfort zone. But with great risk comes great reward! 

The next few days were a bit of struggle. I kept at it and managed to get the basics working. I was definitely getting closer. It was when I stumbled back upon [Building Interruptible and Responsive Interactions](https://developer.apple.com/videos/play/wwdc2014/236/) from WWDC 2014 that I knew I was on the right track. Seamlessly transitioning from gestures to animations was the key achieving the consistency with the Mail.app that I was looking for. The code quickly began to flow. 

![](https://raw.githubusercontent.com/jerkoch/SwipeCellKit/develop/Screenshots/Expansion-Selection.gif)

After the animations were implemented using the shiny new `UIViewPropertyAnimator` class, I spent a lot of time ensuring that the elasticity felt accurate when dragging beyond allowable thresholds. I also made sure that the drag thresholds for selection and destruction behaviors matched the Mail.app.  

![](https://raw.githubusercontent.com/jerkoch/SwipeCellKit/develop/Screenshots/Transition-Border.gif)

For transitions, I started with the Mail.app *border* style. Once that was working, I added support for a *reveal* style transition like the News.app uses, and then added a *drag* style for good measure. 

After adding some final polish by integrating Apple's new Haptic Feedback API, it was time to call it a day and send my creation out into the world.

#### The Final Product

Introducing [SwipeCellKit](https://github.com/jerkoch/SwipeCellKit). A swipeable UITableViewCell based on the stock Mail.app, implemented in Swift.

![](https://raw.githubusercontent.com/jerkoch/SwipeCellKit/develop/Screenshots/Hero.gif)

A swipeable UITableViewCell with support for:

* Left and right swipe actions
* Action buttons with: *text only, text + image, image only*
* Haptic Feedback
* Customizable transitions: *Border, Drag, and Reveal*
* Animated expansion when dragging past threshold

Integration is simple. Set the `delegate` property on `SwipeTableViewCell`:

````swift
func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
    ...
    cell.delegate = self
    return cell
}
````

Then adopt the `SwipeTableViewCellDelegate` protocol:

````swift
func tableView(_ tableView: UITableView, editActionsForRowAt indexPath: IndexPath, for orientation: SwipeActionsOrientation) -> [SwipeAction]? {
    guard orientation == .right else { return nil }

    let deleteAction = SwipeAction(style: .destructive, title: "Delete") { action, indexPath in
        // handle action by updating model with deletion
    }

    // customize the action appearance
    deleteAction.image = UIImage(named: "trash")

    return [deleteAction]
}
````

Optionally, you call implement the `editActionsOptionsForRowAt` method to customize the behavior of the swipe actions:

````swift
func tableView(_ tableView: UITableView, editActionsOptionsForRowAt indexPath: IndexPath, for orientation: SwipeActionsOrientation) -> SwipeTableOptions {
    var options = SwipeTableOptions()
    options.expansionStyle = .destructive
    options.transitionStyle = .border
    return options
}
````

#### Conclusion

So there you have it!  A few years too late, but hopefully *SwipeCellKit* is still valuable to some people out there.

All source code for *SwipeCellKit* is available on [GitHub](https://github.com/jerkoch/SwipeCellKit). It includes a [Demo](https://github.com/jerkoch/SwipeCellKit/tree/develop/Example) app which shamelessly copies the Mail.app *Inbox* interface. 

You can read the full [documentation](http://www.jerkoch.com/SwipeCellKit) which was generated with [Jazzy](https://github.com/realm/jazzy).

Please try it out! Pull requests welcome.