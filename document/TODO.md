# TODO list and daily schedule

**- 일일 진행 -**
**0810**
현재 위치를 기반으로 한 지도 띄우기 성공

**0811**
지도 클릭하면 목적지 설정 후 네비게이션 기능 가능

**0813**
혼자 Simulation하는 것 수정

**0818**
레이아웃 수정 및 내 위치 추가됨

**0819**
학교 위치로 이동하는 버튼 추가 완료
자동 완성 기능 추가했으나 동작 X

**0820**
자동 완성 기능 동작 및 레이아웃 깔끔해짐
검색 기능 완료 - 검색된 결과가 목적지로 설정되서 길 안내 시작 가능

**0821**
getRoute_walking, getRoute_navi_walking 함수를 통해 도보로 예상 시간 및 거리 출력 가능
지도 수정 필요

**0824**
학교 지도 tileset 연동된 style 띄우기 성공

**0826**
rastersource와 vectorsource의 차이점이 대체 뭐냐????

**0928**
ARbutton 추가 및 프로젝트와 연동(아직 기능 구현 X)

**1005**
Unity와 연동 재시도했으나 Unable to start activity ComponentInfo 오류 계속 발생
string.xml 파일에 `<string name="game_view_content_description">Game view</string>` 추가

**1006**
AR 버튼은 작동 but, 카메라 X

**1010**
카메라 된다!
`build.gradle`에서 `ndk` 수정
`libs`폴더에 `arcore_client.aar`, `google_ar_required.aar`, `unityandroidpermission.aar`, `unitygar.aar` 옮김
이후 `implementation`, `manifest` 수정