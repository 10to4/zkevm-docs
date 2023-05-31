# 随机线性组合

<!-- toc -->

## 介绍

在电路中，为例减少约束的数量，我们使用随机线性组合作为一个廉价的哈希函数去为两个场景随机-检查字节：
1. 在254比特域上编码32字节的词（256比特）
2. 在254比特域上累计（或者调整/编码）任意长度字节

第一个场景中，允许我们用单个witness值去存储一个EVM词，而不必担心一个词在这个域上并不匹配的情况。大部分时间我们直接移动这些线性组合词从这里到那里，并且仅当我们需要执行计算或者位操作时我们将编码词语为字节（在每个字节上使用范围检查）来做当前操作。
或者我们也可以用两个witness值存储一个EVM词语，代表高部分和低部分：但是这就使得我们需要将每个词转移到两个witness值。注意从随机线性组合中获得的约束的优化还没有被正确分析。
在第二个场景中，允许我们去很容易得为一笔交易或者merkle (hexary) patricia trie节点在固定数量的witness上做RLP编码，而不必担心RLP编码字节变成任意长或者有限长度（对于MPT节点有一个最大长度，但是交易没有）。每一个累加的witness将进一步得解压/分解/解码成几个字节并作为输入喂给`keccak256`。

> 如果我们进一步问`keccak256`去获得一个累加的witness及它在输入中包含的字节数量就更好了。
>
> **han**

## 随机性的关注

随机性的方式对随机线性组合很重要：如果操作不当，一个恶意的验证者能够找到一个碰撞让一个看起来不正确的witness通过验证（允许凭空造出以太）。

下面是两种方法试图得出一个合理的随机性来减轻风险。

### 1. 使用额外的一轮从承诺的多项式得出随机数
假定在我们的约束系统中我们能够分离所有随机线性组合的witness到不同的多项式中，我们可以：
1. 承诺多项式，但随机线性witness的承诺除外
2. 从公共输入的承诺中产生随机数
3. 继续证明处理。

### 从电路的公开输入从得到随机数

> 更新：我们应该只要跟着传统的Fiat-Shamir（方法1），去承诺并生成挑战数。假定EVM状态转换是确定的，这对恶意验证者是行不通的。

电路的公共输入至少包含：

- 交易元数据（输入）
- 老的状态trie根（输入）
- 新的状态trie根（输出）

尽管事实上新的状态trie根可能是不正确的（在攻击的例子中），由于状态trie根隐含所有包含的字节（包括交易原始数据），如果我们从所有这些中生成随机数，恶意证明者需要首先确定新的（不正确的）状态trie根是什么，并且再找到输入和输出的碰撞。这就限制了碰撞对的可能性因为输入和输出也是固定的。

## 使用随机线性组合的一个小的确定性系统

以下的例子展示了随机线性组合是怎么用来比较使用单个witness值的词语的相等性。
假定一个确定性的虚拟机由两个字节码`PUSH32` 和 `ADD`组成，并且VM作为单个函数`run` 运行描述为：

### 伪代码

```python
def randomness_approach_2(bytecode: list, claimed_output: list) -> int:
    return int.from_bytes(keccak256(bytecode + claimed_output), 'little') % FP

def run(bytecode: list, claimed_output: list):
    """
    run takes bytecode to execute and treat the top of stack as output in the end.
    """

    # Derive randomness
    r = randomness_approach_2(bytecode, claimed_output)

    # Despite an EVM word is 256-bit which is larger then field size, we store it
    # as random linear combination in stack. Top value is in the end of the list.
    stack = []

    program_counter = 0
    while program_counter < len(bytecode):
        opcode = bytecode[program_counter]

        # PUSH32
        if opcode == 0x00:
            # Read next 32 bytes as an EVM word from bytecode
            bytes = bytecode[program_counter+1:program_counter+33]
            # Push random linear combination of the EVM word to stack
            stack.append(random_linear_combine(bytes, r))
            program_counter += 33
        # ADD
        elif opcode == 0x01:
            # Pop 2 random linear combination EVM word from stack
            a, b = stack.pop(), stack.pop()
            # Decompress them into little-endian bytes
            bytes_a, bytes_b = rlc_to_bytes[a], rlc_to_bytes[b]
            # Add them together
            bytes_c = add_bytes(bytes_a, bytes_b)
            # Push result as random linear combination to stack
            stack.append(random_linear_combine(bytes_c, r))
            program_counter += 1
        else:
            raise ValueError("invalid opcode")

    assert rlc_to_bytes[stack.pop()] == claimed_output, "unsatisfied"
```

### [完整的运行代码](./random-linear-combinaion/full-runnable-code.md)

所有随机线性组合或者压缩将在PLONK约束系统中被约束，其中随机数被公共输入喂进去。

这个随机数从输入输出中产生（喂给keccak256），对应于[方法 2](#2-Randomness-from-all-public-inputs-of-circuit)。虽然输入字节中的原始数据代替哈希值，但是假定keacck256 和以太坊上的 merkle (hexary) patricia tries是碰撞冲突的，在两个例子中没有很大的不同。

这个例子至少减少为：**是否恶意证明者可以在确定输入和输出后发现栈的push和pop之间的冲突**。

