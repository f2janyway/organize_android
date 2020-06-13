2020-05-16
===


```
package com.hyundaioilbank.android.map

//https://gun0912.tistory.com/57  클릭시 색 변동 및 레이아웃 관련
class MapActivity : BaseActivityNoViewModel<ActivityMapBinding>(), OnMapReadyCallback,
    GoogleMap.OnMarkerClickListener, GoogleMap.OnMapClickListener {

    val code: String by lazy {
MapActivity

class MapActivity : AppCompatActivity(), OnMapReadyCallback, GoogleMap.OnMarkerClickListener {

    val code: String by lazy {
//        Log.e(TAG, ": ${intent.getStringExtra("stationCode")} < code")
        intent.getStringExtra("stationCode")
    }

    private lateinit var mMap: GoogleMap
    //현재 위치 얻는
    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var lastLocation: Location

    private lateinit var locationCallback: LocationCallback

    private lateinit var locationRequest: LocationRequest
    private var locationUpdateState = false
    private val TAG = "Map"


    var selectedMarker: Marker? = null
    var selectedMarkerItem: MarkerItem? = null
    private val sheetBehavior by lazy {
        BottomSheetBehavior.from(binding.includeMapBottomSheet.bottomSheetParent)
    }
    val bottomInitHeight: Int by lazy {
        sheetBehavior.peekHeight
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        super.setContentView(R.layout.activity_map)

        binding.act = this
        binding.includeMapBottomSheet.act = this
        hideActionBar()
        // Obtain the SupportMapFragment and get notified when the map is ready to be used.
        sheetBehavior.apply {
            //options
            peekHeight = 350
            isFitToContents = false
            halfExpandedRatio = 0.5f
            isHideable = false
            setExpandedOffset(400)
            Log.e(TAG, "onCreate: bottom init height : ${bottomInitHeight}")
        }

        sheetBehavior.addBottomSheetCallback(object : BottomSheetBehavior.BottomSheetCallback() {
            val TAG = "BottomSheet Callback"
            override fun onSlide(bottomSheet: View, slideOffset: Float) {
            }

            override fun onStateChanged(bottomSheet: View, newState: Int) {
                when (newState) {
                    BottomSheetBehavior.STATE_HIDDEN -> {
                        Log.e(TAG, "onStateChanged: hidden ")
                    }
                    BottomSheetBehavior.STATE_EXPANDED -> {
                        Log.e(TAG, "onStateChanged: expanded")
                    }
                    BottomSheetBehavior.STATE_HALF_EXPANDED -> {
                        Log.e(TAG, "onStateChanged: half expanded")
                    }
                    BottomSheetBehavior.STATE_COLLAPSED -> {
                        Log.e(TAG, "onStateChanged: collapesd")
                    }
                    BottomSheetBehavior.STATE_DRAGGING -> {
                        Log.e(TAG, "onStateChanged: dragging")
                    }
                    BottomSheetBehavior.STATE_SETTLING -> {
                        Log.e(TAG, "onStateChanged: setting")
                    }
                }
            }

        })
        val mapFragment = supportFragmentManager
            .findFragmentById(R.id.map) as SupportMapFragment?

        mapFragment!!.getMapAsync(this)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

        locationCallback = object : LocationCallback() {
            override fun onLocationResult(p0: LocationResult?) {
                super.onLocationResult(p0)
                lastLocation = p0!!.lastLocation
//                placeMarkerOnMap(LatLng(lastLocation.latitude, lastLocation.longitude))
            }
        }

        createLocationRequest()
    }



    fun search() {
        val service = retrofit.build().create(ApiService::class.java)
        val callback = service.stationDetail(COILST = code)
        val remain: (StationDetail018) -> Unit = {
            if (it != null) {
                moveMap(it)
            }
        }
        retrofitCallback(call = callback, remains = remain)
    }


    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == REQUEST_CHECK_SETTINGS) {
            if (resultCode == Activity.RESULT_OK) {
                locationUpdateState = true
//                startLocationUpdates()
                search()
            }
        } else
            finish()
    }

    // 2
    override fun onPause() {
        super.onPause()
        fusedLocationClient.removeLocationUpdates(locationCallback)
    }

    // 3
    public override fun onResume() {
        super.onResume()
//        if (!locationUpdateState) {
//            startLocationUpdates()
//        }
    }

    private fun createLocationRequest() {
        // 1
        locationRequest = LocationRequest()
        // 2
        locationRequest.interval = 10000
        // 3
        locationRequest.fastestInterval = 5000
        locationRequest.priority = LocationRequest.PRIORITY_HIGH_ACCURACY

        val builder = LocationSettingsRequest.Builder()
            .addLocationRequest(locationRequest)

        // 4
        val client = LocationServices.getSettingsClient(this)
        val task = client.checkLocationSettings(builder.build())

        // 5
        task.addOnSuccessListener {
            locationUpdateState = true
//            startLocationUpdates()
        }
        task.addOnFailureListener { e ->
            // 6
            if (e is ResolvableApiException) {
                // Location settings are not satisfied, but this can be fixed
                // by showing the user a dialog.
                try {
                    // Show the dialog by calling startResolutionForResult(),
                    // and check the result in onActivityResult().
                    e.startResolutionForResult(
                        this@MapActivity,
                        REQUEST_CHECK_SETTINGS
                    )
                } catch (sendEx: IntentSender.SendIntentException) {
                    // Ignore the error.
                }
            }
        }
    }
// 얘는 현재 위치를 계속 체크 얘두

    private fun startLocationUpdates() {
        //1
        if (ActivityCompat.checkSelfPermission(
                this,
                android.Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(android.Manifest.permission.ACCESS_FINE_LOCATION),
                LOCATION_PERMISSION_REQUEST_CODE
            )
            return
        }
        //2
        fusedLocationClient.requestLocationUpdates(
            locationRequest,
            locationCallback,
            null /* Looper */
        )
    }

    //permission
    companion object {
        private const val LOCATION_PERMISSION_REQUEST_CODE = 1
        private const val REQUEST_CHECK_SETTINGS = 2
    }


    private fun setUpMap() {
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), LOCATION_PERMISSION_REQUEST_CODE
            )
            return
        } else
            search()

    }


    override fun onMapReady(map: GoogleMap?) {
        mMap = map!!

        mMap.uiSettings.isZoomControlsEnabled = true
        mMap.setOnMarkerClickListener(this)
        setUpMap()
        getSampleMarkerItem()
        mMap.isMyLocationEnabled = true
        mMap.uiSettings.isMyLocationButtonEnabled = false
//        map.mapType = GoogleMap.MAP_TYPE_TERRAIN
        // 2
        fusedLocationClient.lastLocation.addOnSuccessListener(this,onSuccessListener)
        mMap.setOnCameraIdleListener(onCameraIdleListener)
    }
    private val onSuccessListener: OnSuccessListener<Location> = OnSuccessListener {
        if (it != null) {
            lastLocation = it
            val currentLatLng = LatLng(it.latitude, it.longitude)
            mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(currentLatLng, 13f))
        }
    }

    private val onCameraIdleListener by lazy {
        GoogleMap.OnCameraIdleListener {
            val center = mMap.cameraPosition.target
            val min = sampleList.minBy {
                calDistance(center.latitude, center.longitude, it.lat, it.lon)
            }
            min!!.isSelected = true
            changeSelectedMarker(addMarker(min))
            Log.e(TAG, "$min")
        }
    }

    private fun calDistance(cLat: Double, cLon: Double, mLat: Double, mLon: Double): Double {
        return sqrt((cLat - mLat).pow(2) + (cLon - mLon).pow(2))
    }

    override fun onMarkerClick(marker: Marker?): Boolean {
        val center = CameraUpdateFactory.newLatLng(marker!!.position)
        mMap.animateCamera(center)
        changeSelectedMarker(marker)
        return false
    }

    override fun onMapClick(p0: LatLng?) {
        changeSelectedMarker(null)
        Log.e(TAG, "$p0 < onMapClick")
    }

    private fun changeSelectedMarker(marker: Marker?) {
        // 선택했던 마커 되돌리기
        if (selectedMarker != null) {
            addMarker(selectedMarker!!, false)
            selectedMarker!!.remove()
        }
        // 선택한 마커 표시
        if (marker != null) {
            selectedMarker = addMarker(marker, true)
            marker.remove()
        }
    }

    val sampleList by lazy { ArrayList<MarkerItem>() }
    val markerRootView by lazy { LayoutInflater.from(this).inflate(R.layout.marker_layout, null) }
    val makerTextView by lazy<TextView> { markerRootView.findViewById(R.id.marker_textview) }
    private fun getSampleMarkerItem() {
        sampleList.add(MarkerItem(37.5674320, 126.9216805, "5,000", false))
        sampleList.add(MarkerItem(37.5570458, 126.9416835, "6,000", false))
        sampleList.add(MarkerItem(37.5475456, 126.9216801, "1,000", false))
        sampleList.add(MarkerItem(37.5555456, 126.9296801, "2,000", false))
        sampleList.add(MarkerItem(37.5505456, 126.9326801, "3,000", false))

        for (i in sampleList) {
            addMarker(i)
        }
    }

    private fun addMarker(markerItem: MarkerItem): Marker {
        val pos = LatLng(markerItem.lat, markerItem.lon)
        makerTextView.text = markerItem.price
        if (markerItem.isSelected) {
            makerTextView.setTextColor(Color.RED)
        } else {
            makerTextView.setTextColor(Color.MAGENTA)
        }
        val markerOptions = MarkerOptions()
        markerOptions.title(markerItem.price)
        markerOptions.position(pos)
        markerOptions.icon(
            BitmapDescriptorFactory.fromBitmap(
                createDrawableFromView(
                    this,
                    markerRootView
                )
            )
        )
        return mMap.addMarker(markerOptions)
    }

    private fun addMarker(marker: Marker, isSelected: Boolean): Marker {
        val lat = marker.position.latitude
        val lon = marker.position.longitude
        val price = marker.title
        val item = MarkerItem(lat, lon, price, isSelected)
        if (isSelected) {
            selectedMarkerItem = item
        }
        return addMarker(item)
    }

    private fun createDrawableFromView(context: Context, view: View): Bitmap? {
        val displayMetrics = DisplayMetrics()
        (context as Activity).windowManager.defaultDisplay.getMetrics(displayMetrics)
        view.layoutParams = ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.WRAP_CONTENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        view.measure(displayMetrics.widthPixels, displayMetrics.heightPixels)
        view.layout(0, 0, displayMetrics.widthPixels, displayMetrics.heightPixels)
        view.buildDrawingCache()
        val bitmap: Bitmap = Bitmap.createBitmap(
            view.measuredWidth,
            view.measuredHeight,
            Bitmap.Config.ARGB_8888
        )
        val canvas = Canvas(bitmap)
        view.draw(canvas)
        return bitmap
    }

    private fun getAddress(latLng: LatLng): String {
        // 1
        // 주소 안되네...
        val geocoder = Geocoder(this)
        val addresses: List<Address>?
        val address: Address?
        var addressText = ""
        try {
            // 2
            addresses = geocoder.getFromLocation(latLng.latitude, latLng.longitude, 1)
            // 3
            if (null != addresses && addresses.isNotEmpty()) {
                address = addresses[0]
                for (i in 0 until address.maxAddressLineIndex) {
                    addressText += if (i == 0) address.getAddressLine(i) else "\n" + address.getAddressLine(
                        i
                    )
                }
            }
        } catch (e: IOException) {
            Log.e("MapsActivity", e.localizedMessage)
        }
        return addressText
    }


    fun moveMap(it: StationDetail018) {
        val latLng = LatLng(
            it.RETMAP[0].NLATIT!!.toDouble(),
            it.RETMAP[0].NLOGIT!!.toDouble()
        )
        mMap.addMarker(
            MarkerOptions().position(
                latLng
            ).title(it.RETMAP[0].MOILST)
        )
        val cameraPosition = CameraPosition.Builder()
            .target(latLng) // Center Set
            .zoom(15f) // Zoom
            .build()
        mMap.moveCamera(CameraUpdateFactory.newLatLng(latLng))
        mMap.animateCamera(CameraUpdateFactory.newCameraPosition(cameraPosition))
    }

    fun closeButton() {
        finish()
    }

    //onClick
    fun getcloseMarker(way: Int /*-1:left,  1:right*/) {
        val selectedLon = selectedMarkerItem!!.lon

        val nextMarkerItem = when (way) {
            -1 -> {
                sampleList
                    .filter { it.lon < selectedLon }
                    .minBy { selectedLon - it.lon }
            }
            1 -> {
                sampleList
                    .filter { it.lon > selectedLon }
                    .minBy { it.lon - selectedLon }
            }
            else -> {
                // 처리 해야함
                sampleList.filter {
                    it.isSelected
                }[0]
            }
        }
        if (nextMarkerItem != null) {
            changeSelectedMarker(addMarker(nextMarkerItem))
            val center =
                CameraUpdateFactory.newLatLng(LatLng(nextMarkerItem.lat, nextMarkerItem.lon))
            mMap.moveCamera(center)
        }
    }

    fun moveToMyLocation(){
        val ll = LatLng(lastLocation.latitude,lastLocation.longitude)
        val my  = CameraUpdateFactory.newLatLng(ll)
        mMap.moveCamera(my)
    }

    fun searchMap(){
        startActivity(Intent(this,FindStationActivity::class.java))
    }
}

```


