float 은 7자리, double 15자리 정도의 정밀도를 가집니다. 그래서 상대적으로 낮은 정밀도의 실수인 경우 float 을, 높은 정밀도의 실수인 경우 double 의 사용을 권장드리는데요. 다만 7자리 미만의 소수점이라기보다는 이렇게 이해하시는 것이 좋습니다.

1234567 이라는 7자리의 수가 있습니다. 이 때 소수점을 어디에 찍느냐에 따라 다음과 같이 수가 달라질 수 있는데요. 이를 10 의 몇 승으로 곱하는 형태로 표현을 하면 우측의 값과 같습니다.

1) 1234567.0 = 1234567 x 10의 0승2) 12345.67 = 1234567 x 10의 -2승  
2) 123.4567 = 1234567 x 10의 -4승  
3) 1.234567 = 1234567 x 10의 -6승  
4) 0.01234567 = 1234567 x 10 의 -8승

즉 1234567 이라는 7자리는 float 을 이용하면 오차 없이 표현이 가능합니다. 아래에 예시 코드를 작성하였으며 주석으로는 실행 결과를 적었습니다.

```csharp
System.out.println(1234567.0f); // 1234567.0
System.out.println(12345.67f); // 12345.67
System.out.println(123.4567f); // 123.4567
System.out.println(1.234567f); // 1.234567
System.out.println(0.01234567f); // 0.01234567
```

이번에는 1234567 앞에 9 를 붙여서 총 8자리의 수인 91234567 을 동일하게 출력해보겠습니다. 이 경우 맨 오른쪽 끝값이 7 임에도 불구하고 주석으로 작성한 실행 결과와 8, 7, 6, 5 로 표현되는 오차가 발생한다는 것을 확인할 수 있습니다.

```csharp
System.out.println(91234567.0f); // 9.1234568E7
System.out.println(912345.67f); // 912345.7
System.out.println(9123.4567f); // 9123.457
System.out.println(91.234567f); // 91.234566
System.out.println(0.91234567f); // 0.91234565
```

이 부분을 감안하셔서 사용하려는 데이터의 정밀도에 따라 float 또는 double 을 선택하시면 되겠습니다.



참고
[인프런 커뮤니티 ## 실수형 float 과 double](https://www.inflearn.com/community/questions/749645/%EC%8B%A4%EC%88%98%ED%98%95-float-%EA%B3%BC-double?srsltid=AfmBOorTNtUXTdgFRHDAQs2rRGZiZCleBpAIBhRnvAZlURRQ3sy6cgPM)


