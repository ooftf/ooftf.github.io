### ScrollView嵌套RecyclerView自动滚动
    相关文章https://blog.csdn.net/beibaokongming/article/details/79581232
- 解决方案一： ScrollView的子viewGroup中增加android:descendantFocusability="blocksDescendants" 属性    
- 解决方案二： ScrollView的直接子view 上添加focusableInTouchMode=true
### ScrollView 嵌套RecyclerView + GridLayoutManager 只显示一行或者不显示，需要滑动RecyclerView 
