# 22-08-28

검색기능


//참고 api-> Elasticsearch에 형태소 분석기 API 를 사용하면 다른 단어와 섞여 있거나 띄어쓰기 오류 등으로 인한 검색 오류를 방지해 줄 수 있다
//단,현제 구상 사이트와 스펙과 맞지 않고, 혼자 처리할 사안이 아님. 팀원과 고려해 볼 사항이고 스펙을 검색 기능 때문에 무리하게 올릴 필요가 없다고 생각함.

H2 DB 테이블 구조 

띄어쓰기로 인한 오류를 없애기 위해선 
PR_TILETLE의 띄어쓰기를 없앤 텍스트와 비교해주기 위해 맨 테이블 끝에 *column PR_TILTE_ALLAS*를 추가 해 보는 것을 고려할 것
Java String API replaceAll("\s", "") -> 정규식 표현 '\s' - > whitespace 1개, "" 제거 = 스페이스를 제거 하고 저장함.



mybatis 자료
<bind> 테그로 WHERE ... LIKE '%...%' %검색어%에서 특수문자와 결합할때 JDBC에서 생기는 인식오류를 escape character(\ -> backslash)를 사용하거나 <bind> 태그를 사용해 
String 해준 변수를 parameter로 작성해주면 되는데 
  아래는 후자의 방법을 사용.

    <select id="search" resultType="Product">
        <bind name="searchWordConcate" value="'%'+searchWord+'%'"></bind>
        <bind name="searchWordNoSpaceConcate" value="'%'+searchWordNoSpace+'%'"></bind>
        SELECT *
        FROM PRODUCT
        WHERE PR_TITLE LIKE #{searchWordConcate} OR PR_TITLE LIKE #{searchWordNoSpaceConcate}
        OR
        PR_TITLE_ALIAS LIKE #{searchWordConcate} OR PR_TITLE_ALIAS LIKE #{searchWordNoSpaceConcate};
    </select>

//우리 조는 mySQL을 사용하기 때문에 mySQL select문으로도 구현이 가능한지 확인해 볼 것.
  
 
  
  검색기능 코드 ProductInfoController.java(이름은 다시 한번 생각해 볼 필요성이 있음) 코드
  
  **view로 부터 전달 받기 위해서 @RequstParam을 사용했지만 굳이 Param을 사용할 필요성이 있는지 한 번 더 조사해 볼 것
  아래 코드는 HttpRequest를 Controller로 전송해주는 과정에서 <input> 태그 name이 아노테이션 value와 일치한다면 parameter로 전달해줌. (<input name="searchWord"...>)
  searchWordNoSpace는 사용자가 검색한 텍스트의 띄어쓰기를 없앤 값을 Product DB의 값과 비교해 검색 정확도를 높이기 위해서 만들어줬다고 함.
  eplaceAll() 메서드의 parameter "\(:backslash)s"는 정규식 표현(RegExp)에서 띄어쓰기를 뜻하며, 
  정규식 표현은 문자열에 특수기호나 패턴을 찾거나 입력 하기위해 지정된 표현식(Expression)
  
      @RequestMapping(value = "/search/result", method = RequestMethod.GET) 
    public String searchGet(@RequestParam("searchWord") String searchWord, Model model) {
        String searchWordNoSpace = searchWord.replaceAll("\s", "");
        
        if (productService.searchCount(searchWord, searchWordNoSpace) == 0) {
            model.addAttribute("searchWord",searchWord);
            return "product/searchResultZero";
        } else {
            List<Product> productList = productService.search(searchWord, searchWordNoSpace);
            model.addAttribute("searchProductList", productList);
            return "product/searchResult";
        }
    }







아래는 프론트 부분의 코드(데이터 흐름을 한 번 생각해 볼 것)





/////////////////////////Logo and  Searchbar JSP///////////////////////////////////////////////////
            <div class="logo-search-wrap">
                <h1 class="logo-title">
                <a href="<c:url value="/home"/>"> 
                    <img id="logo-img" alt="logo" src="/resources/img/Salle.png">
                </a>
                </h1>
                
                <form action="/search/result" method="GET">
                <input type="text" id="searchWord" name="searchWord" placeholder="검색어를 입력하세요"
                    maxlength="50" size="60">
                <!-- <input id="searchButton" type="image" src="" style="width:25px; height:25px;" alt="Submit Form"/> -->
                <button class="searchButton">
                    <img class="searchButtonImg" alt="Submit Form" src="/resources/img/210106_Salle_searchicon.png"/>
                </button>
                </form>
            </div>
/////////////////////////Javascript///////////////////////////////////////////////////

        //activate Enterkey for Search
            var searchWord = document.getElementById('searchWord');
            var sarchButton = document.getElementsByClassName('searchButton');
            searchWord.addEventListener('keyup', function(event) {
                
                if (event.keyCode === null) {
                    event.preventDefault();
                }
                
                if (event.keyCode === 13) {
                    
                    window.location.href = searchButton.href;
                }
            });
