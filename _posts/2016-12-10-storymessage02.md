---
layout: "post"
title: "StoryMessage(02)"
date: "2016-12-10 20:17"
categories: "야생개발자의프로젝트추억팔이"
---

## 4. 메모리가 새고있습니다!!!!!!

이제 동영상의 생성 속도 효율을 위해서 멀티쓰레딩으로 쓰레드 갯수를 조절해가면서 테스트를 진행하였다. 물론 테스트 효율을 위해서 DB에 메세지를 자동으로 인서트해주는 툴도 만들어서 진행하였다. 상황을 보기위해 작업관리자를 열어놓고 살펴보던 중 문제가 발견되었다. 동영상을 생성할 수록 사용 메모리가 증가하는 것이었다. 시작할 때 26메가 정도였던 동영상을 생성할 수록 점점 증가하여 70메가도 넘어가는 상황이 발생하였다. 코드를 아무리 뒤져봐도 메모리릭이 발생하는 위치를 알 수가 없었다. 결국 팀장님께 도움을 요청했다.

팀장님도 코드를 살펴보셨지만, 메모리가 새는 구간을 발견할 수가 없었다. 그리고 얼마 후 팀장님께서 클로즈업과 내가 참조한 ffmpge 샘플 코드를 돌려보셨는데 동일하게 메모리가 새는 것을 발견하셨다고 하셨다. 결국 팀장님께서 응급방책으로 뭔가를 만들어 주셨는데, 그건 내가 만든 프로그램을 4개정도 자동으로 실행시키고 동영상을 일정 갯수를 생성하면 프로그램을 죽였다가 다시 기동시켜주는 프로그램이었다. 이렇게 내가 만든 프로그램은 멀티쓰레딩도 아닌 괴이한 방식의 멀티 프로세스로 돌아가게 된다.

## 5. SKT와의 연동회의.

여기에서 이제 내가 맡은 프로젝트를 설명하자면 앞서 설명했던 이미지에 사용자의 텍스트를 받아 합성하고 브금과 함께 뭉쳐서 동영상을 생성한 뒤 그 동영상을 각 통신사의 MMS서비스를 통해 발송하는 서비스였다. 이전까지 사용자의 메세지를 jpg나 animated gif 이미지에 합성하는 서비스는 있었지만 동영상에 합성하는 없었고 이 프로젝트가 최초로 그 서비스를 구현하려고 있었다. 1차 목표는 SKT의 네이트온 입점. 나는 대표님과 마케팅팀장님 그리고 투자사의 과장님과 함께 네이트온 쪽에 방문하여 연동방식에 대한 회의가 벌어졌다. 난 당연히 이 연동부터는 개발팀장님이나 다른 개발팀원분이 맡아서 해주지 않을까 했지만, 회사에서는 나에게 당연하듯이 맡겼다.

일단 회의의 가장 큰 쟁점은 동영상의 생성시점 이었다. 동영상 렌더링이란 것이 서버에 꽤 큰 부담이 걸리는 작업이었기에 우리 회사에서는 동영상은 사용자가 전송을 확정한 시점에서 생성하여 발송하기를 원했다. 하지만 네이트온에서 문자보내기 등을 해본 사람은 알겠지만 왼쪽에 폰모양에 미리보기가 보인 후 발송을 하여야 한다. 물론 이때 네이트온 측은 생성된 결과물을 전부 받아낸 상태이고 사용자가 발송을 누르면 그때 발송을 하는 시스템이었다. 즉 사용자가 발송을 하기 전에 미리 동영상이 렌더링 되어야 되는 구조였다. 이렇게 되면 발송을 하지않고 그냥 미리보기까지 하는 사람들의 동영상렌더링 부하도 서버에서 감당하는 구조였기에 회사에서는 최대한 피하려고 했지만, 어디 네이트온이 만만한 회사도 아니고 결국 따라가게 되었다.

뭐 그 외의 연동에 관한 내용은 간단했다. 먼저 동영상 선택등의 화면은 우리측에서 웹을 구성하고 네이트온측은 iframe을 뚫어서 보여주는 형태였다. 그리고 사용자의 진행에 따라 동영상이 생성되면 네이트온측의 API에 통해 동영상이 생성된 url과 미리보기 이미지의 url을 전송해주고 그 이후의 상황은 네이트온에서 알아서 처리하는 구조였다.

다만 네이트온 측에서는 사용자가 받는 동영상의 품질을 높이기 위해 기본 통신망의 MMS프로세스를 태우지 않고, 별도의 프로세스로 MMS를 전송하기를 원했다. 기본 통신망의 MMS프로세스는 원래 수신자의 폰에 맞게 동영상의 사이즈등을 변경/압축하는 과정이 존재한다. 하지만 네이트온에서 사용하는 별도의 프로세스는 그런 과정이 없기 때문에 3개의 사이즈의 동영상을 준비하여 수신자의 폰의 화면크기에 적당한 동영상을 가져가서 발송하는 방식으로 진행되었다. 이 역시 나중에 거하게 뒷통수를 치게 된다.

