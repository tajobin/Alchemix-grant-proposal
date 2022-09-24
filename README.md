# Grant Proposal

I have sectioned this proposal into two separate projects, one dealing with the completion of the aggregation bridge and the other dealing with pools and bridges focused on providing additional features that are made possible by the bundling of transactions with aztec-connect.

##  Completing the Aggregation Bridge

To complete the aggregation bridge that was worked on during the money-legos hackathon two main features have to be added. We need to give users the ability to repay their debt with alTokens instead of liquidation their collateral and we also need to add a safety feature that functions as an escape hatch to allow users to exit in a situation where the base exit flow is not viable.

Aztec-bridges requires that the two input variables are exactly equal, it is therefore not possible to input both the debt and the token representing shares in the same bridge interaction, since the debt and token representing their shares are different. We therefore have 4 different way of pay repaying and withdrawing collateral: 

1. Enter with the tokens representing shares, use the built in functionality in the alchemist to liquidate collateral to repay the debt.
2. Enter with the tokens represeting shares, flashlaon the underlying token, swap on a DEX to alTokens and repay the debt. 
3. Use two separate bridge interactions to first repay the debt and then on the second interaction exit with the collateral. This is done by issuing a virtualToken in first interaction that proves that the debt has been repaid and is equal to the shares the users have. They can then interact with the bridge again to exit.  
4. Similar to 2. but returns yieldToken.

Option 3. and 4. are safety measures needed to guaranty that users funds are never trapped and can always be claimed by users. In day to day use users will simply interact with a front-end that calculates if 1. or 2. is to be used with the decision passed in through the ```auxData``` variable.

In the following scenarios 3. and 4. should be used:

Scenario when both 3. and 4. can be used: If ```maximumLoss``` is reached and the only option is to withdraw yieldTokens by repaying the debt.

Scenario when 3. should be used: If the alToken exchange rate is low enough to warrant a flashlaon repayment but the flashlaon route is compromised.

Scenario when 4. should be used: If ```maximumLoss``` is reached and the only option is to withdraw yieldTokens by repaying the debt the but the users does not have the funds to repay the debt and wishes to liquidate collateral.

No grant requested for this bridge. $4000 already received in the money-legos hackathon from alchemix and potentially $2000 from Aztec as payment for a bounty for a requested bridge. 

## Additional Pools and Bridges
The aggregation bridge is built to mirror the standard interactions with alchemix but we could go further and create bridges that bundle multiple transactions into one. 

### Internal Yield Boost
#### Leverage
It is possible to create a leverage position on alchemix by recursively converting debt taken out into collateral. It can also be done by taking out a flashloan to create the position by depositing the loaned funds as collateral and using the debt taken out combined with the users funds to repay the loan. On alchemix a leverage of 2x is possible. 

Alchemix offers a unique leverage position since the collateral are yield-bearing, the "cost" of creating the position is known upfront and the position is never force liquidated. There is no actual cost but since the exchange rate of alTokens fluctuates we can calculate the max loss if the peg increases from its current value to 1. With the current alUSD/DAI rate the max cost is currently ~0.3%.  

When yield harvested the leverage is decreased <2 but it can be brought up to 2 again with the next pool interaction by accounting for the extra debt that can be taken out when taking out a flashloan. Doing this manually without a pool would be both expensive and cumbersome since users would have to interact with the alchemist multiple times or take out a flashloan when yield has been harvesed. Users will not have to interact with the alchemix to rebalance the position after they have entered the pool, instead it is re-balanced each time a new cohort enters. This will be done at no extra cost for the new cohort since they would have performed the exact same actions. 

If the yield tokens is decreased in value and ```maximumLoss``` is reached, users can either wait for it to recover or decide to exit the position at a loss. No re-balancing is done to decrease the leverage since no forced liquidation is possible, users decide on an individual basis if they want to liquidate assets at a loss or wait for a potential recovery.

Leverage pools can be created for all yield tokens currently supported by alchemix, for both the ETH and USD alchemist. The only requirement is that the underlying token can be flashloaned.

Creating leverage position with aggregated funds could put pressure on the altoken peg, as an additional safety feature the admin can set a threshold on each pool.

Requested Grant: $2000

#### Index Pools 

A pool with a basket of shares in yield tokens. It can be re-balanced to stay at a certain ratio with each bridge interaction. This would function as an index of the yield tokens that the alchemist supports with automatic re-balancing. 

Users can create their own "index" by entering into different aggregation pools but the gas cost would be higher and they would have to balance it themselves. We should still be careful to not deploy to many index pools since we still require a decent batch sizes to actually save on gas compared to multiple aggregation interactions.

Requested Grant: $500
### External Yield Boost 

The external yield boosting bridge will support pools strategies that use the debt taken out to earn additional yield. Pools for popular strategies can be created by the admin as long as they follow a specific interface. The goal of these pools is to further bundle user interactions to save even more gas and provide yield strategies that are not directly supported by an alchemist. These pools can also perform pre-defined actions when each cohort enters e.g. to re-balance their positions.  

Additional research will be done to explore what alchemist and aztec users are interested in. It is important that users are interested in these pools to maximize the batch size and save more gas.

This bridge can be launched with 1-3 different pool types with the ability for the admin to add new strategies to match the demand from users.

A A simple example of a pool would be a pool that deposits the alTokens into a curve pool to earn additional yield. The pools can be re-balanced with each cohort to stay at a certain collateralization ratio or to repay the debt as quickly as possibly by using the yield earn on the debt to repay the debt. 

The bridge will allow the admin to deploy new pool strategies but the admin will not be able to atler any pools that have already been deployed.

Requested Grant: $2000

## Continious Funding Pool 

Pools can be created where a community can continuously fund a beneficiary. Cohorts of users enter into the pool and deposits capital and with each entry the harvested yield is sent to the beneficiary. Each users will therefore continuously fund a beneficiary without having to do anything other than depositing to the pool a single time.

These pools can be configured to also take out debt to issue a one time donation to the beneficiary in addition to the earned yield. 

Example: Gitcoin matching pool used as beneficiary in a pool where all the harvested yield is continuously donated to the gitcoin pool.

If users wish to fund a beneficiary and then recoup the cost they should simply use the aggregation bridge and use the debt to fund the beneficiary. The funding pool should be used when users want to continuously fund a beneficiary with their yield.

Requested Grant: $500
