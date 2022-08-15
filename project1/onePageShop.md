# :pushpin: 원페이지 쇼핑몰

</br>

## 1. 제작 기간 & 참여 인원
- 2021년 9월 27일 ~ 10월 13일
- 개인 프로젝트

</br>

## 2. 사용 기술
#### `Front-end`
  - Javascript
  - JQuery
#### `Back-end`
  - Python3
  - Flask
  - MongoDB
  - AWS

</br>

## 3. 핵심 기능 
- 주문자 이름, 주문 개수, 주소, 전화번호를 입력받고 이를 DB에 저장한다.
- 저장된 데이터를 화면 하단에 목록으로 보여준다.
- 실시간 환율을 화면에 보여준다.

<details>
  <summary><b>핵심 기능 상세 설명 펼치기</b></summary>
  <div markdown="1">

<br>
    
  ### 3.0. 기본 구성
  ![](https://www.notion.so/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F04945ba3-ac4a-482e-9f77-a794981de558%2FUntitled.png?table=block&id=3d955162-162a-439b-b12a-693f35bd9f8e&spaceId=83c75a39-3aba-4ba4-a792-7aefe4b07895&width=2000&userId=2f0da12b-1a66-4b50-bcbe-b24c58210e93&cache=v2)

  - **`app.py`**
    - 서버의 역할을 담당
      <details>
        <summary><b>구성코드</b></summary>
        <div markdown="1">

        ``` py
          from flask import Flask, render_template, jsonify, request

          app = Flask(__name__)

          from pymongo import MongoClient

          client = MongoClient('mongodb://test:test@localhost', 27017)
          db = client.dbhomework


          # HTML 화면 보여주기
          @app.route('/')
          def homework():
              return render_template('index.html')

          # 주문하기(POST) API
          @app.route('/order', methods=['POST'])
          def save_order():
              name_receive = request.form['name_give']
              count_receive = request.form['count_give']
              address_receive = request.form['address_give']
              phoneNumber_receive = request.form['phoneNumber_give']

              doc = {
                  'name' : name_receive,
                  'count' : count_receive,
                  'address' : address_receive,
                  'phoneNumber' : phoneNumber_receive
              }
              db.homework.insert_one(doc)

              return jsonify({'msg': '주문 완료!!'})

          # 주문 목록보기(Read) API
          @app.route('/order', methods=['GET'])
          def view_orders():
              orders = list(db.homework.find({}, {'_id': False}))
              return jsonify({'order_list': orders})

          if __name__ == '__main__':
              app.run('0.0.0.0', port=5000, debug=True)
        ```
        </div>
      </details>

  - **`index.html`**
    - 클라이언트에게 직접적으로 보여지는 웹페이지 역할을 담당
      <details>
        <summary><b>구성코드</b></summary>
        <div markdown="1">

        ```html
          <!doctype html>
            <html lang="en">
            <head>
                <!-- Required meta tags -->
                <meta charset="utf-8">
                <meta name="viewport" content="width=device-width, initial-scale=1, shrink-to-fit=no">

                <!-- Bootstrap CSS -->
                <link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/css/bootstrap.min.css"
                    integrity="sha384-Gn5384xqQ1aoWXA+058RXPxPg6fy4IWvTNh0E263XmFcJlSAwiGgFAW/dAiS6JXm" crossorigin="anonymous">

                <!-- Google fonts 추가    -->
                <link rel="preconnect" href="https://fonts.googleapis.com">
                <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
                <link href="https://fonts.googleapis.com/css2?family=Nanum+Gothic:wght@700&display=swap" rel="stylesheet">

                <!-- Optional JavaScript -->
                <!-- jQuery first, then Popper.js, then Bootstrap JS -->
                <script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>
                <script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.12.9/umd/popper.min.js"
                    integrity="sha384-ApNbgh9B+Y1QKtv3Rn7W3mgPxhU9K/ScQsAP7hUibX39j7fakFPskvXusvfa0b4Q"
                    crossorigin="anonymous"></script>
                <script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.0.0/js/bootstrap.min.js"
                    integrity="sha384-JZR6Spejh4U02d8jOt6vLEHfe/JQGiRRSQQxSfFWpi1MquVdAyjUar5+76PVCmYl"
                    crossorigin="anonymous"></script>

                <title>나만의 쇼핑몰</title>
                <meta property="og:title" content="나만의 쇼핑몰" />
                <meta property="og:description" content="맛있는 사과사세요~~🍎" />
                <meta property="og:image" content="{{ url_for('static', filename='ogimage.png') }}" />
                <style>
                    * {
                        font-family: 'Nanum Gothic', sans-serif;
                    }
                    .wrap {
                        width: 700px;
                        margin: 50px auto auto auto;
                    }
                    .img {
                        width: 700px;
                        height: 500px;

                        background-image: url("https://images.unsplash.com/photo-1568702846914-96b305d2aaeb?ixid=MnwxMjA3fDB8MHxwaG90by1wYWdlfHx8fGVufDB8fHx8&ixlib=rb-1.2.1&auto=format&fit=crop&w=1170&q=80");
                        background-position: center;
                        background-size: cover;
                    }
                    .description {
                        font-size: 20px;
                    }
                    .description_price {
                        font-weight: normal;
                        font-size: 20px;
                    }
                    .btn-primary {
                        font-size: 20px;

                        display: block;
                        width: auto;
                        margin: auto;

                        border-radius: 5px;
                    }
                    #dollartowon {
                        color:blue;
                    }
                    .table {
                        margin: 20px auto 20px auto;
                        text-align: center;
                    }
                </style>
                <script>
                    $(document).ready(function () {
                        dollar_to_won();
                        order_listing();
                    })
                    function ordered() {
                        let name = $('#name-text').val()
                        let count = $('#count-text').val()
                        let address = $('#address-text').val()
                        let phoneNumber = $('#phoneNumber-text').val()

                        $.ajax({
                            type: "POST",
                            url: "/order",
                            data: { name_give: name,
                                count_give: count,
                                address_give: address,
                              phoneNumber_give: phoneNumber
                            },
                            success: function (response) {
                                alert(response["msg"]);
                                window.location.reload();
                            }
                        })
                    }
                    function order_listing() {
                        $.ajax({
                            type: "GET",
                            url: "/order",
                            data: {},
                            success: function (response) {
                                let orders = response['order_list'] 
                                for (let i=0; i<orders.length; i++) {
                                    let address = orders[i]['address']
                                    let count= orders[i]['count']
                                    let name = orders[i]['name']
                                    let phoneNumber= orders[i]['phoneNumber']

                                    let temp_html = `<tr>
                                                    <th scope="row">${name}</th>
                                                    <td>${count}</td>
                                                    <td>${address}</td>
                                                    <td>${phoneNumber}</td>
                                                </tr>`
                                    $('#order-list').append(temp_html)
                                }

                            }
                        })
                    }
                    function dollar_to_won() {
                        $.ajax({
                            type: "GET",
                            url: "http://spartacodingclub.shop/sparta_api/rate",
                            data: {},
                            success: function (response) {
                                let rate = response['rate'];
                                let temp_html = `${rate}`;
                                $('#dollarToWon').append(temp_html);

                            }
                        })
                    }
                </script>
            </head>
            <body>
                <div class="wrap">
                    <div class="img"> </div>
                    <div class="description">
                        <h1>사과를 팝니다 <span class="description_price">가격: 1,000원/개</span></h1>
                        <p>이 사과는 먹으면 기분이 좋아지는 효과가 있어요. 이유는 그냥 달고 맛있거든요😁</p>
                        <p>오늘의 환율($ → ₩) : <span id="dollarToWon"> </span> 원</p>
                    </div>
                    <div class="orderBox">
                        <div class="input-group mb-3">
                            <div class="input-group-prepend">
                                <span class="input-group-text" id="inputGroup-sizing-default">주문자 이름</span>
                            </div>
                            <input type="text" id="name-text" class="form-control" aria-label="Default"
                                  aria-describedby="inputGroup-sizing-default">
                        </div>
                        <div class="input-group mb-3">
                            <div class="input-group-prepend">
                                <label class="input-group-text" for="count-text">개수</label>
                            </div>
                            <select class="custom-select" id="count-text">
                                <option selected></option>
                                <option value="1개">1개</option>
                                <option value="3개">3개</option>
                                <option value="6개">6개</option>
                                <option value="12개">12개</option>
                            </select>
                        </div>
                        <div class="input-group mb-3">
                            <div class="input-group-prepend">
                                <span class="input-group-text" id="inputGroup-sizing-default">주소</span>
                            </div>
                            <input type="text" id="address-text" class="form-control" aria-label="Default"
                                  aria-describedby="inputGroup-sizing-default">
                        </div>
                        <div class="input-group mb-3">
                            <div class="input-group-prepend">
                                <span class="input-group-text" id="inputGroup-sizing-default">전화번호</span>
                            </div>
                            <input type="text" id="phoneNumber-text" class="form-control" aria-label="Default"
                                  aria-describedby="inputGroup-sizing-default">
                        </div>
                        <button onclick = "ordered()" type="button" class="btn-primary">주문하기</button>
                    </div>
                    <table class="table">
                        <thead>
                            <tr>
                                <th scope="col">주문자 이름</th>
                                <th scope="col">개수</th>
                                <th scope="col">주소</th>
                                <th scope="col">전화번호</th>
                            </tr>
                        </thead>
                        <tbody id="order-list">

                        </tbody>
                    </table>
                </div>
            </body>
            </html>
        ```
        </div>
      </details>

      <br>

  ### 3.1. 주문하기
  - **서버** 
    - 브라우저에서 보낸 데이터를 이름, 수량, 주소, 전화번호로 구분하여 DB에 저장
    - 저장이 완료되면 "주문 완료" 메세지 return 
      <details>
        <summary><b>사용코드</b></summary>
        <div markdown="1">

        ``` py
          @app.route('/order', methods=['POST'])
          def save_order():
              name_receive = request.form['name_give']
              count_receive = request.form['count_give']
              address_receive = request.form['address_give']
              phoneNumber_receive = request.form['phoneNumber_give']

              doc = {
                  'name' : name_receive,
                  'count' : count_receive,
                  'address' : address_receive,
                  'phoneNumber' : phoneNumber_receive
              }
              db.homework.insert_one(doc)

              return jsonify({'msg': '주문 완료!!'})
        ```

        </div>
      </details>

  - **클라이언트** 
    - 브라우저에서 입력받은 이름, 수량, 주소, 전화번호 데이터를 각 변수에 담아 서버에 **POST** 요청
    - 보낸 데이터가 DB에 정상적으로 저장되었다면 return 받은 메세지를 alert
    - 화면 새로고침

      <details>
        <summary><b>사용코드</b></summary>
        <div markdown="1">

        ``` jsx
          function ordered() {
            let name = $('#name-text').val()
            let count = $('#count-text').val()
            let address = $('#address-text').val()
            let phoneNumber = $('#phoneNumber-text').val()

            $.ajax({
                type: "POST",
                url: "/order",
                data: { name_give: name,
                    count_give: count,
                    address_give: address,
                    phoneNumber_give: phoneNumber
                },
                success: function (response) {
                    alert(response["msg"]);
                    window.location.reload();
                }
            })
          }
        ```

        </div>
      </details>

<br>

  ### 3.2. 주문 목록 보여주기
  - **서버** 
    - DB에 저장된 데이터 전체를 클라이언트에 return

      <details>
        <summary><b>사용코드</b></summary>
        <div markdown="1">

        ``` py
          @app.route('/order', methods=['GET'])
          def view_orders():
              orders = list(db.homework.find({}, {'_id': False}))
              return jsonify({'order_list': orders})
        ```

        </div>
      </details>

  - **클라이언트** 
    - 서버에서 전송한 데이터를 이름, 수량, 주소, 전화번호로 구분하여 변수에 할당
    - append() 활용하여 가져온 데이터와 함께 동적으로 html 추가

      <details>
        <summary><b>사용코드</b></summary>
        <div markdown="1">

        ``` jsx
          $(document).ready(function () {
            order_listing();
          })
          function order_listing() {
              $.ajax({
                  type: "GET",
                  url: "/order",
                  data: {},
                  success: function (response) {
                      let orders = response['order_list'] 
                      for (let i=0; i<orders.length; i++) {
                          let address = orders[i]['address']
                          let count= orders[i]['count']
                          let name = orders[i]['name']
                          let phoneNumber= orders[i]['phoneNumber']

                          let temp_html = `<tr>
                                          <th scope="row">${name}</th>
                                          <td>${count}</td>
                                          <td>${address}</td>
                                          <td>${phoneNumber}</td>
                                      </tr>`
                          $('#order-list').append(temp_html)
                      }
                  }
              })
          }
        ```

        </div>
      </details>

<br>

  ### 3.3. 환율 계산하기
  - **클라이언트**
    - JSON 형식 데이터가 저장된 url에 GET 요청
    - append() 활용하여 가져온 데이터와 함께 동적으로 html 추가

      <details>
        <summary><b>사용코드</b></summary>
        <div markdown="1">

        ``` jsx
        function dollar_to_won() {
            $.ajax({
                type: "GET",
                url: "http://spartacodingclub.shop/sparta_api/rate",
                data: {},
                success: function (response) {
                    let rate = response['rate'];
                    let temp_html = `${rate}`;
                    $('#dollarToWon').append(temp_html);

                }
            })
        }
        ```

        </div>
      </details>

  </div>
</details>

<br>

## 4. 추후 발전방향
  - 간단한 기능을 수행하는 페이지이지만 FE, BE, AWS를 활용한 서비스 런칭까지 배울 수 있었던 프로젝트입니다.
  - 스파르타 코딩클럽 웹개발 종합반 교육 과정에서 최종 결과물로 만든 프로젝트이지만  강의 전체 내용이 녹아들어 있진 않아 복습하면서 여러 기능들을 추가해 볼 예정입니다.
    - 데이터 입력 누락 시 경고 메세지(조건문, 함수 활용)
    - 여러 제품을 추가하여 삭제, 추가(스크롤링, pymongo 활용)
