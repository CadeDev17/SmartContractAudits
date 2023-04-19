-------------------------------------------------- Context Questions --------------------------------------------------
What are the actors in this contract? 
- Anyone: Anyone can propose new minting mechanisms.
- Contributors to the stability reserve: Anyone who contributed more than 3% to the stability reserve can veto new minting mechanisms.
- System participants: They can open and manage positions using the Frankencoin system.
- Qualified pool share holders: They can veto new positions by calling the "deny" method on the position.

    -- What privileges do they have?
    - Anyone can propose new minting mechanisms.
    - Contributors to the stability reserve can veto new minting mechanisms.
    - System participants can open and manage positions.
    - Qualified pool share holders can veto new positions by calling the "deny" method on the position.

What are the assets?
- Frankencoin (ZCHF)
- Other ERC20 collateral that is semi-stable

    -- Where are they held?

Who is allowed access to what and when?

What are the assumed trust relationships?

What is the overall threat model?

    -- Potential attack vectors?
    - Bad collateral ERC20 token
    - Veto without contributing 3%
    - Pool shares not exactly 3x the the equity reserves of the system
    - Hard coded pegs

-------------------------------------------------------- NOTES --------------------------------------------------------

Frankencoin -> (ZCHF) = oracle free, collateralized Swiss Franc stablecoin
    -- Aims to solve issues including trusting centralized third parties, dependence on centralized oracles, and limitations in the types of collateral available to mint the stablecoin.

Anyone can propose new minting mechanisms
    -- Anyone who contributes 3% to the stability reserve can veto new minting mechanism proposals

Two current minting mechanisms:
    -- One is a simple swap contract to convert XCHF (CryptoFranc, fiat-collateralized stablecoin) into ZCHF and back.

    -- The other is a novel collateralized minting mechanism based on auctions. This does NOT depend on external oracles and is very flexible when it comes to the use of collateral. It supports any collateral with sufficient availability on the market. However, its liquidation mechanism is slower than that of other collateralized stablecoins, making it less suitable for highly volatile types of collateral.

Main Concerns:
    -- Minting of Frakencoin tokens (ZCHF)
    -- The purchase of reserve pool shares
    -- The stablecoin conversion between XCHF and ZCHF
    -- Nobody can mint infinite amounts of ZCHF (Frankencoin) 
    -- No one's deposited collateral can be stolen
    -- It should not be possible to mint Frankencoins in a way that the FPS holders disapprove.
    -- The FPS issuance and redemption mechanism should be path-independent, an attacker should not be possible to drain capital from the equity contract through repeated interaction with the issuance and redemption mechanism.
    -- It is impossible for an attacker to steal someone's collateral
    -- It is impossible for a position owner to mint beyond the limit of a position
    -- It is impossible for a position owner to withdraw collateral without having repaid the corresponding amount of previously minted Frankencoins

Auction Mechanism:
    -- Two piles of collateral for sale
        - One from the owner of the position
        - One from the challenger who questions the soundness of the position
        - The final price determines which of the two the bidder gets
    
    -- If the price is high enough, the bidder gets the collateral of the challenger
    -- But, if the price is too low, the bidder gets the collateral from the position owner
        - This ensures that the position owner has no incentive to overbid
        in order to avert a challenge. Doing so would be costly for them as they would have to buy collateral above the market price from the challenger as often as the challenger repeats it.

Pool Shares:
    -- The bonding curve for issuance and redemption is set such that the market cap of all pool shares is always three times the equity reserves of the system, with the supply being proportional to the cubic root of the market cap and the price being proportional to the cubic root squared.

Initiating a position:
    -- Create a completely new one with arbitrary parameters
    -- Clone an existing position

    -- Creating a completely new position is for advanced users and not exposed in the frontend. Cloning an existing position is the faster way of obtaining Frankencoin against a collateral and supported in the frontend

Random Notes:s
-- When someone mints fresh Frankencoins against a collateral, that is considered a position.
    -- A position is owned by exactly one owner. Initially the person who created the position would be the owner but it is transferrable through the Ownable contracts.
    -- Anyone can challenge a position if they believe that the liquidation price is below the true value of the collateral, triggering an auction that serves to purpose of determining the market price of the collateral.

-- The only smart contract that is approved to create new positions is the minting hub