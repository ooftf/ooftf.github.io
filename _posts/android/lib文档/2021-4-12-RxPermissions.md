---
layout: post
author: "ooftf"
tags: Android
---

# RxPermissions
##  isGranted,isRevoked

##  shouldShowRequestPermissionRationale,request,requestEach,requestEachCombined,ensure,ensureEach,ensureEachCombined


isGranted 是否已经获取到权限
isRevoked 和isGranted 相反

ensure和request内部逻辑相似,只不过ensure是配合RxJava.compose使用的


request 仅在使用中允许       isGranted true      不会再询问
request 本次运行允许         isGranted true      会再询问
request 拒绝               isGranted false     会再询问
request 拒绝且不再询问       isGranted false     不会再询问


request 对于多个权限只会返回一个结果，表示是否全都允许  返回结果是Boolean
requestEach            对于多个权限，返回多个结果，但是在得到所有结果之后才执行的  返回结果是Permission
requestEachCombined         对于多个权限只会返回一个结果 返回结果是 Permission
shouldShowRequestPermissionRationale      是否会显示请求权限弹窗
requestEach  仅在使用中允许  granted=true, shouldShowRequestPermissionRationale=false           shouldShowRequestPermissionRationale::subscribe  true
requestEach  本次运行允许    granted=true, shouldShowRequestPermissionRationale=false           shouldShowRequestPermissionRationale::subscribe  true
requestEach  拒绝          granted=false, shouldShowRequestPermissionRationale=true           shouldShowRequestPermissionRationale::subscribe  true
requestEach  拒绝且不再询问  granted=false, shouldShowRequestPermissionRationale=false          shouldShowRequestPermissionRationale::subscribe  false