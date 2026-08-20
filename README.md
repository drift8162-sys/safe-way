apply plugin: 'com.android.application'

android {
    compileSdkVersion 33

    defaultConfig {
        applicationId "com.example.safewaymini"
        minSdkVersion 21
        targetSdkVersion 33
        versionCode 1
        versionName "1.0"

        testInstrumentationRunner "androidx.test.runner.AndroidJUnitRunner"
    }

    buildTypes {
        release {
            minifyEnabled false
            proguardFiles getDefaultProguardFile('proguard-android-optimize.txt'), 'proguard-rules.pro'
        }
    }

    compileOptions {
        sourceCompatibility = 1.8
        targetCompatibility = 1.8
    }

    buildFeatures {
        viewBinding false
    }
}

dependencies {
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.9.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    implementation 'com.google.android.gms:play-services-maps:18.1.0'
    implementation 'com.google.android.gms:play-services-location:21.0.1'
    implementation 'com.google.code.gson:gson:2.10.1'
    implementation 'androidx.core:core-ktx:1.10.1'

    testImplementation 'junit:junit:4.13.2'
    androidTestImplementation 'androidx.test.ext:junit:1.1.5'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.5.1'
}
<?xml version="1.0" encoding="utf-8"?>
<manifest package="com.example.safewaymini"
    xmlns:android="http://schemas.android.com/apk/res/android">

    <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
    <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />

    <application
        android:allowBackup="true"
        android:label="SafeWayMini"
        android:icon="@mipmap/ic_launcher"
        android:roundIcon="@mipmap/ic_launcher_round"
        android:supportsRtl="true"
        android:theme="@style/Theme.AppCompat.Light.NoActionBar">

        <activity android:name=".MainActivity"
            android:exported="true">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
        </activity>

        <meta-data
            android:name="com.google.android.geo.API_KEY"
            android:value="YOUR_GOOGLE_MAPS_API_KEY" />

    </application>
</manifest>
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:id="@+id/rootLayout"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <fragment
        android:id="@+id/mapFragment"
        android:name="com.google.android.gms.maps.SupportMapFragment"
        android:layout_width="0dp"
        android:layout_height="0dp"
        android:layout_marginTop="0dp"
        android:layout_marginBottom="16dp"
        android:tag="map"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintBottom_toTopOf="@+id/buttonPanel"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent" />

    <LinearLayout
        android:id="@+id/buttonPanel"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="12dp"
        android:gravity="center"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintEnd_toEndOf="parent">

        <Button
            android:id="@+id/btnAddDark"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="🌙 Add Dark Area" />

        <Button
            android:id="@+id/btnAddDog"
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:layout_weight="1"
            android:text="🐕 Add Dog Area" />

        <Button
            android:id="@+id/btnClearAll"
            android:layout_width="wrap_content"
            android:layout_height="wrap_content"
            android:layout_marginStart="8dp"
            android:text="🗑 Clear All" />

    </LinearLayout>

</androidx.constraintlayout.widget.ConstraintLayout>
package com.example.safewaymini;

import java.util.UUID;

/**
 * POJO for storing marker information.
 */
public class MarkerData {
    public String id;
    public String type; // "dark" or "dog"
    public double latitude;
    public double longitude;
    public long timestampMs;

    // Default constructor for Gson
    public MarkerData() { }

    public MarkerData(String type, double latitude, double longitude, long timestampMs) {
        this.id = UUID.randomUUID().toString();
        this.type = type;
        this.latitude = latitude;
        this.longitude = longitude;
        this.timestampMs = timestampMs;
    }
}
package com.example.safewaymini;

import android.content.Context;
import android.content.SharedPreferences;

import com.google.gson.Gson;
import com.google.gson.reflect.TypeToken;

import java.lang.reflect.Type;
import java.util.ArrayList;

/**
 * Helper to persist and retrieve marker list as JSON in SharedPreferences.
 */
