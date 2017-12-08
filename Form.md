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
## 오픈 소스 SW에 대한 각 기능별 설명(폴더별)
>  각 소스별 대략적인 설명은 gitgub폴더에 따로 업로드
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
- UCrop.java, UCropActivity.java : 옵션, 어플 오버레이를 조절하는 클래스
- - -
## Bug Report
- - -
## 프로그램 개선사항
- 자동 확대
	- 사용자가 이미지를 편집할 때 화면비를 수정하면 자동으로 이미지가 확대되어 원뵨 상태에서 변동이 생깁니다. 원본의 크기를 유지하고자 하는 사용자는 좌측의 이미지 크기 비율을 수정하는 버튼을 터치하여 직접 바꾸어야 하는 불편함이 있습니다. 화면비를 수정해도 원본 크기 비율을 유지하면 좋겠습니다.
- 여백 기능
	- 이미지를 수정하면 여백이 없도록 자동으로 화면에 꽉 채워지는데 여백을 남기고 싶어하는 사용자도 있기 때문에 여백을 선택사항으로 변경할 수 있으면 좋겠습니다.
- - -
## 소감
**김주영**
> 이번 학기 수업을 들으면서 오픈소스 소프트웨어가 뭔지 자세히 알 수 있었고, 특히 github를 처음에 접하고, 사용할땐 별 흥미가 없었지만, 프로젝트를 하면서 github를 계속해서 사용하다보니 많은 흥미가 생겼고, 과목이 끝난 이후에도 github를 계속 사용해서 블로그를 만드는 등 github를 공부할 것 같습니다. 또한 조장이 되어서 조원들과 같이 협업을 하는 과정도 재미있었고, 조원 애들과 더욱 친해져서 좋았습니다. 나중에 기회가 된다면 github에 올라온 오픈소스 소프트웨어를 가지고 온라인 상에서 다른 사람들과 협업하여 오픈소스 소프트웨어를 만들어볼 것입니다.

**김태완**
> 이 수업을 통해서 오픈소스의 정의와 라이센스, 깃허브에 대해 자세히 알 수 있었습니다. 앞으로 개발하는 프로그램에 대해서는 깃허브를 통해 관리하면서 개인 포트폴리오로 활용할 생각입니다. 이번 겨울방학 때 오픈소스 개발을 하면서 좀 더 능숙하게 깃허브를 다를 수 있도록 좀 더 공부하려고 합니다. 깃허브를 통해서 다양한 오픈소스를 접할 수 있다는 것을 깨달았고, 그것을 실행해보면서 메소드를 분석해보고 모르는 것은 또 검색을 통해서 알아보는 과정이 재미있었습니다.  

**이세은**
> Git은 몇 달 전에 프로젝트를 하면서 처음 알게 되었다. 그때는 팀원과 매일 직접 만나서 파일을 옮기고 피드백을 받았는데 Git이라는 프로그램을 알게 된 후로는 멀리 떨어져서 피드백 할 수 있게 되었다. 당시에는 Branch, Issue 등과 같은 Git의 다양한 기능을 알지 못한 상태로 자료의 수정 기록만 봤지만 수업을 들은 후로 여러 사람들과 더 쉽고 편하게 협업할 수 있게 되었고 앞으로도 쭉 써먹을 수 있는 프로그램을 알게 되어 값진 경험이었다.  
