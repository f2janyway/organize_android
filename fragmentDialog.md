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