## 6. 서버구축.

이제 본격적으로 서버를 구축하라는 명령이 하달되었다. 하지만 난 서버에 대해서 잼병인 기본적으로 응용프로그래밍만 배웠던 초짜 개발자에다가 회사에서도 딱히 서버에 대해서 잘 아는 개발자가 없었다. 회사에서는 결국 서버구성에 대해서 협조사에 자문을 구했고 서버구성에 대한 회의가 열리게 되었다. 우리회사측에는 이사님과 개발팀장님 그리고 나 협조사측에는 개발이사와 개발팀장이 모였다. 당췌 여기에 내가 끼는 게 맞는 지 싶지만 일단은 이것저것 물어보면서 서버구축의 방향성을 잡았다.

일단 렌더링 서버는 2대. 그리고 웹서버는 사용자가 몰릴 가능성이 있기에 2대를 구축하고 L4 스위치를 물리기로 하였다. 마지막으로 DB서버 1대. 이렇게 총 5대가 들어가는 향후에도 딱히 잡아본 적 없는 큰 프로젝트였다. 회사에서는 서버를 임대하였고 전부 Windows서버에 DB는 MS-SQL이었다. 그리고 나는 서버 구축에 들어가게 된다. 일단 DNS를 받고 3개의 서브도메인을 땄다. 1개는 L4에 물릴 웹도메인이며 나머지 2개는 렌더링 서버에 물릴 도메인이었다. 그리고는 IIS셋팅으로 서버를 구축해나가기 시작했다. 물론 기존의 클로즈업 서비스의 서버를 참고하여 셋팅을 잡았다. 지금 생각해보면 클로즈업 서비스는 나에게 참 좋은 교보재였던 것 같기도 하고...

문제는 웹사이트 구성이었다. 사실상 웹사이트 구성을 해본 적이 없었던 나는 ASP/JSP/PHP 등등 웹언어를 다룰 줄을 몰랐다. 유일하게 다룰 줄 아는 건 C# 기반의 ASP.NET뿐. 어차피 서버도 Windows서버였겠다, 나는 그냥 구현을 목적으로 ASP.NET으로 서버를 구축하였다. 일단은 UI부분은 협조사에서 Flash로 만들어 주었다. Flash는 2개였는데 하나는 리스트에서 보여질 기본샘플 동영상을 재생하는 플레이어였다. 그리고 나머지 하나는 동영상에 할당된 캐릭터를 선택하고 각각에 말풍선에 메세지를 입력한 뒤 말풍선이 빈 동영상 위에 적당한 타이밍에 글자를 찍어주어 미리보기를 구현한 에디터였다. 해당 에디터에서 입력한 메세지 등은 DB로 입력되고, 그러면 렌더링 서버의 프로그램이 DB를 와칭하다가 인서트된 로우를 가져와서 동영상을 렌더링하도록 하였다.

여기서 렌더링 서버는 별개의 도메인을 가진 2개의 서버였다. 문제는 서버가 서로 데이터 공유같은 건 고려도 안 되어있으니, 당연히 양측에서 생성된 동영상은 별개의 주소를 가지게 된다. 어차피 동영상의 이름은 메세지가 들어온 로우의 id를 기반으로 생성되니 이름이야 별 문제가 없었지만 어느 서버에 동영상이 생성되었는 지 알 수가 없는 것이었다. 결국 이건 동영상을 생성한 뒤 동영상을 생성한 프로그램이 자기가 존재하는 서버의 도메인을 로우에 써주는 걸로 해결을 봤다. 애초에 우격다짐으로 서버를 짠게 문제였지만, 일단은 돌아가니 별 생각을 안했다.

## 7. 연동의 시작

다시 문제가 발생하였다. 동영상이 완성된 시점을 웹사이트에서 알아내야 되는데, 당시 나의 짧은 지식으로는 어떻게 해야될 지 감이 안 잡혔다. 결국 내가 구현한 방법은 일단은 대기화면을 위한 플래시를 하나 얹어놓고 제로사이즈의 아이프레임을 하나 뚫어 놓았다. 그리고 거기에서 특수한 페이지를 하나 만들었는데, while(1) 안에서 DB컨넥션을 걸어놓고 해당 메세지의 로우의 스테이터스가 완료로 변경될때까지 계속 와칭하다가 완료로 변경되면 while을 빠져나가서 네이트온 API로 리다이렉트 되는 페이지였다. 그리고 그 페이지를 제로사이즈의 아이프레임에 띄우게 하였다.

일단은 문제없이 돌아갔다. 연동테스트도 별 문제없이 돌아갔고. 이제 프로젝트는 오픈까지 순항할 것 같이 보였다.