public class StorageHelper {
    private static final String PREF_NAME = "safeway_pref";
    private static final String KEY_MARKERS = "markers_json";

    private SharedPreferences prefs;
    private Gson gson;

    public StorageHelper(Context ctx) {
        prefs = ctx.getSharedPreferences(PREF_NAME, Context.MODE_PRIVATE);
        gson = new Gson();
    }

    public ArrayList<MarkerData> loadMarkers() {
        String json = prefs.getString(KEY_MARKERS, null);
        if (json == null) return new ArrayList<>();
        Type listType = new TypeToken<ArrayList<MarkerData>>() {}.getType();
        try {
            ArrayList<MarkerData> list = gson.fromJson(json, listType);
            return list != null ? list : new ArrayList<MarkerData>();
        } catch (Exception e) {
            return new ArrayList<>();
        }
    }

    public void saveMarkers(ArrayList<MarkerData> markers) {
        String json = gson.toJson(markers);
        prefs.edit().putString(KEY_MARKERS, json).apply();
    }

    public void clearMarkers() {
        prefs.edit().remove(KEY_MARKERS).apply();
    }
}
[19/11/2025 07:15] Компьютер: package com.example.safewaymini;

import android.Manifest;
import android.content.DialogInterface;
import android.content.pm.PackageManager;
import android.location.Location;
import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;

import com.google.android.gms.location.FusedLocationProviderClient;
import com.google.android.gms.location.LocationCallback;
import com.google.android.gms.location.LocationRequest;
import com.google.android.gms.location.LocationResult;
import com.google.android.gms.location.LocationServices;

import com.google.android.gms.maps.CameraUpdateFactory;
import com.google.android.gms.maps.GoogleMap;
import com.google.android.gms.maps.OnMapReadyCallback;
import com.google.android.gms.maps.SupportMapFragment;

import com.google.android.gms.maps.model.BitmapDescriptorFactory;
import com.google.android.gms.maps.model.LatLng;
import com.google.android.gms.maps.model.MarkerOptions;

import java.util.ArrayList;

/**
 * MainActivity replicates MIT App Inventor components: LocationSensor, Map, TinyDB, Notifier.
 *
 * Minimal comments; explains why for non-obvious choices only.
 */
public class MainActivity extends AppCompatActivity implements OnMapReadyCallback {

    private static final float DEFAULT_ZOOM = 16f;

    private GoogleMap mMap;
    private FusedLocationProviderClient fusedLocationClient;
    private LocationCallback locationCallback;

    private Button btnAddDark;
    private Button btnAddDog;
    private Button btnClearAll;

    private StorageHelper storageHelper;
    private ArrayList<MarkerData> markerList = new ArrayList<>();

    private Location lastKnownLocation;

    private ActivityResultLauncher<String[]> requestPermissionLauncher;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize UI
        btnAddDark = findViewById(R.id.btnAddDark);
        btnAddDog = findViewById(R.id.btnAddDog);
        btnClearAll = findViewById(R.id.btnClearAll);

        storageHelper = new StorageHelper(this);
        markerList = storageHelper.loadMarkers();

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
[19/11/2025 07:15] Компьютер: // Permission launcher for both fine & coarse
        requestPermissionLauncher = registerForActivityResult(
                new ActivityResultContracts.RequestMultiplePermissions(),
                permissions -> {
                    boolean fineGranted = permissions.getOrDefault(Manifest.permission.ACCESS_FINE_LOCATION, false);
                    boolean coarseGranted = permissions.getOrDefault(Manifest.permission.ACCESS_COARSE_LOCATION, false);
                    if (fineGranted  coarseGranted) {
                        enableLocationFeatures();
                    } else {
                        disableLocationFeaturesDueToPermission();
                    }
                });

