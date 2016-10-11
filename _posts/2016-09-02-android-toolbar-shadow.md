---
layout: post
title:  "How to remove android toolbar shadow"
date:   2016-09-02 09:50:52
categories: android toolbar shadow
published: true
comments: true
thread: 20160902095055555
---
做APP之前一直是hybrid这种，所以接触略浅显。最近又上马一个APP项目，android这块只能自己上解决一些问题。
现在都流行沉浸式状态栏了，这两天也去恶补了一些沉浸式状态栏的知识，还是云里雾里。

但是对于使用toolbar后在toolbar上方有一个border原因,
```java
<android.support.design.widget.AppBarLayout
         android:layout_height="wrap_content"
         android:layout_width="match_parent"
         android:theme="@style/AppTheme.AppBarOverlay">

     <android.support.v7.widget.Toolbar
             android:id="@+id/toolbar"
             android:layout_width="match_parent"
             android:layout_height="?attr/actionBarSize"
             android:background="?attr/colorPrimary"
             app:popupTheme="@style/AppTheme.PopupOverlay"/>

 </android.support.design.widget.AppBarLayout>
```
修改后
```java
<android.support.design.widget.AppBarLayout
       android:layout_height="wrap_content"
       android:layout_width="match_parent"
       app:elevation="0dp"
       android:theme="@style/AppTheme.AppBarOverlay">
       <android.support.v7.widget.Toolbar
           android:id="@+id/toolbar"
           android:layout_width="match_parent"
           android:layout_height="?attr/actionBarSize"
           android:background="?attr/colorPrimary"
           app:popupTheme="@style/AppTheme.PopupOverlay"/>
</android.support.design.widget.AppBarLayout>
```
