uCrop (https://github.com/Yalantis/uCrop)
===========
Chosun Univ. Opensource Software Project(01), Group_03, Devsign
------------
![uCrop_main](https://github.com/Yalantis/uCrop/blob/master/preview.gif)
- - -
## 이 프로그램을 고른 이유
 세 명의 조원이 모두 이해하기 쉬운 언어를 찾아본 결과 자바, 안드로이드, cpp등이 있었고, 서로의 의견 조율 결과 자바나  
 안드로이드로 구현된 오픈소스 소프트웨어를 찾게 되었습니다.  
 
 Github 에서 Topic 과 Explore 에서 검색을 통해 uCrop 이라는 안드로이드 이미지 편집 프로그램을 발견하여 소스를  
 대략적으로 보고 회의한 결과, 주제도 흥미롭고 난이도도 적당하여 uCrop 으로 프로젝트 프로그램으로 정했습니다.
 - - -
## Program Summary
이미지를 불러와 확대, 축소, 자르기, 회전 등 이미지 편집을 한 후 편집한 이미지를 원하는 비율로 저장하는 기능을 가진   
라이브리리  
- 커스터마이징  
  사용자에게 자르기 비율을 동적으로 선택할 수 있게 하려면 `withAspectRatio((x, y)`를 호출하는게 아닌 uCrop 빌더   
  클래스가 가진 `withOptions(UCrop,Options options`를 사용하면 됨
  
  아래의 사항들을 변경할 수 있음  
  - 이미지 압축 포맷(예 : PNG, JPEG, WEBP) 압축
  - 이미지 압축 품질(0 ~ 100), 무압축 손실인 PNG는 품질 설정을 무시
  - 모든 제스처가 동시에 활성화되는지 여부
  - 소스 Uri에서 디코딩되고
  - 자르기 프레임 / 격자를 표시할지 여부 토글
  - 자르기 프레임, 행, 열들의 색, 너비, 갯수 설정
  - 직사각형 또는 타원형 자르기 영역 고르기
  - UI 색상(툴바, 상태 표시 줄, 활성 위젯 상태)
  - 기타 등등  
- 호환
  - Library - Android ICS 4.0+ (API 14) (Android GINGERBREAD 2.3+ (API 10) for versions <= 1.3.2)
  - Sample - Android ICS 4.0+ (API 14)
  - CPU - armeabi armeabi-v7a x86 x86_64 arm64-v8a (for versions >= 2.1.2)
- - -

## 결과 화면
<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/KJY/image/result.PNG?raw=true" width="30%">
- 사용자 임의대로 화면 상단 우측의 체크 표시를 터치하면 결과 화면이 나옵니다.
- 편집이 완료되었으면 다운로드를 할 수 있고, 되돌아 갈 경우 메인 화면이 나옵니다.
## 오픈 소스 SW에 대한 각 기능별 설명(폴더별)
>  각 소스별 대략적인 설명은 gitgub폴더에 따로 업로드
- callback
  - BitmapCropCallback.java :
  - BitmapLoadCallback.java : 
  - CropBoundsChangeListener.java : 
  - OverlayViewChangeListener.java : 
- model
  - AspectRatio.java : 이미지 비율에 관한 클래스
  - CropParameters.java : 이미지 편집시 사용되는 인자들을 관리하는 클래스
  - ExifInfo.java : Exif 정보에 관한 클래스
  - ImageState.java : 이미지 상태를 반환하는 클래스
- task
  - BitmapCropTask.java : 이미지를 자르는 기능을 수행하는 클래스
  - BitmapLoadTask.java : 이미지를 로드하는 작업을 수행하는 클래스
- util
  - BitmapLoadUtils.java :  비트맵 로드에 관한 클래스
  - CubicEasing.java : 애니메이션 퀄리티에 관한 클래스
  - EglUtils.java : 안드로이드 버전에 관한 클래스
  - FastBitmapDrawable.java : 비트맵을 불러오는 과정에 대한 클래스
  - FileUtils.java : 비트맵 경로에 관한 클래스
  - ImageHeaderParser.java : EXIF에 관한 클래스
  - RectUtils.java : 비트맵 좌표에 관한 클래스
  - RotationGestureDetector.java : 비트맵 제스처에 관한 클래스
  - SelectedStateListDrawable.java : 비트맵 상태에 관한 클래스
- view
  - widget
    - AspectRatioTextView.java : 어플의 색상 및 텍스트뷰에 관한 클래스
    - HorizontalProgressWheelView.java : 스크롤에 관한 클래스
  - CropImageView.java : 자르기 기능, 자르기 가이드라인 작성 및 이미지 원형 유지 방법을 추가하는 클래스
  - GestureCropImageView.java : 어플을 이용할 때 사용되는 제스쳐를 정의하는 클래스
  - OverlayView.java : 이미지 위에 오버레이를 그리는 클래스
  - TransformImageView.java : 이미지 설정 및 변환에 관한 클래스
  - UCropView.java : 제스쳐와 오버레이에 관한 뷰를 정의하는 클래스
- UCrop.java : 
- UCropActivity.java : 
- - -
## Bug Report
- - -
## 프로그램 개선사항
- - -
## 소감
