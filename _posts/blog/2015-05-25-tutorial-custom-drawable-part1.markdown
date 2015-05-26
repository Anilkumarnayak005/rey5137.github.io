---
layout:     post
title:      "Tutorial: Custom Drawable - Part 1"
subtitle:   "How to create a custom drawable and add it to your view"
date:       2015-05-25 23:00:00
author:     "Rey Pham"
comments: 	true
categories: blog 
tags:		Android Tutorial
---

Sometimes you really want to apply some specific visual to your app, and if you look for answer on the web, most of them is **write your own Drawable class**.  
So what is **Drawable**? Here is class definition: A **Drawable** is a general abstraction for "something that can be drawn".  
Ok, we have **Drawable** class. But how we can use it, or how we can add it to a view? And that what this tutorial is for.

Now, let's try to create a border for ImageView. Here is the result we want:
![]({{ site.baseurl }}/img/tutorial-custom-drawable/border.jpg){: .center-image }
<br />

##BorderDrawable

* First, we create a class extends from Drawable. We also create a constructor to pass all needed attributes.

{% highlight java %}

public class BorderDrawable extends Drawable {

    Paint mPaint;
    int mColor;
    int mBorderWidth;
    int mBorderRadius;

    RectF mRect;
    Path mPath;

    public BorderDrawable(int color, int borderWidth, int borderRadius){
        mPaint = new Paint(Paint.ANTI_ALIAS_FLAG);
        mPaint.setStyle(Paint.Style.FILL);

        mPath = new Path();
        mPath.setFillType(Path.FillType.EVEN_ODD);

        mRect = new RectF();

        mColor = color;
        mBorderWidth = borderWidth;
        mBorderRadius = borderRadius;
    }
}	

{% endhighlight %}

* Override `onBoundsChange(Rect)` method to calculate the Path we will draw later in `draw(Canvas)`.

{% highlight java %}

@Override
protected void onBoundsChange(Rect bounds) {
    mPath.reset();

    mPath.addRect(bounds.left, bounds.top, bounds.right, bounds.bottom, Path.Direction.CW);
    mRect.set(bounds.left + mBorderWidth, bounds.top + mBorderWidth, bounds.right - mBorderWidth, bounds.bottom - mBorderWidth);
    mPath.addRoundRect(mRect, mBorderRadius, mBorderRadius, Path.Direction.CW);
}

{% endhighlight %}

* We also need implement some abstract methods.

{% highlight java %}

@Override
public void draw(Canvas canvas) {
    mPaint.setColor(mColor);
    canvas.drawPath(mPath, mPaint);
}

@Override
public void setAlpha(int alpha) {
    mPaint.setAlpha(alpha);
}

@Override
public void setColorFilter(ColorFilter cf) {
    mPaint.setColorFilter(cf);
}

@Override
public int getOpacity() {
    return PixelFormat.TRANSLUCENT;
}

{% endhighlight %}
<br />

##BorderImageView
* Create a class extends from ImageView class. 

{% highlight java %}

public class BorderImageView extends ImageView{

    BorderDrawable mBorder;

    public BorderImageView(Context context) {
        super(context);

        init(context, null, 0, 0);
    }

    public BorderImageView(Context context, AttributeSet attrs) {
        super(context, attrs);

        init(context, attrs, 0, 0);
    }

	//another constructors ...

    private void init(Context context, AttributeSet attrs, int defStyleAttr, int defStyleRes){
        setWillNotDraw(false);
        mBorder = new BorderDrawable(context.getResources().getColor(R.color.primary), getPaddingLeft(), getPaddingLeft() / 2);
    }
	
}

{% endhighlight %}

In `init()` method, we have to call `setWillNotDraw(false)` so the view will call `onDraw(canvas)` later, or else it will skip. You also see that I use the padding value of ImageView as the border's width.

* Next, we need override `onSizeChanged(int, int, int, int)` method to set the bound of drawable.

{% highlight java %}

@Override
protected void onSizeChanged(int w, int h, int oldw, int oldh) {
    super.onSizeChanged(w, h, oldw, oldh);
    mBorder.setBounds(0, 0, w, h);
}
	
{% endhighlight %}

* And `onDraw(canvas)` to make a call to the `draw(Canvas)` method of drawable.

{% highlight java %}

@Override
protected void onDraw(Canvas canvas) {
    super.onDraw(canvas);
    mBorder.draw(canvas);
}
	
{% endhighlight %}

* That's it. Now we can add this BorderImageView to XML:

{% highlight xml %}

<com.rey.tutorial.widget.BorderImageView
    android:layout_width="96dp"
    android:layout_height="96dp"
    android:src="@drawable/avatar"
    android:scaleType="centerCrop"
    android:padding="8dp"/>
	
{% endhighlight %}

Although you can extend ImageView class and draw the border directly in `onDraw(Canvas)` method, using Drawable is more reusable. In [second part]({{ site.baseurl }}{% post_url /blog/2015-05-26-tutorial-custom-drawable-part2 %}), I will show you how to create a state-based drawable.  

The source code is available on [Github](https://github.com/rey5137/tutorials/tree/add_drawable_to_view).