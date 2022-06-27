# Refactoring
주관적인 리팩토링

# 스프링에서 의존성 주입을 할시 , 생성자 를 이용한 의존성 주입을하자

내생각인데 이렇게 하는거보다 
```java
public class test {
@Autowired
private Service1 service1;

@Autowired
private Service2 service2;

@Autowired
private Service3 service3;

@Autowired
private Service4 service4;

@Autowired
private Service5 service5;


}


```
이게 나을거같다. 물론 공식에서도 이걸 권장하더라 
```java
public class test {
  
  private Service1 service1;
  private Service2 service2;
  private Service3 service3;
  private Service4 service4;
  private Service5 service5;
  
  @Autowired
  public test(Service1 service1 , Service2 service2 , Servicec3 service3 , Service4 service4 , Service5 service5){
    this.service1 = service1;
    this.service2 = service2;
    this.service3 = service3;
    this.service4 = service4;
    this.service5 = service5;
    
  }

}


```
