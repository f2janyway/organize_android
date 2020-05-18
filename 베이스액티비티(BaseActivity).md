2020-05-08
===

sample
```
import android.view.View
import androidx.appcompat.app.AppCompatActivity
import android.widget.FrameLayout
import android.widget.Toolbar
import androidx.appcompat.app.ActionBar
import androidx.constraintlayout.widget.ConstraintLayout
import androidx.databinding.DataBindingUtil
import androidx.databinding.ViewDataBinding
import androidx.lifecycle.ViewModel
import androidx.lifecycle.ViewModelProvider


open class BaseActivity<DB:ViewDataBinding, VM:ViewModel> : AppCompatActivity() {

    lateinit var binding:DB
    lateinit var vmClass :VM
    lateinit var toolbar: androidx.appcompat.widget.Toolbar
    lateinit var actionBar : ActionBar
    val viewModel by lazy {
        ViewModelProvider(
            viewModelStore,
            ViewModelProvider.NewInstanceFactory()
        ).get(vmClass.javaClass)
    }

    override fun setContentView(layoutResID: Int) {

        val baseActivityLayout =
            layoutInflater.inflate(R.layout.activity_base,null) as ConstraintLayout
        val activityContainer = baseActivityLayout.findViewById<FrameLayout>(R.id.layout_container)

        toolbar = baseActivityLayout.findViewById(R.id.toolbar)
        setSupportActionBar(toolbar)
        actionBar = supportActionBar!!
        actionBar.apply {
            title = getString(R.string.default_toolbar)
            setDisplayHomeAsUpEnabled(true)
        }

        binding = DataBindingUtil.inflate(layoutInflater,layoutResID,activityContainer,true)
        super.setContentView(baseActivityLayout)
    }
     fun showActionBar(){
//        actionBar.title = where
        toolbar.visibility = View.VISIBLE
    }
     fun hideActionBar(){
        toolbar.visibility = View.GONE
    }

}

```


2020-05-14
=
되는거

```

abstract class BaseActivity<DB : ViewDataBinding, VM : ViewModel> : AppCompatActivity() {

    lateinit var binding: DB
    lateinit var toolbar: androidx.appcompat.widget.Toolbar
    lateinit var actionBar: ActionBar
    val viewModel by lazy {
        ViewModelProvider(getOwner()).get(insertViewModelClass())
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
//        binding = DataBindingUtil.setContentView(this, getLayoutId())
    }

    override fun setContentView(layoutResID: Int) {

        val baseActivityLayout =
            layoutInflater.inflate(R.layout.activity_base, null) as ConstraintLayout
        val activityContainer = baseActivityLayout.findViewById<FrameLayout>(R.id.layout_container)

        toolbar = baseActivityLayout.findViewById(R.id.toolbar)
        setSupportActionBar(toolbar)
        actionBar = supportActionBar!!
        actionBar.apply {
            title = getString(R.string.default_toolbar)
            setDisplayHomeAsUpEnabled(true)
        }

        binding = DataBindingUtil.inflate(layoutInflater,layoutResID,activityContainer,true)
        super.setContentView(baseActivityLayout)
    }

    abstract fun getLayoutId(): Int
    abstract fun getOwner(): ViewModelStoreOwner
    fun showActionBar() {
//        actionBar.title = where
        toolbar.visibility = View.VISIBLE
    }

    fun hideActionBar() {
        toolbar.visibility = View.GONE
    }

    abstract fun insertViewModelClass(): Class<VM>
}

```