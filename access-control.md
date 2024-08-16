# Access Control and Permissions in NEO N3 Smart Contracts

Access control is a crucial component of smart contract security, ensuring that only authorized users or entities can interact with specific functions. In the context of NEO N3, understanding and implementing robust access control mechanisms is essential to protect your smart contracts from unauthorized access and potential exploits.

## Understanding Access Control in NEO N3

In smart contract development, access control mechanisms are used to define who can perform certain actions within a contract. This involves verifying the identity of users or contracts attempting to interact with the contract's functions. Properly implemented access control ensures that only trusted entities can execute sensitive operations, thereby preventing unauthorized access and maintaining the integrity of the contract.

## Notice the Specific Permissions Way in NEO N3

NEO N3 introduces specific considerations for managing permissions that developers need to be aware of:

### 1. **Use `CheckWitness` Instead of Direct User Comparison**:
   - In NEO N3, instead of comparing a user directly to the transaction sender (e.g., `tx = (Transaction)Runtime.ScriptContainer;` and then `user == tx.Sender`), it is recommended to use the `CheckWitness` function. This method verifies that a particular user or entity has signed the transaction, confirming their authorization. This approach is more secure and aligns with NEO N3's best practices for access control.
   - Compared to EVM-based platforms, while not exactly the same, `tx.Sender` in NEO N3 functions more similarly to `tx.origin` on the EVM platform. Meanwhile, `Runtime.CallingScriptHash` can resemble `tx.sender` in specific scenarios, such as when one contract calls another and the called contract retrieves the caller's script hash through this method.
   - However, it's important to note that when an external user directly invokes a contract, `Runtime.CallingScriptHash` does not return the user's address but rather the script hash of the transaction. In such cases, `tx.Sender` must be used to obtain the external user's address. Because of this, some contract developers choose to initialize their contracts by setting `tx.Sender` — which corresponds to the deployer's address—as the contract's owner during deployment. Consequently, most contracts opt to require users (whether external users or calling contracts) to explicitly provide the authorized user's address as a parameter.
   
#### Example Scenario

In a typical smart contract, you might need to verify if a user has admin privileges before allowing certain actions. An incorrect approach is to directly compare the user with the transaction sender using user == tx.Sender. This method is insecure in NEO N3, as it does not leverage the platform’s best practices for verifying user permissions and can lead to unauthorized access. Here is an example from [DogeRift](https://github.com/DogeRift/DogeRift/blob/50dcbc88d61c63ab1978786b8c0154de44f72f12/contracts/DogeRift/src/DogeRift.cs#L37-L44) in the first verison.

```csharp
private static Transaction Tx => (Transaction) Runtime.ScriptContainer;
private static void VerifyOwner()
{
    ByteString owner = ContractMetadata.Get("Owner");
    if (!Tx.Sender.Equals(owner))
    {
        throw new Exception("Only the contract owner can do this");
    }
}
```

In this scenario, the contract incorrectly assumes that the sender of the transaction (tx.Sender) is the user to be verified. This approach can be bypassed or exploited, especially when dealing with contracts that interact with other contracts. Notice that, as mentioned in our previous discussion on [reentrancy](./reentrancy-attack.md), NEP-11 and NEP-17 token transfers can also trigger callback functions like `onNEP11Payment` and `onNEP17Payment`. If the contract owner interacts with an unknown contract — whether through a simple transfer to a third party or other interactions to malicious contracts could exploit this by invoking functions that rely on this `VerifyOwner` method for authorization. This could result in the malicious contract gaining control over the vulnerable contract.

#### Mitigation Strategies

See [DogeRift](https://github.com/DogeRift/DogeRift/blob/348eb09d73cf8c5add8847d3c8cad090add9bb85/contracts/DogeRift/src/DogeRift.cs#L37-L44)'s revised version. Directly using `CheckWitness` could solve this.

```csharp
private static void VerifyOwner()
{
    ByteString owner = ContractMetadata.Get("Owner");
    if (!Runtime.CheckWitness((Neo.UInt160)owner))
    {
        throw new Exception("Only the contract owner can do this");
    }
}
```


### 2. **Avoid Your Contract Address as the User**:

- When using `CheckWitness`, ensure that your contract logic does not allow other users to designate your contract's own address as the user being verified. If this is not properly checked, the contract might inadvertently pass the `CheckWitness` verification, leading to potential security vulnerabilities. It's essential to validate that only legitimate users or external contracts are verified.

### 3. **Do Not Implement the `verify` Method**:

- It is advisable not to implement the `verify` method in your contracts unless absolutely necessary. The `verify` method, if present, is automatically invoked by the NEO VM, which can introduce unintended behaviors or security risks if not carefully managed. Avoiding this method helps maintain predictable and secure contract behavior.

## Conclusion

Access control and permissions are fundamental to the security of NEO N3 smart contracts. By understanding the specific nuances of NEO N3’s permission model, including the proper use of `CheckWitness` and avoiding risky practices like implementing the `verify` method, developers can significantly reduce the risk of unauthorized access. Always prioritize security in your contract design to build robust and trustworthy applications.

## Further Reading

For more insights into smart contract security on NEO N3, consider reading the following resources:

- [Official NEO Documentation on Smart Contracts](https://docs.neo.org)
- [Smart Contract Access Control Best Practices](https://github.com/securing/SCSVS/blob/master/1.2/0x11-V2-Access-Control.md)
