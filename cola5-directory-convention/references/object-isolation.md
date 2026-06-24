# Object Isolation

对象类型隔离规约。定义各层对象类型的归属、跨层流转规则和转换规约，确保层间依赖方向正确、对象不越界泄露。

## 对象类型总览

| 对象类型 | 后缀 | 归属层 | 用途 |
|---------|------|--------|------|
| Command | `Cmd` | app | 写操作入参（如 `OrderCreateCmd`） |
| Query | `Qry` | app | 读操作入参（如 `OrderListQry`） |
| View Object | `VO` | app | 面向前端的出参（如 `OrderVO`） |
| Data Transfer Object | `DTO` | adapter.api | 服务间调用入参/出参（如 `OrderDTO`、`OrderCreateDTO`、`OrderQueryDTO`） |
| Entity | 无后缀 | domain | 领域实体 / 聚合根（如 `Order`、`OrderItem`） |
| Value Object | `V` | domain | 值对象（如 `MoneyV`、`AddressV`） |
| Domain Event | `Event` | domain | 领域事件（如 `OrderSubmittedEvent`） |
| Data Object | `DO` | infrastructure | 数据库映射对象（如 `OrderDO`） |
| External Response | `Response` | infrastructure | 外部服务响应（如 `PaymentResponse`） |

## 跨层流转矩阵

| 对象类型 | adapter | app | domain | infrastructure |
|---------|---------|-----|--------|----------------|
| Cmd / Qry | ✅ 使用 | ✅ 定义 | ❌ | ❌ |
| VO | ✅ 使用 | ✅ 定义 | ❌ | ❌ |
| DTO（服务间） | ✅ 定义 | ❌ | ❌ | ❌ |
| Entity | ❌ | ✅ 使用 | ✅ 定义 | ✅ 使用 |
| Value Object | ❌ | ✅ 使用 | ✅ 定义 | ✅ 使用 |
| Event | ❌ | ✅ 使用 | ✅ 定义 | ✅ 使用 |
| DO | ❌ | ❌ | ❌ | ✅ 定义 |
| Response（外部） | ❌ | ❌ | ❌ | ✅ 定义 |

> ✅ 定义 = 该类型的归属层，负责创建和维护
> ✅ 使用 = 可以引用该类型
> ❌ = 禁止引用该类型

## 依赖方向与对象归属

```
adapter ──依赖──→ app ──依赖──→ domain ←──依赖── infrastructure
   │                │                │                    │
   │                │                │                    │
   ▼                ▼                ▼                    ▼
 使用 Cmd/Qry/VO  定义 Cmd/Qry/VO  定义 Entity/V/Event 定义 DO/Response
 定义 DTO          使用 Entity      使用（无外部依赖）    使用 Entity
```

### 关键约束

- **adapter 依赖 app**，因此 adapter 可以使用 app 定义的 Cmd / Qry / VO
- **app 依赖 domain**，因此 app 可以使用 domain 定义的 Entity / Value Object（`V`） / Event
- **infrastructure 依赖 domain**，因此 infrastructure 可以使用 domain 定义的 Entity / Value Object（`V`） / Event
- **app 禁止依赖 adapter**，因此 app 无法访问 adapter 定义的 DTO
- **domain 不依赖任何模块**，因此 domain 无法访问任何其他层的对象类型

## 对象转换规约

### 完整转换链

```
Cmd/Qry ──[CmdExe/QryExe]──→ Entity ──[GatewayImpl]──→ DO
   (app 层)                 (domain 层)              (infrastructure 层)

VO ←──[CmdExe/QryExe]── Entity ←──[GatewayImpl]── DO
(app 层)                (domain 层)              (infrastructure 层)

DTO ←──[HttpApi/RpcApi]── Entity
(adapter 层)              (通过 app 层间接获取)

Response ──[Client]──→ Entity
(infrastructure 层)       (domain 层)
```

### 转换规则明细

| 转换方向 | 转换器位置 | 方法命名 | 示例 |
|---------|-----------|---------|------|
| Cmd → Entity | `CmdExe` | `toEntity(cmd)` | `OrderCreateCmdExe.toEntity(cmd)` |
| Entity → VO | `QryExe` / `CmdExe` | `toVO(entity)` | `OrderListQryExe.toVO(order)` |
| Entity → DO | `Converter` / `GatewayImpl` | `toDO(entity)` | `OrderConverter.toDO(order)` |
| DO → Entity | `Converter` / `GatewayImpl` | `toEntity(do)` | `OrderConverter.toEntity(orderDO)` |
| Entity → DTO | `ApiConverter` | `toDTO(entity)` | `OrderApiConverter.toDTO(order)` |
| Response → Entity | `Client` | `toEntity(response)` | `PaymentClient.toEntity(resp)` |

### 转换器归属

| 转换器 | 归属层 | 命名格式 |
|--------|--------|---------|
| CmdExe（含 Cmd→Entity / Entity→VO） | app | `{Action}{Resource}CmdExe` |
| QryExe（含 Entity→VO） | app | `{Action}{Resource}QryExe` |
| Converter（Entity↔DO） | infrastructure | `{Resource}Converter` |
| ApiConverter（Entity→DTO） | adapter.api | `{Resource}ApiConverter` |

## Mandatory 规则

1. Cmd / Qry / VO **必须**定义在 app 模块，adapter 通过依赖 app 使用
2. DTO（服务间调用）**必须**定义在 adapter.api 模块或独立 API jar，**禁止**泄露到 app 或 domain
3. DO **禁止**泄露到 domain 或 app 层
4. External Response **禁止**泄露到 domain 层，必须在 infrastructure 的 Client 或 GatewayImpl 中转换
5. 对象转换逻辑**禁止**散落在业务代码中，必须抽取到对应的转换器
6. **禁止**跨层直接传递非归属对象（如 Controller 直接返回 Entity）

## Recommended 规则

1. 使用 MapStruct 等工具简化 Entity 与 DO 之间的转换
2. 简单转换（1-2 个字段）可直接在 CmdExe / GatewayImpl 中完成，不必强制抽取 Converter
3. DTO 与 VO 字段可能重叠，但应独立定义，不要复用——两者的演进节奏不同
4. 聚合根不要暴露 getter 让外部转换器直接读取内部状态，应提供 `toVO()` / `toDTO()` 等方法