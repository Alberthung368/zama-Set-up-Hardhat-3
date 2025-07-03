3. Turn it into FHEVM
In this tutorial, you'll learn how to take a basic Solidity smart contract and progressively upgrade it to support Fully Homomorphic Encryption using the FHEVM library by Zama.

Starting with the plain Counter.sol contract that you build from the "Write a simple contract" tutorial, and step-by-step, you’ll learn how to:

Replace standard types with encrypted equivalents

Integrate zero-knowledge proof validation

Enable encrypted on-chain computation

Grant permissions for secure off-chain decryption

By the end, you'll have a fully functional smart contract that supports FHE computation.

Initiate the contract
Create the FHECounter.sol file
Navigate to your project’s contracts directory:

Copy
cd <your-project-root-directory>/contracts
From there, create a new file named FHECounter.sol, and copy the following Solidity code into it:

Copy
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

/// @title A simple counter contract
contract Counter {
  uint32 private _count;

  /// @notice Returns the current count
  function getCount() external view returns (uint32) {
    return _count;
  }

  /// @notice Increments the counter by a specific value
  function increment(uint32 value) external {
    _count += value;
  }

  /// @notice Decrements the counter by a specific value
  function decrement(uint32 value) external {
    require(_count >= value, "Counter: cannot decrement below zero");
    _count -= value;
  }
}
This is a plain Counter contract that we’ll use as the starting point for adding FHEVM functionality. We will modify this contract step-by-step to progressively integrate FHEVM capabilities.

Turn Counter into FHECounter
To begin integrating FHEVM features into your contract, we first need to import the required FHEVM libraries.

Replace the current header

Copy
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;
With this updated header:

Copy
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.24;

import { FHE, euint32, externalEuint32 } from "@fhevm/solidity/lib/FHE.sol";
import { SepoliaConfig } from "@fhevm/solidity/config/ZamaConfig.sol";
This imports:

FHE — the core library to work with FHEVM encrypted types

euint32 and externalEuint32 — encrypted uint32 types used in FHEVM

SepoliaConfig — provides the FHEVM configuration for the Sepolia network.
Inheriting from it enables your contract to use the FHE library

Replace the current contract declaration:

Copy
/// @title A simple counter contract
contract Counter {
With the updated declaration :

Copy
/// @title A simple FHE counter contract
contract FHECounter is SepoliaConfig {
This change:

Renames the contract to FHECounter

Inherits from SepoliaConfig to enable FHEVM support

This contract must inherit from the SepoliaConfig abstract contract; otherwise, it will not be able to execute any FHEVM-related functionality on Sepolia or Hardhat.

From your project's root directory, run:

Copy
npx hardhat compile
Great! Your smart contract is now compiled and ready to use FHEVM features.

Apply FHE functions and types
Comment out the increment() and decrement() Functions
Before we move forward, let’s comment out the increment() and decrement() functions in FHECounter. We'll replace them later with updated versions that support FHE-encrypted operations.

Copy
 /// @notice Increments the counter by a specific value
// function increment(uint32 value) external {
//     _count += value;
// }

/// @notice Decrements the counter by a specific value
// function decrement(uint32 value) external {
//     require(_count >= value, "Counter: cannot decrement below zero");
//     _count -= value;
// }
Replace uint32 with the FHEVM euint32 Type
We’ll now switch from the standard Solidity uint32 type to the encrypted FHEVM type euint32.

This enables private, homomorphic computation on encrypted integers.

Replace

Copy
uint32 _count;
and

Copy
function getCount() external view returns (uint32) {
With :

Copy
euint32 _count;
and

Copy
function getCount() external view returns (euint32) {
Replace increment(uint32 value) with the FHEVM version increment(externalEuint32 value)
To support encrypted input, we will update the increment function to accept a value encrypted off-chain.

Instead of using a uint32, the new version will accept an externalEuint32, which is an encrypted integer produced off-chain and sent to the smart contract.

To ensure the validity of this encrypted value, we also include a second argument:inputProof, a bytes array containing a Zero-Knowledge Proof of Knowledge (ZKPoK) that proves two things:

The externalEuint32 was encrypted off-chain by the function caller (msg.sender)

The externalEuint32 is bound to the contract (address(this)) and can only be processed by it.

Replace

Copy
 /// @notice Increments the counter by a specific value
// function increment(uint32 value) external {
//     _count += value;
// }
With :

Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  //     _count += value;
}
Convert externalEuint32 to euint32
You cannot directly use externalEuint32 in FHE operations. To manipulate it with the FHEVM library, you first need to convert it into the native FHE type euint32.

This conversion is done using:

Copy
FHE.fromExternal(inputEuint32, inputProof);
This method verifies the zero-knowledge proof and returns a usable encrypted value within the contract.

Replace

Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  //     _count += value;
}
With :

Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  euint32 evalue = FHE.fromExternal(inputEuint32, inputProof);
  //     _count += value;
}
Convert _count += value into its FHEVM equivalent
To perform the update _count += value in a Fully Homomorphic way, we use the FHE.add() operator. This function allows us to compute the FHE sum of 2 encrypted integers.

Replace

Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  euint32 evalue = FHE.fromExternal(inputEuint32, inputProof);
  //     _count += value;
}
With :

Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  euint32 evalue = FHE.fromExternal(inputEuint32, inputProof);
  _count = FHE.add(_count, evalue);
}
This FHE operation allows the smart contract to process encrypted values without ever decrypting them — a core feature of FHEVM that enables on-chain privacy.

