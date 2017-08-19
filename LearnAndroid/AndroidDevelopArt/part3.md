## 《Android开发艺术第三章》 ##

### View基础知识 ###

##### 什么是View? #####

View是android中所有控件的基类,用于界面层显示的控件，android中ViewGroup也继成View，所以View可以单独的一个控件也可以是组合的控件，通过他们关系组成View的树结构显示在界面中。

##### View的参数 #####

View的位置由四个定点决定(相对于父容器来说的)，分别对应着View的四个属性left top right bottom。left左上角的横坐标，top是左上角的纵坐标，right右下角的横坐标，bottom是右下角的纵坐标

- 宽高与坐标的关系以及如何这四个参数 

> width = right - left;
> height = bottom - top;
> 
> left  =getLeft()
> right =getRight()
> top   =getTop()
> bottom =getBottom()

##### View的额外参数 #####

从Android 3.0开始View增加了x，y，translationX，translationY 其中x,y是View的左上角的坐标而translationX和translationY是View的左上角相对于父容器的偏移量。这四个参数的是相对于父容器的坐标，并且translationX和translationY默认值都是0，并且View给它们get/set方法换算关系如下

> x = left + translationX
> y = top  + translationY

##### MotionEvent与TouchSlop #####

- MotionEvent:

 在手指触摸屏幕后所产生的一系列事件中，典型的时间类型有：

    1. ACTION_DOWN-手指刚接触屏幕
    
    2. ACTION_MOVE-手指在屏幕上移动
    
    3. ACTION_UP-手机从屏幕上松开的一瞬间
    
 正常情况下，一次手指触摸屏幕的行为会触发一系列点击事件，考虑如下几种情况：
    
    1. 点击屏幕后离开松开，事件序列为 DOWN -> UP
    
    2. 点击屏幕滑动一会再松开，事件序列为DOWN->MOVE->...->UP
    
 通过MotionEvent对象我们可以得到点击事件发生的x和y坐标，getX/getY返回的是相对于当前View左上角的x和y坐标，getRawX和getRawY是相对于手机屏幕左上角的x和y坐标。

- TouchSlop:

    TouchSlope是系统所能识别出的可以被认为是滑动的最小距离，获取方式是ViewConfiguration.get(getContext()).getScaledTouchSlope()。

##### VelocityTracker GetureDetector和Scroller #####

- VelocityTracker：用于追踪手指在滑动过程中的速度，包括水平和垂直方向上的速度。

```
    //初始化
	VelocityTracker mVelocityTracker = VelocityTracker.obtain();

	//在onTouchEvent方法中
	mVelocityTracker.addMovement(event);

	//获取速度
	mVelocityTracker.computeCurrentVelocity(1000);

	float xVelocity = mVelocityTracker.getXVelocity();
	//重置和回收

	mVelocityTracker.clear(); //一般在MotionEvent.ACTION_UP的时候调用

	mVelocityTracker.recycle(); //一般在onDetachedFromWindow中调用
```
速度计算公式：速度 = （终点位置 - 起点位置）/ 时间段

获取计算的数度时候需要注意两点：    

   1. 获取速度前必须先计算速度 即调用getXVelocity与getYVelocity前必须先调用computeCurrentVelocity（时间/毫秒数）。
   2. 速度是指一段时间内划过的的像素数，比如1000ms从水平方向划过100个像素，那么水平速度就是100，并且有正负之分 左到右为正，反之为负。
   3. 当不需要的时候，需要调用clear方法来重置并回收内存velocityTracker.clear();
velocityTracker.recycler();

- GetureDetector：

    手势检测，用于辅助检测用户的点击、滑动、长按、双击等行为。

    在日常开发中，比较常用的有:onSingleTapUp(单击)、onFling(快速滑动)、onScroll（拖动）、onLongPress(长按)、onDoubleTap(双击)。

    建议：如果只是监听滑动相关的事件在onTouchEvent中实现或者在view设置的onTouchListener 中的onTouch处理；如果要监听双击这种行为的话，那么就使用GestureDetector。


- Scroller：

    弹性滑动对象，用于View实现弹性滑动

### View滑动 ###

View的滑动可以通过三种方式实现

 - 第一种：通过View本身提供的scrollTo与scrollBy 的方法实现滑动
 - 第二种：通过动画实现给View施加平移效果来滑动
 - 第三种：通过改变View的的Layoutparams重置布局来实现

##### scrollTo/scrollBy #####

