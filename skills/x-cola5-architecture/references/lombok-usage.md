# Lombok Usage

Lombok 在 COLA 5 各层的使用规约。不同层的对象职责不同，Lombok 注解的使用策略也不同。

## 各层注解策略总览

| 层 | 对象类型 | 推荐注解 | 禁止注解 | 原因 |
|----|---------|---------|---------|------|
| domain | Entity（聚合根/实体） | `@Getter` | `@Data`、`@Setter` | 聚合根必须保护内部一致性，禁止外部直接修改状态 |
| domain | Value Object（V） | `@Getter` + `@EqualsAndHashCode` | `@Data`、`@Setter` | 值对象不可变，无 setter |
| domain | Domain Event | `@Getter` + `@EqualsAndHashCode` | `@Data`、`@Setter` | 事件不可变 |
| domain | Domain Service | `@RequiredArgsConstructor` | — | 构造器注入 Gateway |
| app | Cmd | `@Data` + `@Builder` | — | 纯数据载体，需要全量 getter/setter |
| app | Qry | `@Data` + `@Builder` | — | 纯数据载体 |
| app | VO | `@Data` + `@Builder` | — | 纯数据载体 |
| app | Application Service | `@RequiredArgsConstructor` | — | 构造器注入领域服务 |
| app | Executor | `@RequiredArgsConstructor` | — | 构造器注入 |
| infrastructure | DO | `@Data` + `@Builder` + `@NoArgsConstructor` + `@AllArgsConstructor` | — | 数据库映射对象，需要无参构造器（ORM） |
| infrastructure | GatewayImpl | `@RequiredArgsConstructor` | — | 构造器注入 Mapper/Client |
| infrastructure | Client | `@RequiredArgsConstructor` | — | 构造器注入 |
| infrastructure | Converter | 无需 Lombok | — | 静态工具类 |
| client | DTO | `@Data` + `@Builder` + `@NoArgsConstructor` + `@AllArgsConstructor` | — | 服务契约对象，需要无参构造器（反序列化） |
| client | Api 接口 | 无需 Lombok | — | 纯接口 |
| adapter | Controller | `@RequiredArgsConstructor` | — | 构造器注入 Application Service |
| adapter | HttpApi / RpcApi | `@RequiredArgsConstructor` | — | 构造器注入 |

## Domain 层详细规约

### Entity（聚合根 / 实体）

聚合根**禁止**使用 `@Data` 和 `@Setter`。状态变更必须通过业务方法，不能暴露 setter 让外部直接修改。

```java
@Getter
public class Order extends AggregateRoot {
    private String orderId;
    private List<OrderItem> items;
    private OrderStatus status;

    public void addItem(Product product, int quantity) {
        if (this.status != OrderStatus.DRAFT) {
            throw new BizException("ORDER_NOT_DRAFT", "只有草稿订单可添加商品");
        }
        this.items.add(new OrderItem(product, quantity));
    }

    public void submit() {
        if (this.items.isEmpty()) {
            throw new BizException("ORDER_EMPTY", "订单不能为空");
        }
        this.status = OrderStatus.SUBMITTED;
    }
}
```

❌ 禁止：
```java
@Data
public class Order extends AggregateRoot {
    private String orderId;
    private List<OrderItem> items;
    private OrderStatus status;
}
```
问题：`@Data` 生成 `setStatus()`、`setItems()`，外部可绕过业务规则直接修改状态。

### Value Object（V）

值对象不可变，使用 `@Getter` + `@EqualsAndHashCode`，所有字段 `final`。

```java
@Getter
@EqualsAndHashCode
public class MoneyV {
    private final BigDecimal amount;
    private final String currency;

    public MoneyV(BigDecimal amount, String currency) {
        this.amount = amount;
        this.currency = currency;
    }

    public MoneyV add(MoneyV other) {
        return new MoneyV(this.amount.add(other.amount), this.currency);
    }
}
```

### Domain Event

事件不可变，使用 `@Getter` + `@EqualsAndHashCode`。

