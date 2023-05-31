# Tx Circuit

Tx circuit iterates over transactions included in proof to verify each transaction has valid signature. It also verifies the built transaction merkle patricia trie has same root hash as public input.

Main part of Tx circuit will be instance columns whose evaluation values are built by verifier directly. See the [issue](https://github.com/appliedzkp/zkevm-circuits/issues/122) for more details.

To verify if a transaction has valid signature, it hashes the RLP encoding of transaction and recover the address of signer with signature, then verifies the signer address is correct.

It serves as a lookup table for EVM circuit to do random access of any field of transaction.

To prevent any skip of transactions, we verify that the amount of transactions in the Tx circuit is equal to the amount that verified in EVM circuit.

# Tx 电路

Tx 电路在包含在证明内的交易上迭代去验证每一笔交易拥有有效签名。它也验证创建的交易merkle patricia trie拥有与公开输入相同的根哈希。

Tx 电路的主要部分将被实例化成instance列其中计算值由验证者之间创建。这个[issue](https://github.com/appliedzkp/zkevm-circuits/issues/122) 看更多细节。

为了验证一笔交易是否拥有有效的签名，它哈希了交易的RLP编码并使用签名恢复签名者的地址，任何验证签名者地址是正确的。

它作为EVM电路的查找表去做交易上任意域上的随机访问。

为了阻止跳过任意交易，我们验证Tx电路上的交易数量等于在EVM电路上验证的数量。