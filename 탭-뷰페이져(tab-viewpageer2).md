2020-05-08
===

1. viewpager2 는 그냥 recyclerview 랑 같다고 봐도 무방
2. viewpager2 의 itemlayout 의 제일 부모는 match_parent, match_parent 여야함


## fragment 를 사용하는 경우

viewpager 가지고 있는 activity or fragment 또는 다른 파일로
FragmentStateAdapter 을 구현

```
// 이걸 보면 예도 recyclerview adapter 임을 알 수 있다.
public abstract class FragmentStateAdapter extends
        RecyclerView.Adapter<FragmentViewHolder> implements StatefulAdapter
```


```
class MainActivity : AppCompactAcitivity(){

   ...

    private inner class PagerFragment(f: FragmentManager) : FragmentStateAdapter(f, lifecycle) {
        val TAG = "PagerFragment"
        override fun getItemCount(): Int = 3

        override fun createFragment(position: Int): Fragment {

            // 필요에 맞게 fragment return 
            return when(position){
                0->{
                    PagerPointFragment()
                }
                1->{
                    PagerEventFragment()
                }
                //2
                else->{
                    PagerOtherFragment()
                }
            }
        }
    }
    ...
}
```

## tab

일반적인 tabLayout

activity_main.xml
```
<com.google.android.material.tabs.TabLayout
    android:id="@+id/main_tab"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    app:tabIndicatorColor="@color/blue"
    app:layout_constraintTop_toBottomOf="@+id/main_logo"/>

    ...


<androidx.viewpager2.widget.ViewPager2
    android:id="@+id/main_pager"
    android:layout_width="match_parent"
    android:layout_height="0dp"
    app:layout_constraintBottom_toBottomOf="parent"
    app:layout_constraintTop_toBottomOf="@+id/main_tab" />
```

in MainActivity
```
val tabsArr = arrayOf("POINT", "EVENT", "OTHER")
...
binding.apply {
    mainDrawer.setDrawerLockMode(DrawerLayout.LOCK_MODE_UNLOCKED)

    mainPager.adapter = PagerFragment(supportFragmentManager)

    TabLayoutMediator(mainTab, mainPager) { tab, position ->
        tab.text = tabsArr[position]
        mainPager.setCurrentItem(tab.position, true)
    }.attach()
}
```
<br>

아래는   
scrollable 과 의 차이는 그냥 저거    
app:tabMode="scrollable" 이 차이이군...  
방식이 좀 다르니 뭐 참고용
## scrollable tab

```
<com.google.android.material.tabs.TabLayout
  android:theme="@style/Theme.MaterialComponents.Light"
  style="@style/Widget.MaterialComponents.TabLayout"
  android:id="@+id/tabs"
  android:layout_width="match_parent"
  android:layout_height="wrap_content"
  app:layout_constraintTop_toTopOf="parent"

  <!-- 이게 그 역할을 하네 -->
  app:tabMode="scrollable" >


  <!--1-->
  <com.google.android.material.tabs.TabItem

      android:layout_width="wrap_content"
      android:layout_height="match_parent"
       />
   ...

   <!--7-->
  <com.google.android.material.tabs.TabItem

      android:layout_width="wrap_content"
      android:layout_height="match_parent" />

</com.google.android.material.tabs.TabLayout>
```


```
// 탭의 이름을 설정해주고
TabLayoutMediator(tabs, viewpager) { tab, position ->
   tab.text = zeroToEndArr[position]
}.attach()

// 여기서 탭을 설정
tabs.addOnTabSelectedListener(object : TabLayout.OnTabSelectedListener {
   override fun onTabReselected(tab: TabLayout.Tab?) {
   }

   override fun onTabUnselected(tab: TabLayout.Tab?) {
   }

   override fun onTabSelected(tab: TabLayout.Tab?) {
       mListAdapter.setTap(tab!!.position)
//                Log.e("tabpos", tab.position.toString())
   }
})
```