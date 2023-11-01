---
title: "[wooteco] 우아한 테크 코스 프리코스 1주 차 회고"
categories: [wooteco]
tag: ["wooteco", "우테코"]
toc: true
toc_sticky: true
author_profile: false
sidebar:
    nav: "counts"
---

# 📌 1년 전과 지금의 나는 달라졌나

1년 전의 나를 돌아볼 수 있었던 일주일이 쏜살같이 지나갔다. 과거 우테코 5기에 도전했지만 최종 코테까지도 가지 못했던 쓰라린 기억이 있다. 사실 지금 생각해보면 1년 전의 나는 그렇게 간절하지 않았던것 같다. 그냥 우테코라는 좋은 과정이 있구나, 저기 들어가면 좋겠는데? 이런 마음가짐으로 참여했었던것 같다.

1주 차 미션을 공지한 메일을 열어 숫자 야구 미션임을 확인하고 처음 들었던 생각은 "1년 전의 나는 어떻게 만들었을까?"였다. 과거의 잔재를 열어보니 역시 참혹 그 자체였다. 그때 자바를 기본서를 보며 공부한지 2개월쯤 되었을 때였는데 아무런 생각 없이 접근 제어자를 설정하고, 매직 넘버가 남발하던 쓰레기 같았던 코드였다. 당시 프리코스를 진행할 때는 그래도 나름 만족할만한 결과를 낸거 같아 뿌듯했었던것 같았는데 오만이였을까.

10개월 동안 혼자서만 달려왔던 내가 프리코스에 참여하는 수천 명의 사람들과 프로그래밍과 관련된 내용을 얘기할 수 있다는 게 감격스럽기도 하고, 평소에 놓쳤던 부분도 다시 한번 생각하게 되는 일주일이었다. 일주일 동안 커뮤니티에서 많은 내용들을 접하며 학습하고, 스스로 했던 고민들 중 가장 기억나는 몇 가지를 적어보자 한다.

