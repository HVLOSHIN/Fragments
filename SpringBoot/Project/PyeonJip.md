---
tags:
  - fragment
  - project
  - spring
---

# 1. Exception

GlobalException 선언
```java
@Getter  
@AllArgsConstructor  
public class GlobalException extends RuntimeException {  
  
    ErrorCode errorCode;  
}
```

 [[7. @ExceptionHandler#@ControllerAdvice|@ControllerAdvice]]를 통한 할당
```java
@ControllerAdvice  
public class GlobalExceptionHandler {  
    @ExceptionHandler(GlobalException.class)  
    public ResponseEntity<ErrorResponse> handleGlobalException(GlobalException e) {  
        return ErrorResponse.toResponseEntity(e.getErrorCode());  
    }
}
```

에러코드 - [[6. Spring ExceptionResolver#ResponseStatusExceptionResolver| 에러메세지]] 할당
```java
@Getter  
@AllArgsConstructor  
public enum ErrorCode {
	// 장바구니  
	CART_NOT_FOUND(HttpStatus.NOT_FOUND, "CART-01", "장바구니를 찾을 수 없습니다."),  
	CART_ITEM_QUANTITY_INVALID(HttpStatus.BAD_REQUEST, "CART-02", "유효하지 않은 수량입니다.");
  
    private final HttpStatus httpStatus;  
    private final String code;  
    private final String message;  
}	
```

```java
@Repository  
public interface CartRepository extends JpaRepository<Cart, Long> {  
    Optional<List<Cart>> findAllByEmail(String email);
```
레포지토리에서 Optional로 두면 좀더 편하게 예외를 반환할 수 있다.
```java hl:3-4
// 조회  
public List<CartDto> getCartItemsByEmail(String email) {  
    List<Cart> serverCartItems = cartRepository.findAllByEmail(email)  
            .orElseThrow(() -> new GlobalException(ErrorCode.CART_NOT_FOUND));  
    return serverCartItems.stream()  
            .map(this::convertToCartDto)  
            .collect(Collectors.toList());  
}
```

다음에는 이런식으로 처리해봐도 될듯
```java
 public class Exceptions {  
    private final GlobalException CATEGORY_NOT_FOUND = 
    new GlobalException(HttpStatus.NOT_FOUND, "CATEGORY-01", "카테고리를 찾을 수 없습니다.");  
}
```


# 2. 연관관계 매핑
```java
@Entity  
@Getter @Setter  
public class Cart {  
  
    @Id  
    @GeneratedValue(strategy = GenerationType.IDENTITY)  
    private Long id;  
  
    private String email; // 사용자 식별  
  
    private Long optionId; // optionId가 있으면 productId도 필요없다.  
  
    private Long quantity; // 사용자가 선택한 수량 (기본값 : 1)  
    public Cart(String email, Long optionId, Long quantity) {  
    }  
    public Cart() {  
    }
}
```

이번에 연관관계를 하나도 매핑하지 않았다.
#### 장점
1. 단순한 구조 : 엔티티의 구조가 단순해지고, 관리해야 할 필드가 줄어든다.
2. 효율적인 성능 : 복잡한 연관관계 매핑은 JPA의 지연로딩(**Lazy Loading**)이나 즉시 로딩(**Eager Loading**) 전략과 관련된 성능이슈를
   초래할 수 있다.
3. 명시적 쿼리 작성 : SQL 쿼리를 직접 작성해서 필요한 데이터만 가져올 수 있다.
4. 유연한 테이블 구조 변경 : DB 구조를 좀더 유연하게 관리할 수 있다.

#### 단점
1. 중복된 외래 키 관리 : 연관 관계 매핑 없이 외래키를 직접 엔티티에 두면, 
   데이터 무결성을 유지하기 위해 여러 엔티티에서 동일한 외래 키를 중복 관리해야 할 수 있다.
2. JPA의 객체지향적 장점 상실 
3. 응집도 감소 : 직접 연결되지 않기 때문에, 특정 엔티티의 변경이 다른 엔티에 즉시 반영되지 않을 수 있다.