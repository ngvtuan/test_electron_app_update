# Pastel Smart Pooling Proposal

******The Problem We Are Trying to Solve:**

In order to create a Supernode (SN), the user must acquire 5 million PSL coins and then pledge these as collateral to back the SN. The SN is only valid while those coins have not been moved or spent.

However, for security reasons, it's possible to set this up in what is called a "cold/hot" configuration, where the coins are stored in the wallet of a user's home machine (the cold node), and then the remote cloud machine that is running the actual SN instance (the hot node) does not have direct access to the coins, which minimizes the chances of the remote machine getting hacked (which is more likely because it must have a static IP address that anyone can look up and see, making it a target). This works essentially by having the user prove that they own the coins on the cold node by signing a message using the private key of the address (this is the `masternode genkey` command), and the output of this is placed in the `masternode.conf` file on the hot node.

But many users either can't afford to acquire the 5 million PSL and want to stake a smaller amount of coins. But the protocol does not and never will support that, since it's not a PoS network— it uses PoW, with an exception for the SNs that actually do useful services on the network. But we want to offer these users a way to pool together their resources so that several of them can pledge their coins together to get the needed amount. 

One issue to this would be that we would want the group of users to actually pool together more than the required 5 million PSL, say 6 million PSL, so that there would be a cushion in case some of them decided to move their funds.

Another factor here is that there are 3rd party companies that users want to be able to hire to handle running the machines on their behalf, but these companies will only work on a non-custodial basis.

The biggest issue we need to deal with is **security:** ensuring that users who participate in any structure like this are not taking credit risk with their coins, and do not have to trust anyone else. They should always retain full custody and control over their coins at all times.


******Proposed Solution:**

The new smart pooling feature allows multiple users to pool their Pastel coins (PSL) together to create a Supernode. Critically, this pooling does **not** require the users to actually send their coins to gather all of the coins at a single address. Instead, user would merely **pledge** a certain number of coins residing in an address in their PSL wallet to a particular pool by signing a message using the private key associated with that address. So long as the user doesn’t move the pledge coins from that address, they will continue to be a member of the pool and receive their share of net rewards after fees.

The process involves creating and broadcasting various types of blockchain tickets:


1. **Smart Pool Initiation Ticket**: Created by the pool operator, this ticket starts the subscription period and defines the parameters for the smart pool, including the requirement for overcollateralization.
2. **Smart Pool Participation Ticket**: Created by each user who wants to join the pool, this ticket includes the amount of PSL pledged and signatures proving ownership of the PastelID and PSL coins.
3. **Smart Pool Activation Ticket:** Created by the pool operator to finalize the establishing of the Smart Pool with given participants and terms. 

Once the total amount of pledged coins meets the required overcollateralized amount, the operator creates a **Smart Pool Activation Ticket** to launch the Supernode. The Supernode then begins operation and starts earning rewards, which are distributed among the participants in the smart pool net of the fee percentage to the operator of the pool.

To handle the situation where a participant moves their pledged coins, the network continuously monitors the blockchain for transactions involving participant addresses. If a participant's balance decreases, they are invalidated and their share of the reward is removed. The reward shares of the remaining participants are then recalculated. The reward shares of the remaining participants are then recalculated.

If the total pledged amount falls below the required amount, the smart pool enters a "replenishment mode" and a new subscription period begins. If the total pledged amount is not restored by the end of the replenishment period, the smart pool is dissolved, the Supernode is shut down, and the remaining pledged coins are returned to the participants.


----------

******Smart Pool Initiation Ticket Structure:**


    {
        "ticket": {
            "type": "smartpool_init",
            "version": "int",
            "smart_pool_id": "string",
            "subscription_period": "int",
            "required_coins": "int",
            "overcollateralization_factor": "float",
            "operator_fee_percentage": "float",
            "supernode": {
                "supernode_pastel_id": "string"
            },
            "operator": {
                "company": "string",
                "pastelid": "string",
                "operator_fee_address": "string",
                "agreement_signature": "bytes"
            },
            "smart_pool_signature": "bytes"
        }
    }

