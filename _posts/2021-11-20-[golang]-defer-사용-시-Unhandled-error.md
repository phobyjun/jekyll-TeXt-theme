---
title: "[golang] defer 사용 시 Unhandled error"

tags: [golang, go, defer, goland, unhandled error]

excerpt: Go에서 defer 사용 시 에러 핸들링

comments: true!
---

## go defer

Golang에서 defer 사용 시 `Unhandled error` 라는 에러가 발생할 때가 있다. 컴파일 과정에서는 크게 문제가 없지만, 프로그램이 계획대로 돌아가고 있지 않다는 뜻이기 때문에 수정이 필요하다. 그럼 왜 발생하는 것일까? 해결 방법까지 알아보자.

## defer 키워드

[golang.org](golang.org/ref/spec#Defer_statements)에서 설명하는 defer는 다음과 같다.

> defer문은 외부 함수의 끝에 도달해 return문을 실행했거나 해당 고루틴이 panic 상태에 있기 때문에 외부 함수가 반환되는 순간까지 실행이 연기되는 함수를 호출합니다.

간단히 말하면 지연 호출이다. 특정 함수를 감싸고 있는 함수 안에서 가장 나중에, 끝나기 직전에 실행하도록 하는 키워드이다. 간단한 예시를 보자.

![](/assets/img/2021-11-20-1/2.png)

위의 예시는 url을 파라미터로 받아 request를 보낸 후 응답을 반환하는 함수와 이를 실행하는 main 함수이다. `getStatusFromRequest` 함수를 보면 defer 문을 확인할 수 있다.

golang의 http 모듈에서 `Get()`을 통해 request를 보내게 되면 reponse를 받아 처리 후 `.Body.Close()`를 통해 닫아주어야 한다. 이 `Close()` 함수는 결과값이 반환되기 전에 실행되면 안되기 때문에 `defer` 키워드를 통해 결과값이 반환되고 함수가 끝나기 바로 직전에 실행시키는 것이다. 하지만 사진을 보면 연노랑색 박스가 쳐져있는 것을 볼 수 있다.

![](/assets/img/2021-11-20-1/1.png)이 부분에서 바로 Unhandled error를 확인할 수 있다.

## Unhandled error

말 그대로 error handling이 되고 있지 않다는 것이다.

golang에서는 try-catch 와 같이 error를 handling 해주는 구문을 제공해주지 않는다. 개발자가 직접 error를 처리해줘야 한다. 바로 이 부분에서 오류가 발생한다. 위 사진에서 오류가 발생한 부분을 보자.

![](/assets/img/2021-11-20-1/3.png)

`Close()` 함수는 return 값으로 error를 가지고 있는 것을 확인할 수 있다. 즉 위의 코드대로 `defer resp.Body.Close()`라고 사용한다면, `Close()`가 가지는 error를 handling하지 못한다는 것이다.

## 해결방법

익명 함수를 사용해 error를 처리해주면 된다.

![](/assets/img/2021-11-20-1/4.png)

defer문에 익명 함수를 만든 뒤 그 함수 안에서 error를 받아 처리해주면 된다. 오류 메세지는 사라졌고 이제 계획한 대로 코드가 동작할 것이다.