2020-05-21
==
대충 됨
```
package com.hyundaioilbank.android.map

//https://gun0912.tistory.com/57  클릭시 색 변동 및 레이아웃 관련
// have to viewmodel & clustering ( 잘 안됨 내가 짠 코드가 아니라 이해 필)
class MapActivity : BaseActivityNoViewModel<ActivityMapBinding>(), OnMapReadyCallback,
    /*GoogleMap.OnMarkerClickListener,*/ /*GoogleMap.OnMapClickListener,*/
    ClusterManager.OnClusterItemClickListener<MarkerItem> {

    val code: String by lazy {
        intent.getStringExtra("stationCode")
    }

    private lateinit var mMap: GoogleMap

    //현재 위치 얻는
    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var lastLocation: Location

    private lateinit var locationCallback: LocationCallback

    private lateinit var locationRequest: LocationRequest
    private var locationUpdateState = false
    private val TAG = "Map"


    private val sheetBehavior by lazy {
        BottomSheetBehavior.from(binding.includeMapBottomSheet.bottomSheetParent)
    }
    val bottomInitHeight: Int by lazy {
        sheetBehavior.peekHeight
    }

    val clusterManager: MyClusterManager<MarkerItem> by lazy {
        MyClusterManager<MarkerItem>(this, mMap)
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        super.setContentView(R.layout.activity_map)

        binding.act = this
        binding.includeMapBottomSheet.act = this
        hideActionBar()
        // Obtain the SupportMapFragment and get notified when the map is ready to be used.
        sheetBehavior.apply {
            //options
            peekHeight = 350
            isFitToContents = false
            halfExpandedRatio = 0.5f
            isHideable = false
            setExpandedOffset(400)
            Log.e(TAG, "onCreate: bottom init height : ${bottomInitHeight}")
        }

        sheetBehavior.addBottomSheetCallback(object : BottomSheetBehavior.BottomSheetCallback() {
            val TAG = "BottomSheet Callback"
            override fun onSlide(bottomSheet: View, slideOffset: Float) {
            }

            override fun onStateChanged(bottomSheet: View, newState: Int) {
                when (newState) {
                    BottomSheetBehavior.STATE_HIDDEN -> {
                        Log.e(TAG, "onStateChanged: hidden ")
                    }
                    BottomSheetBehavior.STATE_EXPANDED -> {
                        Log.e(TAG, "onStateChanged: expanded")
                    }
                    BottomSheetBehavior.STATE_HALF_EXPANDED -> {
                        Log.e(TAG, "onStateChanged: half expanded")
                    }
                    BottomSheetBehavior.STATE_COLLAPSED -> {
                        Log.e(TAG, "onStateChanged: collapesd")
                    }
                    BottomSheetBehavior.STATE_DRAGGING -> {
                        Log.e(TAG, "onStateChanged: dragging")
                    }
                    BottomSheetBehavior.STATE_SETTLING -> {
                        Log.e(TAG, "onStateChanged: setting")
                    }
                }
            }
        })
        val mapFragment = supportFragmentManager
            .findFragmentById(R.id.map) as SupportMapFragment?
        mapFragment!!.getMapAsync(this)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)

        locationCallback = object : LocationCallback() {
            override fun onLocationResult(p0: LocationResult?) {
                super.onLocationResult(p0)
                lastLocation = p0!!.lastLocation
                Log.e(TAG, "location >> $p0")
//                placeMarkerOnMap(LatLng(lastLocation.latitude, lastLocation.longitude))
            }
        }
    }

    override fun onMapReady(map: GoogleMap?) {
        mMap = map!!


        mMap.uiSettings.isZoomControlsEnabled = true
        setUpMap()
        getSampleMarkerItem()
        mMap.isMyLocationEnabled = true
        mMap.uiSettings.isMyLocationButtonEnabled = false
//        map.mapType = GoogleMap.MAP_TYPE_TERRAIN
        // 2
        fusedLocationClient.lastLocation.addOnSuccessListener(this, onSuccessListener)

        clusterManager.renderer = ClusterRenderer(this, mMap, clusterManager)
//        clusterManager.setOnClusterItemClickListener(this)
        mMap.setOnCameraIdleListener(clusterManager)
        mMap.setOnMarkerClickListener(clusterManager)
        clusterManager.cluster()

    }

    fun search() {
        val service = retrofit.create(ApiService::class.java)
        val callback = service.stationDetail(COILST = code)
        val remain: (StationDetail018) -> Unit = {
            if (it != null) {
                moveMap(it)
            }
        }
        retrofitCallback(call = callback, remains = remain)
    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == REQUEST_CHECK_SETTINGS) {
            if (resultCode == Activity.RESULT_OK) {
                locationUpdateState = true
                search()
            }
        } else
            finish()
    }

    // 2
    override fun onPause() {
        super.onPause()
        fusedLocationClient.removeLocationUpdates(locationCallback)
    }

    //permission
    companion object {
        private const val LOCATION_PERMISSION_REQUEST_CODE = 1
        private const val REQUEST_CHECK_SETTINGS = 2
    }

    private fun setUpMap() {
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), LOCATION_PERMISSION_REQUEST_CODE
            )
            return
        }
    }

    private val onSuccessListener: OnSuccessListener<Location> = OnSuccessListener {
        if (it != null) {
            lastLocation = it
            val currentLatLng = LatLng(it.latitude, it.longitude)
            mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(currentLatLng, 13f))
        }
    }

    private fun calDistance(cLat: Double, cLon: Double, mLat: Double, mLon: Double): Double {
        return sqrt((cLat - mLat).pow(2) + (cLon - mLon).pow(2))
    }

//    override fun onMarkerClick(marker: Marker?): Boolean {
//        val center = CameraUpdateFactory.newLatLng(marker!!.position)
//        mMap.moveCamera(center)
//        return false
//    }

    val sampleList by lazy { ArrayList<MarkerItem>() }
    val markerRootView by lazy { LayoutInflater.from(this).inflate(R.layout.marker_layout, null) }
    val markerTextView by lazy<TextView> { markerRootView.findViewById(R.id.marker_textview) }
    val markerImageView by lazy<ImageView> { markerRootView.findViewById(R.id.marker_category) }
    private fun getSampleMarkerItem() {
        sampleList.add(MarkerItem(37.5674320, 126.9216805, "5,000", false))
        sampleList.add(MarkerItem(37.5570458, 126.9416835, "6,000", false))
        sampleList.add(MarkerItem(37.5475456, 126.9216801, "1,000", false))
        sampleList.add(MarkerItem(37.5555456, 126.9296801, "2,000", false))
        sampleList.add(MarkerItem(37.5505456, 126.9326801, "3,000", false))

        for (i in sampleList) {
            clusterManager.addItems(sampleList)
        }
    }


    private inner class ClusterRenderer(
        context: Context,
        mMap: GoogleMap,
        clusterManager: ClusterManager<MarkerItem>
    ) : DefaultClusterRenderer<MarkerItem>(context, mMap, clusterManager) {
        var cnt = 0;
        override fun onBeforeClusterItemRendered(item: MarkerItem?, markerOptions: MarkerOptions?) {

            val centerItem = getCenterItem()
            selectedMarkerItem = centerItem
            Log.e(TAG, "${centerItem!!.position} < centeredItem-----")
            Log.e(TAG, "$cnt < cnt")

            if (item!!.position == centerItem.position) {
                Log.e(TAG, "${item.position} < it true---------")
                item.isSelected = true
                markerOptions!!
                    .position(item.position)
                    .icon(getBitmapDescriptor(true))
            } else {
                Log.e(TAG, "${item.position} < it false--------")
                item.isSelected = false
                markerOptions!!
                    .position(item.position)
                    .icon(getBitmapDescriptor(false))
            }
            super.onBeforeClusterItemRendered(item, markerOptions)
        }

    }

    @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
    private fun getBitmapDescriptor(isCentered: Boolean): BitmapDescriptor {

        return if (isCentered) {
            markerTextView.setTextColor(Color.RED)
            markerImageView.setImageDrawable(getDrawable(android.R.drawable.arrow_down_float))
            BitmapDescriptorFactory.fromBitmap(
                createDrawableFromView(
                    applicationContext,
                    markerRootView
                )
            )
        } else {
            markerTextView.setTextColor(Color.BLACK)
            markerImageView.setImageDrawable(getDrawable(android.R.drawable.arrow_down_float))
            BitmapDescriptorFactory.fromBitmap(
                createDrawableFromView(
                    applicationContext,
                    markerRootView
                )
            )
        }
    }

    private fun getCenterItem(): MarkerItem? {
        val center = mMap.cameraPosition.target
        val min = sampleList.asSequence().minBy {
            calDistance(
                center.latitude,
                center.longitude,
                it.position.latitude,
                it.position.longitude
            )
        }
        return min
    }


    private fun createDrawableFromView(
        context: Context, view:
        View
    ): Bitmap? {
        val displayMetrics = DisplayMetrics()
        windowManager.defaultDisplay.getMetrics(displayMetrics)
        view.layoutParams = ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.WRAP_CONTENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        view.measure(displayMetrics.widthPixels, displayMetrics.heightPixels)
        view.layout(0, 0, displayMetrics.widthPixels, displayMetrics.heightPixels)
        view.buildDrawingCache()
        val bitmap: Bitmap = Bitmap.createBitmap(
            view.measuredWidth,
            view.measuredHeight,
            Bitmap.Config.ARGB_8888
        )
        val canvas = Canvas(bitmap)
        view.draw(canvas)
        return bitmap
    }

    fun moveMap(it: StationDetail018) {
        val latLng = LatLng(
            it.RETMAP[0].NLATIT!!.toDouble(),
            it.RETMAP[0].NLOGIT!!.toDouble()
        )
        mMap.moveCamera(CameraUpdateFactory.newLatLng(latLng))
//        val cameraPosition = CameraPosition.Builder()
//            .target(latLng) // Center Set
//            .zoom(15f) // Zoom
//            .build()
//        mMap.animateCamera(CameraUpdateFactory.newCameraPosition(cameraPosition))
    }

    //onClick 계속
    fun closeButton() = finish()
    fun searchMap() = startActivity(Intent(this, FindStationActivity::class.java))

    private var selectedMarkerItem: MarkerItem? = null

    // -1:left,  1:right
    fun getcloseMarker(way: Int) {
        val selectedLon = selectedMarkerItem!!.lon

        val nextMarkerItem = when (way) {
            -1 -> {
                sampleList
                    .filter { it.lon < selectedLon }
                    .minBy { selectedLon - it.lon }
            }
            1 -> {
                sampleList
                    .filter { it.lon > selectedLon }
                    .minBy { it.lon - selectedLon }
            }
            else -> {
                // 처리 해야함
                sampleList.filter {
                    it.isSelected
                }[0]
            }
        }
        if (nextMarkerItem != null) {
            val center =
                CameraUpdateFactory.newLatLng(LatLng(nextMarkerItem.lat, nextMarkerItem.lon))
            mMap.moveCamera(center)
        }
    }

    fun moveToMyLocation() {
        val ll = LatLng(lastLocation.latitude, lastLocation.longitude)
        val my = CameraUpdateFactory.newLatLng(ll)
        mMap.moveCamera(my)
    }

    override fun onClusterItemClick(p0: MarkerItem?): Boolean = true

    inner class MyClusterManager<T : ClusterItem>(context: Context?, map: GoogleMap?) :
        ClusterManager<T>(context, map) {
        override fun onCameraIdle() {
            super.onCameraIdle()
            val min = getCenterItem()

            val markers = clusterManager.markerCollection.markers
            for (i in markers) {
                if (i.position == min!!.position) {
                    i.setIcon(getBitmapDescriptor(true))
                } else {
                    i.setIcon(getBitmapDescriptor(false))
                }
            }
        }

        override fun onMarkerClick(marker: Marker?): Boolean {
            mMap.moveCamera(CameraUpdateFactory.newLatLng(marker!!.position))
            return super.onMarkerClick(marker)
        }
    }


}


```