Here's a breakdown of each field in the `smartpool_init` ticket structure and their roles in the overall scheme:


1. **type**: This specifies that the ticket is for initializing a smart pool. It helps in categorizing and identifying the type of ticket on the network.
2. **version**: This is an integer representing the version of the smart pool ticket. It allows for future updates and changes to the ticket structure.
3. **smart_pool_id**: A unique identifier for the smart pool. It helps in linking all tickets and transactions related to this specific smart pool.
4. **subscription_period**: An integer representing the time window (measured in Pastel Blocks) for users to join the smart pool. It helps in setting the boundaries for when the pool can accept new pledges.
5. **required_coins**: This specifies the minimum number of Pastel coins (PSL) that must be pledged to create a Supernode. It sets the base collateral requirement for the smart pool.
6. **overcollateralization_factor**: A number indicating the extra amount of coins needed in addition to the `required_coins`. It provides a safety net in case some participants withdraw their pledges. An `overcollateralization_factor` of 1.5 would mean that, instead of the minimum of 5 million PSL required to create a Pastel Supernode, the Smart Pool would require pledges totaling 1.5*5 million = 7.5 million PSL before it would be enabled. 
7. **operator_fee_percentage**: Specifies the percentage of the rewards that will be taken by the operator as a fee. It defines the compensation for the service provided by the pool operator.
8. **supernode**:
    - **supernode_pastel_id**: This is the PastelID associated with the Supernode. It serves as an identity marker for the Supernode within the Pastel Network.
9. **operator**:
    - **company**: A name given to the third-party company or entity responsible for operating the Supernode.
    - **pastelid:** A PastelID that identifies the operator on the network so they can build up a good reputation over time for reliably and competently managing Smart Pools.
    - **operator_fee_address**: The PSL address where the operator will receive their operator fee for managing the Smart Pool.
    - **agreement_signature**: A digital signature confirming the operator's agreement to the terms and conditions of the smart pool operation.
10. **smart_pool_signature**: A digital signature for the entire smart pool initiation ticket, providing an extra layer of security and verification.


----------

**Smart Pool Participation Ticket Structure**:

    {
        "ticket": {
            "type": "smartpool_part",
            "version": "int",
            "smart_pool_id": "string",
            "participant": {
                "pastel_id": "string",
                "pledged_coins": "int",
                "pledge_signature": "bytes",
                "txid": "string",
                "vout_ind": "int",
                "address_signature": "bytes",
                "reward_address": "string",
                "time_stamp": "string"
            }
        }
    }

Here's an explanation for each field in the `smartpool_part` ticket structure and their respective roles:

1. **type**: Specifies that this ticket is for participating in a smart pool. It helps classify the ticket within the network for easier identification and processing.
2. **version**: An integer that indicates the version of this participation ticket. It allows for adaptability and future revisions to the ticket structure.
3. **smart_pool_id**: A unique identifier that links this participation ticket to a specific smart pool. It ensures that the pledged coins are accounted for in the correct pool.
4. **participant**:
    - **pastel_id**: The unique PastelID of the user participating in the smart pool. It serves to identify the individual or entity making the pledge.
    - **pledged_coins**: The number of Pastel coins (PSL) pledged by the participant. It contributes to the total collateral of the smart pool.
    - **pledge_signature**: A digital signature proving ownership of the pledged coins. It serves as a security measure to confirm the legitimacy of the pledge.
    - **txid**: The transaction ID associated with the pledge. It helps in tracking and auditing the pledge within the blockchain.
    - **vout_ind**: Output index in the transaction, used to uniquely identify the pledged coins within the transaction.
    - **address_signature**: Another digital signature that confirms the address from which the pledge is being made. It adds another layer of verification.
    - **reward_address**: The address where the participant's share of the rewards will be sent. It helps in automating the reward distribution process.
    - **time_stamp**: The time at which the pledge was made. It helps in enforcing the subscription period and other time-sensitive aspects of the smart pool.
    
