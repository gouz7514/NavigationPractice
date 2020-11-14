# NavigationPractice
## 프로젝트명
Mapbox SDK를 활용한 네비게이션 어플리케이션(Navigation Application by using Mapbox SDK)<br>

## 프로젝트 소개
2D 지도를 기반으로 한 네비게이션 및 AR을 통한 길 안내 어플리케이션<br>
![어플 사용 예시](https://user-images.githubusercontent.com/41367134/99142609-fbb92a00-2699-11eb-8ca6-cec562a0bf4e.jpg)

## 프로젝트 내용
### 개요
단국대학교 응용컴퓨터공학과 4학년 종합설계 프로젝트([단국대 학생들을 위한 종합 어플리케이션 repository](https://github.com/TwinkleRing/Capstone-Project))의 일부로서, `Mapbox`를 활용해 지도 및 네비게이션을 나타내고
네비게이션 상의 경로를 AR로 표시함으로서 사용자의 원활한 길 찾기를 가능하게 하는데 그 목적이 있음<br>

### 기술 스택 및 개발 환경
- java
- [mapbox Maps SDK for android v9.3.0](https://docs.mapbox.com/android/maps/overview/)
- [mapbox Navigation SDK](https://docs.mapbox.com/android/navigation/overview/)
- [mapbox Unity SDK v2.1.1](https://docs.mapbox.com/unity/maps/overview/)
- [Google Places API](https://developers.google.com/places/web-service/overview)
- Android Studio v4.1.0, Gradle ver 6.5

### 과정
#### 1. 지도
mapbox android SDK의 사용을 위해 해당 모듈을 `New Module`로 추가

지도를 포함할 액티비티의 xml 파일에 `MapView`를 추가<br>
**activity_main.xml**
```xml
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
```java
// LocationEngine 변수
private LocationEngine locationEngine;
...
@SuppressLint("MissingPermission")
    private void initLocationEngine() {
        locationEngine = LocationEngineProvider.getBestLocationEngine(this);
        LocationEngineRequest request = new LocationEngineRequest.Builder(DEFAULT_INTERVAL_IN_MILLISECONDS)
                .setPriority(LocationEngineRequest.PRIORITY_HIGH_ACCURACY)
                .setMaxWaitTime(DEFAULT_MAX_WAIT_TIME).build();
        locationEngine.requestLocationUpdates(request, callback, getMainLooper());
        locationEngine.getLastLocation(callback);
    }
```

현재 위치 얻어오는 콜백 실행
```java
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
            MainActivity activity = activityWeakReference.get();
            if (activity != null) {
                Toast.makeText(activity, exception.getLocalizedMessage(),
                        Toast.LENGTH_SHORT).show();
            }
        }
    }
```

#### 2. 2D 네비게이션
**2.1 지도 상 클릭시 목적지로 설정 후 경로 생성**
```java
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
```java
private void getRoute_walking(Point origin, Point destination) {
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
#### 3. 경로 추가
네이버, 구글의 길 안내 API를 사용하지 않은 이유는 각각 다음과 같다.
- 네이버 : 정확하지만 큰 도로 우선 안내 -> 샛길, 보행자 전용 도로 안내 X
- 구글 : 국내 도보 경로 제공 X

Mapbox를 사용하면 지도의 자유로운 styling, 자유로운 기능 구현 등이 가능하지만<br>
Mapbox도 원하는 경로를 나타내지 못하는 문제점이 있었다.<br>
**사용자는 실제로 빨간 경로를 보행 가능하나 경로(파란 선)는 그렇지 못한 모습**
![경로 문제 발생](https://user-images.githubusercontent.com/41367134/99142797-d0cfd580-269b-11eb-83b3-323b453539d9.PNG)<br>
이를 해결하기 위해 Mapbox측에 메일을 보냈고 Mapbox는 오픈소스 지도 프로젝트인 [OpenStreetMap](www.openstreetmap.org)을 기반으로 지도를 생성한다는 답장을 받을 수 있었다.<br>
결과적으로, 해당 사이트와 [JOSM](https://josm.openstreetmap.de/)을 이용해 원하는 지역에 원하는 경로를 추가해 지도를 생성할 수 있었다.<br>
[내가 기여한 OpenStreetMap의 변경 내역](https://www.openstreetmap.org/changeset/91498317#map=18/37.32117/127.12767)

#### 4. 장소 자동완성
학교나 내 주변 외에 목적지를 설정할 수 있도록 `Google Places API`를 사용해 장소 검색 기능을 구현함
```java
Places.initialize(getApplicationContext(), "AIzaSyCVXwfS2pdm-KGbqvXc30RB8jGGJZ58mtc");
        // Create a new Places client instance.
        PlacesClient placesClient = Places.createClient(this);
        // Initialize the AutocompleteSupportFragment.
        AutocompleteSupportFragment autocompleteFragment = (AutocompleteSupportFragment)
                getSupportFragmentManager().findFragmentById(R.id.autocomplete_fragment);
        // Specify the types of place data to return.
        autocompleteFragment.setPlaceFields(Arrays.asList(Place.Field.ID, Place.Field.NAME));
        // Set up a PlaceSelectionListener to handle the response.
        autocompleteFragment.setOnPlaceSelectedListener(new PlaceSelectionListener() {
            @Override
            public void onPlaceSelected(Place place) {
                txtView.setText(String.valueOf(place.getName())); // edittext 부분에 목적지 설정됨
            }

            @Override
            public void onError(@NonNull Status status) {
                Log.i(TAG, "An error occurred: " + status);
            }
        });
```
![장소 자동완성](https://user-images.githubusercontent.com/41367134/99143283-9a945500-269f-11eb-8865-68f3e5114a0b.jpg)

#### 5. AR 네비게이션
실제 세계와 지도를 연동해 나타내기 위해 Mapbox의 [World-scale AR](https://docs.mapbox.com/unity/maps/examples/world-scale-ar/)을 사용하였다.
이 후 `Directions.prefab`을 사용해 Direction API의 결과를 바탕으로 출발지와 목적지 정보를 가져오며<br>
결과적으로 출발지(초록색 마커) ~ 목적지(하얀색 마커) 까지 경로(빨간 경로)가 나타는 것을 확인할 수 있다.
![Unity 상에 나타나는 경로](https://user-images.githubusercontent.com/41367134/99142984-56a05080-269d-11eb-88e2-ea6e7527ca31.PNG)

### 결과
- 프로젝트 설계 당시 **설정했던 목표를 달성**할 수 있었다.
- 경로가 나타나지 않는 문제점을 **기업과 메일을 주고 받으며 해결**할 수 있었다.
- AR의 경우 그 정확도가 낮다.
- 정확도 해결을 위해 메일을 보내봤으나 뚜렷한 해결책은 찾지 못하였다. [참고할 만한 자료 : World-scale AR manual alignment](https://docs.mapbox.com/unity/maps/examples/world-scale-manual-align-ar/)

### 참고 자료
- [Mapbox Tutorials](https://docs.mapbox.com/help/tutorials/)
- Mapbox와 주고받은 모든 메일