2020-06-13
=
80 프로 완성

```
class MapActivity : BaseActivity<ActivityMapBinding, MapViewModel>(), OnMapReadyCallback,
    /*GoogleMap.OnMarkerClickListener,*/ /*GoogleMap.OnMapClickListener,*/
    ClusterManager.OnClusterItemClickListener<RETLISTMarker005> {

    var isSearch: Boolean = false

    private lateinit var mMap: GoogleMap

    //현재 위치 얻는
    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var lastLocation: Location

    private lateinit var locationCallback: LocationCallback

    private lateinit var locationRequest: LocationRequest
    private var locationUpdateState = false
    private val TAG = "Map"

    private var centerMarkerItem: RETLISTMarker005? = null
    private val sheetBehavior by lazy {
        BottomSheetBehavior.from(binding.includeMapBottomSheet.bottomSheetParent)
    }

    val clusterManager: MyClusterManager<RETLISTMarker005> by lazy {
        MyClusterManager<RETLISTMarker005>(this, mMap)
    }


    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        super.setContentView(R.layout.activity_map)

        loading(supportFragmentManager,2000)

        networkCheck(this)
        binding.act = this
        binding.includeMapBottomSheet.act = this
        hideActionBar()
        /**
         */
        // Obtain the SupportMapFragment and get notified when the map is ready to be used.
        sheetBehavior.apply {
            val inner = binding.includeMapBottomSheet.bottomSheetInnerConst
            inner.viewTreeObserver.addOnGlobalLayoutListener {
                peekHeight = inner.getChildAt(4).top
            }
            isFitToContents = true
//            halfExpandedRatio = 0.5f
            isHideable = false
//            setExpandedOffset(400)
        }

        sheetBehavior.addBottomSheetCallback(object : BottomSheetBehavior.BottomSheetCallback() {
            val TAG = "BottomSheet Callback"
            override fun onSlide(bottomSheet: View, slideOffset: Float) {
            }

            override fun onStateChanged(bottomSheet: View, newState: Int) {
                when (newState) {
                    BottomSheetBehavior.STATE_HIDDEN -> {
                        Log.e(TAG, "onStateChanged: hidden ")
                    }
                    BottomSheetBehavior.STATE_EXPANDED -> {
                        binding.includeMapBottomSheet.bottomSheetToggleArrow.rotation = 180f
                    }
                    BottomSheetBehavior.STATE_HALF_EXPANDED -> {
                        Log.e(TAG, "onStateChanged: half expanded")
                    }
                    BottomSheetBehavior.STATE_COLLAPSED -> {
                        Log.e(TAG, "onStateChanged: collapesd")
                        binding.includeMapBottomSheet.bottomSheetToggleArrow.rotation = 0f
                    }
                    BottomSheetBehavior.STATE_DRAGGING -> {
                        Log.e(TAG, "onStateChanged: dragging")
                    }
                    BottomSheetBehavior.STATE_SETTLING -> {
                        Log.e(TAG, "onStateChanged: setting")
                    }
                }
            }
        })
        val mapFragment = supportFragmentManager
            .findFragmentById(R.id.map) as SupportMapFragment?
        mapFragment!!.getMapAsync(this)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)


        locationCallback = object : LocationCallback() {
            override fun onLocationResult(p0: LocationResult?) {
                super.onLocationResult(p0)
                lastLocation = p0!!.lastLocation
                Log.e(TAG, "location >> $p0")
//                placeMarkerOnMap(LatLng(lastLocation.latitude, lastLocation.longitude))
            }
        }
        checkMapPermission(this@MapActivity)

        viewModel.stations005.observe(this, Observer {
            Log.e("MapActivity", "onMapReady: $list")
            Log.e("MapActivity", "onMapReady: ${list.count()} <<<<<<<<")
            if (isClean) {
                val cleanList = it.RETLIST.filter {
                    it.ECWASHMBR == "Y"
                }
                if (cleanList.count() > 0) list =
                    cleanList as ArrayList<RETLISTMarker005> else list.clear()
            } else {
                list = it.RETLIST
            }

            clusterManager.clearItems()
//            mMap.clear()
            clusterManager.addItems(list)
            clusterManager.cluster()
//            onMapReady(mMap)

        })
        fusedLocationClient.lastLocation.addOnSuccessListener {
            Log.e("MapActivity", "onCreate:  location $it")
            viewModel.getStations(
                null,
                null,
                "2",
                null,
                latWido = it.latitude.toString(),
                lonGyungdo = it.longitude.toString(),
                filter = null,
                radius = "10"
            )
        }

    }

    override fun onMapReady(map: GoogleMap?) {
        checkMapPermission(this)
        mMap = map!!
        mMap.uiSettings.isZoomControlsEnabled = false
        setUpMap()

        Log.e("MapActivity", "onMapReady: list.count() : ${list.count()}")

        mMap.isMyLocationEnabled = true
        mMap.uiSettings.isMyLocationButtonEnabled = false
//        map.mapType = GoogleMap.MAP_TYPE_TERRAIN
        // 2
        fusedLocationClient.lastLocation.addOnSuccessListener(this, onSuccessListener)

        clusterManager.renderer = ClusterRenderer(this, mMap, clusterManager)
//        clusterManager.setOnClusterItemClickListener(this)
        mMap.setOnCameraIdleListener(clusterManager)
        mMap.setOnMarkerClickListener(clusterManager)

        if (isSearch)
            search()
//        else{
//            //현재 위치에서
//            getCenterItem()
//        }

        clusterManager.cluster()
    }

    private fun bindCenterMarkerBottomSheet() {
        if (centerMarkerItem != null) {
            binding.includeMapBottomSheet.apply {
                bottomSheetStationName.text = centerMarkerItem?.MOILST
                bottomSheetStationAddress.text = centerMarkerItem?.JUSO
                checkCategoryAndShow(centerMarkerItem!!)

            }

        }
    }

    //findstationadapter 중복
    private fun checkCategoryAndShow(it: RETLISTMarker005) {
        binding.includeMapBottomSheet.bottomSheetCateInclude.apply {
            var cnt = 0
            //세차기 시설
            cnt += cateVisibleOrGone(it.ECWASH ?: "N", findStationCateCleaner)
            //세차 멤버쉽
            cnt += cateVisibleOrGone(it.ECWASHMBR ?: "N", findStationCateCleanMembership)
            //경정비 시설 여부
            cnt += cateVisibleOrGone(it.EREPAIR ?: "N", findStationCatePair)
            //편의점
            cnt += cateVisibleOrGone(it.ECVS ?: "N", findStationCateCu)
            cnt += cateVisibleOrGone(it.ESELF ?: "N", findStationCateSelf)
            cnt += cateVisibleOrGone(it.ECARGO ?: "N", findStationCateLikeDump)

            if (cnt == 0) findStationCateBlank.visibility =
                View.VISIBLE else findStationCateBlank.visibility = View.GONE
        }
    }

    //findstationadapter 중복
    private fun cateVisibleOrGone(param: String, view: LinearLayout): Int {
        return if (param == "Y") {
            view.visibility = View.VISIBLE
            1
        } else {
            view.visibility = View.GONE
            0
        }
    }

    fun clickToggle() {
        binding.apply {
            if (sheetBehavior.state != BottomSheetBehavior.STATE_EXPANDED)
                sheetBehavior.state = BottomSheetBehavior.STATE_EXPANDED
            else
                sheetBehavior.state = BottomSheetBehavior.STATE_COLLAPSED
        }
    }


    override fun onRequestPermissionsResult(
        requestCode: Int,
        permissions: Array<out String>,
        grantResults: IntArray
    ) {
//        super.onRequestPermissionsResult(requestCode, permissions, grantResults)
        if (requestCode == MAP_PERMISSION_CODE) {
            if (grantResults.isNotEmpty() && grantResults[0] != PackageManager.PERMISSION_GRANTED) {
                finish()
            }
        }
    }

    fun search() {
        //test  filter 을 여기로 보내려면 intent 로 일일히 다 넣던가 아니면 parceable
        viewModel.getStations(
            searchWord = null,
            radius = "10",
            filter = null,
            lonGyungdo = intent.getStringExtra("lon")!!,
            latWido = intent.getStringExtra("lon")!!,
            whichSearch = "2",
            sigugun = null, sido = null
        )
        isSearch = false
//        onMapReady(mMap)

    }

    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == REQUEST_CHECK_SETTINGS) {
            if (resultCode == Activity.RESULT_OK) {
                locationUpdateState = true
                search()
            }
        } else
            finish()
    }

    // 2
    override fun onPause() {
        super.onPause()
        fusedLocationClient.removeLocationUpdates(locationCallback)
    }

    //permission
    companion object {
        private const val LOCATION_PERMISSION_REQUEST_CODE = 1
        private const val REQUEST_CHECK_SETTINGS = 2
    }

    private fun setUpMap() {
        if (ActivityCompat.checkSelfPermission(
                this,
                Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this,
                arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), LOCATION_PERMISSION_REQUEST_CODE
            )
            return
        }
    }

    private val onSuccessListener: OnSuccessListener<Location> = OnSuccessListener {
        if (it != null) {
            lastLocation = it
            val currentLatLng = LatLng(it.latitude, it.longitude)
            mMap.moveCamera(CameraUpdateFactory.newLatLngZoom(currentLatLng, 13f))
        }
    }

    private fun calDistance(cLat: Double, cLon: Double, mLat: Double, mLon: Double): Double {
        return sqrt((cLat - mLat).pow(2) + (cLon - mLon).pow(2))
    }


    var list = ArrayList<RETLISTMarker005>()
    var cleanList = ArrayList<RETLISTMarker005>()
    val markerRootView by lazy { LayoutInflater.from(this).inflate(R.layout.marker_layout, null) }
    val markerTextView by lazy<TextView> { markerRootView.findViewById(R.id.marker_textview) }
    val markerImageView by lazy<ImageView> { markerRootView.findViewById(R.id.marker_image) }

    private inner class ClusterRenderer(
        context: Context,
        mMap: GoogleMap,
        clusterManager: ClusterManager<RETLISTMarker005>
    ) : DefaultClusterRenderer<RETLISTMarker005>(context, mMap, clusterManager) {
        @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
        override fun onBeforeClusterItemRendered(
            item: RETLISTMarker005,
            markerOptions: MarkerOptions
        ) {


            val centerMarker = getCenterItem()


            Log.e(TAG, "${centerMarker!!.position} < centeredItem-----")

            if (item.position == centerMarker.position) {
                Log.e(TAG, "${item.position} < it true---------")
                item.isSelected = true
                if (binding.includeMapBottomSheet.bottomSheetStationName.text == "") {
                    centerMarkerItem = centerMarker
                    bindCenterMarkerBottomSheet()
                }
                markerOptions
                    .position(item.position)
                    .icon(getBitmapDescriptor(true))
            } else {
                Log.e(TAG, "${item.position} < it false--------")
                item.isSelected = false
                markerOptions
                    .position(item.position)
                    .icon(getBitmapDescriptor(false))
            }

            super.onBeforeClusterItemRendered(item, markerOptions)


        }
    }

    private fun getBitmapDescriptor(isCentered: Boolean): BitmapDescriptor {

        return if (isCentered) {
            markerImageView.apply {
                setBackgroundResource(R.drawable.map_marker_active)
            }
            markerTextView.setTextColor(Color.WHITE)
            markerTextView.visibility = View.VISIBLE
            BitmapDescriptorFactory.fromBitmap(
                createDrawableFromView(
                    applicationContext,
                    markerRootView
                )
            )
        } else {
//            markerTextView.setTextColor(Color.BLACK)
            if (isClean)
                markerImageView.setBackgroundResource(R.drawable.map_marker_2)
            else
                markerImageView.setBackgroundResource(R.drawable.map_marker_1)
            markerTextView.visibility = View.GONE

            BitmapDescriptorFactory.fromBitmap(
                createDrawableFromView(
                    applicationContext,
                    markerRootView
                )
            )
        }
    }

    private fun getCenterItem(): RETLISTMarker005? {
        val center = mMap.cameraPosition.target

        val min = list.asSequence().minBy {
            calDistance(
                center.latitude,
                center.longitude,
                it.position.latitude,
                it.position.longitude
            )
        }
        return min
    }


    private fun createDrawableFromView(
        context: Context, view:
        View
    ): Bitmap? {
        val displayMetrics = DisplayMetrics()
        windowManager.defaultDisplay.getMetrics(displayMetrics)
        view.layoutParams = ViewGroup.LayoutParams(
            ViewGroup.LayoutParams.WRAP_CONTENT,
            ViewGroup.LayoutParams.WRAP_CONTENT
        )
        view.measure(displayMetrics.widthPixels, displayMetrics.heightPixels)
        view.layout(0, 0, displayMetrics.widthPixels, displayMetrics.heightPixels)
        view.buildDrawingCache()
        val bitmap: Bitmap = Bitmap.createBitmap(
            view.measuredWidth,
            view.measuredHeight,
            Bitmap.Config.ARGB_8888
        )
        val canvas = Canvas(bitmap)
        view.draw(canvas)
        return bitmap
    }

    fun moveMap(it: StationDetail018) {
        Log.e("move map", "$it")
        val latLng = LatLng(
            it.RETMAP[0].NLATIT!!.toDouble(),
            it.RETMAP[0].NLOGIT!!.toDouble()
        )
        mMap.moveCamera(CameraUpdateFactory.newLatLng(latLng))
        val searchItem = RETLISTMarker005(
            NLATIT = latLng.latitude.toString(),
            NLOGIT = latLng.longitude.toString()
        )
        list.add(searchItem)
        clusterManager.addItem(searchItem)
    }

    //onClick 계속
    fun closeButton() = finish()
    fun searchMap() = startActivity(Intent(this, FindStationActivity::class.java))


    fun moveToMyLocation() {
        val ll = LatLng(lastLocation.latitude, lastLocation.longitude)
        val my = CameraUpdateFactory.newLatLng(ll)
        mMap.moveCamera(my)
    }

    override fun onClusterItemClick(p0: RETLISTMarker005?): Boolean = true

    inner class MyClusterManager<T : ClusterItem>(context: Context?, map: GoogleMap?) :
        ClusterManager<T>(context, map) {
        @RequiresApi(Build.VERSION_CODES.LOLLIPOP)
        override fun onCameraIdle() {
            super.onCameraIdle()

            val min = getCenterItem()

            Log.e("MyClusterManager", "onCameraIdle: $min")
//            if (min != null) {
            viewModel.getStations(
                null,
                null,
                "2",
                null,
                latWido = mMap.cameraPosition.target.latitude.toString(),
                lonGyungdo = mMap.cameraPosition.target.longitude.toString(),
                filter = null,
                radius = "10"
            )

            val markers = clusterManager.markerCollection.markers
            for (i in markers) {
                if (min != null && i.position == min.position) {
                    i.setIcon(getBitmapDescriptor(true))
                    centerMarkerItem = min
                    bindCenterMarkerBottomSheet()
                    Log.e("MyClusterManager", "onCameraIdle: $i")
                } else {
                    i.setIcon(getBitmapDescriptor(false))
                }
            }
//            }
        }

        override fun onMarkerClick(marker: Marker?): Boolean {
            mMap.moveCamera(CameraUpdateFactory.newLatLng(marker!!.position))
            return super.onMarkerClick(marker)
        }
    }

    fun clickSearh() {
        startActivity(Intent(this@MapActivity, FindStationActivity::class.java))
    }

    override fun getLayoutId(): Int = R.layout.activity_map

    override fun getOwner(): ViewModelStoreOwner = this

    override fun insertViewModelClass(): Class<MapViewModel> = MapViewModel::class.java


    var isClean: Boolean = false
    fun clickCleanMemberShipButton() {
        isClean = !isClean
        binding.isClean = isClean
    }
}

```