controller에서 dto 변환 찬성하는 입장  
- dto는 controller용 dto, service용 dto를 나누지 않는 이상 일반적으로 controller dto만 사용하게됨.
- 그래서 dto가 바뀌면 컨트롤러만 변경에 영향을 받는 것이 타당하다고 생각함. (안그러면 dto 변경시 컨트롤러, 서비스 다 바꿔야할 수도 있음.)

service에서 dto 변환 찬성하는 입장  
- service에서 domain을 반환하고 controller에서 dto를 만들게 되면 controller에서 domain을 알게 됨.
- controller에서 domain을 수정할 일이 발생할 수 있고, domain 변경시 변경 포인트가 service + controller까지 전파될 여지가 있음.


저는 도메인이 변경될 가능성보다는 dto 변경될 가능성이 큰 것도 있고, controller dto/service dto를 구분하는 조직도 있음




참고
[Controller, Service 레이어에서 DTO의 사용](https://shyun00.tistory.com/214)  
[Controller와 Service 레이어 간 요청 & 응답 Dto 사용에 관하여](https://ksh-coding.tistory.com/102)