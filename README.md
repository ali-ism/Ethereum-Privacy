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

Create ring signature using RingSign_User(0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db, 5000000000000000000 *(wei)*, data)

Signature = ["72009855460516685219824033039940398545173912476735849717319342650666294264820","18077985686962909651520089174399279189068128000700366999768005539139229810740","37270461499941877070057267465950549139654588956935278236402174464132096684854","14085983200358974918631556832163130552850932543728434519734637714963811761170","979192615699164691250836073164922764378935352498","520786028573371803640530888255888666801131675076"]

|  transaction hash | 0xaae9ea63466d4735755b0a89abbc2c448948fcff994d2161c7f6e33273570d9e |
|-|-|
|  from | 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 |
|  to | RingMixerV2.RingSign_User(address[],uint256[],uint256[]) 0xd9145CCE52D386f254917e481eB44e9943F39138 |
|  transaction cost | 228301 gas (Cost only applies when called by a contract) |
|  execution cost | 194421 gas (Cost only applies when called by a contract) |
|  hash | 0xaae9ea63466d4735755b0a89abbc2c448948fcff994d2161c7f6e33273570d9e |
|  input | 0x7b84da5d000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000e000000000000000000000000000000000000000000000000000000000000000010000000000000000000000004b20993bc481177ec7e8f571cecae8a9e22c02db00000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000004563918244f4000000000000000000000000000000000000000000000000000000000000000000060000000000000000000000000000000000000000000000000000000000000001503f38a9c967ed597e47fe25643985f032b072db8075426a92110f82df48dfcb5266556a586e327234753778214125442a472d4b6150645367566b59703373362d4b6150645367566b59703373357638792f423f4528482b4d6251655468576d000000000000000000000000ab8483f64d9c6d1ecf9b849ae677dd3315835cb20000000000000000000000005b38da6a701c568545dcfcb03fcb875f56beddc4 |
|  decoded input | { "address[] destination": [ "0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db" ], "uint256[] value": [ { "type": "BigNumber", "hex": "0x4563918244f40000" } ], "uint256[] data": [ { "type": "BigNumber", "hex": "0x01" }, { "type": "BigNumber", "hex": "0x503f38a9c967ed597e47fe25643985f032b072db8075426a92110f82df48dfcb" }, { "type": "BigNumber", "hex": "0x5266556a586e327234753778214125442a472d4b6150645367566b5970337336" }, { "type": "BigNumber", "hex": "0x2d4b6150645367566b59703373357638792f423f4528482b4d6251655468576d" }, { "type": "BigNumber", "hex": "0xab8483f64d9c6d1ecf9b849ae677dd3315835cb2" }, { "type": "BigNumber", "hex": "0x5b38da6a701c568545dcfcb03fcb875f56beddc4" } ] } |
|  decoded output | { "0": "uint256[32]: signature 72009855460516685219824033039940398545173912476735849717319342650666294264820,18077985686962909651520089174399279189068128000700366999768005539139229810740,37270461499941877070057267465950549139654588956935278236402174464132096684854,14085983200358974918631556832163130552850932543728434519734637714963811761170,979192615699164691250836073164922764378935352498,520786028573371803640530888255888666801131675076,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0,0" } |
|  logs | [] |
|  decoded output | { "0": "bool: success true" } |
|  logs | [] |
|  value | 5000000000000000000 wei |

Cash in my Tokens and send them to address B by calling Withdraw(0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db, 5000000000000000000, signature)

|  status | true Transaction mined and execution succeed |
|-|-|
|  transaction hash | 0xedfbdfb8dc3fb7990642b397f24aa9e2454730a91782ea19956dd2ba4cae7548 |
|  from | 0x5B38Da6a701c568545dCfcB03FcB875f56beddC4 |
|  to | RingMixerV2.Withdraw(address[],uint256[],uint256[]) 0xd9145CCE52D386f254917e481eB44e9943F39138 |
|  gas | 3000000 gas |
|  transaction cost | 199412 gas |
|  execution cost | 163548 gas |
|  hash | 0xedfbdfb8dc3fb7990642b397f24aa9e2454730a91782ea19956dd2ba4cae7548 |
|  input | 0x4b1d4622000000000000000000000000000000000000000000000000000000000000006000000000000000000000000000000000000000000000000000000000000000a000000000000000000000000000000000000000000000000000000000000000e000000000000000000000000000000000000000000000000000000000000000010000000000000000000000004b20993bc481177ec7e8f571cecae8a9e22c02db00000000000000000000000000000000000000000000000000000000000000010000000000000000000000000000000000000000000000004563918244f4000000000000000000000000000000000000000000000000000000000000000000069f342243d7b5153ad99f99e4a5ee5c57ca88a61e89e7d18e2adfd275534143f427f7c6fd056eab651e927fad2930ff9cc5f260b8104b6c52cfde07f5d68780345266556a586e327234753778214125442a472d4b6150645367566b59703373361f24624b40a4a00ad1f818088114afb2b850c97901b9b6bfbca73d16370b7c12000000000000000000000000ab8483f64d9c6d1ecf9b849ae677dd3315835cb20000000000000000000000005b38da6a701c568545dcfcb03fcb875f56beddc4 |
|  decoded input | { "address[] destination": [ "0x4B20993Bc481177ec7E8f571ceCaE8A9e22C02db" ], "uint256[] value": [ { "type": "BigNumber", "hex": "0x4563918244f40000" } ], "uint256[] signature": [ { "type": "BigNumber", "hex": "0x9f342243d7b5153ad99f99e4a5ee5c57ca88a61e89e7d18e2adfd275534143f4" }, { "type": "BigNumber", "hex": "0x27f7c6fd056eab651e927fad2930ff9cc5f260b8104b6c52cfde07f5d6878034" }, { "type": "BigNumber", "hex": "0x5266556a586e327234753778214125442a472d4b6150645367566b5970337336" }, { "type": "BigNumber", "hex": "0x1f24624b40a4a00ad1f818088114afb2b850c97901b9b6bfbca73d16370b7c12" }, { "type": "BigNumber", "hex": "0xab8483f64d9c6d1ecf9b849ae677dd3315835cb2" }, { "type": "BigNumber", "hex": "0x5b38da6a701c568545dcfcb03fcb875f56beddc4" } ] } |
|  decoded output | { "0": "bool: success false" } |
|  logs | [] |
|  value | 0 wei |

My address (89.9- ETH, 
