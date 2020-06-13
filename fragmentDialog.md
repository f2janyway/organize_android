2020-06-11
=
fragment
```

class LoadingFragment :DialogFragment(){

    lateinit var binding  : FragmentLoadingBinding
    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = FragmentLoadingBinding.inflate(inflater,container,false)
        //파일접근
        val uri = Uri.parse("android.resource://" + requireContext().packageName +"/" + R.raw.loading )

//배경 투명하게(이미지뷰나 layout에서 해봤자 안됨)
        dialog?.window?.setBackgroundDrawableResource(android.R.color.transparent)

//gif 
        Glide.with(this).asGif().load(uri).placeholder(ContextCompat.getDrawable(requireActivity(),
            R.drawable.ic_launcher_background)).into(binding.loadingImageview)
        return binding.root

    }
    override fun onStart() {
        super.onStart()
        //밖 dim 처리 없애기
        val window = dialog!!.window
        val windowParams: WindowManager.LayoutParams? = window?.attributes
        //여기서 정한다. 0->remove dim all 
        windowParams?.dimAmount = 0.00f
        windowParams?.flags = windowParams?.flags?.or(WindowManager.LayoutParams.FLAG_DIM_BEHIND)
        window?.attributes = windowParams
    }
}
```

2020-06-13
==
PopUpPagerFragmentDialog
```
class PopUpPagerFragmentDialog : DialogFragment() {

    private lateinit var binding: FragmentPopUpBinding
    var realItemCount = 0

    private val popUpAdapter: PopUpPagerAdapter by lazy {
        PopUpPagerAdapter(requireActivity())
    }
    //sync with dot and viewpager

    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        binding = FragmentPopUpBinding.inflate(inflater, container, false)

        binding.fr = this
        (requireActivity() as TestActivity).a += 1

        realItemCount = (requireActivity() as TestActivity).list1.count()
        binding.apply {
            popupPager.adapter = popUpAdapter
            popupPager.setCurrentItem(realItemCount * 10, false)
        }
        binding.apply {
            checkBoxBindTextView(popupCheckbox, checkboxText) {
                // 현재는 TestActivity 로인해
//                (requireActivity() as MainActivity).pref.apply {
//                    Log.e("PopUpPagerFragmentDialog", "onCreateView: pref occur")
//                    if (popupCheckbox.isChecked)
//                        setBoolean("oil-popup-today", true)
//                    else
//                        setBoolean("oil-popup-today", false)
//                }
            }
        }
        return binding.root
    }


    fun clickClose(){
        dismiss()
    }
    fun clickPrev(){
        binding.popupPager.setCurrentItem(binding.popupPager.currentItem -1 ,true)
    }
    fun clickNext(){
        binding.popupPager.setCurrentItem(binding.popupPager.currentItem +1 ,true)
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
        if ((requireActivity() as TestActivity).a == 1)
            params?.height = (size.y * 0.5).toInt()
        else
            params?.height = (size.y * 0.9).toInt()
        dialog?.window?.attributes = params as WindowManager.LayoutParams
        dialog?.window?.setBackgroundDrawableResource(android.R.color.transparent)
    }

    private inner class PopUpPagerAdapter(fa: FragmentActivity) : FragmentStateAdapter(fa) {
        override fun getItemCount(): Int = Int.MAX_VALUE
        override fun createFragment(position: Int): Fragment {
            Log.e("PopUpPagerAdapter", "createFragment: $position")
            return PopUpPageFragment.newInstance(
                position % realItemCount
            )
        }
    }
}
```
PopUpPageFragment (1 or 2 item(s))
`````
class PopUpPageFragment : BaseFragment<PopupItemBinding>() {

//    var url: String = ""
//    var url1:String = ""
//    var oneOrTwo = 1
    var pos = 0
    override fun onCreateView(
        inflater: LayoutInflater, container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View? {
        super.onCreateView(inflater, container, savedInstanceState)
//        url = requireArguments().getString(ARG_ITEM_URL, "")
//        url1 = requireArguments().getString(ARG_ITEM_URL1, "")
//        oneOrTwo = requireArguments().getInt(ARG_POPUP_ONE_TWO)
        pos = requireArguments().getInt(ARG_POPUP_POS)
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        val act  = (requireActivity() as TestActivity)
        if ((requireActivity() as TestActivity).a == 1) {
            Glide.with(view.context)
                .load(act.list1[pos])
                .fitCenter()
                .override(600)
                .apply(RequestOptions().transform(FitCenter(),RoundedCorners(80)))
                .into(binding.popupImage0)
        }else if((requireActivity() as TestActivity).a == 2){
            binding.popupImage1.visibility = View.VISIBLE
            Glide.with(view.context)
                .load(act.list1[pos])
                .fitCenter()
                .override(600)
                .apply(RequestOptions().transform(FitCenter(),RoundedCorners(80)))
                .into(binding.popupImage0)
            Glide.with(view.context)
                .load(act.list1[pos])
                .fitCenter()
                .override(600)
                .apply(RequestOptions().transform(FitCenter(),RoundedCorners(80)))
                .into(binding.popupImage1)

            binding.popupImage0.setOnClickListener {
                Log.e("PopUpPageFragment", "onViewCreated: top")
            }
            binding.popupImage1.setOnClickListener {
                Log.e("PopUpPageFragment", "onViewCreated: bottom")
            }
        }
        Log.e("PopUpPageFragment", "onViewCreated: number in ${(requireActivity() as TestActivity).a}")

    }

    override fun getLayoutId(): Int {
        return R.layout.popup_item
    }

    companion object {
        fun newInstance(popup_pos:Int): PopUpPageFragment =
            PopUpPageFragment().apply {
                arguments = Bundle().apply {
                    //ARG_ITEM_URL 이걸 AlertSlidePageFragment 에서도 쓰니 주의하셈  //꼬였다.
                    putInt(ARG_POPUP_POS, popup_pos )
                }
            }

    }
}
const val ARG_POPUP_POS= "popup-position"
`````