使用scrollTo/scrollBy scrollTo和scrollBy方法只能改变view内容的位置而不能改变view在布局中的位置。 scrollBy是基于当前位置的相对滑动，而scrollTo是基于所传参数的绝对滑动相对于父容器左上角。通过View的getScrollX和getScrollY方法可以得到滑动的距离（android开发艺术探索P131详细描述）。

##### 通过动画实现 #####

使用动画 使用动画来移动view主要是操作view的translationX和translationY属性，既可以使用传统的view动画，也可以使用属性动画，使用后者需要考虑兼容性问题，如果要兼容Android3.0一下版本系统的话推荐使用nineoldandroids。使用动画还存在一个交互问题：在android3.0以前的系统上，view动画和属性动画，新位置均无法触发点击事件，同时，老位置仍然可以触发单击事件。从3.0开始，属性动画的单击事件触发位置为移动后的位置，view动画仍然在原位置

##### 改变布局参数 #####

改变布局参数 通过改变LayoutParams的方式去实现View的滑动是一种灵活的方法。

##### 滑动对比 #####

- scrollTo/scrollBy：操作简单，适合对View的内容滑动。
- 动画：适于无交互的View（也可以有交互的View）和实现复杂的东化效果。
- 改变布局参数:操作复杂，一般用于有交互的View

### 弹性滑动 ###

1. 使用Scroller Scroller的工作原理：Scroller本身并不能实现view的滑动，它需要配合view的computeScroll方法才能完成弹性滑动的效果，它不断地让view重绘，而每一次重绘距滑动起始时间会有一个时间间隔，通过这个时间间隔Scroller就可以得出view的当前的滑动位置，知道了滑动位置就可以通过scrollTo方法来完成view的滑动。就这样，view的每一次重绘都会导致view进行小幅度的滑动，而多次的小幅度滑动就组成了弹性滑动，这就是Scroller的工作原理。

2. 使用延时策略 使用延时策略来实现弹性滑动，它的核心思想是通过发送一系列延时消息从而达到一种渐进式的效果，具体来说可以使用Handler的sendEmptyMessageDelayed(xxx)或view的postDelayed方法，也可以使用线程的sleep方法。 

3. 通过属性动画ValueAnimator来实现然后在addUpdateListener 中的onAnimationUpdate方法中拿到变化的数值可以通过设置scrollBy/scrollTo 或者用来该表Layoutparams达到滑动效果。

### View事件分发的机制 ###

事件分发机制的三个重要方法
   
   - public boolean dispatchTouchEvent(MotionEvent ev)
   - public boolean onInterceptTouchEvent(MotionEvent event)
   - public boolean onTouchEvent(MotionEvent event)
    
dispatchTouchEvent：用来进行事件的分发。如果事件能够传递给当前的View，那么此方法一定会被调用，返回结果受当前View的onTouchEvent和下级View的dispatchTouchEvent方法的影响，表示是否消耗当前事件。

onInterceptTouchEvent：在上述方法内部调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列当中，此方法不会被再次调用，返回结果表示是否拦截当前事件。

onTouchEvent：在dispatchTouchEvent方法中调用，用来处理点击事件，返回结果表示是否消耗当前的事件，如果不消耗，则在同一个事件序列中，当前View无法再次接受到事件。

##### 事件派发过程 #####
当一个点击事件/down事件产生时，他的传递过程如下循序：Activity->Window->View如果view的onTouchEvent 返回false 那么事件会传递给它的父容器的onTouchEvent 如果它也返回false则继续往上传递直到Activity的自己的onTouchEvent 中去如果Activity也是返回false这这一系列事件序就不会传递下去了。



##### View事件派发消费过程 #####
- **view dispatchTouchEvent 事件分析**： 当一个view需要处理事件时候，如果它设置了onTouchListener，那么onTouch方法会被调用。如果onTouch发回true时候那么view的onTouchEvent不会调用反之会被调用由此可见onTouchListener的优先级高于onTouchEvent，如果当前view有点击事件View 的onTouchListener的onTouch返回true则不会相应，反之会相应长点击事件也是一样， 
        public boolean dispatchTouchEvent(MotionEvent event) {
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(event, 0);
        }

        if (onFilterTouchEventForSecurity(event)) {
            //noinspection SimplifiableIfStatement
            ListenerInfo li = mListenerInfo;
            if (li != null && li.mOnTouchListener != null && (mViewFlags & ENABLED_MASK) == ENABLED
                    && li.mOnTouchListener.onTouch(this, event)) {//判断是否有触摸监听事件和监听事件的返回值
                return true;
            }

            if (onTouchEvent(event)) {//如果没有触摸监听事件或则触摸监听事件返回值为false则调用onTouchEvent
                return true;
            }
        }

        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onUnhandledEvent(event, 0);
        }
        return false;//如果onTouchEvent返回false这当前view派发结果就是false就是不消费当前这个事件down move up 中的一次事件
        //如果是down则后面move up事件父容器都不会往下派发了当前view就无法做出响应了 
        }

