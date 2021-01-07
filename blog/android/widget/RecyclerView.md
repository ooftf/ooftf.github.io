### ScrollView嵌套RecyclerView自动滚动
    相关文章https://blog.csdn.net/beibaokongming/article/details/79581232
1. ScrollView的子viewGroup中增加android:descendantFocusability="blocksDescendants" 属性    
2.    ScrollView的直接子view 上添加focusableInTouchMode=true