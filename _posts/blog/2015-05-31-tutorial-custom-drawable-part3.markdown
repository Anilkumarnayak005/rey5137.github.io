---
layout:     post
title:      "Tutorial: Custom Drawable - Part 3"
subtitle:   "How to create an animated drawable."
date:       2015-05-31 23:00:00
author:     "Rey Pham"
comments: 	true
categories: blog 
tags:		Android Tutorial
---

In the last part of this series, we will make the drawable animated between states. Here is the result we want:
![]({{ site.baseurl }}/img/tutorial-custom-drawable/animatedstateborder.gif){: .center-image }
Ok, let's do it.
<br />

##AnimatedStateBorderDrawable
As usual, we need to make some changes on StateBorderDrawable class.

* First, we add a new `duration` parameter to constructor method:

{% highlight java %}

public class AnimatedStateBorderDrawable extends Drawable {

    private boolean mRunning = false;
    private long mStartTime;
    private int mAnimDuration;

    Paint mPaint;
    ColorStateList mColorStateList;
    int mPrevColor;
    int mMiddleColor;
    int mCurColor;
    int mBorderWidth;
    int mBorderRadius;

    RectF mRect;
    Path mPath;

    public BorderDrawable(ColorStateList colorStateList, int borderWidth, int borderRadius, int duration){
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setStyle(Paint.Style.FILL);

        mPath = new Path();
        mPath.setFillType(Path.FillType.EVEN_ODD);

        mRect = new RectF();

        mColorStateList = colorStateList;
        mCurColor = mColorStateList.getDefaultColor();
        mPrevColor = mCurColor;
        mBorderWidth = borderWidth;
        mBorderRadius = borderRadius;
        mAnimDuration = duration;
    }
}	

{% endhighlight %}

You can see that I added some new member variable like `mPrevColor`, `mCurColor`, `mMiddeColor`. Because we will animate the color between 2 states, so we need to know the color of previous and current state. Some variable like `mRunning`, `mStartTime` also needed for storing animation's data.

* Next, we have to implement `android.graphics.drawable.Animatable` interface. There are 3 methods:

{% highlight java %}

@Override
public boolean isRunning() {
    return mRunning;
}

@Override
public void start() {
    resetAnimation();
    scheduleSelf(mUpdater, SystemClock.uptimeMillis() + FRAME_DURATION);
    invalidateSelf();
}

@Override
public void stop() {
    mRunning = false;
    unscheduleSelf(mUpdater);
    invalidateSelf();
}

{% endhighlight %}

`isRunning()` is pretty forward, so let's talk about `start()` method. When we call `start()` method to start running the animation, first we have to call `resetAnimation()` to reset all animation's data:

{% highlight java %}

private void resetAnimation(){
    mStartTime = SystemClock.uptimeMillis();
    mMiddleColor = mPrevColor;
}

{% endhighlight %}

You will see that there are 2 variables need to be updated: `mStartTime` for animation start running time, and `mMiddleColor` for color will be drawn when animation running.
Then, we will schedule a Runnable will be run after a specific duration to update animation's progress and invalidate drawable: 

{% highlight java %}

private final Runnable mUpdater = new Runnable() {

    @Override
    public void run() {
        update();
    }

};

private void update(){
    long curTime = SystemClock.uptimeMillis();
    float progress = Math.min(1f, (float) (curTime - mStartTime) / mAnimDuration);
    mMiddleColor = getMiddleColor(mPrevColor, mCurColor, progress);

    if(progress == 1f)
        mRunning = false;

    if(isRunning())
        scheduleSelf(mUpdater, SystemClock.uptimeMillis() + FRAME_DURATION);

    invalidateSelf();
}

{% endhighlight %}

In the `update()` method, we will calculate the `mMiddleColor` based on the animation's progress and the color of 2 states. Then we check if the animation is completed to continue schedule `mUpdater` or not.

* Next, we update `onStateChange(int[])` and `drawn(Canvas)` methods:

{% highlight java %}

@Override
protected boolean onStateChange(int[] state) {
    int color = mColorStateList.getColorForState(state, mCurColor);

    if(mCurColor != color){
        if(mAnimDuration > 0){
            mPrevColor = isRunning() ? mMiddleColor : mCurColor;
            mCurColor = color;
            start();
        }
        else{
            mPrevColor = color;
            mCurColor = color;
            invalidateSelf();
        }
         return true;
    }

    return false;
}

@Override
public void draw(Canvas canvas) {
    mPaint.setColor(isRunning() ? mMiddleColor : mCurColor);
    canvas.drawPath(mPath, mPaint);
}

{% endhighlight %}

* And we also override `jumpToCurrentState()` and `scheduleSelf(Runnable, long)` methods:

{% highlight java %}

@Override
public void jumpToCurrentState() {
    super.jumpToCurrentState();
    stop();
}

@Override
public void scheduleSelf(Runnable what, long when) {
    mRunning = true;
    super.scheduleSelf(what, when);
}

{% endhighlight %}

`jumpToCurrentState()` method will be called when the view want to skip all the drawable's animation, so we will call `stop()` method to stop animation if it's running.

<br />

##AnimatedStateBorderImageView
We just have to change some few point in the StateBorderImageView class.

* First is the `init()` method. Change from `StateBorderDrawable` to `AnimatedStateBorderDrawable`:

{% highlight java %}

mBorder = new AnimatedStateBorderDrawable(colorStateList, 
        getPaddingLeft(), 
        getPaddingLeft() / 2, 
        context.getResources().getInteger(android.R.integer.config_mediumAnimTime));

{% endhighlight %}

* Next, we need to override `jumpDrawablesToCurrentState()` method to notify drawable when the view want to skip all animations.

{% highlight java %}

@Override
public void jumpDrawablesToCurrentState() {
    super.jumpDrawablesToCurrentState();
    mBorder.jumpToCurrentState();
}
	
{% endhighlight %}

* That's it. Now we can add this AnimatedStateBorderImageView to XML:

{% highlight xml %}

<com.rey.tutorial.widget.AnimatedStateBorderImageView
    android:layout_width="96dp"
    android:layout_height="96dp"
    android:src="@drawable/avatar"
    android:scaleType="centerCrop"
    android:padding="8dp"/>
	
{% endhighlight %}

Let's run and see the result.  

That is the last of this tutorial series. Although the code is simple, but now you know the basic of implementing a fully animated drawable. 

The source code is available on [Github](https://github.com/rey5137/tutorials/tree/add_drawable_to_view).