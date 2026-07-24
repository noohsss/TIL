# 도메인 모델
## **1.1 도메인이란?**

(온라인 서점 도메인 예시)

!스크린샷 2023-10-28 오후 2.07.01.png

- 도메인이란 소프트웨어로 해결하고자 하는 문제 영역을 의미한다.
- 하나의 도메인으로 끝이 아니라 그 밑으로 여러 하위 도메인으로 나뉠 수 있다.
- 도메인이 제공해야 할 모든 기능을 직접 구현하는 것이 아닌, 외부 시스템(결제, 배송)을 연동하는 경우도 있다.
- 도메인마다 고정된 하위 도메인이 존재하는 것은 아니다.

## 1.2 도메인 전문가와 개발자 간 지식 공유

- 각 영역에 있는 전문가는 해당 도메인에 대한 지식과 경험을 바탕으로 원하는 기능 개발을 요구한다.
- 개발자는 이런 요구사항을 분석하고 설계하여 코드를 작성하며 테스트하고 배포한다.
- 코딩보다는 요구사항을 올바르게 이해하는 것이 중요.
    - 개발자와 전문가가 직접 대화하는 것이 좋다.(전달자가 많게 되면 정보의 왜곡, 손실이 발생하기 때문)
- 개발자도 해당 도메인의 지식을 어느정도 갖춰야한다.
- “Garbage in, Garbage out”
    - 소프트웨어 분야에서 유명한 문장으로 ‘잘못된 값이 들어가면 잘못된 결과가 나온다’라는 의미를 갖는다.
    

## 1.3 도메인 모델

(객체 기반 주문 도메인 모델)

!스크린샷 2023-10-28 오후 2.30.15.png

- 도메인 모델에는 다양한 정의가 존재, 도메인 모델은 특정 도메인을 개념적으로 표현한 것이다.
- 주문 모델을 객체 모델로 구성하면 위 그림과 같다.
    - 주문(Order)은 주문번호(orderNumber)와 지불할 총금액(totalAmounts)이 있다.
    - 배송정보(ShippingInfo)를 변경(changeShipping)할 수 있다.
    - 주문을 취소(cancel) 할 수 있다.
- 이처럼 도메인 모델을 사용하면 여러 관계자들이 동일한 모습으로 도메인을 이해하고 도메인 지식을 공유하는데 도움이 된다.
- 도메인 모델을 객체로만 모델링 할 수 있는 것은 아니다.

(상태 다이어그램을 이용한 주문 상태 모델링)

!스크린샷 2023-10-28 오후 2.29.21.png

- 도메인 모델을 표현할 때 클래스 다이어그램이나 상태 다이어그램과 같은 UML 표기법만 사용해야 하는것은 아니다.
    - 관계가 중요한 도메인이라면 그래프를 이용해서 도메인을 모델링 할 수 있다.
    - 계산 규칙이 중요하다면 수학 공식을 활용해서 도메인 모델을 만들 수 있다.
    - 도메인을 이해하는 데 도움이 된다면 표현 방식이 중요하지는 않다.
- 도메인 모델은 기본적으로 도메인 자체를 이해하기 위한 개념 모델이다.
- 하위 도메인과 모델
    - 구현 기술에 맞는 구현 모델이 따로 필요하다.
        - 개념 모델과 구현 모델은 서로 다른 것이지만 구현 모델이 개념 모델을 최대한 따르도록 할 수 있다.
    - 도메인에 따라 용어 의미가 결정되므로 여러 하위 도메인을 하나의 다이어그램에 모델링 하면 안된다.
    - 도메인의 상품을 제대로 이해하는 데 방해가 되기 때문이다.

## 1.4 도메인의 모델 패턴

(아키텍처 구성)

!스크린샷 2023-10-28 오후 3.05.35.png

!스크린샷 2023-10-28 오후 3.05.51.png

- 도메인 모델은 아키텍처 상의 도메인 계층을 객체 지향 기법으로 구현하는 패턴을 말한다.

### !! 도메인 계층

- 도메인의 핵심 규칙을 구현
- 주문 도메인의 경우 ‘출고 전에 배송지를 변경할 수 있다’ 라는 규칙과 ‘주문 취소는 배송 전에만 할 수 있다’라는 규칙을 구현한 코드가 도메인 계층에 위치하게 된다.

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippinginfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippinginfo = newShippingInfo;
    }
    
    // ...
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

