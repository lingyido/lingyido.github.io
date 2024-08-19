# Random Data Integration in NEO N3 Smart Contracts

Random data is often necessary for functions like lotteries, gaming, and other applications that require unpredictability. However, generating random numbers securely on a deterministic blockchain like NEO N3 presents unique challenges. This article delves into the complexities of using random data in NEO N3 smart contracts, provides examples of common pitfalls, and offers strategies for ensuring randomness security.

## Understanding Random Data Integration in NEO N3

In traditional programming environments, random data can be generated using various algorithms and system calls. However, on a blockchain, all operations must be deterministic to ensure consensus among all nodes. This requirement makes secure random number generation particularly challenging, as any predictable or manipulable randomness can be exploited by malicious actors.

NEO N3 provides a built-in function, `System.Runtime.GetRandom`, which generates pseudo-random numbers. While convenient, its usage can be problematic if not properly handled, potentially leading to vulnerabilities.

## Notice the Specific Randomness Processing in NEO N3

When incorporating random data into your NEO N3 smart contract, it’s crucial to recognize the limitations and potential risks. Specifically, developers must ensure that random numbers cannot be predicted or manipulated by malicious users.

One common approach to generating random data is using the `System.Runtime.GetRandom` function. However, due to the deterministic nature of blockchain, this function's output can be influenced or predicted by other factors within the blockchain environment. Therefore, careful consideration is required when using this function in smart contracts, especially in scenarios like lotteries or other games of chance.

## Example Scenario: Simple Lottery with System.Runtime.GetRandom

Let's consider a straightforward lottery implementation where a random number determines if a prize is awarded. The contract generates a random number, and if the number is even, the user wins.

```csharp
public static void ExecuteLottery()
{
    BigInteger randomNumber = Runtime.GetRandom();
    if (randomNumber % 2 == 0)
    {
        // Award the prize
        Transfer(GasToken.Hash, Runtime.ExecutingScriptHash, user, prizeAmount);
    }
}
```

In this example, the contract directly uses `System.Runtime.GetRandom` to generate a number. While this approach is simple, it carries significant risks, as the randomness can be influenced by factors that are potentially predictable or manipulable by malicious actors.

## Mitigation Strategies

To secure random data integration in your NEO N3 smart contracts, consider the following mitigation strategies:

1. **Avoid Using Built-in Random Functions Directly**:
   - Instead of relying on `System.Runtime.GetRandom`, use a more secure method, such as oracle or a two-step lottery process. In this approach, the contract first commits to a random number by storing the hash of a block or transaction. In a subsequent transaction, the contract uses the revealed block hash to determine the lottery outcome. This method leverages the unpredictability of block hashes, which are difficult to manipulate. Note, this is only an example for showing how commit-reveal works. Do not use this in practice cause relying only on `blockhash` is not good enough.

   Example:
   ```csharp
   // Commit phase: Store the block hash for the lottery (in transaction 1)
   Storage.Put(Storage.CurrentContext, "lotteryCommit", Blockchain.GetHeight());

   // Reveal phase: Use the stored block hash to determine the outcome (in transaction 2)
   BigInteger blockHeight = (BigInteger)Storage.Get(Storage.CurrentContext, "lotteryCommit");
   Block block = Ledger.GetBlock((uint)blockHeight);
   BigInteger randomNumber = block.Hash.ToBigInteger();
   if (randomNumber % 2 == 0)
   {
       // Award the prize
       Transfer(GasToken.Hash, Runtime.ExecutingScriptHash, user, prizeAmount);
   }
   ```

2. **Continue Using System.Runtime.GetRandom with Precautions**:
   - If you must use `System.Runtime.GetRandom`, ensure strict checks and controls:
     1. **Strictly Verify Entry Scripts**: Ensure that the user's entry script is verified to prevent any unauthorized code execution. Also, avoid any operations, especially transfers, that could trigger a callback or reentrancy, as highlighted in the reentrancy attack discussion.
     2. **Control Gas Usage to Prevent Side-Channel Attacks**: Make sure the gas consumption is consistent across different execution paths. This prevents attackers from using gas consumption as a side-channel to deduce the randomness. Limiting the gas usage and ensuring that only profitable transactions are executed can help mitigate this risk.

   ```csharp
   public static void ExecuteLotterySafely()
   {
       if (!Runtime.CheckWitness(user)) return;
       BigInteger randomNumber = Runtime.GetRandom();
       
       // Gas usage control logic
       if (randomNumber % 2 == 0)
       {
           Runtime.BurnGas(100); // Ensure gas usage consistency
           // Award the prize
           Transfer(GasToken.Hash, Runtime.ExecutingScriptHash, user, prizeAmount);
       }
       else
       {
           Runtime.BurnGas(100); // Ensure gas usage consistency
       }
   }
   ```

## Conclusion

Integrating random data into NEO N3 smart contracts requires careful consideration to avoid potential vulnerabilities. By understanding the limitations of blockchain randomness and applying secure strategies, developers can protect their contracts from exploitation. Whether avoiding built-in random functions or implementing additional safeguards, it’s crucial to prioritize security in any application relying on randomness. There is always a balance between absolute fairness and efficiency; for simple tasks like selecting attributes for a commemorative NFT, the more user-friendly built-in randomness may suffice. However, when significant financial stakes are involved, no amount of security is too much.

## Further Reading

For more insights on smart contract security and random data integration in NEO N3, consider exploring the following resources:

- [NEO Core Developer's Paper Blockchain_Random_Number](https://github.com/Jim8y/jinghui.me/blob/master/content/publication/Blockchain_Random_Number%20(28).pdf)
- [Some NEO issues that may related](https://github.com/neo-project/neo/issues/2693)
- [How can I securely generate a random number in my smart contract?](https://ethereum.stackexchange.com/questions/191/how-can-i-securely-generate-a-random-number-in-my-smart-contract/83616#83616)
