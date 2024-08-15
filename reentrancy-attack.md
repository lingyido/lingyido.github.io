# Reentrancy Attacks in NEO N3 Smart Contracts

Reentrancy attacks are a significant threat to the security of smart contracts. These attacks occur when a contract makes an external call to another contract, and the attacker exploits this by re-entering the original contract before the first execution is completed. This can lead to unexpected behavior, including the draining of funds or unauthorized state changes.

## Understanding Reentrancy in NEO N3

In the NEO N3 platform, smart contracts interact with external contracts or services, which can expose them to reentrancy risks. A typical scenario involves a contract that updates its state after making an external call. If the external contract allows for a callback into the original contract, an attacker could repeatedly trigger the function, causing the original contract's state to be altered multiple times.

## Notice the NEP-17 and NEP-11 Standards

On the NEO N3 platform, token standards like NEP-17 and NEP-11 inherently support callbacks. This means that if the token recipient is a smart contract, there's a potential for reentrancy attacks. Both NEO and GAS, which are native assets on NEO N3 and compliant with the NEP-17 standard, are susceptible to this risk. Unlike Ethereum’s ERC-20 and ERC-721 tokens, where reentrancy risks are well-known on ETH instead of other tokens, the callback mechanism in NEO’s NEP-17 and NEP-11 introduces unique considerations that developers need to be aware of.

### Example Scenario

Consider a contract with a `Withdraw` function that sends assets to a recipient:

```csharp
public static bool Withdraw(UInt160 recipient, BigInteger amount)
{
    if (GAS.Transfer(Runtime.ExecutingScriptHash, recipient, amount))
    {
        Storage.Put(Storage.CurrentContext, recipient, GetBalance(recipient) - amount);
        return true;
    }
    return false;
}
```

In this scenario, if the `Transfer` method in the external contract `GAS` allows for a callback into `onNep17Payment`, an attacker could repeatedly drain funds by re-entering the contract before the state is updated.

## Mitigation Strategies

To prevent reentrancy attacks, consider the following approaches, using them all is better:

1. **following the checks-effects-interactions mode**:
   Avoid state changes after external calls. Ensure that all state updates occur before any external call is made, especially the token transfer.

   ```csharp
   public static bool Withdraw(UInt160 recipient, BigInteger amount)
   {
       BigInteger currentBalance = GetBalance(recipient);
       if (currentBalance < amount) return false;
       
       Storage.Put(Storage.CurrentContext, recipient, currentBalance - amount);
       
       if (!GAS.Transfer(Runtime.ExecutingScriptHash, recipient, amount))
       {
           Storage.Put(Storage.CurrentContext, recipient, currentBalance);
           return false;
       }
       return true;
   }
   ```

2. **Use Mutex or Flags**:
   Implement a mutex or flag to prevent re-entry during execution.

   ```csharp
   
   [NoReentrant] // notice that this feature is provided by the [Neo.Compiler.CSharp](https://github.com/neo-project/neo-devpack-dotnet/blob/08d0e6ae73ed4a1b4ec8d7d5b4adee5f4bedfdb8/src/Neo.SmartContract.Framework/Attributes/NoReentrantAttribute.cs)
   public static bool Withdraw(UInt160 recipient, BigInteger amount)
   {
       BigInteger currentBalance = GetBalance(recipient);
       if (currentBalance < amount) return false;
       
       Storage.Put(Storage.CurrentContext, recipient, currentBalance - amount);
       
       if (!GAS.Transfer(Runtime.ExecutingScriptHash, recipient, amount))
       {
           Storage.Put(Storage.CurrentContext, recipient, currentBalance);
           return false;
       }
       return true;
   }
   ```

## Conclusion

Reentrancy attacks can be catastrophic if left unchecked. By understanding the risks and applying these mitigation strategies, developers can protect their NEO N3 smart contracts from such vulnerabilities. Always prioritize security in your development process to build robust and secure blockchain applications.

## Further Reading

* use claim pattern instead of transferring pattern
* multiple contracts reentrancy