- 위 코드는 주문 도메인의 일부 기능을 도메인 모델 패턴으로 구현한 코드이다.
- 주문 상태를 표현하는 OrderState 는 배송지를 변경할 수 있는지를 검사할 수 있는 IsShippingChangeable() 메서드를 제공한다.
- 주문 대기중 (PAYMENT_WATING) 상태와 상품 준비 중(PREPARING) 상태의 isShippingChangeable() 메서드는 true를 리턴한다.
- OrderState는 주문 대기 중이거나 상품 준비 중에는 배송지를 변경할 수 있다는 도메인 규칙이다.

```java
public class Order {
    private OrderState state;
    private ShippingInfo shippinginfo;

    public void changeShippingInfo(ShippingInfo newShippingInfo) {
        if (!state.isShippingChangeable()) {
            throw new IllegalStateException("can't change shipping in " + state);
        }
        this.shippinginfo = newShippingInfo;
    }
    
     public boolean isShippingChangeable() {
            return state == OrderState.PAYMENT_EAITING ||
								statae == OrderState.PREPARING;
        }
			...
}

public enum OrderState {
    PAYMENT_WAITING, PREPARING, SHIPPED, DELIVERING, DELIVERY_COMPLETED;
}
```

- 배송지 변경이 가능한지를 판단할 규칙이 주문 상태와 다른 정보를 함께 사용한다면 OrderState만으로는 배송지 변경 가능 여부를 판단할 수 없으므로 위와 같이 수정해야 한다.
- 핵심 규칙을 구현한 코드는 도메인 모델에만 위치하기 때문에 규칙이 바뀌거나 규칙을 확장해야 할 때 다른 코드에 영향을 덜 주고 변경 내역을 모델에 반영할 수 있게 된다.

### **!! 개념 모델과 구현 모델**

- 개념모델 : 순수하게 문제를 분석한 결과물
- 데이터 베이스, 트랜잭션 처리, 성능, 구현 기술과 같은 것을 고려하지 있지 않다.
- 때문에 개념 모델을 코드에 그대로 사용할 수 없다.(구현 모델로 전환 필요)
- 처음부터 완벽한 개념 모델 보다는 프로젝트 초기에 개요 수준의 개념 모델로 도메인에 대한 전체 윤곽을 이해하고, 구현하는 과정에서 구현 모델로 발전시켜야 한다.

## 1.5 도메인 모델 도출

- 도메인을 이해하고 이를 바탕으로 도메인 모델을 초안으로 만들어야 코드를 작성할 수 있다.
- 구현을 시작하려면 도메인에 대한 초기 모델이 필요하다.
- 도메인 모델 도출 예제
    - 최소 한 종류 이상의 상품을 주문해야 한다.
    - 한 상품을 한 개 이상 주문할 수 있다.
    - 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.
    - 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.
    - 주문할 때 배송지 정보를 반드시 지정해야 한다.
    - 배송지 정보는 받는 사람 이름, 전화번호, 주소로 구성된다.
    - 출고를 하면 배송지를 변경할 수 있다.
    - 출고 전에 주문을 취소할 수 있다.
    - 고객이 결제를 완료하기 전에는 상품을 준비하지 않는다.
    
- 다음 요구사항은 주문 항목이 어떤 데이터로 구성되는지 알려준다.
    - 한 상품을 한 개 이상 주문할 수 있다.
    - 각 상품의 구매 가격 합은 상품 가격에 구매 개수를 곱한 값이다.

```java
public class OrderLine {
	private Product product;
	private int price;
	private int quantity;
	private int amounts;
	
	public OrderLine(Product product, int price, int quantity) {
		this.product = product;
		this.price = price;
		this.quantity = quantity;
		this.amounts = calculateAmounts();
	}

	private int calculateAmounts() {
		return price * quantity;
	}

	public int getAmounts() {...}
}
```

- 두 요구사항에 따르면 주문 항목을 표현하는 OrderLine은 적어도 주문할 상품, 상품의 가격, 구매 개수를 포함해야 한다.
    - 주문할 상품 : product 필드
    - 상품의 가격 : price 필드
    - 구매 개수 : quantity 필드
- 구매 항목의 구매 가격도 제공해야 한다.
    - calculateAmounts()