        // Map setup
        SupportMapFragment mapFragment = (SupportMapFragment) getSupportFragmentManager()
                .findFragmentById(R.id.mapFragment);
        if (mapFragment != null) {
            mapFragment.getMapAsync(this);
        } else {
            Toast.makeText(this, "Map fragment failed to load.", Toast.LENGTH_LONG).show();
        }

        // Buttons
        btnAddDark.setOnClickListener(v -> addMarkerAtCurrentLocation("dark"));
        btnAddDog.setOnClickListener(v -> addMarkerAtCurrentLocation("dog"));
        btnClearAll.setOnClickListener(v -> confirmClearAll());

        // Location callback: center map on every valid update
        locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(@NonNull LocationResult result) {
                if (result == null) {
                    Toast.makeText(MainActivity.this, "Location unavailable.", Toast.LENGTH_SHORT).show();
                    return;
                }
                Location loc = result.getLastLocation();
                if (loc != null) {
                    lastKnownLocation = loc;
                    centerMapOnLocation(loc);
                } else {
                    Toast.makeText(MainActivity.this, "Location unavailable.", Toast.LENGTH_SHORT).show();
                }
            }
        };

        // Finally, ask permissions (or enable if already granted)
        checkAndRequestPermissions();
    }

    private void checkAndRequestPermissions() {
        boolean fine = ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED;
        boolean coarse = ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED;
        if (fine  coarse) {
            enableLocationFeatures();
        } else {
            requestPermissionLauncher.launch(new String[] {
                    Manifest.permission.ACCESS_FINE_LOCATION,
                    Manifest.permission.ACCESS_COARSE_LOCATION
            });
        }
    }

    private void enableLocationFeatures() {
        btnAddDark.setEnabled(true);
        btnAddDog.setEnabled(true);
        startLocationUpdates();
        if (mMap != null) {
            try {
                mMap.setMyLocationEnabled(true); // requires permission; checked above
            } catch (SecurityException ignored) { }
        }
    }

    private void disableLocationFeaturesDueToPermission() {
        btnAddDark.setEnabled(false);
        btnAddDog.setEnabled(false);
        Toast.makeText(this, "Location permission denied — location features disabled.", Toast.LENGTH_LONG).show();
    }

    private void startLocationUpdates() {
        try {
            LocationRequest req = LocationRequest.create();
            req.setInterval(5000);
            req.setFastestInterval(2000);
            req.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
            fusedLocationClient.requestLocationUpdates(req, locationCallback, getMainLooper());
        } catch (SecurityException e) {
            // Permission might have been revoked after check
            disableLocationFeaturesDueToPermission();
        }
    }

    private void stopLocationUpdates() {
        fusedLocationClient.removeLocationUpdates(locationCallback);
    }
