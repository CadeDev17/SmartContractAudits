# Practice Audit for Y2K Finance (Code4rena)

# Findings Summary:

| ID      | Title | Severity | 
| ------- | ----- | -------- |
| G-01 | Using boolean values for storage variables incurs overhead   | Gas      |
| G-02 | require() statements can be converted into custom errors and save gas   | Gas      |
| G-03 | Using > 0 costs more gas than != 0 when used on a uint in a require() statement   | Gas      |
| G-04 | Overused modifiers can be converted into a function to save gas   | Gas      |
| G-05 | ++i costs less gas compared too i++   | Gas      |
| G-06 | Setting a state variable to its default value is a waste of gas and should be avoided   | Gas      |
| G-07 | x += 1 is LESS gas efficient compared too x = x - 1   | Gas      |

## Full Report


# Gas Findings:

[G-01] - Using boolean values for storage variables incurs overhead

You can avoid 100 gas for by avoiding the extra SLOAD and then you can also save ~20,000 units of gas
when you are changing from False -> True after having been true in the past. 

The way that you can avoid this is by usint uint256(1) for true and uint256(2) to indicate when something is false.
Using this method allows you to avoid the extra SLOAD to read the slots contents, replace the bits that are taken up by the boolean and then writing it again.

Example:
```solidity
    if(insrVault.idExists(epochEnd) == false || riskVault.idExists(epochEnd) == false)

    // Can be converted too:

    if(insrVault.idExists(epochEnd) == 2 || riskVault.idExists(epochEnd) == 2)
```

Controller.sol - [Line-211](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Controller.sol#L211)

[G-02] - require() statements can be converted into custom errors and save gas

When a require() statement fails, it causes the transaction to revert and any gas used in the execution of the transaction up to that point is lost. However, if you use custom errors instead of require() statements, you can control when an error is thrown and potentially save gas by avoiding the unnecessary execution of code.

Example:
```solidity
    require((shares = previewDeposit(id, assets)) != 0, "ZERO_SHARES");

    // Can be converted too:

    if (shares = previewDeposit(id, assets) != 0){
        throw new Error("zero_shares") // or call error function/modifier
    }
```

SemiFungibleVault.sol [Line-91](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/SemiFungibleVault.sol#L91), [Line-116](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/SemiFungibleVault.sol#L116), [Line-165](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L165), [Line-96](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/rewards/StakingRewards.sol#L96), [Line119](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/rewards/StakingRewards.sol#L119), [Line-217](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/rewards/StakingRewards.sol#L217), [Line-226](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/rewards/StakingRewards.sol#L226), [Lines-23-25](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/oracles/PegOracle.sol#L23-L25), [Lines-98-106](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/oracles/PegOracle.sol#L98-L106), [Lines-121-129](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/oracles/PegOracle.sol#L121-L129)


[G-03] - Using > 0 costs more gas than != 0 when used on a uint in a require() statement

!= 0 costs 6 less GAS compared to > 0 for unsigned integers in require statements with the optimizer enabled. Detailed Explanation

```solidity
    require(msg.value > 0, "ZeroValue");

    // Can be converted too:

    require(msg.value != 0, "ZeroValue");
```

Vault.sol [Line-187](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L187)

rewards/StakingRewards.sol [Line-134](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/rewards/StakingRewards.sol#L134)

[G-04] - Overused modifiers can be converted into a function to save gas

The reason for this is that each time a modifier is used, it is essentially "copied and pasted" into the code at the location where it is called. If you define the logic of the modifier as a function, you can simply call the function at the location where the modifier would normally be used. This can help save gas by avoiding the need to "copy and paste" the modifier code each time it is used.

The most used modifiers in this contract are the onlyFactory() and marketExists() modifiers, those could be converted to functions which would save gas by not copying/pasting so many times throughout the code. Then in the functions where these modifiers were previously called, you can just invoke these functions immediately within the function block and it will work exactly the same but cost much less gas.

```solidity
    modifier onlyFactory() {
        if(msg.sender != factory)
            revert AddressNotFactory(msg.sender);
        _;
    }

    // Can be converted too

    function onlyFactory() {
        if(msg.sender != factory)
            revert AddressNotFactory(msg.sender);
        _;
    }
```

Vault.sol [Line-63](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L63), [Line-79](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L79)

VaultFactory.sol [Line-129](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/VaultFactory.sol#L129)

[G-05] - ++i costs less gas compared too i++

++i costs less gas compared to i++ for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled. This is because pre-increment can be implemented as a single opcode, while post-increment or i += 1 requires multiple opcodes to execute.

Vault.sol [Line-443](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/Vault.sol#L443)

[G-06] - Setting a state variable to its default value is a waste of gas and should be avoided

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas. In the code linked below, the initialization of uint256 marketIndex = 0 can be removed as that is its implied value anyways and doesn't need to be defined.

VaultFactory.sol [Line-159](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/VaultFactory.sol#L159)

[G-07] x += 1 is LESS gas efficient compared too x = x - 1

```solidity
    marketIndex += 1;

    // can be converted to:

    marketIndex = marketIndex + 1
```

VaultFactory.sol [Line-195](https://github.com/code-423n4/2022-09-y2k-finance/blob/2175c044af98509261e4147edeb48e1036773771/src/VaultFactory.sol#L195)