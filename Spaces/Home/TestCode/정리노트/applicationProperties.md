# 테스트 코드 수행 시 application.properties

테스트 코드 수행 시 src/test/resources/application.properties파일이 없다면

main의 application.properties을 그대로 갖고 온다.

**자동으로 가져오는 옵션의 범위는 application.properties까지다.**

**application-~~.properties는 갖고오지 않는다.**

application.properties에 가짜로 설정값을 넣어 주거나 test/하위에 만들어줘야 한다.