---
title: "EVM-Based Blockchains Payment Gateway"
date: 2023-08-24T17:58:22+03:30
---

In this post we are going to understand how a basic payment gateway in Ethereum Virtual Machine based blockchain would work.
<br/>
As you migh know the main langauge to write an Ethereum smart contract is Solidity so we're gonna use Solidity.
<br/>
![off-chain signing](/posts/images/off-chain.svg)
<br/>
note that the payment should be stored in database off-chain, otherwise we cannot track what happened to the transaction.
<br/>
now let's dive in what happens on-chain, how should we control the flow.
<br/>
### an overview:
1. define an address who is the owner of the contract
2. know the public key which signs the data.
3. check if the payment ID is paid or not.
4. check if the payment is still valid (expiration)
5. have a mechanism to validate the signature of the payment
6. be able to pause the smart contract if any attacks or problems had occured

<br/>
now let's dive into coding part.
<br/>
<br/>

```solidity
pragma solidity ^0.8.18;

contract Invoice {
    address public owner;
    address public signer;
    bool public functioning;
    mapping(bytes32 => bool) public isPaid;

    event PaymentAccepted(bytes32 indexed hash, uint time, uint value);

    // settin owner, signer and functioning(being able to pause the contract) on construction
    constructor(address _signer) {
        owner = msg.sender;
        functioning = true;
        signer = _signer;
    }

    // modifier to restrict access on contract methods
    modifier onlyAdmin() {
        require(msg.sender == owner, "only admin can do this");
        _;
    }

    // modifier to only let the method work if the contract is functioning
    modifier onlyFunctioning {
        require(functioning == true, "contract is not functioning right now");
        _;
    }

    // being able to set the signer again by the owner of the contract
    function setSigner(address _signer) public onlyAdmin {
        signer = _signer;
    }

    // being able to set the owner again
    function setOwner(address _owner) public  onlyAdmin {
        owner = _owner;
    }
}
```
<br/>
this is the basic functionality that we needed for restricting and keeping contract safe.
<br/>
now we should add a method to pay and another to validate the payment.
<br/>
<br/>

```solidity
pragma solidity ^0.8.18;

contract Invoice {
    // rest of the code we wrote above

    function pay(
        bytes12 ID,
        uint expiration,
        bytes32 providedHash,
        address receiver,
        uint8 _v,
        bytes32 _r,
        bytes32 _s
    ) public payable onlyFunctioning {
        require(validatePayment(ID, msg.value, expiration, receiver, providedHash, _v, _r, _s), "payment not valid");
        payable(receiver).transfer(msg.value);
        isPaid[providedHash] = true;
        emit PaymentAccepted(providedHash, block.timestamp, msg.value);
    }

    function validatePayment(
        bytes12 ID,
        uint value,
        uint expiration,
        address receiver,
        bytes32 providedHash,
        uint8 _v,
        bytes32 _r,
        bytes32 _s,
    ) public view returns(bool valid) {
        require(isPaid[providedHash] == false, "already been paid");
        require(block.timestamp <= expiration, "payment is expired");
        bytes memory prefix = "\x19Ethereum Signed Message:\n32";

        // order is important and should be the same in off-chain app
        bytes32 ourHash = keccak256(abi.encodePacked(ID, value, receiver, expiration));
        bytes32 payloadHash = keccak256(abi.encodePacked(prefix, ourHash));
        require(ourHash == providedHash, "hash mismatch");
        require(ecrecover(payloadHash, _v, _r, _s) == signer, "signature mismatch");
        return true;
    }
}
```
<br/>
and this is the very simple implentation of a simple payment gateway.
<br/>
for the simplicity sake I didn't implemented the code for ERC-20 integration but it's possible too.
<br/>
<br/>

## Engage and Connect
Wish you enjoyed reading this post in. If you’re interested in collaboration, you can find me on GitHub at [GitHub Profile](https://github.com/blackestwhite). I also maintain a blog in Persian at [blackestwhite.github.io/fa](https://blackestwhite.github.io/fa). Feel free to get in touch with me via email at pesaregoal@gmail.com. I’m always open to new opportunities and discussions.