> 1주 차 미션을 수행하며 학습했던  Notion링크와 블로그 글들을 첨부한다
>
> - [Notion: 왜 일급 컬렉션을 사용할까?](https://ykmxxi.notion.site/cda7ae6c4ce34f03b1145a55bbbf7967?pvs=4)
> - [Notion: 왜 모든 원시값과 문자열을 포장할까?](https://ykmxxi.notion.site/343447ee124f4e2b9e06bda78e5cd0c8?pvs=4)
> - [Notion: MVC 패턴](https://ykmxxi.notion.site/MVC-0496b91fae5e408a9cdce7b278f7a54b?pvs=4)
> - [JDK 17 까지의 변천사 by TestCode](https://ykmxxi.github.io/%EC%9E%90%EB%B0%94/java-jdk17/)
> - [Java Enum](https://ykmxxi.github.io/%EC%9E%90%EB%B0%94/java-enum/)
> - [Java final & 불변 객체](https://ykmxxi.github.io/%EC%9E%90%EB%B0%94/java-final-immutable-object/)



# 📌 뭐든 꼼꼼히

1주차 미션을 끝낸 다음날 코드 리뷰에 답하기 위해 내 PR을 까보니 여기저기 오타가 보이기 시작했다. 커밋 메시지에서도 오타가 존재했고, 리드미에도 오타가 존재했다. 마감 전까지 수십번 확인한거 같았는데 면밀히 살피지 못한거 같다. 사실 뭐든 체크를 할 때 오랫동안 앉아서 한번에 여러번 스크롤 하다보면 집중력은 무너지고 대충대충 넘기는 상황이 발생했던것 같다. 다음부터 검토를 할때는 시간 간격을 두고 한 번씩만 검토를 진행하도록 바꿔봐야 겠다.

또 치명적인 실수가 하나 있었다. 코드 컨벤션을 정확히 준수하고 있다 생각했는데 오잉?

<center><img src="/assets/images/wooteco/precourse/1.png"></center>

space와 tab을 혼용해 치명적인 실수를 했다. Mac 에서 확인했을 때는 몰랐는데 Window 환경에서 커밋을 쭉 둘러보니 들여쓰기가 8 space가 아니라 8 tab으로 적용해 사용했더니 위 같은 코드가 되었다.

역시나 인텔리제이 옵션을 확인해보니 스페이스가 아닌 tab을 적용하고 있었다. IDE와 Github에서 스페이스로 적용되었는 것처럼 보였는데 맥북이 아닌 Window 컴퓨터로 보니 우주 코드가 만들어져 있었다. 이 부분은 나중에 확인해 봐야겠다.

# 📌 리팩토링 체크 리스트 만들기

이건 1주 차 미션을 진행하며 느낀 점인데, 리팩토링 체크 리스트가 없으니 무분별한 리팩토링이 남발했던 것 같았다. 또 놓친 부분도 있었던거 같다.

2주 차 미션을 진행할 때는 어떻게라도 돌아가는 쓰레기를 만드는 구현이 끝나면 README에 따로 리팩토링 체크 리스트를 두어 적용해야 겠다.

# 📌 상수만 모아서 getter만 남발할거면 enum을 쓰지마

 과거의 나는 상수를 모아 getter를 남발하는 형식으로 enum을 사용했다. 상수를 모아서 관리하니까 괜히 유지 보수성도 올라가고 여러 클래스가 생기니 나도 클래스 분리 실력이 늘었다는 착각까지 했었던것 같다.

```java
public enum ErrorMessage {

    ERROR_MESSAGE1("에러 1 발생"),
    ERROR_MESSAGE2("에러 2 발생"),
    ERROR_MESSAGE3("에러 3 발생");

    private final String message;

    ErrorMessage(final String message) {
        this.message = message;
    }

    public String getMessage() {
        return message;
    }
    
}

```

뭔가 enum을 잘못 사용하고 있다는 생각이 들어 enum에 대해 학습을 시작했다. enum도 객체를 프로그래밍 언어에서 구현하는 방법 중 하나이며 스스로 상태와 행위를 관리할 수 있어야 하는데 값만 퍼 나르는 용도로 사용하고 있었다.

계속 학습을 하면서 느낀 점은 enum은 매력적인 기술이라는 것이다. 문맥을 담아 의도를 명확하게 드러낼 수 있고, 함수형 인터페이스와 합을 이루면 아름다워진다는 등 많은 장점이 있었지만 일단 스스로 상태와 행위를 관리하는 enum으로 만드는 것에 집중하기로 결정했다.

enum에게도 메시지를 보낼 수 있도록 설계하니 생각보다 많은 코드에서 getter의 남발이 줄었고 코드에서 의도가 명확하게 드러났다(나만의 착각인가).

```java
public enum GameResultMessage {
	BALL("볼"), STRIKE("스트라이크"), NOTHING("낫싱");

	private final String message;

	GameResultMessage(final String message) {
		this.message = message;
	}

	public static String createMessage(final GameResult gameResult) {
		if (gameResult.isNothing()) {
			return NOTHING.message;
		}
		int ballCount = gameResult.getBallCount();
		int strikeCount = gameResult.getStrikeCount();
		if (gameResult.existOnlyBall()) {
			return createCountMessage(ballCount, BALL);
		}
		if (gameResult.existOnlyStrike()) {
			return createCountMessage(strikeCount, STRIKE);
		}
		return String.join(" ", createCountMessage(ballCount, BALL), createCountMessage(strikeCount, STRIKE));
	}

	private static String createCountMessage(final int count, final GameResultMessage gameResultMessage) {
		return String.join("", count + gameResultMessage.message);
	}

}
```

굳이 상수만 모아서 getter만 남발할 거면 오히려 상수를 사용하는 클래스에 모아두는 것이 좋아 보인다. 이 상수가 어떤 곳에서 사용되는지 정확하니까. 만약 상수로도 메시지를 보낼 수 있다면 enum 도입을 고려해보자.

다음 목표는 enum을 함수형 인터페이스와 결합해 사용해보는 것이다. 언제쯤 도입할 수 있을지는 모르겠지만 이번 프리코스에서 한번 쯤은 존재하지 않을까?



# 📌 개발에 완벽함이 존재할까

항상 처음부터 완벽히 하려는 습관을 버리기가 참 힘들다. README의 기능 목록을 완벽하게 만들고 싶고, 시작부터 완벽한 설계를 하고 싶고 욕망이 끝이 없다. 완벽함을 버리자.

프리코스 미션을 보면 기능 요구사항과 제한사항이 모호하다. 코수타에서도 언급했듯이 현장의 요구사항은 명확하게 내려오지 않는다. 정답이 없는 요구사항들에 대해 어떻게 반응하고 어떻게 해결해 나갈 것인지, 스스로의 고민하는 힘을 길르는 것이 우테코가 우리에게 바라는 점이 아닐까?

완벽한 정답을 찾기 보다는 모호한 문제와 맞닥뜨렸을 때 스스로 고민하고 최선의 판단을 내리려고 노력해 나가는 행동을 습관으로 만들어야 겠다. 물론 다른 사람들과 열띈 토론을 통해 어설픈 완벽함이라도 추구해야겠지만