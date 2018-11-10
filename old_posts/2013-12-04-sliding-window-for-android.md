---
id: 29
title: Sliding Window for Android
author: rahular
layout: post
guid: http://rahular.com/?p=29
excerpt: A simple tutorial for adding Sliding Windows to Android apps
permalink: /sliding-window-for-android/
categories:
  - android
---
In this post I will be showing you how to create a sliding window for your Android apps. A sliding window is extremely helpful when you have too many options in your app&#8217;s menu and want some more real estate to put them in a cleaner way. Doing this was quite a pain until <a href="https://github.com/jfeinstein10" target="_blank">Jeremy Feinstein</a> wrote a beautiful library, which made it so much simpler! So here&#8217;s how you do it (I will be using Eclipse).

* First of all you need to download the library which is available <a href="https://github.com/jfeinstein10/SlidingMenu" target="_blank">here</a>
* Now in Eclipse, go to File &#8212; Import &#8212; Android &#8212; &#8216;Existing Android code into workspace&#8217;
* Browse for the location of the library (it is named &#8216;library&#8217; inside the master directory)
* Clean build the project just in case there are any errors
* Create a new Android Project in the regular way


    {% include image.html url="../res/framed_Screenshot_2013-06-17-23-17-48.png" description="" scale="50%" %}

When you run the project without changing anything, the app should look something like what is shown above. (In case there are is any dependency error, it is likely to be caused by the &#8216;android-support-v4.jar&#8217; which is automatically referenced in the project. If this error occurs, just delete this file from the &#8216;libs&#8217; directory of your project.)Next you will have to link the library to your project so as to reuse the existing code. To do this, right clock on your project, go to &#8216;Properties&#8217; and select &#8216;Android&#8217; on the left pane. In the &#8216;library&#8217; section, click on &#8216;Add&#8217; and add the library imported previously.Now comes a little bit of coding.

Go to values &#8212; dimens.xml and add the following 

```java
<dimen name="slidingmenu_offset">60dp</dimen> 
<dimen name="shadow_width">15dp</dimen>
```
            
In the &#8216;drawable&#8217; directory, create an XML file called &#8216;shadow.xml&#8217; and add the following 

```java
<shape xmlns:android="http://schemas.android.com/apk/res/android" >
    <gradient
        android:centerColor="#11000000"
        android:endColor="#33000000"
        android:startColor="#00000000" />
</shape>
```

Create a dummy layout and name it &#8216;slide_view&#8217; into which all the sliding menu content can go

In the &#8216;onCreate&#8217; method of the main activity, add the following
    
```java
SlidingMenu menu = new SlidingMenu(this);
menu.setMode(SlidingMenu.LEFT);
menu.setTouchModeAbove(SlidingMenu.TOUCHMODE_FULLSCREEN);
menu.setShadowWidthRes(R.dimen.shadow_width);
menu.setShadowDrawable(R.drawable.shadow);
menu.setBehindOffsetRes(R.dimen.slidingmenu_offset);
menu.setFadeDegree(0.35f);
menu.attachToActivity(this, SlidingMenu.SLIDING_CONTENT);
menu.setSecondaryMenu(R.layout.slide_view);
```

Now when you run the app, it must look something like this

{% include image.html url="../res/framed_Screenshot_2013-06-17-23-18-13.png" description="" scale="50%" %}

{% include image.html url="../res/framed_Screenshot_2013-06-17-23-18-18.png" description="" scale="50%" %}
    
That should do it. All the code is self-explanatory. If you guys have any questions, feel free to comment below and I&#8217;ll try to answer them.
