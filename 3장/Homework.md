# OrderCheckout 리팩터링

## 예제 소스

파일 이름 예: `Main.java` (한 파일로 실행 가능)
```java
import java.util.*;
import java.math.BigDecimal;

class Main {
    public static void main(String[] args) {
        OrderCheckout svc = new OrderCheckout();
        // 정상 케이스
        List<String> errors = new ArrayList<>();
        String receipt = svc.checkout("vip001 , A-100 , 2 , COUPON10", true, errors);
        System.out.println("RECEIPT=" + receipt);
        System.out.println("ERRORS=" + errors);

        // 입력 에러 케이스
        List<String> e2 = new ArrayList<>();
        System.out.println("BAD=" + svc.checkout(" ,A-100,-1,", false, e2));
        System.out.println("ERRORS2=" + e2);
    }
}
```


OrderCheckout.java

```java
/**
 * 문제 의도: 한 함수에서 너무 많은 일을 함
 * - 플래그 인자(vip)
 * - 출력 인자(errors)
 * - 매직 넘버와 문자열
 * - 파싱/검증/가격/재고/할인/저장/영수증조립/로깅을 한 함수에서 처리
 * - 부작용(내부 상태 lastOrderId)
 * - 예외 대신 오류코드/문자열로 제어
 */
class OrderCheckout {
    private String lastOrderId; // 사이드 이펙트 예시

    /**
     * raw: "userId , itemId , qty , coupon"
     * vip: VIP 여부(플래그 인자)
     * errors: 오류 메시지를 채워 넣는 출력 인자
     */
    public String checkout(String raw, boolean vip, List<String> errors) {
        // 로깅(임시)
        System.out.println("[CHECKOUT START] raw=" + raw + " vip=" + vip);

        // 파싱
        String[] parts = raw.split(",");
        if (parts.length < 3) {
            errors.add("INVALID_FORMAT");
            return "ERROR";
        }
        String userId = parts[0].trim();
        String itemId = parts[1].trim();
        String qtyStr = parts[2].trim();
        String coupon = (parts.length >= 4) ? parts[3].trim() : "";

        // 검증
        if (userId.isEmpty()) errors.add("USER_REQUIRED");
        if (itemId.isEmpty()) errors.add("ITEM_REQUIRED");
        int qty = 0;
        try {
            qty = Integer.parseInt(qtyStr);
        } catch (Exception ex) {
            errors.add("QTY_NOT_NUMBER");
        }
        if (qty <= 0) errors.add("QTY_POSITIVE");

        if (!errors.isEmpty()) return "ERROR";

        // 가격(매직넘버 다수)
        int baseUnit = itemId.startsWith("A") ? 100 : 150; // A면 100원, 아니면 150원
        int shipping = itemId.startsWith("A") ? 500 : 1200; // KR 가정
        int weightPerUnit = 2; // kg
        int weight = weightPerUnit * qty;
        int extra = (weight > 10) ? 800 : 0;

        BigDecimal sum = BigDecimal.valueOf(baseUnit)
                .multiply(BigDecimal.valueOf(qty))
                .add(BigDecimal.valueOf(shipping))
                .add(BigDecimal.valueOf(extra));

        // VIP/쿠폰 할인(분기 중첩 + 플래그 인자)
        if (vip) {
            sum = sum.multiply(BigDecimal.valueOf(0.9)); // 10% 할인
        }
        if ("COUPON10".equalsIgnoreCase(coupon)) {
            sum = sum.subtract(BigDecimal.TEN); // 10원 고정 차감
        }

        // 재고 확인(임시)
        if (qty > 20) {
            errors.add("OUT_OF_STOCK");
            return "ERROR";
        }

        // 저장(모킹) + 사이드이펙트
        String orderId = UUID.randomUUID().toString();
        System.out.println("SAVE: order=" + orderId + " user=" + userId + " item=" + itemId + " qty=" + qty + " sum=" + sum);
        lastOrderId = orderId;

        // 영수증 문자열 조립
        String receipt = "OK:" + orderId + ":" + userId + ":" + itemId + ":" + qty + ":" + sum;
        System.out.println("[CHECKOUT END] receipt=" + receipt);
        return receipt;
    }

    public String getLastOrderId() {
        return lastOrderId;
    }
}
```


## 리팩터링 목표(“함수” 원칙 집중)

1.  **작고 한 가지 일만 하는 함수**로 분해
-   파싱 / 검증 / 가격계산 / 할인 / 배송비 / 재고확인 / 저장 / 영수증 조립 / 로깅을 **의미 있는 이름**의 작은 함수들로 나누기.
-   **한 수준의 추상화**만 유지(상세와 개요 섞지 않기).

2.  **플래그·출력 인자 제거**
-   `vip(boolean)`, `errors(List)` 제거.
-   VIP 여부, 쿠폰, 사용자/상품/수량 등은 **파라미터 객체(Record/DTO)** 또는 **요청 모델**로 묶기.
-   오류는 **예외**로 처리하고, 호출부에서 **필요한 최소 범위**만 `try-catch`.
    
3.  **커맨드-쿼리 분리 & 부작용 최소화**
-   파싱/검증은 **상태 변경 없이** 수행 후 **불변 DTO** 반환.
-   저장은 저장만 하도록 분리하고, `lastOrderId` 같은 숨은 상태 의존 제거(필요시 명시적 반환).

4.  **매직 넘버/문자열 상수화 & 정책 캡슐화**
-   단가, 배송비, 무게, 임계값, 쿠폰/할인 정책을 상수/전략으로 분리.
-   할인/배송비는 전략(또는 간단 맵/정책 객체)으로 설정해 **확장에 열려** 있게.



## 최소 테스트(수정 후, 예시 메인)

```java
public class MainRefTest {
    public static void main(String[] args) {
        // 1) 정상
        var svc = new RefactoredOrderService(/* 정책/리포지토리 주입 */);
        var req = new CheckoutRequest("vip001","A-100",2,true,"COUPON10");
        Receipt r = svc.checkout(req);
        System.out.println(r); // OK:...

        // 2) 검증 실패
        try {
            svc.checkout(new CheckoutRequest("", "A-100", -1, false, ""));
            System.out.println("FAIL: should have thrown");
        } catch (ValidationException e) {
            System.out.println("PASS: " + e.getMessage());
        }

        // 3) 재고 부족
        try {
            svc.checkout(new CheckoutRequest("u1","B-100", 99, false, ""));
            System.out.println("FAIL: should have thrown");
        } catch (OutOfStockException e) {
            System.out.println("PASS: OUT_OF_STOCK");
        }
    }
}
```


## 제출물 가이드

- 변경된 **소스 전체**(요청/결과 DTO, 서비스, 정책/전략, 저장 모킹 리포지토리 등).
- **리팩터링 요약 문서(5줄 내외)**
- github 파일 push후 링크 슬랙 공유 😀😀😀