----------

**Reward Distribution with Operator's Fee:**
When a smart pool Supernode earns a reward, the operator's fee is first deducted based on the `**operator_fee_percentage**` specified in the Smart Pool Initiation Ticket. For example, if the total reward is 1,250 PSL and the operator's fee is 5%, the operator would receive 1,250*0.05 = 62.5 PSL.

The remaining 1,250 - 62.5 = 1,187.5 PSL would then be distributed among the participants based on their share in the pool. For details on how exiting the pool affects rewards, see the "Voluntary Exit Mechanism" section.


1. The operator creates and broadcasts the Smart Pool Initiation Ticket, starting the subscription period.
2. Any user who wants to participate in the pool creates and broadcasts a Smart Pool Participation Ticket during the subscription period. This ticket includes the unique identifier of the Smart Pool they want to join (the `**smart_pool_id**`), as well as their PastelID, the number of PSL coins they are pledging, and their signatures.
3. If a participant moves their pledged coins, their participation in the Smart Pool is invalidated.
4. The Pastel Network verifies the signatures on each ticket and the fulfillment of all conditions.
5. Now the total amount of PSL that needs to be pledged to activate the Supernode would be `**required_coins * overcollateralization_factor**`. This way, if any of the users withdraw their funds, as long as the total pledged amount is still above the `**required_coins**` value, the Supernode remains operational.
6. This also means that the system needs to continuously monitor the pledged amount to ensure it is still above the required threshold and to notify the participants if the pledged amount is close to the threshold or falls below it. In the latter case, the system could initiate a new subscription period to replenish the pool.

Once the required amount of 6 million PSL is pledged, the operator can move to launch the Supernode. Here are the steps:


1. **Create a Smart Pool Activation Ticket**: The operator would create a new type of ticket that signifies the activation of the smart pool. This ticket would reference the `**smart_pool_id**` and include a list of the valid participants (i.e., those who have not moved their pledged coins). The operator would sign this ticket with their PastelID.

**Smart Pool Activation Ticket Structure**:

    {
        "ticket": {
            "type": "smartpool_activate",
            "version": "int",
            "smart_pool_id": "string",
            "participants": [
                {
                    "pastel_id": "string",
                    "address": "string"
                }
            ],
            "operator_signature": "bytes"
        }
    }
    


1. **Broadcast the Activation Ticket**: The operator would then broadcast this ticket to the network. Nodes in the network would verify the ticket, confirming that all the participants are valid (i.e., they have not moved their pledged coins) and that the operator's signature is valid.
2. **Launch the Supernode**: Once the activation ticket is verified and added to the blockchain, the operator can launch the Supernode. The launch process would involve running the necessary software on a server and pointing it to the `**supernode_address**` from the initial smart pool ticket.
3. **Verification of the Supernode**: The network would then verify the Supernode, checking that it is running correctly and that it has the correct amount of PSL pledged to it.
4. **Start of Supernode Operation**: Once the Supernode is verified, it can begin operation. It will now be part of the Pastel Network and can begin providing services and earning rewards.

This process ensures that the launch of the Supernode is transparent and verifiable. The use of the activation ticket allows the network to confirm that all the conditions for launching the Supernode have been met.

Currently, the code in **pasteld** to validate a newly created SN checks the Supernode’s *genkey* and *txid/vout* against the blockchain data the ensure that this txid/vout contains *exactly* 5 million PSL and that the *genkey* matches that *txid/vout*. We would need to modify that logic to also support this alternative path to validate a new SN, since there would no longer be a single txid/vout that contains precisely 5 million PSL. 

Instead, there would be this set of valid, signed smart tickets that prove that the group of users pledging at least 6 million PSL. Not that, if users pledge more than 6 million PSL, it would still work, but it would depress the returns to each of the users. So users would have an incentive to only subscribe to pools that aren't excessively oversubscribed so as to maximize their own individual returns.

