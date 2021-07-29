---
title: ContentProvider
published: false
---

ContentProvider 通过 uri 来标识其它应用要访问的数据，通过 ContentResolver 的增、删、改、查方法实现对共享数据的操作。还可以通过注册 ContentObserver 来监听数据是否发生了变化来对应的刷新页面。下面分别说说每个类的作用。

UriMatcher 用来匹配 uri

## ContentProvider

content://xxx.xxx.xxx/student
其中 xxx.xxx.xxx 为 authorities 部分

## ContentResolver

## ContentObserver