Grant FHE Permissions
This step is critical! You must grant FHE permissions to both the contract and the caller to ensure the encrypted _count value can be decrypted off-chain by the caller. Without these 2 permissions, the caller will not be able to compute the clear result.

To grant FHE permission we will call the FHE.allow() function.

Replace
Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  euint32 evalue = FHE.fromExternal(inputEuint32, inputProof);
  _count = FHE.add(_count, evalue);
}
With :
Copy
/// @notice Increments the counter by a specific value
function increment(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  euint32 evalue = FHE.fromExternal(inputEuint32, inputProof);
  _count = FHE.add(_count, evalue);

  FHE.allowThis(_count);
  FHE.allow(_count, msg.sender);
}
We grant two FHE permissions here — not just one. In the next part of the tutorial, you'll learn why both are necessary.

Convert decrement() to its FHEVM equivalent
Just like with the increment() migration, we’ll now convert the decrement() function to its FHEVM-compatible version.

Replace :

Copy
/// @notice Decrements the counter by a specific value
function decrement(uint32 value) external {
  require(_count >= value, "Counter: cannot decrement below zero");
  _count -= value;
}
with the following :

Copy
/// @notice Decrements the counter by a specific value
/// @dev This example omits overflow/underflow checks for simplicity and readability.
/// In a production contract, proper range checks should be implemented.
function decrement(externalEuint32 inputEuint32, bytes calldata inputProof) external {
  euint32 encryptedEuint32 = FHE.fromExternal(inputEuint32, inputProof);

  _count = FHE.sub(_count, encryptedEuint32);

  FHE.allowThis(_count);
  FHE.allow(_count, msg.sender);
}
The increment() and decrement() functions do not perform any overflow or underflow checks.

Compile FHECounter.sol
From your project's root directory, run:

Copy
npx hardhat compile
Congratulations! Your smart contract is now fully FHEVM-compatible.

Now you should have the following files in your project:

contracts/FHECounter.sol — your Solidity smart FHEVM contract

test/FHECounter.ts — your FHEVM Hardhat test suite written in TypeScript

In the next tutorial, we’ll move on to the TypeScript integration, where you’ll learn how to interact with your newly upgraded FHEVM contract in a test suite.
