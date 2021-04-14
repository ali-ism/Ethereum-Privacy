# Ethereum-Privacy

## RingMixer

This solution implements an Ethereum smart contract which enables the transaction of ETH without revealing the sender by using ring signatures. All transactions are done by calling functions of the contract and the transfer of tokens is done internally. This contract provides the following transaction functionalities:
- *Deposit*/*DepositN*: convert ETh to RingMixer tokens 1:1 and deposit them in the given destination address(es).
- *Withdraw*: cash in the RingMixer tokens to ETH and send them to the given destination address.

The contract also has some utility functions to support elliptic curve operations and ring signature generation and verification. It also holds several hash maps to hold data necessary for the system to function:
- *KeyImageUsed*: history of previously used Key Images. A Key Image is a mechanism introduced to ring signatures to prevent double spending. A key image is included in the ring signature for every transaction output. It is computed using a one-way function of the one-time secret and public keys of the transaction.
- *token_balance*: records the RingMixer token account balances.
- *lookup_pubkey_by_balance*, *lookup_pubkey_by_balance_populated* and *lookup_pubkey_by_balance_count*: hold the history and balances of accounts who have participated in previous transactions since these would be suitable mix-in addresses for ring signatures.

The *Deposit* transactions do not require ring signatures since they are just converting ETH to RingMixer tokens and depositing them into an empty account and it is not sending ETH directly to the new account. *Withdraw* transactions are signed by ring signatures. To do this, the user can build a ring signature by using the *RingSign_User* function. This function takes as argument a message (destination and value of the transaction) and the necessary ring signature data in the format (index of the true pubkey, one time private key, [random number for each public key], [public keys]). All mix-in addresses must have enough balance to pay the transaction. Also, if sending to multiple destinations the sum of all outputs must also match the balance since eventually only one of the mix-in addresses will pay the transaction. The provided ring signature is checked against the history of Key Images and the signature is verified and added to the Key Image history before committing the transaction.

The ring signature that is used is of the format (keyimage, starting ring segment (c), [random number for each public key], [public keys]). Therefore, it is expected that the size of this signature grows linearly with the number of mix-in accounts.