- ** view onTouchEvent 事件的分析 ** 在view在不可用的情况（disabled）下照样会消耗事件onTouchEvent返回值和view的viewflags （CLICKABLE||LONG_CLICKABLE）相关。在view可用的情况下进一步判断是否有view的代理有调用代理处理本次事件代理返回true则当前一个事件消费掉否则继续往下判断当前的view的viewflags 如果当前  viewFlags==CLICKABLE或viewFlags==LONG_CLICKABLE当前view处理本次的事件并且最后返回true消费了事件不管它是否是disabled状态当然肯定是enable不然下不来这里判断并且，如果view设置点击事件或则长点击事件viFlags=CLICKABLEew或viFlags=LONG_CLICKABLE 由down-up事件走完了就死执行performClick（）方法执行点击事件然后调用view的onclick事件

不完全代码如下
   
     public boolean onTouchEvent(MotionEvent event) {
        final int viewFlags = mViewFlags;
        //第一步判断是否可用
        if ((viewFlags & ENABLED_MASK) == DISABLED) {
            if (event.getAction() == MotionEvent.ACTION_UP && (mPrivateFlags & PFLAG_PRESSED) != 0) {
                setPressed(false);
            }
            // A disabled view that is clickable still consumes the touch
            // events, it just doesn't respond to them.
            return (((viewFlags & CLICKABLE) == CLICKABLE ||
                    (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE));
        }
        //第二部判断是否有代理与代理是否消费事件
        if (mTouchDelegate != null) {
            if (mTouchDelegate.onTouchEvent(event)) {
                return true;
            }
        }
        //第三部判断是否可点击或长点击
        if (((viewFlags & CLICKABLE) == CLICKABLE ||
                (viewFlags & LONG_CLICKABLE) == LONG_CLICKABLE)) {
            switch (event.getAction()) {
                case MotionEvent.ACTION_UP:  
                    ...省略几行 
                   performClick();//由此可见当前view没有Up事件是不会相应点击事件的
                break；
                case MotionEvent.ACTION_DOWN:
                  ....省略几十行 
                 break;
                case MotionEvent.ACTION_CANCEL:
                  ....省略几行
                break
                case MotionEvent.ACTION_MOVE:
                  ....省略几行 
                break;

            return true;  //默认返回true
            }
         return false;//其他情况返回false
       ｝

    
    public boolean performClick() {
        sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);

        ListenerInfo li = mListenerInfo;
        if (li != null && li.mOnClickListener != null) {
            playSoundEffect(SoundEffectConstants.CLICK);
            li.mOnClickListener.onClick(this);//调用点击事件回调通知
            return true;
        }

        return false;
    }

##### ViewGroup事件派发消费过程 #####

- **viewGroup dispatchTouchEvent 事件分析**： 第一步：先清除被标记（mFirstTouchTarget）的第一个子view 处理初始化down设置的目标 第二部检查是否拦截根据当前事件否是MotionEvent.ACTION_DOWN事件或者当前有标记的目标mFirstTouchTarget如果没有则拦截否者继续判断
mGroupFlags==FLAG_DISALLOW_INTERCEPT的判断也就是子view通过parent调用viewgroup的requestDisallowInterceptTouchEvent方法设置的如果设置了这个viewgroup的标记就不拦截否则接着调用自己的onInterceptTouchEvent判断是否需要拦截默认值是不拦截的。现在分开两条分支一条是拦截一条是不拦截，先看不拦截的情况，在不拦截的情况呢先遍历子view判断子View是否能够接收到点击事件。由两点来衡量，子view是否在播放动画和事件是否落在子元素的区域。如果满足条件这两个条件那么事件往下派发通过dispatchTransformedTouchEvent得到新的处理事件结果是true则view赋值给mFirstTouchTarget后调返回得到结果 拦截也是直接调用dispatchTransformedTouchEvent不过 mFirstTouchTarget是等于null的然后都是通过dispatchTransformedTouchEvent进一步处理接着判断传入来的view是否是null如果是null则调用handled = super.dispatchTouchEvent(event)调用父类的dispatchTouchEvent就是View的dispatchTouchEvent上面已分析。否者就是调用handled = child.dispatchTouchEvent(event)调用目标子view的 进行派发也是view的dispatchTouchEvent同上返回处理结果给如果子view返回false则还需要父类View的派发给自己的onTouchEvent处理在返回给上一层如果子view处理就直接返回给上一层

不完全代码如下
```
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (mInputEventConsistencyVerifier != null) {
            mInputEventConsistencyVerifier.onTouchEvent(ev, 1);
        }
        boolean handled = false;
        if (onFilterTouchEventForSecurity(ev)) {//第一步清除mFirstTargetView
            final int action = ev.getAction();
            final int actionMasked = action & MotionEvent.ACTION_MASK;

            // Handle an initial down.
            if (actionMasked == MotionEvent.ACTION_DOWN) {
                // Throw away all previous state when starting a new touch gesture.
                // The framework may have dropped the up or cancel event for the previous gesture
                // due to an app switch, ANR, or some other state change.
                cancelAndClearTouchTargets(ev);
                resetTouchState();
            }

            // Check for interception.//第二步判断当前事件是否是down或mFirstTouchTarget
            不为空。如果都不成立拦截
            final boolean intercepted;
            if (actionMasked == MotionEvent.ACTION_DOWN
                    || mFirstTouchTarget != null) {
                //判断子view是否调用父容器的requestDisallowInterceptTouchEvent方法
                final boolean disallowIntercept = (mGroupFlags & FLAG_DISALLOW_INTERCEPT) != 0;
                if (!disallowIntercept) {
                    intercepted = onInterceptTouchEvent(ev);
                    ev.setAction(action); // restore action in case it was changed
                } else {
                    intercepted = false;
                }
            } else {
                // There are no touch targets and this action is not an initial down
                // so this view group continues to intercept touches.
                intercepted = true;
            }
            ....省略几行
            if (!canceled && !intercepted) {//是取消并且不拦截继续往下
             ...省略n行
                  // 遍历出处理事件的子view
                  for (int i = childrenCount - 1; i >= 0; i--) {
                            final int childIndex = customOrder ?
                                    getChildDrawingOrder(childrenCount, i) : i;
                            final View child = children[childIndex];
                            //处理事件的子view的条件根据动画和位置判断
                             if (!canViewReceivePointerEvents(child)
                                    || !isTransformedTouchPointInView(x, y, child, null)) {
                                continue;
                            }
                            //得到处理事件的子view
                            newTouchTarget = getTouchTarget(child);
                            if (newTouchTarget != null) {
                                // Child is already receiving touch within its bounds.
                                // Give it the new pointer in addition to the ones it is handling.
                                newTouchTarget.pointerIdBits |= idBitsToAssign;
                                break;
                            }

                            resetCancelNextUpFlag(child);
                            //进一步派发给子view处理
                            if (dispatchTransformedTouchEvent(ev, false, child, idBitsToAssign)) {//触摸区域的子view消费当前的事件
                                // Child wants to receive touch within its bounds.
                                mLastTouchDownTime = ev.getDownTime();
                                mLastTouchDownIndex = childIndex;
                                mLastTouchDownX = ev.getX();
                                mLastTouchDownY = ev.getY();
                                //给找到了消费事件子view赋值给mFirstTouchTarget通过addTouchTarget方法
                                newTouchTarget = addTouchTarget(child, idBitsToAssign);
                                alreadyDispatchedToNewTouchTarget = true;
                                break;
                            }

                            }
                            ....在省略n行                
             }
            // 进一步判断如果没有找到子view或找到子view而且没有消费事件也就是mFirstTouchTarget没有赋值则继续派发给viewgroup的view层dispatchTouchEvent在调用在按照view的逻辑派发一边上述已经描述清除了view的派发过程
            if (mFirstTouchTarget == null) {
                // No touch targets so treat this as an ordinary view.
                handled = dispatchTransformedTouchEvent(ev, canceled, null,
                        TouchTarget.ALL_POINTER_IDS);
            } else {
            
            }
    return handled；
}

伪代码 具体过程ViewGroup中有
  private boolean dispatchTransformedTouchEvent(MotionEvent event, boolean cancel,
            View child, int desiredPointerIdBits) {
        final boolean handled;
            if (child == null) {
                handled = super.dispatchTouchEvent(event);
            } else {
                handled = child.dispatchTouchEvent(event);
            }
return handled;

private TouchTarget addTouchTarget(View child, int pointerIdBits) {
        TouchTarget target = TouchTarget.obtain(child, pointerIdBits);
        target.next = mFirstTouchTarget;
        mFirstTouchTarget = target;
        return target;
    }

｝
```
##### Activity事件派发过程 #####

- **Activity dispatchTouchEvent** 

```
    public boolean dispatchTouchEvent(MotionEvent ev) {
        if (ev.getAction() == MotionEvent.ACTION_DOWN) {
            onUserInteraction();
        }
        if (getWindow().superDispatchTouchEvent(ev)) {//调用Window的派发得到返回结果如果为true则派发完成否则调用自己onTouchEvent消费事件
            return true;
        }
        return onTouchEvent(ev);
    }
```
首先是事件交给Activity所附属的Window进行派发返回true，事件结束反之调用自己的onTouchEvent处理。派发给window过程中是通过window的实现类PhoneWindow来superDispatchTouchEvent处理它是通过mDecor来调用superDispatchTouchEvent而mDecor是DecorView是集成了FrameLayout的一个viewGroup然后在传到我们setContentView的ViewGroup中的。


##### 总结 #####


-  viewgroup 的onInterceptTouchEvent事件不是每次都调用的，如果想提前处理事件在 dispatchTouchEvent中处理，前提事件能传到当前的viewgGroup中
-  viewgroup中的 viewgroupFlags 设置FLAG_DISALLOW_INTERCEPT滑动冲突可以使用这个在子view条用 parent.requestDisallowInterceptTouchEvent方法
-  onInterceptTouchEvent 事件不要拦击 down事件不然后续的系列事件会没办法传给子view在冲突问题中时候还有就是Up事件也不能拦截否则子view没办法相应点击事件，也可用通过move事件来解决冲突的问题一般提倡用onInterceptTouchEvent处理事件冲突， parent.requestDisallowInterceptTouchEvent来解决冲突比较麻烦

### View的滑动冲突 ###

##### 常见的滑动冲突场景 #####

-  外部滑动方向与内部滑动方向不一致，比如ViewPager中包含ListView
- 外部滑动方向与内部滑动方向一致
- 上面两种情况的嵌套

##### 滑动冲突的处理规则 #####

可以根据滑动距离和水平方向形成的夹角；或者根据水平和竖直方向滑动的距离差；或者两个方向上的速度差等。

##### 滑动冲突的解决方式 #####
- 外部拦截法

    ```
    public boolean onInterceptTouchEvent(MotionEvent event) {
    	boolean intercepted = false;
    	int x = (int) event.getX();
   		int y = (int) event.getY();

    	switch (event.getAction()) {
    	case MotionEvent.ACTION_DOWN: {
    	    intercepted = false;
    	    break;
    	}
    	case MotionEvent.ACTION_MOVE: {
     	   int deltaX = x - mLastXIntercept;
     	   int deltaY = y - mLastYIntercept;
     	   if (父容器需要拦截当前点击事件的条件，例如：Math.abs(deltaX) > Math.abs(deltaY)) {
       	     intercepted = true;
       	 } else {
        	    intercepted = false;
      	  }
       	 break;
    	}
    	case MotionEvent.ACTION_UP: {
     	   intercepted = false;
        	break;
    	}	
    	default:
        	break;
    	}

    	mLastXIntercept = x;
    	mLastYIntercept = y;

    	return intercepted;
	}
    ```
- 内部拦截法

父容器不拦截任何事件，所有的事件都传递给子元素，如果子元素需要此事件就直接消耗掉，否则就由父容器进行处理，这种方法和Android中的事件分发机制不一样，需要配合requestDisallowInterceptTouchEvent方法才能正常工作， 在子view中 dispatchTouchEvent 处理。 

```
	public boolean dispatchTouchEvent(MotionEvent event) {
    	int x = (int) event.getX();
    	int y = (int) event.getY();

    	switch (event.getAction()) {
    	case MotionEvent.ACTION_DOWN: {]
    	    getParent().requestDisallowInterceptTouchEvent(true);
        	break;
    	}
    	case MotionEvent.ACTION_MOVE: {
        	int deltaX = x - mLastX;
        	int deltaY = y - mLastY;
        	if (当前view需要拦截当前点击事件的条件，例如：	Math.abs(deltaX) > Math.abs(deltaY)) {
            		getParent().requestDisallowInterceptTouchEvent(false);
        	}
        	break;
    	}
    	case MotionEvent.ACTION_UP: {
        	break;
    	}
    	default:
        	break;
    	}

    	mLastX = x;
    	mLastY = y;
    	return super.dispatchTouchEvent(event);
	}
```