- 다음 요구사항은 Order와 OrderLine과의 관계를 알려준다
    - 최소 한 종류 이상의 상품을 주문해야 한다.
    - 총 주문 금액은 각 상품의 구매 가격 합을 모두 더한 금액이다.

```java
public class Order {
	private List<OrderLine> orderLines;
	private Money totalAmounts;

	public Order(List<OrderLine> orderLines) {
		setOrderLines(orderLines);
	}

	private void setOrderLines(List<OrderLine> orderLines) {
		verifyAtLeastOneOrMoreOrderLines(orderLines);
		this.orderLines = orderLines;
		calculateTotalAmounts();
	}

	private void verifyAtLeastOneOrMoreOrderLines(List<OrderLine> orderLines) {
		if(orderLines == null || orderLines.isEmpty()) {
			throw new IllegalArgumentException("no OrderLine");
		}	
	}

	private void calculateTotalAmounts() {
		int sum = orderLines.stream()
												.mapToInt(x -> x.getAmounts())
												.sum();
		this.totalAmounts = new Money(sum);
	}

	..// 다른 메서드
}
```

- Order는 한 개 이상의 OrderLine을 가질 수 있으므로 Order를 생성할 때 OrderLine 목록을 List로 전달한다.
- 생성자에서 호출하는 setOrderLines() 메서드는 요구사항에 정의한 제약 조건을 검사한다.

(내용이 많아 다른 예제는 생략)

## 1.6 엔티티와 밸류

!스크린샷 2023-10-28 오후 3.50.00.png

- 도출한 모델은 크게 엔티티와(entity) 밸류(value)로 구분할 수 있다.
- 엔티티와 밸류를 제대로 구분해야 도메인을 올바르게 설계하고 구현할 수 잇다.
- 이 둘이 차이를 명확하게 이해하는 것은 도메인 구현에서 중요하다.

### 1.6.1 엔티티

!스크린샷 2023-10-28 오후 3.54.30.png

- 엔티티의 가장 큰 특징은 식별자를 가진다는 것이다.
    - 식별자는 엔티티 객체마다 고유해서 각 엔티티는 서로 다른 식별자를 가진다.
    - 엔티티의 식별자는 바뀌지 않는다.
- 엔티티의 식별자는 바뀌지 않고 고유하다.
    - 두 엔티티 객체의 식별자가 같으면 두 엔티티는 같다고 판단할 수 있다.
    - 식별자를 이용해서 equals() 메서드와 hashCode() 메서드를 구현할 수 있다.
    

### 1.6.2 엔티티의 식별자 생성

- 엔티티의 식별자를 생성하는 시점은 도메인의 특징과 사용하는 기술에 따라 달라진다.
    - 특정 규칙에 따라 생성
    - UUID나 Nano ID와 같은 고유 식별자 생성기 사용
    - 값을 직접 입력
    - 일련번호 사용(시퀀스나 DB의 자동 증가 컬럼 사용)
    

```java
// 엔티티를 생성하기 전에 식별자 생성
String orderNumber = orderRepository.generateOrderNumber();

Order order = new Order(orderNumber, ....);
orderRepository.save(order);
```

- 자동 증가 컬럼을 제외한 다른 방식은 식별자를 먼저 만들고 엔티티 객체를 생성 할 때 식별자를 전달한다.

```java
Article article = new Article(author, title, ...);
articleRepository.save(article); // DB에 저장한 뒤 구한 식별자를 엔티티에 반영
Long savedArticleId = article.getId(); // DB에 저장한 후 식별자 참조 가능
```

- 자동 증가 컬럼은 DB 테이블에 데이터를 삽입해야 비로소 값을 알 수 있으므로 테이블에 데이터를 추가하기 전까지는 식별자를 알 수 없다.
- 따라서 엔티티 객체를 생성할 때 식별자를 전달 할 수 없다.

### 1.6.3 밸류 타입

!스크린샷 2023-10-28 오후 4.08.47.png

- 위 그림에서 receiverName 필드와 receiverPhoneNumber 필드는 서로 다른 두 데이터를 담고 있지만 두 필드는 개념적으로 받는 사람을 의미한다.
- shippingAddress1, shippingAddress2, shippingZipcode 필드는 주소라는 하나의 개념을 표현한다.
- 밸류 타입은 개념적으로 완전한 하나를 표현할 때 사용한다.