[19/11/2025 07:15] Компьютер: private void centerMapOnLocation(Location loc) {
        if (mMap == null  loc == null) return;
        LatLng latLng = new LatLng(loc.getLatitude(), loc.getLongitude());
        mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(latLng, DEFAULT_ZOOM));
    }

    private void addMarkerAtCurrentLocation(String type) {
        if (lastKnownLocation == null) {
            Toast.makeText(this, "Current location unavailable — cannot add marker.", Toast.LENGTH_SHORT).show();
            return;
        }
        double lat = lastKnownLocation.getLatitude();
        double lng = lastKnownLocation.getLongitude();
        long ts = System.currentTimeMillis();

        MarkerData md = new MarkerData(type, lat, lng, ts);
        markerList.add(md);
        storageHelper.saveMarkers(markerList);

        addMarkerToMap(md);
        Toast.makeText(this, type.equals("dark") ? "Added 🌙 Dark area" : "Added 🐕 Dog hazard", Toast.LENGTH_SHORT).show();
    }

    private void addMarkerToMap(MarkerData md) {
        if (mMap == null) return;
        LatLng pos = new LatLng(md.latitude, md.longitude);
        MarkerOptions opts = new MarkerOptions()
                .position(pos)
                .title(md.type.equals("dark") ? "🌙 Dark area" : "🐕 Dog hazard");

        if ("dark".equals(md.type)) {
            opts.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_AZURE));
        } else {
            opts.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_ORANGE));
        }
        mMap.addMarker(opts);
    }

    private void restoreMarkersOnMap() {
        if (mMap == null) return;
        mMap.clear();
        for (MarkerData md : markerList) {
            addMarkerToMap(md);
        }
    }

    private void confirmClearAll() {
        new AlertDialog.Builder(this)
                .setTitle("Confirm")
                .setMessage("Remove all markers and clear storage?")
                .setPositiveButton("Yes", (dialog, which) -> {
                    markerList.clear();
                    storageHelper.clearMarkers();
                    if (mMap != null) mMap.clear();
                    // Try to re-enable MyLocation layer if permission available
                    if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                             ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
                        try {
                            mMap.setMyLocationEnabled(true);
                        } catch (SecurityException ignored) { }
                    }
                    Toast.makeText(this, "All markers cleared.", Toast.LENGTH_SHORT).show();
                })
                .setNegativeButton("No", (dialog, which) -> dialog.dismiss())
                .show();
    }

    @Override
    public void onMapReady(@NonNull GoogleMap googleMap) {
        mMap = googleMap;
        // Restore saved markers (map might be ready after onCreate)
        restoreMarkersOnMap();
[19/11/2025 07:15] Компьютер: // Try enabling MyLocation layer if permission granted
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                 ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
            try {
                mMap.setMyLocationEnabled(true);
            } catch (SecurityException ignored) { }
        } else {
            disableLocationFeaturesDueToPermission();
        }

        // If we already have a last location, center map
        if (lastKnownLocation != null) {
            centerMapOnLocation(lastKnownLocation);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        // restart location updates if permission exists
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                 ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
            startLocationUpdates();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        stopLocationUpdates();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopLocationUpdates();
    }
}
[19/11/2025 07:16] Компьютер: // File: app/src/main/java/com/example/safewaymini/MainActivity.java
[19/11/2025 07:16] Компьютер: package com.example.safewaymini;

import android.Manifest;
import android.content.DialogInterface;
import android.content.pm.PackageManager;
import android.location.Location;
import android.os.Bundle;
import android.widget.Button;
import android.widget.Toast;

import androidx.activity.result.ActivityResultLauncher;
import androidx.activity.result.contract.ActivityResultContracts;
import androidx.annotation.NonNull;
import androidx.appcompat.app.AlertDialog;
import androidx.appcompat.app.AppCompatActivity;
import androidx.core.content.ContextCompat;

import com.google.android.gms.location.FusedLocationProviderClient;
import com.google.android.gms.location.LocationCallback;
import com.google.android.gms.location.LocationRequest;
import com.google.android.gms.location.LocationResult;
import com.google.android.gms.location.LocationServices;

import com.google.android.gms.maps.CameraUpdateFactory;
import com.google.android.gms.maps.GoogleMap;
import com.google.android.gms.maps.OnMapReadyCallback;
import com.google.android.gms.maps.SupportMapFragment;

import com.google.android.gms.maps.model.BitmapDescriptorFactory;
import com.google.android.gms.maps.model.LatLng;
import com.google.android.gms.maps.model.MarkerOptions;

import java.util.ArrayList;

/**
 * MainActivity replicates MIT App Inventor components: LocationSensor, Map, TinyDB, Notifier.
 *
 * Minimal comments; explains why for non-obvious choices only.
 */
public class MainActivity extends AppCompatActivity implements OnMapReadyCallback {

    private static final float DEFAULT_ZOOM = 16f;

    private GoogleMap mMap;
    private FusedLocationProviderClient fusedLocationClient;
    private LocationCallback locationCallback;

    private Button btnAddDark;
    private Button btnAddDog;
    private Button btnClearAll;

