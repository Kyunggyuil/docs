---
layout: post
title: 하드웨어 블록 만들기
category: 'Entryjs'
order: 4
---

하드웨어 블록은 기본적으로 일반 블록과 생성 및 사용법이 같으나 아래와 같은 몇가지 특징이 있습니다.

1. 블록색상 고정(#00979D)
1. 하드웨어 연결 프로그램에 값 읽고 쓰기
    1. Entry.hw.sendQueue (값 보내기)
    1. Entry.hw.portData (값 읽기)
1. 하드웨어 초기값 설정
    1. Entry.(하드웨어이름).setZero
1. isNorFor와 Class의 활용
    1. isNotFor
    2. Class

위의 내용에 대한 설명은 다음과 같습니다.


---  

### 블록생상 고정

하드웨어 블록은 현재는 `#00979D`로 고정해서 사용하도록 강제하고 있습니다. 이점 양해부탁드리며 앞으로 소스 개발시 해당 부분 변동 없이 처리 부탁드립니다.

### 하드웨어 연결프로그램에 값 읽고 쓰기

#### 1. Entry.hw.sendQueue (값 보내기)

블록 명세를 보면 func에 블록의 기본 행동을 정의하도록 되어 있습니다. 엔트리 프로그램에서는 정의된 주기대로 websocket을 통해 하드웨어에 데이터를 전달하는데 이때 전달하는 값이 Entry.hw.sendQueue의 값입니다. text형태의 어떠한 값이라고 sendQueue를 통해 전달할수 있고 적절하게 하드웨어에서 받아서 처리 한다면 어떠한 형태의 text도 전달 가능합니다. 다만, 성능이슈가 발생할수 있으므로 최대한 적은 데이터를 보내주시고 JSON형태의 데이터를 권장합니다.

##### 사용 예
{% highlight js %}
function (sprite, script) {
    // Port 라는 key에 '1'이라는 데이터를 하드웨어 프로그램에 보냄.
    Entry.hw.sendQueue['Port'] = '1';
    // 다음 블록으로 진행
    return script.callRetrurn();
}
{% endhighlight %}

#### 2. Entry.hw.portData (값 읽기)

똑같이 func에 사용하는 값으로 주기적으로 하드웨어에서 들어오는 data값 입니다. 값을 읽어서 화면에 표기해야 하는 블록의 경우 해당 값을 읽어서 처리해야 합니다. 기본적으로 text정보 또는 JSON정보가 포함되어 들어 오게 되어 있습니다. 하드웨어 프로그램에서 처리하여 보내준 데이터가 그대로 들어오도록 구성되어 있으니 하드웨어 모듈을 작성하실때 해당 부분을 염두해 두고 만드시면 좋을 것 같습니다.

##### 사용 예
{% highlight js %}
function (sprite, script) {
    // Port 라는 key값을 가진 정보를 읽는다.
    var result = Entry.hw.portData['Port'];
    // 값을 전달함.
    return result;
}
{% endhighlight %}

### 하드웨어 초기값 설정

엔트리 프로그램을 주기적으로 시작하기와 정지하기를 사용자가 수행하게 되고 정지하는 순간 엔트리 작품의 상태가 초기로 돌아가도록 되어 있습니다. 이에 맞춰서 하드웨어도 초기값(아무것도 동작하지 않았던 최초의 상태)으로 설정해줘야 합니다. 물론, 개발방향에 따라 해당기능이 필요 없을수도 있습니다. 이것은 개발자 본인이 판단하시고 작성해 주시면 됩니다.

#### Entry.(하드웨어명).setZero

기본적으로 entryjs/src/blocks/(작성하신 하드웨어 오브젝트 정의 스크립트).js 이곳에 작성한 자바스크립에서 setZero를 정의하도록 합니다. (block_arduino.js롤 참고해 보시는것도 좋습니다.) 해당 소스에는 Entry.hw.sendQueue에 초기값을로 동작할 데이터를 세팅 하고 Entry.hw.update()를 실행시켜서 하드웨어쪽으로 데이터를 보내도록 정의하면 됩니다.

##### 사용예
{% highlight js %}
setZero: function() {
    // Port라는 key값을 가진데이터의 초기값은 0
    Entry.hw.sendQueue['Port'] = '0';
    // 해당 데이터를 하드웨어 전달한다.
    Entry.hw.update();
}
{% endhighlight %}

### isNorFor와 Class의 활용

isNotFor와 Class는 사용자에게 블록을 알맞게 보여주는 역할을 합니다.

#### isNotFor

isNotFor의 역할은 해당 하드웨어가 연결되었을때 해당 하드웨어를 사용자에게 보여주는 역할을 하게 되어있습니다. isNotFor에 적힐 데이터는 Entry.(적성한 하드웨어 오브젝트).name 이 매칭되도록 되어 있습니다. 해당 몇칭이 대소문자 까지 정확히 매칭되어야 해당 하드웨어가 연결되었을때 정확히 표현 할수 있습니다. 다만 해당 내용을 작성하지 않을 경우 항상 표시되도록 되는데 이렇게 작성하시면 저희쪽 반영이 Reject사유가 되니 빠짐없이 채워주시기 바랍니다.

#### Class  

Class의 경우에는 블록들간의 구분선을 보여주는 역할을 합니다. 기본적으로 데이터 get과 set정도의 구분이나 특정 상황에서만 쓰는 블록들간의 그룹핑을 해주는 역할이라고 보시면 됩니다. 기본적으로 (entryjs)/extren/util/static.js category에 등록된 블록 순서대로 블록이 표현되며 해당 class가 변경될때마다 구분선이 생기므로 static.js에 category를 넣어주실때 순서를 잘 생각하고 작성해주시면 됩니다.