```java
@Getter
@EqualsAndHashCode
public class OrderSubmittedEvent {
    private final String orderId;
    private final OffsetDateTime submittedAt;

    public OrderSubmittedEvent(String orderId) {
        this.orderId = orderId;
        this.submittedAt = OffsetDateTime.now();
    }
}
```

### Domain Service

使用 `@RequiredArgsConstructor` 实现构造器注入。

```java
@Service
@RequiredArgsConstructor
public class OrderDomainService {
    private final OrderGateway orderGateway;
    private final ProductGateway productGateway;

    public Order createOrder(String customerId, List<String> productIds) {
        List<Product> products = productGateway.findByIds(productIds);
        Order order = new Order(customerId);
        products.forEach(p -> order.addItem(p, 1));
        return order;
    }
}
```

## App 层详细规约

### Cmd / Qry / VO

纯数据载体，使用 `@Data` + `@Builder`。

```java
@Data
@Builder
public class OrderCreateCmd {
    private Long customerId;
    private List<OrderItemCreateCmd> items;
}
```

```java
@Data
@Builder
public class OrderListQry {
    private String status;
    private Integer pageNum;
    private Integer pageSize;
}
```

```java
@Data
@Builder
public class OrderVO {
    private Long orderId;
    private String orderNo;
    private String status;
    private BigDecimal totalAmount;
}
```

### Application Service / Executor

使用 `@RequiredArgsConstructor` 实现构造器注入。

```java
@Service
@RequiredArgsConstructor
public class OrderService {
    private final OrderDomainService orderDomainService;
    private final OrderGateway orderGateway;

    @Transactional(rollbackFor = Exception.class)
    public Long createOrder(OrderCreateCmd cmd) {
        Order order = cmd.toEntity();
        orderDomainService.processOrder(order);
        orderGateway.save(order);
        return order.getOrderId();
    }
}
```

## Infrastructure 层详细规约

### DO（数据对象）

使用 `@Data` + `@Builder` + `@NoArgsConstructor` + `@AllArgsConstructor`。

- `@NoArgsConstructor`：ORM 框架（MyBatis / JPA）需要无参构造器反序列化
- `@AllArgsConstructor`：`@Builder` 需要全参构造器
- `@Builder`：方便测试和手动构造

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class OrderDO {
    private Long id;
    private String orderNo;
    private String status;
    private BigDecimal totalAmount;
    private OffsetDateTime createdAt;
    private OffsetDateTime updatedAt;
}
```

### GatewayImpl / Client

使用 `@RequiredArgsConstructor` 实现构造器注入。

```java
@Component
@RequiredArgsConstructor
public class OrderGatewayImpl implements OrderGateway {
    private final OrderMapper orderMapper;
    private final OrderItemMapper orderItemMapper;
}
```

## Client 模块详细规约

### DTO

使用 `@Data` + `@Builder` + `@NoArgsConstructor` + `@AllArgsConstructor`。

- `@NoArgsConstructor`：Fastjson2 反序列化需要无参构造器
- `@AllArgsConstructor`：`@Builder` 需要全参构造器

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class OrderDTO {
    private Long orderId;
    private String orderNo;
    private String status;
    private BigDecimal totalAmount;
    private List<OrderItemDTO> items;
}
```

## Adapter 层详细规约

### Controller / HttpApi / RpcApi

使用 `@RequiredArgsConstructor` 实现构造器注入。

```java
@RestController
@RequiredArgsConstructor
@RequestMapping("/api/v1/orders")
public class OrderController {
    private final OrderService orderService;
}
```

## @Slf4j 使用规约

| 层 | 是否允许 | 说明 |
|----|---------|------|
| domain | ❌ 禁止 | Domain 不依赖 Slf4j，不记录日志 |
| app | ✅ 允许 | Application Service / Executor 可记录用例级日志 |
| infrastructure | ✅ 允许 | GatewayImpl / Client 可记录外部调用日志 |
| adapter | ✅ 允许 | Controller / Listener 可记录请求日志 |
| client | ❌ 禁止 | Client 不依赖 Slf4j |

