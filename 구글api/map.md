2020-05-16
===
MapActivity
```

class MapActivity : AppCompatActivity(), OnMapReadyCallback, GoogleMap.OnMarkerClickListener {

    val code: String by lazy {
//        Log.e(TAG, ": ${intent.getStringExtra("stationCode")} < code")
        intent.getStringExtra("stationCode")
    }

    private lateinit var mMap: GoogleMap
    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var lastLocation: Location

    private lateinit var locationCallback: LocationCallback

    private lateinit var locationRequest: LocationRequest
    private var locationUpdateState = false
    private val TAG = "Map"

    private val sheetBehavior by lazy {
        BottomSheetBehavior.from(bottom_sheet_parent)
    }
    val bottomInitHeight : Int by lazy {
        sheetBehavior.peekHeight
    }
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_station_map)
     
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
                placeMarkerOnMap(LatLng(lastLocation.latitude, lastLocation.longitude))
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

        map.uiSettings.isZoomControlsEnabled = true
        map.setOnMarkerClickListener(this)


//        val lat = 37.554467
//        val lng = 126.928441
//        val sydney = LatLng(lat, lng)
//        mMap.addMarker(MarkerOptions().position(sydney).title("Marker in Sydney"))
        //zoom 0 - 20 or 13
//        mMap.moveCamera(CameraUpdateFactory.newLatLng(sydney,12.0f))


        setUpMap()

        map.isMyLocationEnabled = true
//        map.mapType = GoogleMap.MAP_TYPE_TERRAIN
        // 2
        fusedLocationClient.lastLocation.addOnSuccessListener(this) { location ->
            //3 Got last known location. In some rare situations this can be null.
            if (location != null) {
                lastLocation = location
                val currentLatLng = LatLng(location.latitude, location.longitude)
//                Log.e(TAG, "lat - ${currentLatLng.latitude} : long - ${currentLatLng.longitude}")
                placeMarkerOnMap(currentLatLng)
                map.animateCamera(CameraUpdateFactory.newLatLngZoom(currentLatLng, 13f))
            }
        }

    }

    private fun getAddress(latLng: LatLng): String {
        // 1
        // 주소 안되네...
        val geocoder = Geocoder(this)
        val addresses: List<Address>?
        val address: Address?
        var addressText = ""

        val addressT = StringBuilder()
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

//                    if (i == 0)
//                        addressT.append(address.getAddressLine(i))
//                    else
//                        addressT.append("\n" + address.getAddressLine(i))
                }
            }
        } catch (e: IOException) {
            Log.e("MapsActivity", e.localizedMessage)
        }

        return addressText
    }


    private fun placeMarkerOnMap(location: LatLng) {
        // 1
        val markerOptions = MarkerOptions().position(location)
        // 안되네...ㅠ
//        markerOptions.icon(
//            BitmapDescriptorFactory.fromBitmap(
//                BitmapFactory.decodeResource(resources, R.mipmap.)
//            )
//        )
        val titleString = getAddress(location)
//        Log.e(TAG, "$titleString <<< titleString")
        markerOptions.title(titleString)
        // 2
        mMap.addMarker(markerOptions)
    }

    fun moveMap(it: StationDetail018) {
        val latLng = LatLng(
            it.RETMAP[0].NLATIT!!.toDouble(),
            it.RETMAP[0].NLOGIT!!.toDouble()
        )
//        Log.e(TAG, "moveMap: ${latLng}")
//        Log.e(TAG, "${it.RETMAP[0]}")
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

    override fun onMarkerClick(p0: Marker?): Boolean {
        return false
    }

}

```

activity_station_map.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.coordinatorlayout.widget.CoordinatorLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".map.MapActivity">
    <androidx.fragment.app.FragmentContainerView
        android:name="com.google.android.gms.maps.SupportMapFragment"
        android:id="@+id/map"
        android:layout_width="match_parent"
        android:layout_height="match_parent"/>

    <include layout="@layout/map_bottom_sheet"
        />


</androidx.coordinatorlayout.widget.CoordinatorLayout>
```
map_bottom_sheet.xml
```
<?xml version="1.0" encoding="utf-8"?>
<androidx.core.widget.NestedScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/bottom_sheet_parent"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@drawable/corner"
    app:layout_behavior="com.google.android.material.bottomsheet.BottomSheetBehavior">

    <androidx.constraintlayout.widget.ConstraintLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content">

        ...

        </LinearLayout>
    </androidx.constraintlayout.widget.ConstraintLayout>
</androidx.core.widget.NestedScrollView>
```