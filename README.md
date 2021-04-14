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

###Test Run and Gas Costs

**Init**
My address: 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 (100 ETH, 0 RM)

My private key: 503f38a9c967ed597e47fe25643985f032b072db8075426a92110f82df48dfcb (from https://ethereum.stackexchange.com/questions/78040/is-there-a-way-to-view-the-private-key-of-test-account-in-remix-javascript-vm)

Address A: 0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2 (100 ETH, 0 RM)

Address B: 0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db (100 ETH, 0 RM)

**Deploy Contract**
My address (99.99 ETH, 0 RM)

**Credit A with 5 Tokens**
Deposit(0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2) with value = 5 ETH

| status | true Transaction mined and execution succeed |
|-|-|
|  transaction hash | 0x8e796f80df8c972f9f4b85b8cd13e54b3dfa575a783b0113a8fe6eb8dd009c09 |
|  from | 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 |
|  to | RingMixerV2.Deposit(address) 0xd9145CCE52D386f254917e481eB44e9943F39138 |
|  gas | 3000000 gas |
|  transaction cost | 44079 |
|  execution cost | 21399 gas |
|  hash | 0x8e796f80df8c972f9f4b85b8cd13e54b3dfa575a783b0113a8fe6eb8dd009c09 |
|  input | 0x8ce0bd46000000000000000000000000ab8483f64d9c6d1ecf9b849ae677dd3315835cb2 |
|  decoded input | { "address destination": "0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2" } |
|  decoded output | { "0": "bool: success true" } |
|  logs | [] |
|  value | 5000000000000000000 wei |

My address (94.99 ETH, 0 RM)

A (100 ETH, 5 RM) by calling token_balance(0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2)

**Credit myself with 5 Tokens**
Deposit(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4) with value = 5 ETH

|  status | true Transaction mined and execution succeed |
|-|-|
|  transaction hash | 0xa143ce44c9a3f2411f574aa3207c9a8a7f058d915e7e7b1e841f3422b7263bda |
|  from | 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 |
|  to | RingMixerV2.Deposit(address) 0xd9145CCE52D386f254917e481eB44e9943F39138 |
|  gas | 3000000 gas |
|  transaction cost | 44079 gas |
|  execution cost | 21399 gas |
|  hash | 0xa143ce44c9a3f2411f574aa3207c9a8a7f058d915e7e7b1e841f3422b7263bda |
|  input | 0x8ce0bd460000000000000000000000005b38da6a701c568545dcfcb03fcb875f56beddc4 |
|  decoded input | { "address destination": "0x5B38Da6a701c568545dCfcB03FcB875f56beddC4" } |
|  decoded output | { "0": "bool: success true" } |
|  logs | [] |
|  value | 5000000000000000000 wei |

My address (89.99 ETH, 5 RM) by calling token_balance(0x5B38Da6a701c568545dCfcB03FcB875f56beddC4)

**Credit B with 5 ETH using my RM Tokens**
Using address A as a mix-in address.

Random keys for each address:
- 5266556A586E327234753778214125442A472D4B6150645367566B5970337336
- 2D4B6150645367566B59703373357638792F423F4528482B4D6251655468576D

data = ["1","0x503f38a9c967ed597e47fe25643985f032b072db8075426a92110f82df48dfcb","0x5266556A586E327234753778214125442A472D4B6150645367566B5970337336","0x2D4B6150645367566B59703373357638792F423F4528482B4D6251655468576D","0xAb8483F64d9C6d1EcF9b849Ae677dD3315835cb2","0x5B38Da6a701c568545dCfcB03FcB875f56beddC4"]

Create ring signature using RingSign_User(4B20993Bc481177ec7E8f571ceCaE8A9e22C02db, 5000000000000000000 *(wei)*, data)
