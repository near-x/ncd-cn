# Promise与跨合约调用 (by Coffee)

**跨合约调用**指某个合约去调用另一个合约方法的操作，该操作是**异步**的，调用方称为 `predecessor`，被调用方称为 `receiver`



### Review

### Action

* [x] `Transfer` （转账 **NEAR**）

* [x] `FunctionCall`（转账 **NEP141 Token**、**跨合约调用**）
* [x] `Deploy`
* [ ] `Stake`
* [ ] `CreateAccount`
* [ ] `DeleteAcount`
* [x] `AddKey`
* [x] `DeleteKey`



## Promise

near_sdk version >= 4.0.0

* 三种 API
  * 低级：env
    * [burrowland/upgrade.rs at main · NearDeFi/burrowland (github.com)](https://github.com/NearDeFi/burrowland/blob/main/contract/src/upgrade.rs)
  
  * 中级：Promise
  
  * 高级：定义 trait，只适用于单个 `FunctionCall`
* 回调函数
  * near_sdk 4.0.0 正式版前后写法的区别
  * `env::promise_result`
  * `#[callback_result]`
  * `#[callback_unwrap]`
* 返回 Promise 与 Value 的区别



## 跨合约调用详解



### Transaction

`Transaction` 有 3 个重要的参数：

* `signer_id: AccountId`

* `receiver_id: AccountId`

* `actions: Array<Action>`

一笔  `Transaction` 就是包含**一个或多个**  `Action` 的数组，节点按顺序执行数组中的 `Action` ，执行具有原子性

[Batch Transaction](https://explorer.testnet.near.org/transactions/HfE7ZJrkGTiwnVKz15KpAqnKJEuN3yxC7LHUoVbvqJ16)



### Rceipt

`Receipt` 有 3 个重要的参数：

* `predecessor_id: AccountId`

* `receiver_id: AccountId`

* `actions: Array<Action>`



对比一下可以发现，`Receipt` 是 `Transaction` 的另外一种形式，一笔 `Transaction` 需要先转化为对应的 `Receipt` 才能执行，通常该转换需要消耗一个区块时间。

`Receipt` 如果包含 `FunctionCall` ， 执行完毕后可能产生新的 `Receipt`（因为合约方法可能包含跨合约调用或其他的 `Action `调用）

**一笔交易的完整流程：签名发起 ` Transaction` -> `Transaction` 转 `Receipt` -> 执行 `Receipt` 产生新的 `Receipt` -> ... -> 直到所有 `Receipt` 执行完毕**

<img width="801" alt="receipt" src="https://user-images.githubusercontent.com/46699230/192817732-9427220e-c673-42ad-b6b2-0ffffe0ea605.png">

[NEAR Explorer | Transaction](https://explorer.near.org/transactions/CkbjbMFsduiZuU5mFRtVLA5hQvJ1uA4P6PVBtWtWGhvp)

[NEAR Explorer | Block](https://explorer.near.org/blocks/ECoy9A83ejqAkNT5kgtbSRdC4JErSsWez74uifKCdWZD)

### Example

alice.near 在 ref-finance 用 wnear 兑换 oct，整个流程经历 5 个区块

https://explorer.testnet.near.org/transactions/GPhqfQEQZABc3M2DGNPzHMYXpZoNUPQmpsBidcz4ygVA

<img width="1078" alt="example" src="https://user-images.githubusercontent.com/46699230/192817655-d25745c0-e391-4181-a30f-a3ddb5ba96c6.png">



## 更多信息

[关于 Near 合约开发中 Promise 的思考和总结 - SegmentFault](https://segmentfault.com/a/1190000041842290)
