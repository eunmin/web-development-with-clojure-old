## Clojure Web Stack

- 지난 장에서 간단에 웹 어플리케이션에 대해서 알아봤고 이 장에서는 좀 더 자세히 알아보자.
- 클로저 웹 스택은 여러가지 종류가 있고 각자의 스타일에 맞게 선택하면 된다.
- 여기서는 실제 어플리케이션에서 많이 사용되고 있는 Ring/Compojure 스택에 대해서 알아볼 것이다.
- 지난 장에서 사용자들이 메시지를 남기도 다른 사람들이 볼 수 있는 간단한 웹 어플리케이션을 보여줬다. 그러면서 필요한 부분의 디렉토리 구조나 파일등을 보았다. 하지만 자세히 살펴보지는 않았다. 이번 장에서는 방명록 어플리케이션을 완전히 이해할 수 있도록 자세히 알아보도록 하자.
- 클로저 웹 스택은 자바 HTTP 서블릿 어플리케이션 위에 작성되어 있어서 Jetty, ClassFish, Tomcat 같은 서블릿 컨테이너에서 실행할 수 있다.
- 클로저 어플리케이션은 독립적으로 실행할 수도 있고 어플리케이션 서버에서 자바 어플리케이션과 함께 배포되어 되어 실행 할 수 도있다.
- 클로저 어플리케이션은 AWS, 구글 앱앤진, Heroky, Jelastic 같은 JVM 어플리케이션을 지원하는 클라우드 서비스에 배포할 수 있다.
- 서블릿은 HTTP 프로토콜을 기반으로 웹 요청을 받고 해당 요청에 대한 응답을 생성한다.
- HTTP는 웹 어플리케이션에 필요한 쿠키, 세션, URL Rewriting와 같은 많은 기능을 제공한다.
- 서블릿은 자바에서 사용하기 위해서 만들어져있기 때문에 클로저에서 직접적으로 사용하기는 불편한다.
- Rails, Django등의 프레임워크와는 다르게 클로저 웹 스택은 하나의 통합 프레임워크가 없다. 대신 여러개의 라이브러리들을 조합해서 어플리케이션을 만든다. 이 책에서는 웹 어플리케이션을 만들 때 자주 사용하는 몇 가지 라이브러리를 다룬다.
- Ring은 서블릿 레퍼로 동작한다.
- Compojure는 특정 URL에 대앙하는 request 핸들러 함수들을 매핑한다.

## Ring으로 Request 라우팅하기

- Ring은 커다란 어플리케이션을 작성하기 위해서 HTTP를 작고 모듈화된 API들로 추상화하는 것을 목적으로 만들어 졌다.
- Ruby나 Python을 사용해 봤다면 WSGI나 Rack 라이브러리와 비슷하다는 것을 알 수 있을 것이다.
- 많은 웹 어플리케이션 라이브러리들이 Ring을 기반으로 작성되었다.
- 어플리케이션을 작성하는 동안 Ring을 직접 사용할 일은 거의 없지만 개발이나 문제해결을 위해 Ring을 설계에 대해서 잘 이해하고 있으면 유용할 것이다.
- Ring은 4가지의 기본 컴포넌트로 구성 된다. : hander, requeset, response, middleware

### Request 핸들링

- Ring은 요청과 응답을 클로저 맵으로 나타낸다.
- 핸들러는 요청을 처리하기 위한 함수다. 요청 맵을 인자로 받고 응답 맵을 리턴해 준다.
  ```Clojure
  (defn handler [request-map]
    {:status 200
     :headers {"Content-Type" "text/html"}
     :body (str "<html><body> your IP is : "
                (:remote-addr request-map)
                "</body></html>")})
  ```
- 위의 예제는 자주 사용되는 패턴이기 때문에 Ring에서는 response 함수 같은 핼퍼를 제공한다.
  ```Clojure
  (defn handler [request-map]
    (response
      (str "<html><body> your IP is : "
            (:remote-addr request-map)
            "</body></html>")))
  ```

#### Request와 Response 맵

- Request와 Response 맵에는 서버 Port, URI, Remote address, body의 content type등의 정보를 가지고 있다. 맵의 키는 HTTP RFC와 서블릿 API를 기반으로 하고 있다.

##### Request 맵

- 아래는 Request에 대한 키이다. 요청에서 모든 키가 나올것이라고 보장하지는 않는다. (예 :ssl-client-cert 같은 것은 없을 수도 있다.)
  - `:server-port` - 요청을 처리하는 서버의 포트
  - `:server-name` - 서버의 IP 주소나 이름
  - `:remote-addr` - 클라이언트의 IP 주소
  - `:query-string` - 요청의 쿼리 스트링
  - `:scheme` - http나 https 같은 프로토콜
  - `:request-method` - `:get`, `:head`, `:options`, `:put`, `:post`, `:delete`와 같은 HTTP 메서드
  - `:request-string` - 요청의 쿼리 스트링 (`:query-string`과 같다?)
  - `:content-type` - 요청 body의 MIME type
  - `:content-length` - 요청 body이 byte
  - `:character-encoding` - 요청의 문자 인코딩
  - `:headers` - 요청 해더 맵
  - `:body` - 요청 body의 input stream
  - `:context` - root로 배포 되지 않았을 때 나타나는 어플리케이션 context ???
  - `:uri` - 서버의 URI path; 여기에는 `:context`가 포함 되어 있다. ???
  - `:ssl-client-cert` - 클라이언트 SSL certificate
- middleware에서는 여기 나온 기본 키 이외에 어플리케이션에 따라 키를 추가할 수 있다.

##### Response 맵

- Response 맵에는 3개의 키를 가진다.
  - `:status` - HTTP 응답 status
  - `:headers` - HTTP 응답 header 맵
  - `:body` - 응답의 body
- status는 HTTP RFC에 있는 status code를 나타낸다.
- headers는 HTTP-header의 key/value 쌍의 맵을 가진다. 키 값은 문자열 또는 문자열의 시퀀스를 가진다.
- 마지막으로 body는 문자열 또는 시퀀스, file, input stream을 가진다. body는 응답 코드와 적합한 본문이어야 한다.
- body가 문자열이면 그대로 클라이언트에 전달된다. 만약 시퀀스이면 각 항목을 문자열로 바꿔서 클라이언트에 전달한다. 파일이나 input stream이라면 파일이나 input stream의 내용을 클라이언트에게 전달한다.
