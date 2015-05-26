---
layout:     post
title:      "Tutorial: Custom Drawable - Part 2"
subtitle:   "How to create a state-based drawable."
date:       2015-05-26 22:00:00
author:     "Rey Pham"
comments: 	true
categories: blog 
tags:		Android Tutorial
---

In [first part]({% post_url /blog/2015-05-25-tutorial-custom-drawable-part1 %}), I have shown you how to create a simple drawable and add it to a view. Now, we will try to customize it. Here is the result:
![]({{ site.baseurl }}/img/tutorial-custom-drawable/stateborder.gif){: .center-image }
You can see that the border's color changed when we click on the view. Ok, let's do it.
<br />

##StateBorderDrawable
We need to make some changes on BorderDrawable class.

* First, we change the int color parameter to ColorStateList parameter.

{% highlight java %}

public class StateBorderDrawable extends Drawable {

    Paint mPaint;
	ColorStateList mColorStateList;
    int mColor;
    int mBorderWidth;
    int mBorderRadius;

    RectF mRect;
    Path mPath;

    public BorderDrawable(ColorStateList colorStateList, int borderWidth, int borderRadius){
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setStyle(Paint.Style.FILL);

        mPath = new Path();
        mPath.setFillType(Path.FillType.EVEN_ODD);

        mRect = new RectF();

        mColorStateList = colorStateList;
        mColor = mColorStateList.getDefaultColor();
        mBorderWidth = borderWidth;
        mBorderRadius = borderRadius;
    }
}	

{% endhighlight %}

* Override `isStateful()` method to return true to indicate that this Drawable want to be notified when view' state changed.

{% highlight java %}

@Override
public boolean isStateful() {
    return true;
}

{% endhighlight %}

* We also have to override `onStateChange(int)` method to handle state changed event.

{% highlight java %}

@Override
protected boolean onStateChange(int[] state) {
    int color = mColorStateList.getColorForState(state, mColor);
    if(mColor != color){
        mColor = color;
        invalidateSelf();
        return true;
    }

    return false;
}

{% endhighlight %}

You can see in the code that we retrieve a color based on the current state of view. Then we check if the new color is different with current color. If yes, we update color and call `invalidateSelf()` method to request a re-drawn. 

<br />

##StateBorderImageView
We also need to modify the BorderImageView class.

* First is the `init()` method.

{% highlight java %}

private void init(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes){
    setWillNotDraw(false);
    int[][] states = new int[][]{
            {-android.R.attr.state_pressed},
            {android.R.attr.state_pressed}
    };
    int[] colors = new int[]{
            context.getResources().getColor(R.color.primary),
            context.getResources().getColor(R.color.accent)
    };
    ColorStateList colorStateList = new ColorStateList(states, colors);

    mBorder = new StateBorderDrawable(colorStateList, getPaddingLeft(), getPaddingLeft() / 2);
    mBorder.setCallback(this);
}

{% endhighlight %}
To keep it simple, I create a ColorStateList with only 2 states: **not pressed** and **pressed**. For dynamically, you can declare a custom color attribute for your view and read the ColorStateList value from it.  
Another different is we have to call `setCallback(Callback)` method on drawable object so when drawable is invalidated, it can request the view to re-drawn.

* Next, we need to override `drawableStateChanged()` method to notify drawable when state's changed.

{% highlight java %}

@Override
protected void drawableStateChanged() {
    super.drawableStateChanged();
    mBorder.setState(getDrawableState());
}
	
{% endhighlight %}

* And `verifyDrawable(Drawable)`.

{% highlight java %}

@Override
protected boolean verifyDrawable(Drawable dr) {
    return super.verifyDrawable(dr) || dr == mBorder;
}
	
{% endhighlight %}
When a drawable request the view to re-drawn (via `Callback#invalidateDrawable(Drawable)` method), the view will check if the drawable belongs to it before invalide itself.

* Okay. Now we can add this StateBorderImageView to XML:

{% highlight xml %}

<com.rey.tutorial.widget.StateBorderImageView
    android:layout_width="96dp"
    android:layout_height="96dp"
    android:src="@drawable/avatar"
    android:scaleType="centerCrop"
    android:padding="8dp"/>
	
{% endhighlight %}

Let's run and see the result.  

Now you know how to create a state-based drawable. In the last part, we will go to next level and make it animate between states.

The source code is available on [Github](https://github.com/rey5137/tutorials/tree/add_drawable_to_view).