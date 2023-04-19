# Audit for Frankencoin (Code4rena)

# Findings Summary:

| ID      | Title | Severity | 
| ------- | ----- | -------- |
| M-01    | Unchecked return value for .transfer / .transferFrom call  | Medium |
| M-02    | Unbounded loop length could cause denial of service  | Medium |
| M-03    | Possibly dividing by 0 if the totalSupply ever returns a zero  | Medium |
| G-01    | x += y costs more gas than x = x + y for state variables  | Gas |
| G-02    | Initializing variables to their default value is a waste of gas  | Gas |


## Full Report

# [M-01] - Unchecked return value for .transfer / .transferFrom call
Checking the return values for any sort of transfer call is very important to make sure that the transfer executed properly. It is usually good to add a require statement that checks the return values to make sure everything worked properly. You could also swap out the .transfer() with the safeTransfer() method unless you are absolutely sure that the given token transfer reverts all changes in case of failure

```solidity
    collateral.transferFrom(msg.sender, address(this), newCollateral - colbal);
```

### Recommendation:

- Use a require statement to check the return values from the transfer or just use the safeTransfer() method to transfer the tokens.

Position.sol - [Line-138](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L138), [Line-228](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L228), [Line-253](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L253), [Line-269](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L269)

# [M-02] - Unbounded loop length could cause denial of service
Loops that do not have a fixed number of iterations, for example, loops that depend on storage values, have to be used carefully: Due to the block gas limit, transactions can only consume a certain amount of gas. Either explicitly or just due to normal operation, the number of iterations in a loop can grow beyond the block gas limit which can cause the complete contract to be stalled at a certain point.

```solidity
    function votes(address sender, address[] calldata helpers) public view returns (uint256) {
        uint256 _votes = votes(sender);
        // helpers array is not limited by its length
        for (uint i=0; i<helpers.length; i++){
            address current = helpers[i];
            require(current != sender);
            require(canVoteFor(sender, current));
            for (uint j=i+1; j<helpers.length; j++){
                require(current != helpers[j]); // ensure helper unique
            }
            _votes += votes(current);
        }
        return _votes;
    }
```

### Recommendation:

- Use a require statement or conditional checking to make sure that the helpers array does not exceed a certain length

Equity.sol - [Line-192](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192), [Line-196](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L196), [Line-312](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L312)

# [M-03] - Possibly dividing by 0 if the totalSupply ever returns a zero
If the total supply ever becomes zero, it could potentially disrupt the functioning of other contract features.

```solidity
    return VALUATION_FACTOR * zchf.equity() * ONE_DEC18 / totalSupply();
```

### Recommendation:

- Do some sort of validation before the calculation to make sure that the returned value from calling the totalSupply() function is not zero

Equity.sol - [Line-109](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L109)

# Gas Findings:

# [G-01] - x += y costs more gas than x = x + y for state 
```solidity
    limit -= reduction + _minimum;
```

Position.sol - [Line-99](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L99), [Line-196](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L196), [Line-242](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L242), [Line-295](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L295), [Line-309](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L309), [Line-330](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Position.sol#L330)

# [G-02] - Initializing variables to their default value is a waste of gas
```solidity
    for (uint i=0; i<helpers.length; i++)
```

If a variable is not set/initialized, it is assumed to have the default value (0 for uint, false for bool, address(0) for address…). Explicitly initializing it with its default value is an anti-pattern and wastes gas. In the similar code above, the initialization of uint256 i = 0 can be simplified to uint256 i.

Equity.sol - [Line-192](https://github.com/code-423n4/2023-04-frankencoin/blob/1022cb106919fba963a89205d3b90bf62543f68f/contracts/Equity.sol#L192)


