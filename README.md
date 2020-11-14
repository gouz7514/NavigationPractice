# NavigationPractice
### 프로젝트명
Mapbox SDK를 활용한 네비게이션 어플리케이션(Navigation Application by using Mapbox SDK)<br>

### 프로젝트 소개
2D 지도를 기반으로 한 네비게이션 및 AR을 통한 길 안내 어플리케이션<br>
![어플 사용 예시](https://user-images.githubusercontent.com/41367134/99141899-14721180-2693-11eb-8f87-4106ab6fb723.jpg)

### 프로젝트 내용
#### 개요
단국대학교 응용컴퓨터공학과 4학년 종합설계 프로젝트([단국대 학생들을 위한 종합 어플리케이션 repository](https://github.com/TwinkleRing/Capstone-Project))의 일부로서, `Mapbox`를 활용해 지도 및 네비게이션을 나타내고
네비게이션 상의 경로를 AR로 표시함으로서 사용자의 원활한 길 찾기를 가능하게 하는데 그 목적이 있음<br>

#### 기술 스택 및 개발 환경
- java
- [mapbox Maps SDK for android v9.3.0](https://docs.mapbox.com/android/maps/overview/)
- [mapbox Navigation SDK](https://docs.mapbox.com/android/navigation/overview/)
- [mapbox Unity SDK v2.1.1](https://docs.mapbox.com/unity/maps/overview/)
- [Google Places API](https://developers.google.com/places/web-service/overview)
- Android Studio v4.1.0
- Gradle ver 6.5

#### 과정
###### 1. 지도
`mapbox android SDK`의 사용을 위해 해당 모듈을 `New Module`로 추가

지도를 포함할 액티비티의 xml 파일에 `MapView`를 추가<br>
`activity_main.xml`
```
<com.mapbox.mapboxsdk.maps.MapView
        android:id="@+id/mapView"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintHorizontal_bias="0.0"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:layout_constraintVertical_bias="1.0"
        mapbox:mapbox_cameraTargetLat="37.321643"
        mapbox:mapbox_cameraTargetLng="127.126756"
        mapbox:mapbox_cameraZoom="16">
    </com.mapbox.mapboxsdk.maps.MapView>
```

위치 업데이트를 위한 `LocationEngine` 개체의 사용
```
// LocationEngine 변수
private LocationEngine locationEngine;
...
@SuppressLint("MissingPermission")
    private void initLocationEngine() {
        Log.e(TAG,"initLocationEngine 실행");
        locationEngine = LocationEngineProvider.getBestLocationEngine(this);
        LocationEngineRequest request = new LocationEngineRequest.Builder(DEFAULT_INTERVAL_IN_MILLISECONDS)
                .setPriority(LocationEngineRequest.PRIORITY_HIGH_ACCURACY)
                .setMaxWaitTime(DEFAULT_MAX_WAIT_TIME).build();
        locationEngine.requestLocationUpdates(request, callback, getMainLooper());
        locationEngine.getLastLocation(callback);
    }
```

현재 위치 얻어오는 콜백 실행
```
class MainActivityLocationCallback implements LocationEngineCallback<LocationEngineResult> {
        private final WeakReference<MainActivity> activityWeakReference;
        MainActivityLocationCallback(MainActivity activity) {
            this.activityWeakReference = new WeakReference<>(activity);
        }
        /**
         * The LocationEngineCallback interface's method which fires when the device's location has changed.
         * @param result the LocationEngineResult object which has the last known location within it.
         */

        @Override
        public void onSuccess(LocationEngineResult result) {
            Log.e(TAG,"MainActivityLocationCallback onSuccess 실행");
            MainActivity activity = activityWeakReference.get();
            if (activity != null) {
                Location location = result.getLastLocation();
                if (location == null) {
                    return;
                }
                // Create a Toast which displays the new location's coordinates
                La = result.getLastLocation().getLatitude();
                Lo = result.getLastLocation().getLongitude();

                // Pass the new location to the Maps SDK's LocationComponent
                if (activity.mapboxMap != null && result.getLastLocation() != null) {
                    activity.mapboxMap.getLocationComponent().forceLocationUpdate(result.getLastLocation());
                }
            }
        }

        /**
         * The LocationEngineCallback interface's method which fires when the device's location can not be captured
         *
         * @param exception the exception message
         */
        @Override
        public void onFailure(@NonNull Exception exception) {
            Log.e("LocationChangeActivity", exception.getLocalizedMessage());
            MainActivity activity = activityWeakReference.get();
            if (activity != null) {
                Toast.makeText(activity, exception.getLocalizedMessage(),
                        Toast.LENGTH_SHORT).show();
            }
        }
    }
```

###### 2. 2D 네비게이션
**2.1 지도 상 클릭시 목적지로 설정 후 경로 생성**
```
@Override
    public boolean onMapClick(@NonNull LatLng point) {
        if (destinationMarker != null) {
            mapboxMap.removeMarker(destinationMarker);
        }
        destinationMarker = mapboxMap.addMarker(new MarkerOptions().position(point));
        destinatonPosition = Point.fromLngLat(point.getLongitude(), point.getLatitude());
        originPosition = Point.fromLngLat(Lo, La);
        getRoute_walking(originPosition, destinatonPosition);
        getRoute_navi_walking(originPosition, destinatonPosition);
        startButton.setEnabled(true);
        startButton.setBackgroundResource(R.color.mapboxBlue);
        arButton.setEnabled(true);
        arButton.setBackgroundResource(R.color.mapboxBlue);
        return false;
    }
```
**2.2 길찾기 메소드**
```
private void getRoute_walking(Point origin, Point destination) {
        Log.e(TAG,"getRoute 실행");
        client = MapboxDirections.builder()
                .origin(origin)
                .destination(destination)
                .overview(DirectionsCriteria.OVERVIEW_FULL)
                .profile(DirectionsCriteria.PROFILE_WALKING) //길찾기 방법(본 프로젝트에서는 도보로 설정)
                .accessToken(getString(R.string.access_token))
                .build();

        client.enqueueCall(new Callback<DirectionsResponse>() {
            @Override
            public void onResponse(Call<DirectionsResponse> call, Response<DirectionsResponse> response) {
                if (response.body() == null) {
                    return;
                } else if (response.body().routes().size() < 1) {
                    return;
                }
                // Print some info about the route
                currentRoute = response.body().routes().get(0);
                Log.e(TAG, "Distance: " + currentRoute.distance());

                int time = (int) (currentRoute.duration()/60);
                double distances = (currentRoute.distance()/1000);

                distances = Math.round(distances*100)/100.0;

                Toast.makeText(getApplicationContext(), String.format("예상 시간 : " + String.valueOf(time)+" 분 \n" +
                        "목적지 거리 : " +distances+ " km"), Toast.LENGTH_LONG).show();
            }
            @Override
            public void onFailure(Call<DirectionsResponse> call, Throwable throwable) {
                Toast.makeText(MainActivity.this, "Error: " + throwable.getMessage(), Toast.LENGTH_SHORT).show();
            }
        });
    }

    private void getRoute_navi_walking (Point origin, Point destinaton) {
        // TODO : https://docs.mapbox.com/android/navigation/overview/map-matching/
        NavigationRoute.builder(this).accessToken(Mapbox.getAccessToken())
                .profile(DirectionsCriteria.PROFILE_WALKING)
                .origin(origin)
                .destination(destinaton).
                build().
                getRoute(new Callback<DirectionsResponse>() {
                    @Override
                    public void onResponse(Call<DirectionsResponse> call, Response<DirectionsResponse> response) {
                        if (response.body() == null) {
                            return;
                        } else if (response.body().routes().size() ==0) {
                            return;
                        }
                        currentRoute = response.body().routes().get(0);
                        if (navigationMapRoute != null) {
                            navigationMapRoute.removeRoute();
                        } else {
                            navigationMapRoute = new NavigationMapRoute(null, mapView, mapboxMap, R.style.NavigationMapRoute);
                        }
                        navigationMapRoute.addRoute(currentRoute);
                    }
                    @Override
                    public void onFailure(Call<DirectionsResponse> call, Throwable t) {
                    }
                });
    }
```
###### 3. 경로 추가


###### 4. 장소 자동완성

###### 5. AR 네비게이션