# Data Storage Security in NEO N3 Smart Contracts

Data storage is a critical aspect of smart contract development, as improper handling can lead to vulnerabilities, data loss, or unauthorized access. In NEO N3, understanding and securing your contract’s storage is crucial to maintaining the integrity and security of your application. **Remember, all data on the blockchain is publicly accessible, so do not attempt to store secrets or sensitive information within your contract's storage.**

## Understanding Data Storage in NEO N3

In NEO N3, data storage is managed using key-value pairs within a contract's storage context. Each piece of data is associated with a unique key, and these keys are often prefixed to organize and separate different types of data. This approach ensures that data is efficiently stored and retrieved, and prevents conflicts between different pieces of information within the same contract. However, improper handling of storage prefixes and keys can lead to vulnerabilities such as data corruption, unauthorized access, or unintended overwrites.

## Notice the Specific Storage Processing in NEO N3

When working with data storage in NEO N3, it's essential to carefully plan where and how data is stored. Specifically, developers should use a prefix + key strategy to organize and secure stored items.

For example, when implementing a token contract complying with NEP-17, using distinct prefixes for different types of data (e.g. allowances, metadata) ensures that there’s no overlap or accidental overwriting of critical data. This practice also helps in maintaining clarity and avoiding conflicts, particularly when the contract grows in complexity or integrates with other systems.

## Example Venerable Scenario

Consider an implementation of an NEP-17 token contract where you need to store balances and allowances securely. Here’s an example of how to use prefixes for storing data:

```csharp
public class MyToken : Nep17Token
{
    private static readonly byte[] GasBalancePrefix = new byte[] { 0x00 };
    private static readonly byte[] AllowancePrefix = new byte[] { 0x01 };

    // Method to retrieve GAS balance
    public static BigInteger GetGasBalance(UInt160 account)
    {
        byte[] key = GasBalancePrefix.Concat(account.ToArray());
        return Storage.Get(Storage.CurrentContext, key).ToBigInteger();
    }

    // Method to set GAS balance
    private static void SetGasBalance(UInt160 account, BigInteger balance)
    {
        byte[] key = GasBalancePrefix.Concat(account.ToArray());
        Storage.Put(Storage.CurrentContext, key, balance);
    }

    // Additional methods for allowances, etc., using AllowancePrefix
}
```

In this example, `0x00` is used as the prefix for storing user balances, and `0x01` is used for allowances. It seems that everything is organized well. You should notice that `0x00` and `0x01` has been used by the parent class `Nep17Token`'s parent class [TokenContract](https://github.com/neo-project/neo-devpack-dotnet/blob/master/src/Neo.SmartContract.Framework/TokenContract.cs). Nep-11 also used the `0x02` to `0x04` for specific purpose.

## Mitigation Strategies

To further enhance storage security in NEO N3, consider the following strategies:

1. **Read all Dependent Contracts and Start Prefixes from Unused Prefix for New Data Types**:
   - When introducing new types of data to your token contract, start with prefixes like `0x10` to ensure separation from existing data types like balances and allowances. This practice keeps the storage well-organized and minimizes the risk of conflicts.

2. **Avoid Using 0xff as a Prefix and Read all Attributes Your Contracts Use**:
   - The prefix `0xff` is typically reserved for special purposes, such as marking [NoReentrantAttribute](https://github.com/neo-project/neo-devpack-dotnet/blob/00832fdaca2f670322d8a48758f0f23e944e7c7b/src/Neo.SmartContract.Framework/Attributes/NoReentrantAttribute.cs#L23) tags in contracts that using this attribute. Using this prefix for other data types could lead to unintended interactions or vulnerabilities, so it’s best to avoid it altogether.

3. **Do not Use Strings and Characters as Key Prefix at the Same Time**:
   - Some develpers like use a string like `owner` or `balance` as the key prefix. It's also OK to develope in this way. While you should make sure no two keys could have potential conflicts.

4. **Use Fixed-Length Keys**
    - Whenever possible, avoid using variable-length keys, especially when combining prefixes with multiple keys (e.g., prefix + key1 + key2). Fixed-length keys ensure consistency and prevent key collisions, making the storage structure more predictable and reducing the risk of data conflicts.

## Conclusion

Data storage security in NEO N3 smart contracts is essential to maintaining the integrity and safety of your applications. By understanding how storage works, using prefix + key strategies, and avoiding common pitfalls like reusing sensitive prefixes, developers can build robust and secure contracts. Always prioritize careful planning and auditing of your storage practices to prevent data-related vulnerabilities.

## Further Reading

For more information on smart contract storage and security in NEO N3, check out the following resources:

- [NEO Documentation on Storage](https://docs.neo.org/docs/n3/reference/scapi/framework/services/Storage.html)
- [NEP-17 Token Standard Specification](https://developers.neo.org/docs/n3/develop/write/nep17)
