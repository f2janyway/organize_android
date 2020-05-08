2020-05-08
===


[참고 블로그](https://kor45cw.tistory.com/5)

## 1. @POST
1. @POST("/posts") : POST 방식의 통신이며, 주소는 위와 같은 방식입니다.

2. @FieldMap HashMap<String, Object> parameters : Field 형식을 통해 넘겨주는 값들이 여러개일 때 FieldMap을 사용합니다. Retrofit에서는 Map보다는 HashMap형식을 쓰기를 권장하고 있습니다.

3. @FormUrlEncoded : @Field 형식을 사용할 때는 Form이 Encoding되어야 합니다. 따라서 FormUrlEncoded라는 Annotation을 해주어야 합니다.

4. @Field 형식의 경우에는 주로 POST 방식의 통신을 할때 사용합니다. GET 방식에서는 사용이 불가능
<pre>
<code>
interface ApiService{
    ...
    @FormUrlEncoded
    @POST(IF001_LOGIN)
    fun login(
        @Field("MIDXXX") MIDXXX: String? = null,
        @Field("NBIMIL") NBIMIL: String? = null,
        @Field("MGN") MGN: String? = "R",
        @Field("NAPPVER") NAPPVER: String? = null,
        @Field("NOSTYPE") NOSTYPE: String? = null,
        @Field("NOSVER") NOSVER: String? = null,
        @Field("ELOGGB") ELOGGB: String? = null
    ): Call<Login001>
    ...
}
</code>
</pre>

## 2. @GET
```
@GET("/posts/{userId}") 
Call<ResponseGet> getFirst(@Path("userId") String id);

@GET("/posts") 
Call<List<ResponseGet>> getSecond(@Query("userId") String id);

```

## 3. @Multipart

```
@Multipart
@POST("/upload")
fun postImage(@Part image:MultipartBody.Part, @Part("username") id:RequestBody): Call<ResponseBody>
```
**usage**
```
//this method just upload image
fun uploadToServer(path: String?) {
     val service = ApiService.retrofit.create(ApiService::class.java)
     //여기선 이미지 파일 경로
     val file = File(path)
     Log.e("file path", "" + file + " <-path , name->" + file.name)
     val requestFile : RequestBody = RequestBody.create(MediaType.parse("multipart/form-data"), file)
     val part : MultipartBody.Part= MultipartBody.Part.createFormData("img", file.name, requestFile)
     val id : RequestBody = RequestBody.create(MediaType.parse("text/plain"), username)
//        val name= RequestBody.create(MediaType.parse("text/plain"), "name")
     service.postImage(part, id).enqueue(object : Callback<ResponseBody> {
         override fun onFailure(call: Call<ResponseBody>, t: Throwable) {
             Toast.makeText(this@Chat, "upload fail", Toast.LENGTH_SHORT)
                 .show()
             Log.e("fail", call.request().toString())
         }

         override fun onResponse(
             call: Call<ResponseBody>,
             response: Response<ResponseBody>
         ) {
             Toast.makeText(this@Chat, "upload success", Toast.LENGTH_SHORT)
                 .show()
         }
     })
 }
```