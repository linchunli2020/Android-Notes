滑动冲突的解决方式:

1.外部拦截法

所谓外部拦截法是指点击事件都先经过父容器的拦截处理，如果父容器需要此事件就拦截，如果不需要此事件就不拦截，这样就可以解决滑动冲突的问题。这种方法比较符合点击事件的分发机制。
外部拦截法需要重写父容器的onInterceptTouchEvnent方法，在内部做相应的拦截即可，这种方法的伪代码如下所示；

<img width="419" alt="image" src="https://user-images.githubusercontent.com/67937122/161198084-de55089a-c7fe-4f9c-b1a2-f3dd533317cb.png">


上述代码是外部拦截法的典型逻辑，针对不同的滑动冲突，只需要修改父容器需要当前点击事件这个条件即可，其他均不需做修改并且也不能修改。这里对上述代码再描述一下，在onInterceptTouchEvent方法中，

首先是ACTION_DOWN这个事件，父容器必须返回false，即不拦截ACTION_DOWN事件，这是因为一旦父容器拦截了ACTION_DOWN，那么后续的ACTION_MOVE和AVTION_UP事件都会直接交由父容器处理，这样事件没法再传递给子元素了；

其次是ACTION_MOVE事件，这个事件可以根据需要来决定是否拦截，如果父容器需要拦截就返回true，否则返回false；		    

最后是ACTION_UP事件，这里必须要返回false，因为ACTION_UP事件本身没有太多意义。

考虑一种情况，假设事件交给子元素处理，如果父容器在ACTION_UP时返回了true，就会导致子元素无法接收到ACTION_UP事件，这个时候子元素中的onClick事件就无法触发，但是父容器比较特殊，
一旦它开始拦截任何一个事件，那么后续的事件都会交给它来处理，而ACTION_UP作为最后一个事件也必定可以传递给父容器，即便父容器的onInterceptTouchEvent方法在ACTION_UP时返回了false。


2.内部拦截法

内部拦截法是指父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就交给父容器进行处理，这种方法和Android中的事件分发机制不一致，
需要配合requestDisallowInterceptTouchEvent方法才能正常工作，使用起来较外部拦截法稍显复杂。它的伪代码如下，我们需要重写子元素的dispatchTouchEvent方法：

<img width="491" alt="image" src="https://user-images.githubusercontent.com/67937122/161198148-85b85eef-71d9-4b90-89d0-75de919d80d0.png">



上述代码是内部拦截法的典型代码，当面对不同的滑动策略时只需要修改里面的条件即可，其他不需要做改动而且也不能有改动。除了子元素需要做处理外，
父元素也要默认拦截除了ACTION_DOWN之外的其他事件，这样当子元素调用parent.requestDisallowInterceptTouchEvent(false)方法时，父元素才能继续拦截所需的事件。

为什么父容器不能拦截ACTION_DOWN事件呢？

因为ACTION_DOWN事件并不受FLAG_DISALLOW_INTERCEPT这个标记位的控制，所以一旦父容器拦截ACTION_DOWN事件，那么所有的事件都无法传递到子元素中去，这样内部拦截就无法起作用了。