Here's a high-level outline of how that modification might look:


1. **Detect Smart Pool Activation**: The first step is to detect when a Smart Pool Activation ticket has been submitted to the network. This could be done by checking the `**type**` field of incoming tickets.
2. **Verify Participants**: For each participant in the Activation ticket, the validation process should check if there is a valid Smart Pool Participation ticket from that user (verified by matching PastelID and address, and checking the signatures), and if the pledged coins have not been moved.
3. **Calculate Total Pledged Coins**: The validation process should then add up the total pledged coins from all valid participants. If this total is not at least equal to the `**required_coins**` multiplied by the `**overcollateralization_factor**`, the Activation ticket is not valid.
4. **Verify Operator's Fee**: Check that the `**operator_fee_percentage**` is within an acceptable range (e.g., 3 - 15%).
5. **Verify Operator's Signature**: The validation process should also check the operator's signature on the Activation ticket to ensure that it is valid.
6. **Supernode Validation**: If the Activation ticket is valid, the system should proceed with the normal process of setting up the Supernode, but with the modified logic that the collateral does not need to come from a single txid/vout, but instead can come from the multiple addresses of the participants.

This modification would likely involve adding new functions to `**pasteld**` for handling the new ticket types, as well as modifying the existing Supernodevalidation functions to include the logic for checking Smart Pool Activation tickets.

It would also be important to think through the implications of this change for other parts of the Pastel Network software. For example, the logic for distributing rewards to Supernodes would need to be adjusted to account for the fact that rewards for a smart pool Supernode need to be distributed among multiple participants. The current reward system of Pastel Network, which sends 1250 PSL to the top-ranked Supernode, could be adapted to distribute rewards among the participants of a smart pool as follows:


1. **Track Smart Pools**: The network would need to keep track of which Supernodes are regular ones and which are smart pools. This could be done by checking the `**type**` field of the Supernode’s activation ticket.
2. **Calculate Individual Rewards**: When a smart pool Supernode earns a reward, the network would calculate the share of the reward for each participant. This would be done by dividing the total reward (1250 PSL) by the total amount of PSL pledged to the smart pool, and then multiplying by the amount of PSL each participant pledged. This would give each participant a reward proportional to their contribution to the smart pool.
3. **Distribute Rewards**: The network would then distribute the rewards to each participant. This could be done by creating a special type of transaction that sends the calculated reward amount to each participant's address.

This approach would ensure that each participant in a smart pool receives a fair share of the rewards, proportional to their contribution. However, it would require significant changes to the Pastel Network software, including the logic for tracking Supernodes and distributing rewards.
Additionally, there are other factors to consider. For example, what happens if a participant moves their pledged coins after the smart pool has been activated? It's a complex problem, but it can be managed in an efficient way without creating a multitude of additional blockchain tickets. The main idea is to actively monitor the participant addresses in the network itself.

When a user pledges their coins and becomes a participant of the pool, they provide an address from which the pledged coins are associated. The network can keep track of these addresses and monitor for any transactions involving them.
Here's a high-level approach:


