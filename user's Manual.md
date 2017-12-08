uCrop
=============
User's Manual
-------------
이 프로그램은 사용자의 저장소에서 이미지를 불러오거나 웹에서 임의의 이미지를 불러옵니다.  
이미지의 크기 비율, 각도, 화면비를 수정할 수 있습니다.  
작업을 완료한 후에는 이미지를 사용자의 저장소에 저장할 수 있습니다.  
결과가 만족스럽지 않은 경우 처음으로 되돌아갈 수 있습니다.  

## 메인 화면  
<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/master/image/uCrop_main.png?raw=true" width="30%">

**①**`PICK & CROP`은 사용자의 휴대폰 내 저장소에서 이미지 파일을 불러옵니다.  
**②**`CROP RANDOM IMAGE`는 웹에 있는 임의의 이미지 파일을 불러옵니다.  

**③ Aspect ratio**  
`Dynamic`은 기본적인 비율입니다.  
`Image source`는 비율을 원본 이미지로 계산합니다.  
`X to Y`에 값을 입력하면 X : Y 비율로 이미지를 수정합니다.  

**④**`Max cropped image size`는 수정한 이미지의 최대 해상도를 설정합니다.  

**⑤**`Compression settings`는 수정한 이미지의 확장자를 설정합니다.  

**⑥**`Quality`는 수정한 이미지를 저장할 때 퀄리티를 설정합니다.

**⑦**`Hide bottom UI controls`를 체크하면 편집화면에서 크기 비율, 각도, 화면비를 수정할 수 없습니다. 원본 이미지의 비율을 유지하고 해상도만 변경할 때 사용합니다.  
- - -

## 이미지 편집화면
<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/master/image/zoom.gif?raw=true" width="30%">  

사용자는 이미지의 크기 비율을 수정할 수 있습니다.  
각각의 이미지마다 최소 비율은 상이합니다.  

<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/master/image/rotation.gif?raw=true" width="30%">  

사용자는 이미지의 각도를 수정할 수 있습니다.  
화면 우측의 버튼을 터치하면 우측으로 90도 회전합니다.  
하단의 바에서 미세하게 조정할 수 있습니다.  
좌측의 X 버튼을 터치하면 최초의 상태로 되돌아갑니다.  

<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/master/image/crop.gif?raw=true" width="30%"> 

사용자는 이미지의 화면비를 수정할 수 있습니다.  
화면비는 1:1, 3:4, ORIGINAL(원본), 3:2, 16:9가 있습니다.  

<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/master/image/square_30to50.PNG?raw=true" width="30%">  

메인 화면의 Aspect ratio 영역에서 Square을 선택할 경우 화면 크기 비율과 각도만 수정이 가능합니다.  


- - -

## 결과 화면
<img src="https://github.com/ReDvGaMe/chosun_OSSW_devsign/blob/master/image/result.PNG?raw=true" width="30%">

사용자 임의대로 화면 상단 우측의 체크 표시를 터치하면 결과 화면이 나옵니다.  
편집이 완료되었으면 다운로드를 할 수 있고, 되돌아 갈 경우 메인 화면이 나옵니다.
