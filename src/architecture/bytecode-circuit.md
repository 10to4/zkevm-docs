# 字节码电路

字节码电路在合约字节码上迭代以验证每一个字节由有效的哈希。

它作为 EVM 电路的查找表来在字节码的任意索引上做随机访问。

# 实现

- [spec](https://github.com/appliedzkp/zkevm-specs/blob/master/specs/bytecode-proof.md)
    - [python](https://github.com/appliedzkp/zkevm-specs/blob/master/src/zkevm_specs/bytecode.py)
- [circuit](https://github.com/appliedzkp/zkevm-circuits/tree/main/zkevm-circuits/src/bytecode_circuit)