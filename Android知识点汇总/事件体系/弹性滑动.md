为了避免 滑动的生硬，可以采用弹性滑动，提高用户体验。这里主要有 ：Scroller、动画、延时三种方式。

如下是 Scroller的典型使用，主要是 invalidate方法起的作用。

        Scroller mScroller = new Scroller(context);

        /**
         * 滑动到指定位置
         *
         * @param destX  X 滑动距离
         * @param destY  Y 滑动距离
         */
        private void smoothScrollTo(int destX, int destY) {
            //滑动起点X
            int scrollX = getScrollX();
            //滑动起点Y
            int scrollY = getScrollY();
            //1000 ms内慢慢滑向 （destX，destY）
            mScroller.startScroll(scrollX, scrollY, destX, destY, 1000);
            //重绘
            invalidate();
        }
         /**
         * 使View 不断重绘
         */
        @Override
        public void computeScroll() {
            /**
             *  computeScrollOffset 方法通过时间流逝百分比计算 scrollX和scrollY 
             *  返回true 表示滑动未结束
             */
            if (mScroller.computeScrollOffset()) {
                //滑动到当前位置，通过小幅度滑动实现弹性滑动
                scrollTo(mScroller.getCurrX(), mScroller.getCurrY());
                //再次重绘
                postInvalidate();
            }
        }


如下 Scroller 的滑动原理（相关方法的调用过程）：

<img width="605" alt="image" src="https://user-images.githubusercontent.com/67937122/161180995-756e6bb5-c3e9-4e56-8653-16c19d7f0368.png">


对于延时达到弹性滑动，主要是利用 了Handler 或者 View 的 postDelayed 方法，或者线程的 sleep方法。




