# :pushpin: 원페이지 쇼핑몰
> 링크 : [kimcno3.shop](http://kimcno3.shop/)

</br>

## 1. 제작 기간 & 참여 인원
- 2021년 9월 27일 ~ 10월 13일
- 개인 프로젝트

</br>

## 2. 사용 기술
#### `Back-end`
  - Python3
  - Flask
  - BeautifulSoup4
  - NoSQL
  - MongoDB
  - AWS
#### `Front-end`
  - Javascript
  - jQeury
  - Ajax


</br>

## 3. 핵심 기능 
- 주문자 이름, 주문 개수, 주소, 전화번호를 입력받고 이를 DB에 저장한다.
- 저장된 데이터를 화면 하단에 목록으로 보여준다.
- 실시간 환율을 화면에 보여준다.

<details>
  <summary><b>핵심 기능 설명 펼치기</b></summary>
  <div markdown="1">

<br>

  <!-- ### 3.1. 전체 흐름
  ![전체 흐름](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F04945ba3-ac4a-482e-9f77-a794981de558%2FUntitled.png?table=block&id=3d955162-162a-439b-b12a-693f35bd9f8e&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=2000&userId=2f0da12b-1a66-4b50-bcbe-b24c58210e93&cache=v2)

  - **서버** 
    - flask를 통해 API을 구축
    - 클라이언트가 요청하는 데이터를 API를 통해 전송

  - **클라이언트** 
    - 원하는 데이터를 서버에 요청
    - API를 통해 데이터를 받아오고 화면에 출력한다.

  - **DataBase** 
    - 주문 내역을 구분하여 저장
    - 저장된 데이터를 필요한 형태로 찾아준다. -->

  ### 3.1. 주문하기
  - **서버** :pushpin: [코드 확인]()
    - 브라우저에서 보낸 데이터를 이름, 수량, 주소, 전화번호로 구분하여 DB에 저장
    - 저장이 완료되면 "주문 완료" 메세지 return 
  - **클라이언트** :pushpin: [코드 확인]()
    - 브라우저에서 입력받은 이름, 수량, 주소, 전화번호 데이터를 각 변수에 담아 서버에 **POST** 요청
    - 보낸 데이터가 DB에 정상적으로 저장되었다면 return 받은 메세지를 alert
    - 화면 새로고침

  ### 3.2. 주문 목록 보여주기
  - **서버** :pushpin: [코드 확인]()
    - DB에 저장된 데이터 전체를 클라이언트에 return

  - **클라이언트** :pushpin: [코드 확인]()
    - 서버에서 전송한 데이터를 이름, 수량, 주소, 전화번호로 구분하여 변수에 할당
    - temp_html, append() 활용하여 가져온 데이터와 함께 동적으로 html 추가

  ### 3.3. 환율 계산하기
  - **클라이언트** :pushpin: [코드 확인]()
    - JSON 형식 데이터가 저장된 url에 GET 요청
    - temp_html, append() 활용하여 가져온 데이터와 함께 동적으로 html 추가

  </div>
</details>

</br>

## 4. 핵심 트러블 슈팅
### 4.1. 컨텐츠 필터와 페이징 처리 문제
- 저는 이 서비스가 페이스북이나 인스타그램 처럼 가볍게, 자주 사용되길 바라는 마음으로 개발했습니다.  
때문에 페이징 처리도 무한 스크롤을 적용했습니다.

- 하지만 [무한스크롤, 페이징 혹은 “더보기” 버튼? 어떤 걸 써야할까](https://cyberx.tistory.com/82) 라는 글을 읽고 무한 스크롤의 단점들을 알게 되었고,  
다양한 기준(카테고리, 사용자, 등록일, 인기도)의 게시물 필터 기능을 넣어서 이를 보완하고자 했습니다.

- 그런데 게시물이 필터링 된 상태에서 무한 스크롤이 동작하면,  
필터링 된 게시물들만 DB에 요청해야 하기 때문에 아래의 **기존 코드** 처럼 각 필터별로 다른 Query를 날려야 했습니다.

<details>
<summary><b>기존 코드</b></summary>
<div markdown="1">

``` jsx
// 코드 작성란
```

</div>
</details>

- 이 때 카테고리(tag)로 게시물을 필터링 하는 경우,  
각 게시물은 최대 3개까지의 카테고리(tag)를 가질 수 있어 해당 카테고리를 포함하는 모든 게시물을 질의해야 했기 때문에  
- 아래 **개선된 코드**와 같이 QueryDSL을 사용하여 다소 복잡한 Query를 작성하면서도 페이징 처리를 할 수 있었습니다.

<details>
<summary><b>개선된 코드</b></summary>
<div markdown="1">

~~~ py
# 코드 작성란
~~~
</div>
</details>

<br>

## 5. 그 외 트러블 슈팅
<details>
<summary>트러블1</summary>
<div markdown="1">

</div>
</details>

<br>

## 6. 추후 발전방향
>