1. **Monitoring Participant Addresses**: The nodes in the network continuously monitor the blockchain. If a transaction is detected that involves one of the participant addresses, the nodes will check if the balance of the pledged coins has decreased. This can be done by analyzing the outputs of the transaction.
2. **Invalidating Participants**: If a decrease in the balance of the pledged coins is detected, the participant is invalidated. This means that their share is removed from the total pool, and they will no longer receive rewards. There's no need for a new ticket here; the network just updates its internal state.
3. **Recalculating Rewards**: When a participant is invalidated, the reward shares of the remaining participants are recalculated. This is done by dividing the total reward by the new total amount of PSL pledged to the smart pool (after removing the invalidated participant's pledge), and then multiplying by the amount of PSL each remaining participant pledged. This operation can be performed whenever a reward is about to be distributed, so it doesn't require any additional storage or tickets.
4. **Distributing Adjusted Rewards**: The adjusted rewards are then distributed to the remaining participants as usual.

One complication to all this would be if the total amount of coins in the smart pool falls below the required 5 million PSL, the Supernode would no longer be valid and the operation would be disrupted. To handle this situation, we introduce a system where the smart pool goes into a "replenishment mode" when the total pledged amount dips below the required level. Here's how it works:


1. **Detecting Insufficient Pledged Coins**: The network continues to monitor transactions and invalidates participants who move their pledged coins. If the total pledged amount falls below 5 million PSL, the network flags the smart pool for replenishment.
2. **Notifying Participants**: When a smart pool is flagged for replenishment, the network can notify the remaining participants and the operator. This could be done through an automated message or alert. It could also be made visible on a dashboard or interface where participants monitor their smart pool.
3. **Replenishment Period**: The smart pool would then enter a replenishment period. During this time, new participants can join the pool or existing participants can increase their pledge to bring the total pledged amount back to the required level.
4. **Restoration or Dissolution**: If the total pledged amount is restored to at least 5 million PSL by the end of the replenishment period, the smart pool resumes normal operation. If not, the smart pool is dissolved, the Supernode is shut down, and the remaining pledged coins are returned to the participants.

This approach ensures that the Supernode can continue to operate as long as there's enough pledged coins, and provides a mechanism for restoring the pledged amount if it falls below the required level. It also allows participants to withdraw their pledged coins if the smart pool can't be replenished, so their coins aren't locked in a non-functioning pool.

**Voluntary Exit Mechanism**
In the smart pool system, users maintain control over their pledged coins at all times. This allows for a straightforward exit mechanism but also necessitates a set of penalties to maintain the pool's stability.

Exit Procedure:
To exit the smart pool, a participant merely needs to move their pledged coins to a different address. The system's active monitoring will detect this action and immediately invalidate the participant from the smart pool.

Penalties for Early Exit:

1. **Forfeited Rewards**: Participants who exit the pool early will forfeit any "earned but not yet distributed" rewards.
2. **PastelID Reliability Score**: Exiting early may also mark the participant's PastelID as less reliable, affecting their ability to join future smart pools.

Reward Redistribution
Upon early withdrawal by a participant, the system will automatically recalculate the reward shares for the remaining participants and the operator. The rewards that would have gone to the exiting participant are instead distributed pro rata among the remaining participants and the operator.

This structure allows for a frictionless entry and exit while also providing disincentives for destabilizing the pool. It respects user autonomy over their coins while protecting the interests of both the operator and the remaining participants.


----------

**Monitoring the Pledged Addresses:**

**Data Structures**

1. **Key/Value Store**: Use an efficient in-memory data structure like a hash table to store the pledged addresses and their corresponding pledge amounts. The address would be the key, and the pledged amount would be the value.
2. **Transaction Queue**: Maintain a queue of incoming transactions that affect the addresses in the Key/Value store. This could be a simple First-In-First-Out (FIFO) queue.

**Monitoring Process**

1. **Transaction Parsing**: As new blocks are added to the blockchain, parse each transaction to check if any of the input addresses match the addresses in the Key/Value store.
2. **Queue Addition**: If a match is found, add the transaction to the Transaction Queue.
3. **Batch Processing**: Periodically or after a certain number of transactions are queued, process the Transaction Queue to update the Key/Value store.
    1. **Dequeue and Check**: Dequeue a transaction and sum up the total outputs that go back to the same address (if any).
    2. **Balance Check**: Compare this sum with the pledge amount from the Key/Value store. If the sum is less than the pledged amount, flag this address for removal from the smart pool.
    3. **Update Store**: If needed, update the pledged amount in the Key/Value store.
4. **Notification**: If an address is flagged for removal, notify the rest of the network or take whatever action is prescribed by the protocol for such an event.
