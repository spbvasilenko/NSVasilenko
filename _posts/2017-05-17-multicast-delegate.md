---
layout: post
title: Multicast Delegate
excerpt: "About how to implement a simple and beautiful multicast delegate on the swift and how to use it."
categories: [swift, architecture]
comments: true
---

Hello! Today I would like to talk about the implementation of the multicast delegate to the swift and its use.

Imagine this case, you have to make your class a singleton that has a delegate, but at the same time we become hostage to what will happen if we want to subscribe to this singleton delegate in different places? Correctly! It will not work!
There are many solutions how to fix this problem. For example, use notification, but I think it's the worst out of all. If we don't want to give up the delegate pattern in this case, I would have proposed to implement the multicast delegate. How to do it? In this we will now look and see how to use it in practice.

So, what is multicast delegate? It's a delegate that points to several methods.

First, we need to understand for yourself that should be able to multicast delegate class?
Assume that:
- Add a new delegate
- To remove a delegate from storage
- Invoke delegates

Secondly, as our multicast delegate will store weak references to the delegates? For this we can use NSHashTable is modeled after NSSet but provides different options, in particular to support weak relationships.

So let's start!

- Create a generic class MulticastDelegate, which has the property of delegates of type NSHashTable:

{% highlight swift %}
class MulticastDelegate<T> {
	private let delegates: NSHashTable<AnyObject>
}
{% endhighlight %}

- Implement the initializer. Note the parameter `strongReferences` - you ask "Why is that? Delegates always weak?". No, not always. For example, delegates can be strong for CAAnimation delegate.

{% highlight swift %}
init(strongReferences: Bool = false) {
    delegates = strongReferences ? NSHashTable<AnyObject>(): NSHashTable<AnyObject>.weakObjects()
}
{% endhighlight %}

- Now we can implement methods to add and remove delegates:

{% highlight swift %}
func addDelegate(_ delegate: T) {
    delegates.add(delegate as AnyObject)
}
    
func removeDelegate(_ delegate: T) {
    delegates.remove(delegate as AnyObject)
}
{% endhighlight %}

- It now remains to implement invoking delegates. The method that the parameters will make the unit that accepts a delegate:

{% highlight swift %}
func invokeDelegates(_ invocation: (T) -> ()) {
    for delegate in delegates.allObjects {
        invocation(delegate as! T)
    }
}
{% endhighlight %}

- Some more swift syntax improvements:

{% highlight swift %}
func +=<T> (left: AFMulticastDelegate<T>, right: T) {
    left.addDelegate(right)
}

func -=<T> (left: AFMulticastDelegate<T>, right: T) {
    left.removeDelegate(right)
}

precedencegroup MulticastPrecendence {
    associativity: left
    higherThan: TernaryPrecedence
}
infix operator |> : MulticastPrecendence
func |><T> (left: AFMulticastDelegate<T>, right: (T) -> ()) {
    left.invokeDelegates(right)
}
{% endhighlight %}

We're done. Well, now how do I use it? It's simple!

{% highlight swift %}
class MessagesService: {
    let delegate = MulticastDelegate<MessagesServiceDelegate>()
}
{% endhighlight %}

{% highlight swift %}
// Create service instance
let messagesService = MessagesService.instance
// Subscribe to service delegate
messagesService.delegate += self
{% endhighlight %}

{% highlight swift %}
// In any place of our service:
delegate |> { $0.messagesService(self, didReceivedMessage: channelMessage, inChannel: channel) }
{% endhighlight %}

Looks great!
