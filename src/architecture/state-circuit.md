# 状态电路

<!-- toc -->

# 介绍

状态电路在EVM电路的随机读写访问记录上迭代来验证每一片数据在不同写入中都是一致的。这也验证了从老的到新的，来自于公共输入的状态merkle patricia trie根哈希对应于有效的交易。
为了验证数据是否一致，首先要验证所有访问记录都由他们唯一标识组成并按照访问顺序排序。然后验证写入的两个记录是一致的。也验证了数据是正确的格式。
它作为EVM电路的查找表去保持随机读写访问是一致的。
为了阻止人访问记录的意恶意插入，我们也验证了在状态电路上随机读写访问记录的数量等于EVM电路的数量（`rw_counter`的最终值）。
# 内容

## 读写单元分组

The first thing to ensure data is consistent between different writes is to give each data an unique identifier, then group data chunks by the unique identifier. And finally, then sort them by order of access `rw_counter`.

Here are all kinds of data with their unique identifier:
要确认数据在不同写入之间保持一致的第一件事是给每一个数据一个唯一标识，然后通过唯一标识将数据分组。并且最终，按照访问`rw_counter`的顺序对它排序。


| 标签                       | 唯一标识                            | 值                                |
| ------------------------- | ---------------------------------------- | ------------------------------------- |
| `TxAccessListAccount`     | `(tx_id, account_address)`               | `(is_warm, is_warm_prev)`             |
| `TxAccessListAccountStorage` | `(tx_id, account_address, storage_slot)` | `(is_warm, is_warm_prev)`             |
| `TxRefund`                | `(tx_id)`                                | `(value, value_prev)`                 |
| `Account`                 | `(account_address, field_tag)`           | `(value, value_prev)`                 |
| `AccountStorage`          | `(account_address, storage_slot)`        | `(value, value_prev)`                 |
| `AccountDestructed`       | `(account_address)`                      | `(is_destructed, is_destructed_prev)` |
| `CallContext`             | `(call_id, field_tag)`                   | `(value)`                             |
| `Stack`                   | `(call_id, stack_address)`               | `(value)`                             |
| `Memory`                  | `(call_id, memory_address)`              | `(byte)`                              |

在它们的组和值上不同标签由不同的约束。
为了方便大部分标签也保证前面的值`*_prev`，这有助于减少EVM电路在执行与当前值`diff`的写入时的查找工作，或者执行一个还原写。

## Lazy 初始化

EVM内存隐式得扩展，例如，当内存是空的并且它遇到一个`mload(32)`， EVM 首先扩展到内存大小为`64`，然后再加载刚刚初始化的字节压到栈里，这里值总是为`0`。
隐式扩展行为使得在EVM电路中即使简单的`MLOAD` 和 `MSTORE` 也变得复杂了，所以我们用一个技巧通过约束每一个记录单元的第一条记录为写操作或者值为`0`去外包状态电路的工作。它节约了扩展内存的工作量并忽略那些未使用过的内存，仅使用过的内存地址将用`0`初始化，以此作为lazy初始化。

> 这些内容也用在另一个例子中：opcode `SELFDESTRUCT` 也可以更新数据的变量数量。重置`balance`, `nonce`, `code_hash`和每一个`storage_slot`即使步骤中没有使用。所以账户下的每一个状态，我们可以添加一个`revision_id`来处理这样的例子，细节看[设计笔记，可回退写的回退笔记，SELFDESTRUCT](./reversible-write-reversion2.md#selfdestruct)。
> ==TODO== 将此转化为一个问题进行讨论
>
> **han**

## Trie opening and incrementally update

# 约束

## `main`

==TODO== 解释这些标签

<!--
##### `tx_access_list_account`
##### `tx_access_list_storage_slot`
##### `tx_refund`
##### `account_nonce`
##### `account_balance`
##### `account_code_hash`
##### `account_storage`
##### `call_state`
##### `stack`
##### `memory`
 -->

# 实现

- [spec](https://github.com/appliedzkp/zkevm-specs/blob/master/specs/state-proof.md)
    - [python](https://github.com/appliedzkp/zkevm-specs/blob/master/src/zkevm_specs/state.py)
- [circuit](https://github.com/appliedzkp/zkevm-circuits/blob/main/zkevm-circuits/src/state_circuit.rs)
