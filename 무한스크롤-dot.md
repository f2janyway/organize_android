2020-05-23
=
activity(xml은 constraintlayout만 있다.)
```
override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_test)
        //show 해준다. 
        val fr = AlertDialogFragment.newInstance(3)
        fr.show(supportFragmentManager,"start")
    }
```
AlertDialogFragment
```

// TODO: Customize parameter argument names
const val ARG_ITEM_COUNT = "item_count"
const val ARG_ITEM_URL = "item_url"

class AlertDialogFragment : DialogFragment() {

    private lateinit var bindingf: FragmentAlertPagerBinding
    var realItemCount = 0
    //test url
    val list = listOf(
        "https://www.thescoop.co.kr/news/photo/202001/37822_50999_827.jpg",//차
        "https://p.bigstockphoto.com/GeFvQkBbSLaMdpKXF1Zv_bigstock-Aerial-View-Of-Blue-Lakes-And--227291596.jpg",//산
        "https://image.shutterstock.com/image-photo/butterfly-grass-on-meadow-night-260nw-1111729556.jpg",
        "https://encrypted-tbn0.gstatic.com/images?q=tbn%3AANd9GcT0E-eUZJkBWnSwhBm-UifO9jR8PFu12F16IRIIYtuMARkCbC7c&usqp=CAU"
    )
    private val alertFragmentAdapter: ScreenSlidePagerAdapter by lazy {
        ScreenSlidePagerAdapter(activity as TestActivity)
    }
    //sync with dot and viewpager
    private val onPageChangeCallback = object :
        ViewPager2.OnPageChangeCallback() {
        override fun onPageSelected(position: Int) {
            super.onPageSelected(position)
            val real = position % realItemCount
            for (i in 0 until realItemCount) {
                (bindingf.alertDotLayout[i] as ImageView).apply {
                    if (real == i)
                        setImageDrawable(
                            ContextCompat.getDrawable(
                                requireContext(),
                                R.drawable.ic_brightness_1_blue_24dp
                            )
                        )
                    else
                        setImageDrawable(
                            ContextCompat.getDrawable(
                                requireContext(),
                                R.drawable.ic_brightness_1_grey_24dp
                            )
                        )
                }
            }
        }
    }

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        bindingf = FragmentAlertPagerBinding.inflate(inflater, container, false)
//        realItemCount = requireArguments().getInt(ARG_ITEM_COUNT)
        realItemCount = list.size
        bindingf.apply {
            alertPager.adapter = alertFragmentAdapter
            alertPager.setCurrentItem(realItemCount * 10, false)
            initAutoScroll()
        }
        createDot()
        //set dot change
        bindingf.alertPager.registerOnPageChangeCallback(onPageChangeCallback)
        return bindingf.root
    }
    private fun initAutoScroll(){
        var cnt = 0
        GlobalScope.launch(Dispatchers.Main) {
            do {
                delay(1000)
                bindingf.alertPager.setCurrentItem(realItemCount * 10 + (++cnt), true)
                Log.e("AlertDialogFragment", "onCreateView: cnt : $cnt")
            } while (cnt < 5)
            if (cnt > 4) this.cancel()
        }
    }

    private fun createDot() {
        for (i in 0 until realItemCount) {
            val imageview = ImageView(context)
            imageview.id = i * imageview.hashCode()

            imageview.setImageDrawable(
                ContextCompat.getDrawable(
                    requireContext(),
                    if (i == 0) R.drawable.ic_brightness_1_blue_24dp else R.drawable.ic_brightness_1_grey_24dp
                )
            )
            bindingf.alertDotLayout.addView(imageview)
        }
    }

    override fun onResume() {
        super.onResume()
        setDialogSize()
    }

    private fun setDialogSize() {
        val windowManager =
            requireActivity().getSystemService(Context.WINDOW_SERVICE) as WindowManager
        val display = windowManager.defaultDisplay
        val size = Point()
        display.getSize(size)

        val params = dialog?.window?.attributes

        params?.width = (size.x * 0.9).toInt()
        params?.height = (size.y * 0.9).toInt()
        dialog?.window?.attributes = params as WindowManager.LayoutParams
    }

    private inner class ScreenSlidePagerAdapter(fa: FragmentActivity) : FragmentStateAdapter(fa) {
        override fun getItemCount(): Int = Int.MAX_VALUE
        override fun createFragment(position: Int): Fragment {
            return AlertSlidePageFragment.newInstance(list[position % realItemCount])
        }

    }

    companion object {
        fun newInstance(itemCount: Int): AlertDialogFragment =
            AlertDialogFragment().apply {
                arguments = Bundle().apply {
                    putInt(ARG_ITEM_COUNT, itemCount)
                }
            }

    }
}
```
FragmentAlertPagerBinding(fragment_alert_pager.xml)
```
<androidx.constraintlayout.widget.ConstraintLayout
        android:id="@+id/alert_layout"
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <androidx.viewpager2.widget.ViewPager2
            android:id="@+id/alert_pager"
            android:layout_width="0dp"
            android:layout_height="0dp"
            ...
            tools:listitem="@layout/fragment_alert_pager_item" />

<!--dot indicator container -->
        <LinearLayout
            android:id="@+id/alert_dot_layout"
            android:orientation="horizontal"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:gravity="center"
            app:layout_constraintTop_toBottomOf="@+id/alert_pager"
            app:layout_constraintBottom_toTopOf="@id/alert_close" >

        </LinearLayout>
```
AlertSlidePageFragment
```
class AlertSlidePageFragment : BaseFragment<FragmentAlertSlidePageBinding>() {


    var url : String = ""
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        super.onCreateView(inflater, container, savedInstanceState)
        url = requireArguments().getString(ARG_ITEM_URL,"")
        Log.e("AlertSlidePageFragment", "onCreateView: url :$url")
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        Glide.with(view.context)
            .load(url)
            .placeholder(
                ContextCompat.getDrawable(
                    view.context,
                    R.drawable.ic_launcher_background
                )
            ).into(binding.alertSlideFramgementImageview)

    }

    override fun getLayoutId(): Int {
        return R.layout.fragment_alert_slide_page
    }
    companion object {
        fun newInstance(url: String): AlertSlidePageFragment =
            AlertSlidePageFragment().apply {
                arguments = Bundle().apply {
                    putString(ARG_ITEM_URL, url)
                }
            }

    }
}
```
AlertSlidePageFragment 의 xml 은 imageView 하나 있다.

<pre>
viewpager2 를 이용하면서 처음에 adapter을 recyclerview.adapter을 직접 사용했다.(slidepagefragment을 이용 안했음)
당시엔 일반적인 리사이클 뷰처럼 스크롤시 position 이 일정하게 변하지 않았다. 그래서 fragment로 바꿨더니 잘된다.
아마 기본 adpater에서 onBindViewHolder()에서 또는 스크롤 계산 이상으로 잘 안됐던 **것** 같다.
기본적인 방법으로는 안된다. 


</pre>
