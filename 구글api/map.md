2020-05-16
===
```
package com.hyundaioilbank.android.map

//import android.location.Geocoder
import android.Manifest
import android.app.Activity
import android.content.Context
import android.content.Intent
import android.content.IntentSender
import android.content.pm.PackageManager
import android.graphics.Bitmap
import android.graphics.Canvas
import android.graphics.Color
import android.location.Address
import android.location.Geocoder
import android.location.Location
import android.os.Bundle
import android.util.DisplayMetrics
import android.util.Log
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.FrameLayout
import android.widget.TextView
import androidx.core.app.ActivityCompat
import com.google.android.gms.common.api.ResolvableApiException
import com.google.android.gms.location.*
import com.google.android.gms.maps.CameraUpdateFactory
import com.google.android.gms.maps.GoogleMap
import com.google.android.gms.maps.OnMapReadyCallback
import com.google.android.gms.maps.SupportMapFragment
import com.google.android.gms.maps.model.*
import com.google.android.gms.tasks.OnSuccessListener
import com.google.android.material.bottomsheet.BottomSheetBehavior
import com.hyundaioilbank.android.BaseActivityNoViewModel
import com.hyundaioilbank.android.R
import com.hyundaioilbank.android.StationDetail018
import com.hyundaioilbank.android.databinding.ActivityMapBinding
import com.hyundaioilbank.android.findstation.FindStationActivity
import com.hyundaioilbank.android.retrofit.ApiService
import com.hyundaioilbank.android.retrofit.ApiService.Companion.retrofit
import com.hyundaioilbank.android.retrofit.ApiService.Companion.retrofitCallback
import kotlinx.android.synthetic.main.map_bottom_sheet.*
import java.io.IOException
import java.lang.Math.pow
import kotlin.math.pow
import kotlin.math.sqrt

//https://gun0912.tistory.com/57  클릭시 색 변동 및 레이아웃 관련
class MapActivity : BaseActivityNoViewModel<ActivityMapBinding>(), OnMapReadyCallback,
    GoogleMap.OnMarkerClickListener, GoogleMap.OnMapClickListener {

    val code: String by lazy {
=======
MapActivity
```

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
        /**
         * @see [] https://www.raywenderlich.com/230-introduction-to-google-maps-api-for-android-with-kotlin#toc-anchor-005'
         */
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