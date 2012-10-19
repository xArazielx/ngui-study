# NGUI HUD 만들기
… myevan (송영진)

HUD 는 게임 화면에 항상 떠 있는 위젯들을 말합니다. 시야를 최대한 확보하기 위해 각 모서리를 기준으로 분산되어 배치 되어 있는 경우가 많습니다. 모서리를 기준으로 위젯을 배치하기 하기위해서 NGUI 는 Anchor 라는 것을 제공합니다.

## Anchor 설정

오브젝트 계층 구조 화면을 다시 확인해보겠습니다. UI Root (2D) 아래 Camera 와 Panel 사이에 Anchor 라는 것이 존재합니다. 

![custom_package](Images/editor.hierachy.ngui.png?raw=true)

Anchor 를 선택한 다음 오른쪽 Inspector 창을 확인해보도록 하겠습니다.

![custom_package](Images/editor.inspector.ngui.anchor.png?raw=true)

Anchor 에서 가장 중요한 속성은 `Side` 입니다. 디폴트는 Center 로 화면 중앙에 게임 진행에 있어 매우 중요한 정보(메시지 상자)를 표시할때 주로 사용됩니다. Side 는 Center 외에 TopLeft, Top, TopRight, Left, Right, BottomLeft, Bottom, BottomRight 등을 선택할 수 있습니다. Top 은 위쪽 중앙, Left 는 왼쪽 중앙, Right 는 오른쪽 중앙, Bottom 은 가운데 중앙을 의미합니다.

HUD 는 항상 표시되는 정보이기 때문에 화면 중앙보다는 모서리를 택하는 경우가 많습니다. 
이번 챕터에서는 사용자 정체성을 표시하는 HUD 를 TopLeft 기준으로 배치해보도록 하겠습니다.

## 이미지 리소스

NGUI 는 내부에 기본 스킨을 가지고 있지 않기 때문에 화면에 위젯을 출력하기 위해서는 이미지 리소스가 필요합니다. 포토샵을 잘 다루시거나 직업이 그래픽 디자이너이시라면 간단히 하나 만드셔서 사용하면 되겠지만, 그래픽에 소질이 없는 저 같은 프로그래머라면 리소스를 어디서 구해야할지 한숨부터 나오실겁니다. 다행히 저는 훌륭한 그래픽 디자이너 imp17(이경주)님을 와이프로 두고 있어서 비교적 수월하게 리소스를 받을 수 있었습니다. - o-)b 

![custom_package](Images/imp17.hud.nameboard.png?raw=true)

NGUI 용 이미지는 이렇게 가로로 길 필요는 없는데, 그 이유에 대해서는 다음에 다루도록 하겠습니다. 


## 아틀라스 만들기

NGUI 는 **Atlas** 를 이용해 이미지를 출력합니다. Atlas 는 여러가지 이미지들이 모여있는 커다란 이미지를 말합니다. Atlas 와 배치 렌더링 기능을 사용하면 한번의 DP 호출로 모든 UI 를 그릴 수도 있기 때문에 프로그래머들이 매우 좋아하는 기능 중 하나입니다. 참고로 DP 는 그래픽카드로 하여금 삼각형들을 그리도록 하게하는 기능을 말하는데, 삼각형 1개를 그리나 몇백개를 그리나 비슷한 시간이 소요되는 것이 특징입니다. (물론 수 만개 레벨이라면 1개 그릴때에 비해서 좀더 많은 시간이 걸립니다) UI 구성요소에 많이 사용되는 사각형은 달랑 2개 삼각형으로 출력 되기 때문에, 아무 생각 없이 UI 구성요소들을 DP 1번에 1씩 그리다보면 얼마안되는 UI 출력이 3D 지형 출력에 근접하는 부하가 발생하기도 합니다. 

다시 본론으로 돌아와서 NGUI Atlas 를 만들어보도록 하겠습니다. Project 패널에서 컨텍스트 메뉴 Create > `Folder` 를 클릭해서 HUD 라는 폴더를 하나 생성합니다.

![custom_package](Images/editor.project.ngui.hud.png?raw=true)

HUD 아래 **Atlases** 폴더와 **Textures** 폴더를 추가로 생성합니다. Atlas 를 생성하기 위해 `Atlases 폴더를 선택한 상태`에서 메인 메뉴 NGUI > `Atlas Maker` 를 클릭합니다. 

![custom_package](Images/editor.menu.ngui.atlas_maker.png?raw=true)

처음 Atlas 를 생성하는 것은 매우 간단합니다. New Atlas 부분에 이름을 적은 다음 `녹색 Create 버튼`을 클릭하면 끝입니다. 여기에서는 Atlas 이름을 HUD 라고 이름 짓도록 하겠습니다. (Unity3D 에서는 이름 짓기에 거의 제약이 없기 때문에 New Atlas 처럼 띄어쓰기를 포함해서 이름을 지어도 되지만, 향후 리소스 관리 도구를 만들어야 할지도 모르는 누군가를 위해 가능하면 띄어쓰기는 자제하시는 편이 좋습니다. )

![custom_package](Images/editor.ngui.atlas_maker.png?raw=true)

Project 폴더에 HUD.mat 과 HUD.prefab 이 생성된 것을 확인한 다음 `Textures 폴더에 HUD 이미지`를 넣습니다. Finder 에서 이미지를 드래그해서 넣어도 되고, 컨텍스트 메뉴에서 Reveal in Finder 를 클릭해 Finder 를 오픈한 다음 복사해도 무방합니다.

이미지를 선택한 다음 Atlas Maker 를 다시 확인해 보면 Add/Update All 이라는 녹색 버튼이 보이고 Sprites 아래에는 아까 선택한 이미지 이름이 보이실겁니다.

![custom_package](Images/editor.ngui.atlas_maker.add.png?raw=true)

`Add/Update All` 버튼을 누르면 방금 이미지가 추가된 HUD.png 파일이 Atlases 폴더에 추가됩니다.

![custom_package](Images/editor.project.hud.atlas.imp17.png?raw=true)

