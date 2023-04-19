## Full Report


# Gas Findings:

# [G-01] - x += 1 is LESS gas efficient compared too x = x - 1 and x++

```solidity
    // Example
    underlyingValue +=
        (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
            derivatives[i].balance()) /
        10 ** 18;

    // Can be converted too:

    underlyingValue =
    underlyingValue + (derivatives[i].ethPerDerivative(derivatives[i].balance()) *
        derivatives[i].balance()) /
    10 ** 18;
```

SafEth.sol - [Lines-71-75](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L72-L75), [Line-95](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L95), [Line-172](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L172), [Line-188](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L188), [Line-192](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L192)

# [G-02] - Duplicate require/revert statements can be converted into a modifier or function to save gas on deployment

```solidity
    // Can be converted into a modifier or function
    require(sent, "Failed to send Ether");
```

WstEth.sol - [Line-77](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/WstEth.sol#L77)


# Below this are findings that I found before I read the automated findings report and should NOT be counted for awards


[G-03] - Require statements can be converted into custom errors to save gas

When a require() statement fails, it causes the transaction to revert and any gas used in the execution of the transaction up to that point is lost. However, if you use custom errors instead of require() statements, you can control when an error is thrown and potentially save gas by avoiding the unnecessary execution of code.

```solidity 
    // Example
    require(rethBalance2 > rethBalance1, "No rETH was minted");

    // Can be converted too:

    if (rethBalance2 > rethBalance1) {
        throw new Error("No rETH was minted);
    }
```

Reth.sol - [Line-113](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/Reth.sol#L113), [Line-200](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/Reth.sol#L200)

SafEth.sol - [Lines-64-66](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L64-L66), [Line-109](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L109), [Line-127](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L127)

SfrxEth.sol - [Line-87](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/SfrxEth.sol#L87)

WstEth.sol - [Line-66](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/WstEth.sol#L66), [Line-77](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/derivatives/WstEth.sol#L77)

[G-04] - Setting a state variable to its default value is a waste of gas and should be avoided

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas. In the code linked below, the initialization of uint256 marketIndex = 0 can be removed as that is its implied value anyways and doesn't need to be defined.

```solidity
    // Example 1:
    uint256 underlyingValue = 0;

    // Can be converted too:
    
    uint256 underlyingValue;

    // Example 2:
    for (uint i = 0; i < derivativeCount; i++)

    // Can be converted too:

    for (uint i; i < derivativeCount; i++)

```

SafEth.sol - [Line-68](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L68), [Line-71](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L71), [Line-83](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L83), [Line-113](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L113), [Line-140](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L140), [Line-147](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L147), [Line-170](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L170), [Line-171](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L171), [Line-191](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L191)

[G-05] - ++i costs less gas compared too i++

++i costs less gas compared to i++ for unsigned integer, as pre-increment is cheaper (about 5 gas per iteration). This statement is true even with the optimizer enabled. This is because pre-increment can be implemented as a single opcode, while post-increment or i += 1 requires multiple opcodes to execute.

```solidity
    // Example
    for (uint i = 0; i < derivativeCount; i++)

    // Can be converted too:

    for (uint i; i < derivativeCount; ++i)
```

SafEth.sol - [Line-71](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L71), [Line-84](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L84), [Line-113](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L113), [Line-140](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L140), [Line-147](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L147), [Line-171](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L171), [Line-191](https://github.com/code-423n4/2023-03-asymmetry/blob/d016bef40dc745b6a16e18d83bd4f7d776245903/contracts/SafEth/SafEth.sol#L191)