```java
@Service
@Slf4j
@RequiredArgsConstructor
public class OrderService {
    public Long createOrder(OrderCreateCmd cmd) {
        log.info("Creating order for customer: {}", cmd.getCustomerId());
    }
}
```

## @Builder 使用规约

`@Builder` 适用于纯数据载体（Cmd / Qry / VO / DTO / DO），**禁止**在 Entity 上使用。

| 对象类型 | @Builder | 原因 |
|---------|----------|------|
| Cmd / Qry / VO | ✅ 推荐 | 纯数据载体，Builder 提升可读性 |
| DTO | ✅ 推荐 | 纯数据载体 |
| DO | ✅ 推荐 | 方便测试构造 |
| Entity | ❌ 禁止 | Builder 模式绕过业务规则校验，允许构造不合法对象 |
| Value Object | ❌ 禁止 | 值对象应通过显式构造器创建，确保不变性 |
| Domain Event | ❌ 禁止 | 事件应通过业务方法触发，不应手动构建 |

❌ 禁止：
```java
@Builder
public class Order extends AggregateRoot {
    private String orderId;
    private OrderStatus status;
}
Order order = Order.builder().status(OrderStatus.SUBMITTED).build();
```
问题：`@Builder` 允许跳过业务规则直接构造 `SUBMITTED` 状态的订单。

## @EqualsAndHashCode 使用规约

| 对象类型 | @EqualsAndHashCode | 说明 |
|---------|-------------------|------|
| Value Object | ✅ 必须 | 值对象按属性判等 |
| Domain Event | ✅ 推荐 | 事件按属性判等 |
| Entity | ❌ 禁止 | 实体按标识（ID）判等，`@EqualsAndHashCode` 按属性判等会导致语义错误 |
| DTO / Cmd / Qry / VO / DO | ❌ 不需要 | `@Data` 已包含 `@EqualsAndHashCode`，无需额外声明 |

## 注解组合速查

### 纯数据载体（Cmd / Qry / VO）

```java
@Data
@Builder
public class OrderCreateCmd { ... }
```

### 需要无参构造器的数据载体（DTO / DO）

```java
@Data
@Builder
@NoArgsConstructor
@AllArgsConstructor
public class OrderDTO { ... }
```

### 领域实体

```java
@Getter
public class Order extends AggregateRoot { ... }
```

### 不可变值对象 / 事件

```java
@Getter
@EqualsAndHashCode
public class MoneyV {
    private final BigDecimal amount;
    private final String currency;
}
```

### 依赖注入的服务类

```java
@Service
@RequiredArgsConstructor
@Slf4j
public class OrderService { ... }
```

## Mandatory 规则

1. Domain Entity（聚合根 / 实体）**禁止**使用 `@Data` 和 `@Setter`，状态变更必须通过业务方法
2. Domain Entity **禁止**使用 `@Builder`，避免绕过业务规则构造不合法对象
3. Value Object（V 后缀）**禁止**使用 `@Setter`，所有字段必须 `final`
4. Domain 模块**禁止**使用 `@Slf4j`
5. DTO / DO 使用 `@Data` + `@Builder` 时，**必须**同时加 `@NoArgsConstructor` + `@AllArgsConstructor`
6. Client 模块**禁止**使用 `@Slf4j`

## Recommended 规则

1. 服务类（Application Service / GatewayImpl / Controller）统一使用 `@RequiredArgsConstructor` 构造器注入，不用 `@Autowired`
2. Cmd / Qry / VO 使用 `@Data` + `@Builder`，不需要 `@NoArgsConstructor` / `@AllArgsConstructor`（前端通过 setter 赋值）
3. Value Object 和 Domain Event 使用 `@Getter` + `@EqualsAndHashCode`，不使用 `@Value`（`@Value` 生成全参构造器但字段为泛型数组时不友好）
4. Domain Service 使用 `@RequiredArgsConstructor` 注入 Gateway
5. 避免在 `@ToString` 中包含懒加载字段或循环引用字段