(위 그림을 밸류 타입을 사용하여 수정한 코드)

```java
public class ShippingInfo {
	private Recevier receiver;
	private Address address;
}

public class Receiver {
    private String name;
    private String phoneNumber;

    public Receiver(String name, String phoneNumber) {
        this.name = name;
        this.phoneNumber = phoneNumber;
    }

    public String getName() {
        return name;
    }

    public String getPhoneNumber() {
        return phoneNumber;
    }
}

public class Address {
    private String address1;
    private String address2;
    private String zipcode;

    public Address(String address1, String address2, String zipcode) {
        this.address1 = address1;
        this.address2 = address2;
        this.zipcode = zipcode;
    }
}
```

- 도메인 개념과 관련된 데이터를 모아서 하나의 클래스 형태로 만든다.
- 밸류 타입을 사용함으로써 가독성이 향상된다.

### 1.6.4 엔티티 식별자와 밸류 타입

- 엔티티 식별자는 단순한 문자열로 구성된 경우가 많지만 도메인에서 특별한 의미를 지니는 경우가 많기 때문에 식별자를 위한 밸류 타입을 사용해서 의미가 잘 드러나도록 할 수 있다.

### 1.6.5 도메인 모델에 set 메서드 넣지 않기

- get/set 메서드를 습관적으로 만드는 경우가 있지만, 좋지 않은 버릇이다.
- 특히 set 메서드는 필드값만 변경하고 끝나기 때문에 상태 변경과 관련된 도메인 지식이 코드에서 사라지게 된다.
- 예를들어 배송지 정보를 새로 변경한다는 의미의 changeShippingInfo() 메서드가 있다면, setShippingInfo() 메서드는 단순히 배송지 값을 설정한다는 것을 의미한다.
- set 메서드의 또 다른 문제는 도메인 객체를 생성할 때 온전하지 않은 상태가 될 수 있다.
    - 핵심 프로퍼티가 누락된 도메인 객체가 생성되어 문제를 일으킬 수 있다.
    - 위 상황을 방지하려면 생성 시점에 필요한 데이터를 모두 받아야 한다.
    - 또한 생성자로 필요 데이터들을 모두 받으므로 호출하는 시점에서 필요한 데이터를 검증할 수 있다.
- 불변 밸류 타입을 사용하면 자연스럽게 밸류 타입에는 set 메서드를 구현하지 않는다.
    - set 메서드를 구현해야 할 특별한 이유가 없다면 불변 타입의 장점을 살릴 수 있도록 불변으로 구현한다.

### !! DTO의 get/set 메서드

- Data Transfer Object의 약자로 프레젠테이션 계층과 도메인 계층이 데이터를 주고받을 때 사용하는 일종의 구조체이다.
- get/set 메서드를 제공해도 도메인 객체의 데이터 일관성에 영향을 줄 가능성이 높지 않다.
- 요즘은 set 메서드가 아닌 private 필드에 직접 값을 할당할 수 있는 기능을 제공하는 프레임워크나 개발 도구가 존재한다.
- set 메서드를 만드는 대신 해당 기능을 최대한 활용하여 불변의 장점을 DTO까지 확장 시키자.

## 1.7 도메인 용어와 유비쿼터스 언어

- 코드를 작성할 때 도메인에서 사용하는 용어는 매우 중요.
- 도메인 용어를 코드에 반영하지 않으면 그 코드는 개발자에게 코드의 의미를 해석해야 하는 부담을 준다.
- 클래스, 필드, 메서드 등의 이름을 작성할 때 도메인과 관련된 용어를 사용해 도메인 규칙을 코드로 작성하게 되면 의미를 해석하는 과정에서 발생하는 문제도 줄어든다.
- 알맞은 영단어를 찾는 것은 쉽지 않은 일이지만 시간을 들여 찾는 노력을 해야 한다.

### !! 유비쿼터스 언어

- 전문가, 관계자, 개발자가 도메인과 관련된 공통의 언어를 만들고 이를 대화, 문서, 도메인 모델, 코드, 테스트 등 모든 곳에서 같은 용어를 사용한다.
- 소통 과정에서 발생하는 용어의 모호함을 줄일 수 있고 불필요한 해석 과정을 줄일 수 있다.