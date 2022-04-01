实现View的滑动主要有如下三种方式:

      scrollTo /scrollBy :适合对view 的内容改变；

      动画： 主要用于没有交互的View 和实现复杂的动画效果；

      改变布局参数：操作稍微复杂，适合有交互的View 。

1.1. 通过View 的 ScrollTo/ScrollBy 方法

      View 源码中的相关实现：
      
      /**
         * Set the scrolled position of your view. This will cause a call to
         * {@link #onScrollChanged(int, int, int, int)} and the view will be
         * invalidated.
         * @param x the x position to scroll to
         * @param y the y position to scroll to
         */
        public void scrollTo(int x, int y) {
            if (mScrollX != x || mScrollY != y) {
                int oldX = mScrollX;
                int oldY = mScrollY;
                mScrollX = x;
                mScrollY = y;
                invalidateParentCaches();
                onScrollChanged(mScrollX, mScrollY, oldX, oldY);
                if (!awakenScrollBars()) {
                    postInvalidateOnAnimation();
                }
            }
        }

          /**
         * Move the scrolled position of your view. This will cause a call to
         * {@link #onScrollChanged(int, int, int, int)} and the view will be
         * invalidated.
         * @param x the amount of pixels to scroll by horizontally
         * @param y the amount of pixels to scroll by vertically
         */
        public void scrollBy(int x, int y) {
            scrollTo(mScrollX + x, mScrollY + y);
        }


其中 scrollBy调用的是 scrollTo，它实现了使用当前位置的相对滑动，而scrollTo是基于所传参数的绝对滑动。在滑动过程中，mScrollX 的值等于 View 左边缘 和 View 内容左边缘在水平方向的距离，而 mScrollY 则是View 上边缘和 View 内容上边缘在竖直方向的距离。他们都是以像素单位。如果从 左往右/从上往下 滑动，mScrollX/mScrollY 为正。
scrollBy 和 scrollTo 只能改变 View 内容的位置而不能改变View 在布局中的位置。

如下滑动过程中，mScrollX/mScrollY 取值情况：

<img width="587" alt="image" src="https://user-images.githubusercontent.com/67937122/161180819-28b71f3c-fb2a-4bc8-ace9-03b274b4e339.png">



1.2. 使用动画

通过动画为View 添加平移效果，View 的 tanslationX 和 tanslationY 属性，可以采用传统的动画和属性动画。动画不能真正的改变 View 的位置，只是移动的是他的影像，如果在新位置有点击事件，则无效。但是在android 3.0以后属性动画解决了该问题。

1.3. 改变布局参数

通过改变View 的LayoutParams 使得 View 重新布局实现滑动。这里以 把 Button 水平移动 100px 为例。可以改变 Button 的 marginLeft,或者在其左边放一个宽度为0 的view,当要平移时改变他的宽度，使其被挤到右边（加入Button的父布局为LinearLayout），实现滑动。如下是改变 LayoutParams的方式：
      ViewGroup.MarginLayoutParams params = (ViewGroup.MarginLayoutParams) mButton.getLayoutParams();
      params.width += 100;
      params.leftMargin += 100;
      mButton.setLayoutParams(params);
      // 或者
      mButton.requestLayout();




