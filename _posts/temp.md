 ## 竖屏切换成横屏
```
⇢ onPause[]
⇠ onPause[0ms]="void"
⇢ onStop[]
⇠ onStop[0ms]="void"
⇢ onSaveInstanceState[outState="Bundle[{}]"]
⇠ onSaveInstanceState[0ms]="void"
⇢ onRetainCustomNonConfigurationInstance[]
⇠ onRetainCustomNonConfigurationInstance[0ms]="null"
⇢ isChangingConfigurations[]
⇠ isChangingConfigurations[0ms]="true"
⇢ onDestroy[]
⇠ onDestroy[0ms]="void"
⇢ onDetachedFromWindow[]
⇠ onDetachedFromWindow[0ms]="void"
⇢ <init>[]
⇠ <init>[1ms]="void"
⇢ onCreate[savedInstanceState="Bundle[{android:viewHierarchyState=Bundle[{android:views={16908290=android.view.AbsSavedState$1@b8c12cc, 2131230768=androidx.appcompat.widget.Toolbar$SavedState@e265915, 2131230770=android.view.AbsSavedState$1@b8c12cc, 2131230776=android.view.AbsSavedState$1@b8c12cc, 2131230839=android.view.AbsSavedState$1@b8c12cc, 2131230977=android.view.AbsSavedState$1@b8c12cc}}], androidx.lifecycle.BundlableSavedStateRegistry.key=Bundle[{}], android:lastAutofillId=1073741823, android:fragments=android.app.FragmentManagerState@b6842a}]"]
⇢ getLastNonConfigurationInstance[]
⇠ getLastNonConfigurationInstance[0ms]="androidx.activity.ComponentActivity$NonConfigurationInstances@310b8f6"
⇠ onCreate[39ms]="void"
⇢ onStart[]
⇠ onStart[0ms]="void"
⇢ onRestoreInstanceState[savedInstanceState="Bundle[{android:viewHierarchyState=Bundle[{android:views={16908290=android.view.AbsSavedState$1@b8c12cc, 2131230768=androidx.appcompat.widget.Toolbar$SavedState@e265915, 2131230770=android.view.AbsSavedState$1@b8c12cc, 2131230776=android.view.AbsSavedState$1@b8c12cc, 2131230839=android.view.AbsSavedState$1@b8c12cc, 2131230977=android.view.AbsSavedState$1@b8c12cc}}], androidx.lifecycle.BundlableSavedStateRegistry.key=Bundle[{}], android:lastAutofillId=1073741823, android:fragments=android.app.FragmentManagerState@b6842a}]"]
⇠ onRestoreInstanceState[0ms]="void"
⇢ onResume[]
⇠ onResume[2ms]="void"
⇢ onAttachedToWindow[]
⇠ onAttachedToWindow[0ms]="void"
⇢ onWindowFocusChanged[hasFocus="true"]
⇠ onWindowFocusChanged[0ms]="void"
```

## 横屏切换成竖屏
```
⇢ onPause[]
⇠ onPause[1ms]="void"
⇢ onStop[]
⇠ onStop[0ms]="void"
⇢ onSaveInstanceState[outState="Bundle[{}]"]
⇠ onSaveInstanceState[0ms]="void"
⇢ onRetainCustomNonConfigurationInstance[]
⇠ onRetainCustomNonConfigurationInstance[0ms]="null"
⇢ isChangingConfigurations[]
⇢ onDetachedFromWindow[]
⇠ onDetachedFromWindow[0ms]="void"
⇢ <init>[]
⇠ <init>[0ms]="void"
⇢ onCreate[savedInstanceState="Bundle[{android:viewHierarchyState=Bundle[{android:views={16908290=android.view.AbsSavedState$1@b8c12cc, 2131230768=androidx.appcompat.widget.Toolbar$SavedState@776b8c, 2131230770=android.view.AbsSavedState$1@b8c12cc, 2131230776=android.view.AbsSavedState$1@b8c12cc, 2131230839=android.view.AbsSavedState$1@b8c12cc, 2131230977=android.view.AbsSavedState$1@b8c12cc}}], androidx.lifecycle.BundlableSavedStateRegistry.key=Bundle[{}], android:lastAutofillId=1073741823, android:fragments=android.app.FragmentManagerState@fe5a2d5}]"]
⇢ getLastNonConfigurationInstance[]
⇠ getLastNonConfigurationInstance[0ms]="androidx.activity.ComponentActivity$NonConfigurationInstances@76a9751"
⇠ onCreate[34ms]="void"
⇢ onStart[]
⇠ onStart[1ms]="void"
⇢ onRestoreInstanceState[savedInstanceState="Bundle[{android:viewHierarchyState=Bundle[{android:views={16908290=android.view.AbsSavedState$1@b8c12cc, 2131230768=androidx.appcompat.widget.Toolbar$SavedState@776b8c, 2131230770=android.view.AbsSavedState$1@b8c12cc, 2131230776=android.view.AbsSavedState$1@b8c12cc, 2131230839=android.view.AbsSavedState$1@b8c12cc, 2131230977=android.view.AbsSavedState$1@b8c12cc}}], androidx.lifecycle.BundlableSavedStateRegistry.key=Bundle[{}], android:lastAutofillId=1073741823, android:fragments=android.app.FragmentManagerState@fe5a2d5}]"]
⇠ onRestoreInstanceState[1ms]="void"
⇢ onResume[]
⇠ onResume[1ms]="void"
⇢ onAttachedToWindow[]
⇠ onAttachedToWindow[0ms]="void"
⇢ onWindowFocusChanged[hasFocus="true"]  // 只要焦点从Activity中失去或者得到就会被调用，所以会被调用多次
⇠ onWindowFocusChanged[0ms]="void"
```

## 设置 <activity android:configChanges="keyboardHidden|orientation|screenSize"/>
### 横屏变竖屏

 ⇢ onConfigurationChanged[newConfig="{1.0 310mcc260mnc [zh_CN_#Hans,en_US] ldltr sw360dp w672dp h336dp 480dpi nrml long land finger qwerty/v/v dpad/v winConfig={ mBounds=Rect(0, 0 - 2160, 1080) mAppBounds=Rect(0, 0 - 2016, 1080) mWindowingMode=fullscreen mDisplayWindowingMode=fullscreen mActivityType=standard mAlwaysOnTop=undefined mRotation=ROTATION_90} s.5}"]
 ⇠ onConfigurationChanged[2ms]="void"
 
#### 竖屏变横屏

 ⇢ onConfigurationChanged[newConfig="{1.0 310mcc260mnc [zh_CN_#Hans,en_US] ldltr sw360dp w360dp h648dp 480dpi nrml long port finger qwerty/v/v dpad/v winConfig={ mBounds=Rect(0, 0 - 1080, 2160) mAppBounds=Rect(0, 0 - 1080, 2016) mWindowingMode=fullscreen mDisplayWindowingMode=fullscreen mActivityType=standard mAlwaysOnTop=undefined mRotation=ROTATION_0} s.7}"]
⇠ onConfigurationChanged[0ms]="void"