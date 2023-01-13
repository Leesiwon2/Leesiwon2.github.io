---
title:  "[study] val(), text(), html() 차이"
layout: single
categories: 
    - study
comments: true
date: 2023-01-12
last_modified_at: 2023-01-13
author: "Lee Cool"
permalink: categories/study/val-text-html
---


### val(), text(), html() 차이

데이터를 가져올 때 특수문자가 있을 경우,  

진행중인 프로젝트에서 DB에서 가져온 값은 똑같은데 전체 grid 에는 제대로 표시가 되는데 상세 팝업 화면에서는 특수코드로 변환이 되었다.

> __1. 전체 Grid 화면__  
그리드에 데이터를 테이블 형식으로 만들어서 넣는 방식
![grid](/img/grid.PNG)  
````javascript
let txtDiv += '<td>data.list.productName</td>';
$('#list').append(txtDiv);
````  
<br>

> __2. 상세 팝업 화면__  
> 테이블 내의 정해진 위치에 값을 넣는 방식
![detail](/img/detail.PNG)  
````javascript
 $('#productNm').val(data.list.productName);
 ````

> 상세 팝업 화면에서 값을 넣을 때, html()을 사용하니까 정상적으로 보인다.  
![successData](/img/successData.PNG) 
````javascript
 $('#productNm').html(data.list.productName);
 ````

이를 통해서 val(), text(), html() 의 차이점이 궁금했다.  
세가지 모두 평소에 자주 쓰지만, 정확한 차이점을 구분해보고 싶었다.
<br>
<br>
## __val()__

 - 일치하는 요소 집합에서 첫 번째 요소의 현재 값을 가져오거나 일치하는 모든 요소의 값을 설정합니다. [출처:https://api.jquery.com/] 
 <br>
 - 주로 input이나 textarea의 값을 가져오거나 세팅할 때 사용한다.
<br>
<br>

## __text()__

- 하위 항목을 포함하여 일치하는 요소 집합에서 각 요소의 결합된 텍스트 콘텐츠를 가져오거나 일치하는 요소의 텍스트 콘텐츠를 설정합니다.[출처:https://api.jquery.com/] 
- 해당 요소의 텍스트 콘텐츠를 가져오거나 세팅할 때 사용한다.

<br>
<br>

## __html()__

- 일치하는 요소 집합에서 첫 번째 요소의 HTML 콘텐츠를 가져오거나 일치하는 모든 요소의 HTML 콘텐츠를 설정합니다. [출처:https://api.jquery.com/] 
- 해당 요소의 html 콘텐츠를 가져오거나 세팅할 때 사용한다.
<br>
<br>


## 예시 

````javascript
<span id="test">
    <div id="divSample">안녕하세요. 
        <input id ="inputSample" value="cool" />입니다.
    </div>
</span>
````

1. $('#inputSample').val() 일 경우, 출력 결과
   > cool 

2. $('#test').text() 일 경우, 출력 결과
   > 안녕하세요. 입니다.

3. $('#test').html() 일 경우, 출력 결과 
   >````javascript
   ><div id="divSample">안녕하세요. <input id="inputSample" value="cool">입니다.</div>
   >````