    private StorageHelper storageHelper;
    private ArrayList<MarkerData> markerList = new ArrayList<>();

    private Location lastKnownLocation;

    private ActivityResultLauncher<String[]> requestPermissionLauncher;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        // Initialize UI
        btnAddDark = findViewById(R.id.btnAddDark);
        btnAddDog = findViewById(R.id.btnAddDog);
        btnClearAll = findViewById(R.id.btnClearAll);

        storageHelper = new StorageHelper(this);
        markerList = storageHelper.loadMarkers();

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this);
[19/11/2025 07:16] Компьютер: // Permission launcher for both fine & coarse
        requestPermissionLauncher = registerForActivityResult(
                new ActivityResultContracts.RequestMultiplePermissions(),
                permissions -> {
                    boolean fineGranted = permissions.getOrDefault(Manifest.permission.ACCESS_FINE_LOCATION, false);
                    boolean coarseGranted = permissions.getOrDefault(Manifest.permission.ACCESS_COARSE_LOCATION, false);
                    if (fineGranted  coarseGranted) {
                        enableLocationFeatures();
                    } else {
                        disableLocationFeaturesDueToPermission();
                    }
                });

        // Map setup
        SupportMapFragment mapFragment = (SupportMapFragment) getSupportFragmentManager()
                .findFragmentById(R.id.mapFragment);
        if (mapFragment != null) {
            mapFragment.getMapAsync(this);
        } else {
            Toast.makeText(this, "Map fragment failed to load.", Toast.LENGTH_LONG).show();
        }

        // Buttons
        btnAddDark.setOnClickListener(v -> addMarkerAtCurrentLocation("dark"));
        btnAddDog.setOnClickListener(v -> addMarkerAtCurrentLocation("dog"));
        btnClearAll.setOnClickListener(v -> confirmClearAll());

        // Location callback: center map on every valid update
        locationCallback = new LocationCallback() {
            @Override
            public void onLocationResult(@NonNull LocationResult result) {
                if (result == null) {
                    Toast.makeText(MainActivity.this, "Location unavailable.", Toast.LENGTH_SHORT).show();
                    return;
                }
                Location loc = result.getLastLocation();
                if (loc != null) {
                    lastKnownLocation = loc;
                    centerMapOnLocation(loc);
                } else {
                    Toast.makeText(MainActivity.this, "Location unavailable.", Toast.LENGTH_SHORT).show();
                }
            }
        };

        // Finally, ask permissions (or enable if already granted)
        checkAndRequestPermissions();
    }

    private void checkAndRequestPermissions() {
        boolean fine = ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED;
        boolean coarse = ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED;
        if (fine  coarse) {
            enableLocationFeatures();
        } else {
            requestPermissionLauncher.launch(new String[] {
                    Manifest.permission.ACCESS_FINE_LOCATION,
                    Manifest.permission.ACCESS_COARSE_LOCATION
            });
        }
    }

    private void enableLocationFeatures() {
        btnAddDark.setEnabled(true);
        btnAddDog.setEnabled(true);
        startLocationUpdates();
        if (mMap != null) {
            try {
                mMap.setMyLocationEnabled(true); // requires permission; checked above
            } catch (SecurityException ignored) { }
        }
    }

    private void disableLocationFeaturesDueToPermission() {
        btnAddDark.setEnabled(false);
        btnAddDog.setEnabled(false);
        Toast.makeText(this, "Location permission denied — location features disabled.", Toast.LENGTH_LONG).show();
    }

    private void startLocationUpdates() {
        try {
            LocationRequest req = LocationRequest.create();
            req.setInterval(5000);
            req.setFastestInterval(2000);
            req.setPriority(LocationRequest.PRIORITY_HIGH_ACCURACY);
            fusedLocationClient.requestLocationUpdates(req, locationCallback, getMainLooper());
        } catch (SecurityException e) {
            // Permission might have been revoked after check
            disableLocationFeaturesDueToPermission();
        }
    }

    private void stopLocationUpdates() {
        fusedLocationClient.removeLocationUpdates(locationCallback);
    }
