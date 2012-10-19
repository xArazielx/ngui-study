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
이번 챕터에서는 사용자 정체성을 표시하는 HUD 를 `TopLeft` 기준으로 배치해보도록 하겠습니다.

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


## 위젯 만들기

아틀라스도 만들어졌으니 이제 위젯을 추가해보도록 하겠습니다. 메인 메뉴 NGUI > `Create a Widget` 을 클릭합니다.

![custom_package](Images/editor.menu.ngui.create_a_widget.png?raw=true)

위젯 툴은 지정한 Atlas 로 UI 구성요소를 쉽게 만들 수 있도록 도와줍니다.
 
![custom_package](Images/editor.ngui.widget_tool.png?raw=true)

편리하게도 Atlas 에 방금만든 HUD 가 이미 선택되어 있습니다. 나중에 Atlas 가 여러개 생기면 Atlas 버튼을 눌러 변경할 수 있습니다. 

Atlas 선택은 필수이지만, Font 는 선택하지 않아도 무방합니다. Font 에 대해서는 다음번에 다루어보도록 하겠습니다.
Template 는 UI 구성 요소들을 미리 만들어 놓은 것입니다. 우리가 출력할 것은 이미지이므로 `Sprite` 를 선택하도록 하겠습니다.

오호~ Color 부분이 Sprite 로 변경되었습니다. 게다가 방금 만든 스프라이트로 자동 선택되어있군요! 다른 스프라이트 선택에 대해서는 다른 이미지를 추가할때 다시 설명드리도록 하겠습니다.

![custom_package](Images/editor.ngui.widget_tool.sprite.png?raw=true)

위젯을 생성하려면 `Add To` 라는 녹색 버튼을 클릭하면 됩니다. 오른쪽에 써있는 Panel 은 오브젝트 계층 구조 중 UI Root(2D) > Camera > Anchor 아래 있는 Panel 에 추가한다는 의미입니다. 오브젝트 계층 구조 트리에서 드래그해서 Panel 위치에 드래그하면 다른 오브젝트의 자식으로 추가되도록 변경할 수 있습니다.


## 위젯 확인

위젯을 생성하면 Panel 아래 Sprite 가 추가된 것을 볼 수 있습니다. 

![custom_package](Images/editor.scene.hud.imp17.png?raw=true)

뭔가 미심쩍다면 Scene 옆에 Game 을 클릭해보도록 합니다.

![custom_package](Images/editor.game.hud.imp17.png?raw=true)

헐 (-_-); 아까 분명히 TopLeft 기준으로 Anchor 를 설정했는데… - ㅁ-)/  ㅛ
삭제해버려 NGUI!!

하시지 마시고… 화면 상단위에 있는 플레이 버튼을 눌러보시기 바랍니다.

![custom_package](Images/editor.play-pause-stop.png?raw=true)

뭔가 나온 것 같긴한데..;; 버그 같기도 하지만… 프리 버전의 정상적인 결과입니다.

![custom_package](Images/editor.game.hud.imp17.running.png?raw=true)

NGUI 개발사도 먹고 살아야 하니까요~ 햐햐~ 
로고가 보기 싫으신 분들은 메인 메뉴 > Window > Asset Store 에서 구매하시면 바로 해결이 가능합니다~
평소 가격은 $95 이지만, 2~3달에 1번씩 $65 정도로 할인해서 판매하기도 하므로 기회를 잘 노리시길 바랍니다.

저는 NGUI 정식 버전을 구매한 상황이지만;; 프리 버전으로 실습하시는 분들을 위해 당분간은 계속 프리 버전으로 진행해 나가도록 하겠습니다.

각설하고요~ NGUI 로고 때문에 잘 안보이긴 하지만 애초에 기대했던 것과는 다른 모양으로 출력되고 있습니다.
그 이유는 위젯의 `Pivot` 때문입니다. 우리말로는 중심점이라고 부르기도 합니다.

![custom_package](Images/editor.inspector.ngui.sprite.png?raw=true)

Anchor 처럼 Pivot 이 **Center**로 저장되어있어서 화면 좌상단에 이미지 중심 위치가 오도록 출력된것입니다.
해결 방법은 간단합니다. Center 를 `TopLeft` 로 변경해주면 끝입니다.

![custom_package](Images/editor.game.hud.imp17.pivot.topleft.png?raw=true)

와하하 ~(-ㅁ-)~ 제대로 된 위치에 출력 성공이니다. 하지만 그래도 뭔가 좀 이상하네요? 테두리가 좀 두껍죠?

원인은 이전 글에서 `Main Camera 를 삭제`했기 때문입니다.

> 액티브된 카메라만큼 화면에 중첩 렌더링이 되기 때문에 3D 공간을 사용하지 않고 2D 기반 게임을 만든다면 Main Camera는 삭제하는 것이 좋습니다

Main Camera 는 화면을 깨끗하게 지워주는 반면에 NGUI 카메라는 깊이 버퍼만 지우기 때문에 알파 부분이 이상하게 표시되는 것입니다.

![custom_package](Images/editor.inspector.ngui.camera.png?raw=true)

Camera 컴포넌트의 Clear Flags 가 **Depth only** 로 되어있는 것을 확인할 수 있습니다.

해결 방법은 간단합니다. Clear Flags 를 SkyBox 나 SolidColor 로 변경하거나 화면을 가득채우는 이미지 스프라이트를 출력해야 합니다. 취향에 따라 선택하시면 되겠습니다~ 실제 게임에서는 2D 게임의 경우는 이미 화면을 채우는 경우가 많고 3D 게임의 경우는 3D 로 렌더링한 화면을 지우고 UI 찍으면 안되기 때문에 Depth 만 지우는 것으로 보입니다.

지금은 와이프가 자고 있는데다, 아직 위젯은 스프라이트 밖에 모르기 때문에 Clear Flags 를 수정하는 것으로 해결해보겠습니다.

![custom_package](Images/editor.game.hud.imp17.clean.png?raw=true)

짝짝짝 >ㅁ<)~




