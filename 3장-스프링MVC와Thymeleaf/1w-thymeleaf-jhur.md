## 3.2 Thymleaf의 기본 사용법

- Thymeleaf 는 JSP 와 비슷하다.

SampleDTO 클래스

```java
package org.zerock.ex3.dto;

import lombok.Builder;
import lombok.Data;

import java.time.LocalDateTime;

@Data
@Builder(toBuilder = true)
public class SampleDTO {
    private Long sno;
    private String first;
    private String last;
    private LocalDateTime regTime;
}
```

- 여기서 @Data 는 Getter/Setter, toString(), equals(), hashCode() 를 자동으로 생성해준다.

SampleContoller - 작성된 Sample DTO의 객체를 Model에 추가해 전달

```java
package org.zerock.ex3.controller;

import lombok.extern.log4j.Log4j2;
import org.springframework.stereotype.Controller;
import org.springframework.ui.Model;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.zerock.ex3.dto.SampleDTO;

import java.time.LocalDateTime;
import java.util.List;
import java.util.stream.Collectors;
import java.util.stream.IntStream;

@Controller
@RequestMapping("/sample")
@Log4j2
public class SampleController {
    @GetMapping("/ex1")
    public void ex1() {
        log.info("ex1........");
    }
    @GetMapping({"/ex2"})
    public void exModel(Model model){
        List<SampleDTO> list = IntStream.rangeClosed(1, 20).asLongStream().
                mapToObj(i ->{
                    SampleDTO dto = SampleDTO.builder()
                            .sno(i)
                            .first("First.." + i)
                            .last("Last.." + i)
                            .regTime(LocalDateTime.now())
                            .build();
                    return dto;
                }).collect(Collectors.toList());
        model.addAttribute("list", list);
    }
}
```

ex2.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <ul>
        <li th:each="dto, state : ${list}">
            [[${state.index}]] --- [[${dto}]]
        </li>
    </ul>
</body>
</html>
```

- 반복문 처리 →  th:each = "변수: ${목록}"
- '[[ ]]' 은 인라인 표현식으로 별도로 태그 속성으로 지정하지 않고 사용하고자 할 때 유용하게 쓰임
- 반복문의 상태(state)객체: 이거 이용하면 순번이나 인덱스 번호, 홀수 짝수 지정가능.
- state.index 는 0부터 시작, state.count 는 1부터 시작

결과

![스크린샷 2021-04-23 오후 7 33 39](https://user-images.githubusercontent.com/60052127/115864655-2477f380-a472-11eb-9ac3-e114ef8bb475.png)

### 반복문 처리

th:each = "변수: ${목록}"

### 제어문 처리

th:if ~ unless

```html
<ul>
  <li th:each="dto, state : ${list}">
      <span th:if="{dto.sno % 5 == 0}" th:text="${'----' + dto.sno}"></span>
      <span th:unless="{dto.sno % 5 == 0}" th:text="${dto.first}"></span>
  </li>
</ul>
```

- sno가 5로 나눈 나머지가 0인 경우에는 sno 만을 출력하고 그렇지 않다면 SampleDTO 의 first 를 출력하라

삼항연산자 스타일 사용 가능

```html
<ul>
  <li th:each="dto, state : ${list}" th:text="{dto.sno % 5 == 0}?
${dto.sno} : ${dto.first}">
  </li>
</ul>
```

### inline 속성

- 주로 자바스크립트 처리에서 유용!

SampleController 일부

```java
@GetMapping({"/exInline"})
    public String exInline(RedirectAttributes redirectAttributes){
        log.info("exInline.....");

        SampleDTO dto = SampleDTO.builder()
                .sno(100L)
                .first("First..100")
                .last("Last..100")
                .regTime(LocalDateTime.now())
                .build();
        redirectAttributes.addFlashAttribute("result", "success");
        redirectAttributes.addFlashAttribute("dto", dto);

        return "redirect:/sample/ex3";
    }
    @GetMapping("/ex3")
    public void ex3(){
        log.info("ex3");
    }
```

- 브라우저에서 sample/exInline 호출하면 sample/ex3 으로 리다이렉션됨.

ex3.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <h1 th:text="${result}"></h1>
    <h1 th:text="${dto}"></h1>
    <script th:inline="javascript">
        var msg = [[${result}]];
        var dto = [[${dto}]];
    </script>
</body>
</html>
```

- 자바스크립트 생성시킴
- 문자열은 자동으로 "" 처리가 되어있고 dto 는 JSON 으로 만들어짐.

결과

![스크린샷 2021-04-23 오후 8 01 30](https://user-images.githubusercontent.com/60052127/115864758-470a0c80-a472-11eb-81bd-0dc863add2c7.png)


- th:block ⇒ 이거는 html hidden 태그랑 비슷한 역할함. 화면에는 안보이는데 코드로서의 역할은 한다.

### 링크 처리

- Thymeleaf 의 링크는 '@{}' 를 이용해서 사용함.

아까 ex2의 SampleController의 GetMapping에 "/exLink" 추가

exLink.html

```html
<!DOCTYPE html>
<html lang="en" xmlns:th="http://www.thymeleaf.org">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
<ul>
    <li th:each="dto : ${list}">
        <a th:href="@{/sample/exView}">[[${dto}]]</a>
    </li>
</ul>
</body>
</html>
```

결과

![스크린샷 2021-04-23 오후 8 20 43](https://user-images.githubusercontent.com/60052127/115864775-51c4a180-a472-11eb-8b11-4799648bcf47.png)


- 파라미터 추가하고 싶을 때

```html
<ul>
    <li th:each="dto : ${list}">
        <a th:href="@{/sample/exView(sno=$(dto.sno)}">[[${dto}]]</a>
    </li>
</ul>
```

- path 만들고 싶을 때

```html
<ul>
    <li th:each="dto : ${list}">
        <a th:href="@{/sample/exView/{sno}(sno=$(dto.sno)}">[[${dto}]]</a>
    </li>
</ul>
```
