2020-05-08
===
```
class MainViewModel : ViewModel{

    ...

    private val _eventList = MutableLiveData<List<RETLIST002>>()
    val eventList:LiveData<List<RETLIST002>>
        get() = _eventList

    fun loadEventList(){
        val service = ApiService.retrofit.build().create(ApiService::class.java)
        val callback = service.eventList()

        val remain: (EventList002) -> Unit = {
                _eventList.value = it.RETLIST
        }
        ApiService.retrofitCallback(callback, remains = remain)
    }
    ...


```

ex> in MainActivity
```
override fun onViewCreated(view: View, savedInstanceState:Bundle?){
 evnetViewModel.eventList.observe(this, Observer {
        eventAdapter.setList(it)
    })
    evnetViewModel.loadEventList()
}
```

