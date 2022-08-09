* [#1 도메인 모델 시작](#1-도메인-모델-시작)
   * [도메인 모델](#도메인-모델)
   * [도메인 모델 패턴](#도메인-모델-패턴)
   * [도메인 모델 도출](#도메인-모델-도출)
   * [엔티티와 밸류](#엔티티와-밸류)
      * [엔티티](#엔티티)
      * [엔티티의 식별자 생성](#엔티티의-식별자-생성)
      * [밸류 타입](#밸류-타입)
      * [엔티티 식별자와 밸류 타입](#엔티티-식별자와-밸류-타입)
      * [도메인 모델에 set 넣지 않기](#도메인-모델에-set-넣지-않기)
   * [도메인 용어](#도메인-용어)



# #1 도메인 모델 시작

- 온라인 서점 구현하기
  - 개발자 입장에서 온라인 서점은 구현해야 할 소프트웨어의 대상이 된다.
  - 온라인 서점 소프트웨어는 온라인으로 책을 판매하는 데 필요한 상품조회, 구매, 결제, 배송 추적 등의 기능을 제공해야 한다.
  - 온라인 서점은 소프트웨어로 해결하고자 하는 문제 영역, 즉 도메인에 해당한다.
- 도메인은 여러 하위 도메인으로 구성된다.
  - '온라인 서점' 이라는 큰 도메인 하위에 여러가지 하위 도메인들
    - 혜택: 쿠폰이나 특별 할인과 같은 서비스를 제공한다.
    - 카탈로그: 고객에게 구매할 수 있는 상품 목록을 제공한다.
    - 주문: 고객의 주문을 처리한다.
    - 배송: 고객에게 구매한 상품을 전달하는 일련의 과정을 처리한다.
    - 회원, 리뷰, 결제, 정산 등등 ...
  - 한 하위 도메인은 다른 하위 도메인과 연동하여 완전한 하나의 기능을 제공한다.
    - ex: 고객이 물건을 구매
      - 주문 -> 결제 -> 배송 -> 혜택
- 특정 도메인을 위한 소프트웨어라고 해서 도메인이 제공해야할 모든 기능을 구현하는 것은 아니다.
  - ex: 온라인 쇼핑몰에서 결제모듈은 PG사를 통해서 한다.

## 도메인 모델

- 도메인 모델
  - 도메인 모델은 특정 도메인을 개념적으로 표현한 것이다.
  - 도메인 모델을 사용하면 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는데 도움이 된다.

[![1-1](https://github.com/sup2is/dev-note/raw/master/architecture/images/ddd-start/1-1.jpeg)](https://github.com/sup2is/dev-note/blob/master/architecture/images/ddd-start/1-1.jpeg)

[![1-2](https://github.com/sup2is/dev-note/raw/master/architecture/images/ddd-start/1-2.jpeg)](https://github.com/sup2is/dev-note/blob/master/architecture/images/ddd-start/1-2.jpeg)

- 도메인 모델의 표현 방식은 도메인을 이해하는 데 도움이 된다면 방식이 무엇인지는 중요하지 않다.
- 개념모델과 구현모델
  - 개념 모델: 순수하게 문제를 분석한 결과물
  - 구현 모델: 개념모델을 기반으로 구현한 결과물
  - 처음부터 완벽한 개념 모델을 만들기보다는 전반적인 개요를 알 수 있는 수준으로 개념모델을 작성해야 한다. 개념 모델은 도메인에 대한 전체 윤곽을 이해하는 데 집중하고 구현하는 과정에서 개념 모델을 구현 모델로 점진적으로 발전시켜 나가야한다.
- 하위 도메인과 모델
  - 각 하위 도메인이 다루는 영역은 서로 다르기 때문에 같은 용어라도 하위 도메인마다 의미가 달라질 수 있다.
  - 모델의 각 구성요소는 특정 도메인을 한정할 때 비로소 의미가 완전해지기 때문에 각 하위 도메인마다 별도로 모델을 만들어야 한다.

## 도메인 모델 패턴

- 일반적인 애플리케이션의 아키텍처

[![1-3](https://github.com/sup2is/dev-note/raw/master/architecture/images/ddd-start/1-3.png)](https://github.com/sup2is/dev-note/blob/master/architecture/images/ddd-start/1-3.png)

| 계층           | 설명                                                         |
| -------------- | ------------------------------------------------------------ |
| ui, 또는 표현  | 사용자의 요청을 처리하고 사용자에게 정보를 보여준다. 여가서 사용자는 소프트웨어를 사용하는 사람뿐만 아니라 외부 시스템도 사용자가 될 수 있다. |
| 응용           | 사용자가 요청한 기능을 실행한다. 업무 로직을 직접 구현하지 않으며 도메인 계층을 조합해서 기능을 실행한다. |
| 도메인         | 시스템이 제공할 도메인의 규칙을 구현한다.                    |
| 인프라스트럭처 | 데이터베이스나 메시징 시스템과 같은 외부 시스템과의 연동을 처리한다. |

- 마틴 파울러의 도메인 모델
  - 여기서의 도메인 모델은 아키텍처상의 도메인 계층을 객체 지향 기법으로 구현하는 패턴을 말한다.

> 도메인 모델의 두가지 의미
>
> 1. 도메인 자체를 표현하는 개념적인 모델
> 2. 도메인 계층을 구현할 때 사용하는 객체 모델

- 도메인 계층 구현하기

  - 도메인 계층은 도메인의 핵심 구현을 구현한다.

  - 주문 도메인

    - "출고 전에 배송지를 변경할 수 있다."
    - "주문 취소는 배송 전에만 할 수 있다."

  - 코드로 주문 도메인 표현하기

  - ``` java
    public class Order {
    	private OrderState state;
    	private ShippingInfo shippingInfo;
    
    	public void changeShippingInfo(ShippingInfo newShippingInfo) {
    		if (!state.isShippingChangeable()) {
    			throw new IllegalStateException("can't change shipping in " + state);
    		}
    		this.shippingInfo = newShippingInfo;
    	}
    
    	public void changeShipped() {
    		// 로직 검사
    		this.state = OrderState.SHIPPED;
    	}
    	...
    }
    
    public enum OrderState {
    	PAYMENT_WAITING {
    		public boolean isShippingChangeable() {
    			return true;
    		}
    	},
    	PREPARING {
    		public boolean isShippingChangeable() {
    			return true;
    		}
    	},
    	SHIPPED, DELIVERING, DELIVERY_COMPLETED;
    
    	public boolean isShippingChangeable() {
    		return false;
    	}
    }
    ```

- 중요 업무 규칙 구현하기

  - 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.

## 도메인 모델 도출

- 도메인을 모델링 할 때 기본이 되는 작업
  - 모델을 구성하는 핵심 구성 요소 찾기
  - 규칙 찾기
  - 기능 찾기
- 주문 도메인과 관련된 몇가지 요구사항
  - 최소 한 종류 이상의 상품을 주문해야 한다.
  - 한 상품을 한 개 이상 주문할 수 있다.
  - 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
  - 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.
  - 주문할 때 배송지 정보를 반드시 지정해야 한다.
  - 배송지 정보는 받는 사람 이름, 전화번호, 주소로 구성된다.
  - 출고를 하면 배송지 정보를 변경할 수 없다.
  - 출고 전에 주문을 취소할 수 있다.
  - 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.
- 위 요구사항을 기반으로 도출한 주문의 네가지 기능
  1. 출고 상태로 변경하기
  2. 배송지 정보 변경하기
  3. 주문 취소하기
  4. 결제 완료로 변경하기

``` java
public class Order {
	public void changeShipped() { ... }
	public void changeShippingInfo(ShippingInfo newShipping) { ... }
	public void cancel() { ... }
	public void completePayment() { ... }
}
```

- 다음 요구사항은 주문 항목이 어떤 데이터로 구성되는지 알려준다.
  - 한 상품을 한 개 이상 주문할 수 있다.
  - 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.
- 이를 구현한 OrderLine

``` java
public class OrderLine {
  private Product product; //상품
  private int price;  //가격
  private int quantity;  //수량
  private int amounts; //구매가격
  
  public OrderLine(Prodect product, int price, int quantity) {
    this.product = product;
    this.private = price;
    this.quantity = quantity;
    this.amounts = calculateAounts();
  }
  
  //구매 가격 구하기
  private int calculateAounts() {
    return price * quantity;
  }
  
  public int getAmounts() {  }
}
```

- 다음 요구사항은 Order와 OrderLine의 관계를 알려준다.
  - 최소 한 종류 이상의 상품을 주문해야 한다.
  - 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
- 이를 반영한 Order

``` java
public class Oreder {
  private List<OrderLine> ordrLines;
  private int totalAmounts;
  
  public Order(List<OrederLine> orderLines) {
    setOrderLines(orderLines);
  }
  
  private void setOrderLines(List<OrderLine> orderLines) {
    verifyAtLeastOneOrMoreOrderLines(orderLines);
    this.orderLines = orderLines;
    calculateTotalAmounts();
  }
  
  private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLine) {
    if(orderLines == null || orderLines.isEmpty()) {
      throw new Exception("no OrderLine");
    }
  }
  
  private void calculateTotalAmounts() {
    this.totalAmounts = new Money(orderLines.stream()
      .mapToInt(x -> x.getAmounts().getValue()).sum();
  }
  //....
}
```

- 다음 요구사항은 Order와 ShippingInfo(배송지 정보)의 관계를 알려준다.
  - 주문할 때 배송지 정보를 반드시 지정해야 한다.

``` java
public class ShipingInfo {
	private String receiverName;
	private String receiverPhoneNumber;
	private String shipingAddress1;
	private String shipingAddress2;
	private String shipingZipcode;

  ...
}

public class Order {
	private List<OrderLine> orderLines;
	private int totalAmounts;
	private ShippingInfo shippingInfo;
	
	public Order(List<OrderLine> orderLines, ShippingInfo shippingInfo){
		setOrderLiens(orderLines);
		setShippingInfo(shippingInfo);
	}
	private void setShippingInfo(ShippingInfo shippingInfo){
		if (shippingInfo == null)
			throw new IllegalArgumentException("no shippingInfo");
		this.shippingInfo = shippingInfo;
	}
...
}
```

- 아래 요구사항으로 OrderStatus(주문의 상태)를 도출 할 수 있다.
  - 출고를 하면 배송지 정보를 변경할 수 없다.
  - 출고 전에 주문을 취소할 수 있다.
  - 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.

``` java
public enum OrderState {
	PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED, CANCELED;	
}

public class Order{
	private OrderState state;
	
	public void changeShippingInfo(ShippingInfo newShippingInfo) {
		verifyNotYetShipped();
		setShippingInfo(newShippingInfo);
	}
	public void cancel(){
		verifyNotYetShipped();
		this.state = OrderState.CANCELED;
	}
	private void verifyNotYetShipped(){
		if(state != OrderState.PAYMENT_WAITING && state != OrderState.PREPARING)
			throw new IllegalStateException("already shippped");
	}
	...
}
```

- 문서화하기
  - 문서화를 하는 주된 이유는 지식을 공유하기 위함이다.
  - 코드보다 상위 수준에서 정리한 문서를 참조하는 것이 소프트웨어 전반을 빠르게 이해하는데 도움이 된다.
  - 코드도 역시 문서화의 대상이 된다. 도메인 지식이 잘 묻어나도록 코드를 작성해야 한다.

## 엔티티와 밸류

- 엔티티와 밸류 구분하기
  - 도출한 모델은 크게 엔티티와 밸류로 구분할 수 있다.
  - 엔티티와 밸류를 제대로 구분해야 도메인을 올바르게 설계하고 구현할 수 있기 때문에 이 둘의 차이를 명확하게 이해하는 것은 도메인을 구현하는데 있어 중요하다.

### 엔티티

- 엔티티와 식별자
  - 엔티티의 가장 큰 특징은 식별자를 갖는다는 것이다.
  - 식별자는 엔티티 객체마다 고유하고 각 엔티티는 서로 다른 식별자를 가진다.
    - ex: 주문 도메인에서 고유한 주문번호
  - 엔티티를 생성하고 엔티티의 속성을 바꾸고 엔티티를 삭제할 때까지 식별자는 유지된다.
  - 엔티티의 식별자는 고유하기 때문에 같은 식별자를 갖는다면 두 엔티티는 같다고 판단할 수 있다.

### 엔티티의 식별자 생성

- 엔티티의 식별자를 생성하는 시점은 도메인의 특징과 사용하는 기술에 따라 달라진다.

- 식별자를 지정하는 방법

  - 특정 규칙에 따라 생성

    - ex: 현재시간을 조합 20230529094644, 20220702094644

  - uuid 사용

    - ``` java
      UUID uuid = UUID.randomUUID();
      // 615fsdf34-c342-5scd-d33d-123145sadfa 와 같은 문자열
      ```

  - 값을 직접 입력

    - ex: 회원의 id 또는 email

  - 시퀀스나 db의 자동 증가 컬럼 사용, 일련번호

    - ex: auto-increment 칼럼

### 밸류 타입

- 밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다.
- ShipingInfo 클래스를 밸류 타입으로 리팩토링하기

``` java
public class ShipingInfo {
	private String receiverName; // Receiver
	private String receiverPhoneNumber; // Receiver
	private String shipingAddress1; // Address
	private String shipingAddress2; // Address
	private String shipingZipcode;  // Address

  ...
}	
```

- 받는사람(Receiver) 를 밸류타입으로 구현하기

``` java
public class Receiver {
	private String name;
	private String phoneNumber;
	
	public Receiver(String name, String phoneNumber){
		this.name = name;
		this.phoneNumber = phoneNumber;
	}

	public String getName(){
		return name;
	}
	public String getPhoneNumber(){
		return phoneNumber;
	}
}
```

- 주소(Address)를 밸류타입으로 구현하기

``` java
public class Address{
	private String address1;
	private String address2;
	private String zipcode;
	//생성자, getter
}
```

- ShipingInfo 클래스에 구현한 Receiver와 Address를 적용하기

``` java
public class ShipingInfo {
  private Receiver receiver;
  private Address address;
  
  ...
}
```

- 밸류타입이 꼭 두 개 이상의 데이터를 가져야하는 것은 아니다. 의미를 명확하게 표현하기 위해 밸류 타입을 사용하는 경우도 있다.

``` java
public class Money{
	private int value;

	public Money(int value){
		this.money = money;
	}
	//... getter
	
	//새로운 기능 추가 가능
	public Money add(Money money){
		return new Money(this.value + money.value);
	}
	public Money multiply(int multiplier) {
		return new Money(this.value * multiplier);
	}
}
public class OrderLine {
	private Product product;
	private int price;
	private int quantity;
	private int amounts;
...
}

// price와 amounts에 Money 클래스 적용하기

public class OrderLine{
	private Product product;
	private Money price;
	private int quantity;
	private Money amounts;
...
}
```

- 이처럼 밸류 타입은 코드의 이미를 더 잘 이해할 수 있도록 도와준다.
- 밸류타입과 불변
  - 밸류 객체의 데이터를 변경할 때는 기존 데이터를 변경하기보다는 변경한 데이터를 갖는 새로운 밸류 객체를 생성하는 방식을 선호한다.
  - 데이터의 변경 기능을 제공하지 않는 타입을 불변이라고 표현한다.
  - 불변 타입을 사용하면 보다 안전한 코드를 작성할 수 있다.
  - 밸류 타입을 사용해서 어떠한 문제가 있는지 분석하고 문제를 해결하는 방법보다 그냥 '밸류타입은 불변으로 작성' 이라는 공식을 갖고 있으면 여러가지 문제를 고민하지 않고 해결할 수 있다. (개인적인 생각)
- 밸류타입 객체가 같은지 비교할 때는 모든 속성이 같은지 비교해야 한다.

### 엔티티 식별자와 밸류 타입

- 식별자는 단순한 문자열이 아니라 도메인에서 특별한 의미를 지니는 경우가 많기 때문에 식별자를 위한 밸류 타입을 사용해서 의미가 잘 들어나도록 할 수 있다.

``` java
public class Order{
	//private String id;
	private OrderNo id;
	...
}
```

### 도메인 모델에 set 넣지 않기

- 도메인 모델에 무작정 get/set을 추가하는 습관
  - 이런 습관은 좋지 않다.
  - 특히 set 메서드는 도메인의 핵심 개념이나 의도를 코드에서 사라지게 한다.
- setX()의 경우 도메인 지식을 부여하기 힘들기 때문에 적절한 이름을 선정하는것이 중요하다.
  - setShippingInfo() -> changeShippingInfo()
  - setOrderStatus() -> completePayment()
  - 메서드 명에 도메인 지식을 녹이는게 중요하다.
  - 만약 set 메서드가 private 으로 외부에 공개되지 않는다면 set이라는 이름을 사용해도 괜찮다.
- 밸류 타입은 불변이어야 하기 때문에 set과 같은 변경 메서드를 구현하지 않아야 한다.

## 도메인 용어

- 코드를 작성할 때 도메인에서 사용하는 용어는 매우 중요하다.
  - https://martinfowler.com/bliki/UbiquitousLanguage.html

``` java
public enum OrderState {
	PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED, CANCELED;	
}
```

- 도메인 용어의 장점
  - 별다른 해석을 위한 이해 과정이 줄어든다.
  - 가독성이 올라가서 코드를 분석하고 이해하는 시간을 절약한다.
  - 도메인 용어를 사용해서 도메인 규칙을 코드로 작성하게 되므로 버그도 줄어든다.
- 알맞은 영어 단어 찾기
  - 도메인 용어를 정하는데 높은 허들은 바로 코드가 '영어' 라는 점이다.
  - 알맞은 영어 단어를 찾는 것은 쉽지 않은 일이지만 시간을 들여 찾는 노력을 해야 한다.