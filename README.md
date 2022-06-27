# !규칙 

![tetet](https://user-images.githubusercontent.com/69393030/175870821-5d1c11f6-5407-4f8c-ac0a-e718ffb1601a.png)
![tessssss](https://user-images.githubusercontent.com/69393030/175870917-6c98c78f-668a-4d77-966f-5fb7054ca753.png)

주관적인 리팩토링 ( 진짜 주관적임 다맞는건 아님 )

개발하다보니 뭔가 이렇게하면 좀더 좋지않을까? 라는생각임 오조오억번 이야기하지만 "주관적" 이라고 

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


# 어떤값을 불러올때 반복적으로 메서드를 호출하지말자
이게 뭔말이냐면 
내가 현업에서 스프링 시큐리티로 로그인한 유저 권한을 가져와서 , 권한에 맞게 메뉴를 보여줘야하는게 있는데 예를들어서 이런 코드가 있다고 가정하자

```java
public class ReturnMenu {

	public String returnMenuList(List<MenuVO> menuList) throws JsonProcessingException {
		SecurityAuth security = new SecurityAuth();
		ObjectMapper mapper = new ObjectMapper();
		String resultJsonString = "";
		if(menuList != null) {				
			if(security.getAuth().equals("ROLE_ADMIN")) {
				resultJsonString = mapper.writeValueAsString(menuList);
				
			}else if(security.getAuth().equals("ROLE_USER")) {
				menuList.stream().filter(item->!item.getMenuDc().equals("환경설정")).collect(Collectors.toList());
				resultJsonString = mapper.writeValueAsString(menuList);
			}else if(security.getAuth().equals("ROLE_ANONYMOUS")) {
				return null;
			}
		}
		return resultJsonString;
	}
}

```
지금보면 security.getAuth() 라는 함수는 로그인한 사용자의 권한을 가져오는 메서드이다. 
if , else if  로직 타면서 getAuth() 라는 메서드 호출을 총 3번한다 저렇게 하는 거보다는 

```java

public class ReturnMenu {

	public String returnMenuList(List<MenuVO> menuList) throws JsonProcessingException {
		SecurityAuth security = new SecurityAuth();
		ObjectMapper mapper = new ObjectMapper();
		String resultJsonString = "";
		String userAuth = security.getAuth();
		if(menuList != null) {				
			if(userAuth.equals("ROLE_ADMIN")) {
				resultJsonString = mapper.writeValueAsString(menuList);
				
			}else if(userAuth.equals("ROLE_USER")) {
				menuList.stream().filter(item->!item.getMenuDc().equals("환경설정")).collect(Collectors.toList());
				resultJsonString = mapper.writeValueAsString(menuList);
			}else if(userAuth.equals("ROLE_ANONYMOUS")) {
				return null;
			}
		}
		return resultJsonString;
	}
}


```

이렇게 유저 권한을 미리 가져와서 , 변수에 담고 ,비교를 하는것이 메서드 호출 3번하는거보다 메서드 1번 호출 하는게 더 빠를거같다..(아님말고..)


# 주제와 관련된 클래스 , 그에 해당하는 메서드만을 선언하자 - Single-responsibility principle

이건또 뭔소리냐?

```java
class ReturnMenu {

public String returnMenuList(List<MenuVO> menuList) throws JsonProcessingException {
		SecurityAuth security = new SecurityAuth();
		ObjectMapper mapper = new ObjectMapper();
		String resultJsonString = "";
		String userAuth = security.getAuth();
		if(menuList != null) {				
			if(userAuth.equals("ROLE_ADMIN")) {
				resultJsonString = mapper.writeValueAsString(menuList);
				
			}else if(userAuth.equals("ROLE_USER")) {
				menuList.stream().filter(item->!item.getMenuDc().equals("환경설정")).collect(Collectors.toList());
				resultJsonString = mapper.writeValueAsString(menuList);
			}else if(userAuth.equals("ROLE_ANONYMOUS")) {
				return null;
			}
		}
		return resultJsonString;
	}
	
// 이거 금지 
public String getAuth() {
		String userAuth = "";
		
		SecurityContext context = SecurityContextHolder.getContext();
		Authentication authentication = context.getAuthentication();
		Collection<? extends GrantedAuthority> authorities = authentication.getAuthorities();
		Iterator<? extends GrantedAuthority> iter = authorities.iterator();
		
		while (iter.hasNext()) {
		    GrantedAuthority auth = iter.next();
		    userAuth = auth.getAuthority();
		}
		return userAuth;
	}

}



```
보이는가 ? ReturnMenu 라는 유틸 클래스에 메뉴 리턴해주는 메서드만 있어야할거같은데 왜 권한 을 가져오는 메서드도 같이 있을까? 
이렇게 하는것보다는 차라리 하나의 주제를 가진 클래스가 그 하나의 주제를 바탕으로 동작하는 메서드들을 선언하는게 맞지 않냐?
라는게 본인 생각이다.

이는 마치 하나의 함수에서는 하나의 기능이 작동해야하는원리와 같다 보면된다.

``` java 
// 빙고 
public List<?> getUser(){
return queryBuilder.findAll();
}

// 이거 안됌 
public List<?> getUser(){
	List<?> userList = queryBuilder.findAll();
	List<?> fileList = queryBuilder.findAll();

	HashMap<String , Object> resultMap = new HashMap<String , Object>();
	resultMap.put("userList" ,userList);
	resultMap.put("fileList" ,fileList);
return resultMap;
}

```
하나의 메서드에서는 하나의 기능만 하도록한다. 

객체 지향 프로그래밍에서 단일 책임 원칙이란 모든 클래스는 하나의 책임만 가지며, 클래스는 그 책임을 완전히 캡슐화해야 함을 일컫는다. 클래스가 제공하는 모든 기능은 이 책임과 주의 깊게 부합해야 한다.