[19/11/2025 07:16] Компьютер: private void centerMapOnLocation(Location loc) {
        if (mMap == null  loc == null) return;
        LatLng latLng = new LatLng(loc.getLatitude(), loc.getLongitude());
        mMap.animateCamera(CameraUpdateFactory.newLatLngZoom(latLng, DEFAULT_ZOOM));
    }

    private void addMarkerAtCurrentLocation(String type) {
        if (lastKnownLocation == null) {
            Toast.makeText(this, "Current location unavailable — cannot add marker.", Toast.LENGTH_SHORT).show();
            return;
        }
        double lat = lastKnownLocation.getLatitude();
        double lng = lastKnownLocation.getLongitude();
        long ts = System.currentTimeMillis();

        MarkerData md = new MarkerData(type, lat, lng, ts);
        markerList.add(md);
        storageHelper.saveMarkers(markerList);

        addMarkerToMap(md);
        Toast.makeText(this, type.equals("dark") ? "Added 🌙 Dark area" : "Added 🐕 Dog hazard", Toast.LENGTH_SHORT).show();
    }

    private void addMarkerToMap(MarkerData md) {
        if (mMap == null) return;
        LatLng pos = new LatLng(md.latitude, md.longitude);
        MarkerOptions opts = new MarkerOptions()
                .position(pos)
                .title(md.type.equals("dark") ? "🌙 Dark area" : "🐕 Dog hazard");

        if ("dark".equals(md.type)) {
            opts.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_AZURE));
        } else {
            opts.icon(BitmapDescriptorFactory.defaultMarker(BitmapDescriptorFactory.HUE_ORANGE));
        }
        mMap.addMarker(opts);
    }

    private void restoreMarkersOnMap() {
        if (mMap == null) return;
        mMap.clear();
        for (MarkerData md : markerList) {
            addMarkerToMap(md);
        }
    }

    private void confirmClearAll() {
        new AlertDialog.Builder(this)
                .setTitle("Confirm")
                .setMessage("Remove all markers and clear storage?")
                .setPositiveButton("Yes", (dialog, which) -> {
                    markerList.clear();
                    storageHelper.clearMarkers();
                    if (mMap != null) mMap.clear();
                    // Try to re-enable MyLocation layer if permission available
                    if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                             ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
                        try {
                            mMap.setMyLocationEnabled(true);
                        } catch (SecurityException ignored) { }
                    }
                    Toast.makeText(this, "All markers cleared.", Toast.LENGTH_SHORT).show();
                })
                .setNegativeButton("No", (dialog, which) -> dialog.dismiss())
                .show();
    }

    @Override
    public void onMapReady(@NonNull GoogleMap googleMap) {
        mMap = googleMap;
        // Restore saved markers (map might be ready after onCreate)
        restoreMarkersOnMap();
[19/11/2025 07:16] Компьютер: // Try enabling MyLocation layer if permission granted
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                 ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
            try {
                mMap.setMyLocationEnabled(true);
            } catch (SecurityException ignored) { }
        } else {
            disableLocationFeaturesDueToPermission();
        }

        // If we already have a last location, center map
        if (lastKnownLocation != null) {
            centerMapOnLocation(lastKnownLocation);
        }
    }

    @Override
    protected void onResume() {
        super.onResume();
        // restart location updates if permission exists
        if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) == PackageManager.PERMISSION_GRANTED
                 ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_COARSE_LOCATION) == PackageManager.PERMISSION_GRANTED) {
            startLocationUpdates();
        }
    }

    @Override
    protected void onPause() {
        super.onPause();
        stopLocationUpdates();
    }

    @Override
    protected void onDestroy() {
        super.onDestroy();
        stopLocationUpdates();
    }
}
