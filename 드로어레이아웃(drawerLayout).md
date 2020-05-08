2020-05-08
===

# 아주 기초만

열고 닫기
```
//버튼에 단다.
fun openDrawer() {
    if (!binding.mainDrawer.isDrawerOpen(GravityCompat.START))
        binding.mainDrawer.openDrawer(GravityCompat.START)
}
```

```
class MainActivity : AppCompatActivity(),
    NavigationView.OnNavigationItemSelectedListener

    ...
    //드로어블 메뉴 클릭 이벤트
    override fun onNavigationItemSelected(item: MenuItem): Boolean {
        Log.e("itemclick", item.itemId.toString())ㄴ
        return true
    }
```

```
//설정으로 drawer 반응 제어
mainDrawer.setDrawerLockMode(DrawerLayout.LOCK_MODE_UNLOCKED)
```


activity_main.xml
```
<androidx.drawerlayout.widget.DrawerLayout
...>

...
    <!-- 예가 숨겨졌다 나옴 -->
    <com.google.android.material.navigation.NavigationView
        android:id="@+id/navigation"
        android:layout_width="wrap_content"
        android:layout_height="match_parent"
        android:layout_gravity="start"
        app:headerLayout="@layout/navi_header"
        app:menu="@menu/navigation_menu" />
</androidx.drawerlayout.widget.DrawerLayout>
```

<br>
navi_header.xml 는 아주 일반적인 xml

<br>
navigation_menu.xml

```
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <group android:checkableBehavior="single">
        <item
            android:id="@+id/account"
            android:icon="@drawable/ic_launcher_foreground"
            style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
            android:title="계정"
            />
        <item
            android:id="@+id/setting"
            android:icon="@drawable/ic_launcher_foreground"
            style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
            android:title="설정"
            />
        <item
            android:id="@+id/logout"
            android:icon="@drawable/ic_launcher_foreground"
            style="@style/TextAppearance.AppCompat.Widget.ActionBar.Title"
            android:title="로그아웃"
            />
    </group>
</menu>
```
