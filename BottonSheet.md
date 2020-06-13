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

2020-06-13 / sat
=

## 방식2
fragment 이용

```
class AppInfoBottomSheetFragment : BottomSheetDialogFragment() {

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        // Inflate the layout for this fragment
        //필요없음 fragmentDialog(super) 사용안하면 라운드 모양만들기 위해 필요
       // dialog?.window?.setBackgroundDrawable(ColorDrawable(Color.TRANSPARENT))
        return inflater.inflate(R.layout.fragment_app_info_bottom_sheet, container, false)
    }


    override fun onActivityCreated(savedInstanceState: Bundle?) {
        super.onActivityCreated(savedInstanceState)
        val button = dialog?.findViewById<Button>(R.id.dialog_button)
        val title = dialog?.findViewById<TextView>(R.id.dialog_title)
        val contents = dialog?.findViewById<TextView>(R.id.dialog_contents)
        if(requireArguments().getString(ARG_BOTTOM_SHEET_CATE) == "point"){
            title ?.text= "포인트 적립/소멸 기준"
            contents ?.text = requireActivity().getString(R.string.point_acc_expire)
        }
        button?.setOnClickListener {
            dialog?.dismiss()
        }
    }
    companion object{
        fun newInstance(which: String): AppInfoBottomSheetFragment =
            AppInfoBottomSheetFragment().apply {
                arguments = Bundle().apply {
                    putString(ARG_BOTTOM_SHEET_CATE, which)
                }
            }
    }
}
```


fragment_app_info_bottom_sheet.xml
```
<androidx.core.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="wrap_content">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:background="@android:color/transparent"
        android:orientation="vertical"
        android:padding="10dp"
        tools:context=".bottomsheet.AppInfoBottomSheetFragment">

        <!-- TODO: Update blank fragment layout -->

        <TextView
            android:id="@+id/dialog_title"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_gravity="left"
            android:layout_marginTop="10dp"
            android:gravity="center"
            android:text="앱 이용안내"
            android:textColor="@color/black"
            android:textSize="25sp"

            />

        <TextView
            android:id="@+id/dialog_contents"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:layout_marginTop="5dp"
            android:text="@string/app_info"
            android:textColor="@color/black"
            android:textSize="20sp" />

        <Button
            android:id="@+id/dialog_button"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginStart="10dp"
            android:layout_marginEnd="10dp"
            android:layout_marginBottom="20dp"
            android:background="@drawable/button_azure"
            android:text="확인"
            android:textColor="@color/white"
            android:textSize="20sp" />
    </LinearLayout>
</androidx.core.widget.NestedScrollView>
```

라운드 모양을 위한 style작업 ; 다른 방법이 있는지는 잘 모름.
```
 <style name="AppBottomSheetDialogTheme"
        parent="Theme.Design.Light.BottomSheetDialog">
        <item name="bottomSheetStyle">@style/AppModalStyle</item>
    </style>

    <style name="AppModalStyle"
        parent="Widget.Design.BottomSheet.Modal">
        <item name="android:background">@drawable/corner</item>
    </style>
```