# TODO list and daily schedule

**- 일일 진행 -**<br>
**0810**<br>
현재 위치를 기반으로 한 지도 띄우기 성공<br>

**0811**<br>
지도 클릭하면 목적지 설정 후 네비게이션 기능 가능<br>

**0813**<br>
혼자 Simulation하는 것 수정<br>

**0818**<br>
레이아웃 수정 및 내 위치 추가됨<br>

**0819**<br>
학교 위치로 이동하는 버튼 추가 완료<br>
자동 완성 기능 추가했으나 동작 X<br>

**0820**<br>
자동 완성 기능 동작 및 레이아웃 깔끔해짐<br>
검색 기능 완료 - 검색된 결과가 목적지로 설정되서 길 안내 시작 가능<br>

**0821**<br>
getRoute_walking, getRoute_navi_walking 함수를 통해 도보로 예상 시간 및 거리 출력 가능<br>
지도 수정 필요<br>

**0824**<br>
학교 지도 tileset 연동된 style 띄우기 성공<br>

**0826**<br>
rastersource와 vectorsource의 차이점이 대체 뭐냐????<br>

**0928**<br>
ARbutton 추가 및 프로젝트와 연동(아직 기능 구현 X)<br>

**1005**<br>
Unity와 연동 재시도했으나 Unable to start activity ComponentInfo 오류 계속 발생<br>
string.xml 파일에 `<string name="game_view_content_description">Game view</string>` 추가<br>

**1006**<br>
AR 버튼은 작동 but, 카메라 X<br>

**1010**<br>
카메라 된다!<br>
`build.gradle`에서 `ndk` 수정<br>
`libs`폴더에 `arcore_client.aar`, `google_ar_required.aar`, `unityandroidpermission.aar`, `unitygar.aar` 옮김<br>
이후 `implementation`, `manifest` 수정<br>