2020-05-12
=
그냥 raw text 를 불러올때 json 이 아니다.
```
 val retrofit = Retrofit.Builder().baseUrl(BASE_URL)
//            .addConverterFactory(GsonConverterFactory.create(gson))
            .addConverterFactory(ScalarsConverterFactory.create())



```

String을 받아올때
gson 이용시 불편함
<br>구현 안해봄
[stackoverflow][https://link](https://stackoverflow.com/questions/39918814/use-jsonreader-setlenienttrue-to-accept-malformed-json-at-line-1-column-1-path)


2020-05-08
===

```
// base url 은 '/' 로 끝나야함
const val BASE_URL_OIL = "https://github.com:10443/mktapp/control/"
companion object {
    val retrofit = Retrofit.Builder().baseUrl(BASE_URL_OIL)
        .addConverterFactory(GsonConverterFactory.create())

    // T 는 callback 의 대상이 됨
    fun <T> retrofitCallback(call: Call<T>, remains: (T) -> Unit) {
        call.enqueue(object : Callback<T> {
            override fun onFailure(call: Call<T>, t: Throwable) {
                val TAG = "RetrofitCallback"
                Log.e(TAG, "$t");
            }
            override fun onResponse(call: Call<T>, response: Response<T>) {
                val data = response.body()
                remains(data!!)
            }
        })
    }
}
```
  **usage**
```
fun loadEventList(){
    val service = ApiService.retrofit.build().create(ApiService::class.java)
    val callback = service.eventList()

    val remain: (EventList002) -> Unit = {
            _eventList.value = it.RETLIST
    }
    ApiService.retrofitCallback(callback, remains = remain)
}
```
<br>

retrofit
함수화
```
fun enqueueCall(query:String){
    
    val call = service.searchImages(getString(R.string.pixa_key), query, 1, true)
    call.enqueue(object : Callback<Photo>{
        override fun onFailure(call: Call<Photo>, t: Throwable) {
        }

        override fun onResponse(call: Call<Photo>, response: Response<Photo>) {
            val list = response.body()?.hits ?: listOf(Hits())
            val pager = ViewPager2(this@TestBottomActivity)
            pager.adapter = ViewPagerAdapter(list)
            pager_container.addView(pager)
        }
    })
}
```