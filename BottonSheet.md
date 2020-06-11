2020-05-08
===

# Test 더 해봐야.


두가지 방식이 있음.   
1. 레이아웃 아래 숨겨놓고 하는 것
   1. 뒤의 화면과 같은 화면
   2. bottom layout 의 최상위는 CoordinatorLayout
2. BottomSheetDialogFragment() 상속한 fragment 이용
   1. 기본적으로 뒷배경 어두워짐 (dim? 처리)
   2. backstack이 프래그먼트가 차지하고 있는듯
   3. 그래서 CoordinatorLayout 상관없음


## 1 방식

activity_next.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".NextActivity">
   <Button
    android:id="@+id/btnPersistent"
    android:layout_width="wrap_content"
    android:layout_height="wrap_content"
    android:layout_gravity="center" />

    <!-- Adding bottom sheet after main content -->
    <include layout="@layout/bottom_sheet1" />


</androidx.coordinatorlayout.widget.CoordinatorLayout>
```

<br>

bottom_sheet1.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.core.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/bottomSheet"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:behavior_hideable="false"
    app:behavior_peekHeight="10dp"
    app:layout_behavior="com.google.android.material.bottomsheet.BottomSheetBehavior">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@drawable/coner"
        android:orientation="vertical"
        android:padding="12dp"
        >

        <androidx.recyclerview.widget.RecyclerView
            android:id="@+id/bottom_recycler"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />


    </LinearLayout>
</androidx.core.widget.NestedScrollView>
```

<br>

coner.xml
```
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="rectangle">
    <solid android:color="@android:color/holo_blue_dark"/>
    <corners android:topLeftRadius="20dp"
        android:topRightRadius="20dp"/>
</shape>
```

in activity
```

val sheetBehavior = BottomSheetBehavior.from(bottomSheet)


sheetBehavior.apply {
    //options
   peekHeight = 0
   isFitToContents = false
   halfExpandedRatio = 0.3f

}

...

//options
btnPersistent.setOnClickListener {
   if (sheetBehavior.state != BottomSheetBehavior.STATE_EXPANDED) {
       sheetBehavior.state = BottomSheetBehavior.STATE_EXPANDED
       setBtnCloseSheet()
   } else {
       sheetBehavior.state = BottomSheetBehavior.STATE_COLLAPSED
       setBtnExpandSheet()
   }
}

...

sheetBehavior.addBottomSheetCallback(object : BottomSheetBehavior.BottomSheetCallback() {
   override fun onSlide(bottomSheet: View, slideOffset: Float) {
   }

   override fun onStateChanged(bottomSheet: View, newState: Int) {
       when (newState) {
           BottomSheetBehavior.STATE_HIDDEN -> {
           }
           BottomSheetBehavior.STATE_EXPANDED -> {
               setBtnCloseSheet()
           }
           BottomSheetBehavior.STATE_HALF_EXPANDED -> {
               Log.e("halfExpanded","<<<<<<<<<<<<<<")
           }
           BottomSheetBehavior.STATE_COLLAPSED -> {
               setBtnExpandSheet()
           }
           BottomSheetBehavior.STATE_DRAGGING -> {
           }
           BottomSheetBehavior.STATE_SETTLING -> {
           }
       }
   }
})
```

2020-06-11 
=
peakheight 정하는 법(특정 레이아웃 높이에 따라)
```
 sheetBehavior.apply {
    val inner = binding.includeMapBottomSheet.bottomSheetInnerConst
    inner.viewTreeObserver.addOnGlobalLayoutListener {
       //children 의 높이에 따라 정하는데 정확히 기준 잘...
       //부모 레이아웃(bottomSheetInnerConst) 이 만약 LinearLayout 
       //이면 알기가 쉽다.
       peekHeight = inner.getChildAt(4).top
       //확인용
       for(i in inner.children){
           Log.e("MapActivity", "onCreate: children : $i")
       }
      }
 }

```