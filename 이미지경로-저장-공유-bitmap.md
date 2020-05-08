2020-05-08
===

## manifest 설정

권한 코드(permission) 필요
 ```
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />

```
## 이미지 경로
```
private fun getRealPathFromURI(contentURI: Uri?) {

    var file: String?
    if ("content" == contentURI!!.scheme) {
        val cursor: Cursor? = contentResolver.query(
            contentURI,
            arrayOf(MediaStore.Images.ImageColumns.DATA),
            null,
            null,
            null
        )
        cursor?.moveToFirst();
        file = cursor?.getString(0)
        cursor?.close()
    } else {
        file = contentURI.path
    }
    Log.e("Chosen path ", "Chosen path = $file");
//        return file
    uploadToServer(file)
}
```
<br>

## 이미지 uri from bitmap
```
private fun getImageUriFromBitmap(context: Context, bitmap: Bitmap): Uri? {
    val bytes = ByteArrayOutputStream()
    bitmap.compress(Bitmap.CompressFormat.JPEG, 100, bytes)
    var path: String? = null
    try {
        //insertImage deprecated
        path = MediaStore.Images.Media.insertImage(context.contentResolver,
                                                    bitmap,
                                                    "Title",
                                                    null)
    } catch (e: Exception) {
        e.printStackTrace()
        Toast.makeText(context, getString(R.string.permission), Toast.LENGTH_SHORT).show()
    }
    return Uri.parse(path) ?: null
}
```

<br>

## 이미지 저장
```
fun saveImage() {
    val mBitmap = (photoView.drawable as BitmapDrawable).bitmap

    val uri = getImageUriFromBitmap(activity!!, mBitmap!!)
    val path = /*ContextWrapper(context).getDir("images", Context.MODE_PRIVATE)*/
        getRealPathFromUri(uri!!)
    //val filepath = path + File.separator.toString() + "${UUID.randomUUID()}.jpg"
    val file = File(path!!)
    Log.e("file", file.absolutePath)
    try {
        if (!file.exists()) {
            file.createNewFile()
            Log.e("file not exists", file.absolutePath)
        }
        val fos = FileOutputStream(file)
        // job 을 control 하면 더 좋음
        GlobalScope.launch(Dispatchers.IO) {
            mBitmap.compress(Bitmap.CompressFormat.JPEG, 100, fos)
            fos.close()
            this.launch(Dispatchers.Main) {
                Toast.makeText(activity!!, "다운로드 됨", Toast.LENGTH_SHORT).show()
            }
        }
    } catch (e: IOException) {
        e.printStackTrace()
    }
}
```
<br>

## (이미지) 공유
```
private fun shareBitmap() {
    val mBitmap = (photoView.drawable as BitmapDrawable).bitmap

    val stream = ByteArrayOutputStream()
    mBitmap.compress(Bitmap.CompressFormat.JPEG, 10, stream)
    val shareIntent = Intent()

    //여기선 no used 
    val compressedBitmap = BitmapFactory.decodeByteArray(stream.toByteArray(),
                                                            0,
                                                            stream.toByteArray().size)
    shareIntent.apply {
        action = Intent.ACTION_SEND
        putExtra(Intent.EXTRA_STREAM, getImageUriFromBitmap(activity!!, mBitmap)
            /*Bitmap.createScaledBitmap(compressedBitmap, compressedBitmap.width /6, compressedBitmap.height/6, false) 쓸데 없구만 uri 로 보내야지*/)
        type = "image/*"
    }
    startActivity(Intent.createChooser(shareIntent, getString(R.string.share)))
}
```