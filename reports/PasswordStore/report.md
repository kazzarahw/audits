# PasswordStore - Security Audit <!-- omit from toc -->
*kazzarahw @ 2024-08-18*

## Table of Contents <!-- omit from toc -->

- [High Severity Findings](#high-severity-findings)
	- [\[H-1\] `PasswordStore::s_password` variable is not truly secret](#h-1-passwordstores_password-variable-is-not-truly-secret)
	- [\[H-2\] `PasswordStore::setPassword` lacks owner authorization](#h-2-passwordstoresetpassword-lacks-owner-authorization)
- [Medium Severity Findings](#medium-severity-findings)
- [Low Severity Findings](#low-severity-findings)
- [Informational Findings](#informational-findings)
	- [\[I-1\] `PasswordStore::getPassword` has incorrect NatSpec](#i-1-passwordstoregetpassword-has-incorrect-natspec)
	- [\[I-2\] `PasswordStore::SetNetPassword` event name has a typo](#i-2-passwordstoresetnetpassword-event-name-has-a-typo)


## High Severity Findings

### [H-1] `PasswordStore::s_password` variable is not truly secret

**Description**:

All smart contract state variables are held in `memory`. This means that even if the variable is declared `private`, the raw hex value can still be read by anyone as all aspects of the blockchain are transparent.

**Impact**:

Everyone is able to read the contract's password, even without a transaction.

**Proof**:

```bash
make anvil # run Anvil instance
make deploy # deploy PasswordStore to chain
cast storage 0x5FbDB2315678afecb367f032d93F642f64180aa3 1 | cast parse-bytes32-string # get the `s_password` variable as hex then decode it to a string
```

**Patch**:

This likely cannot be fixed with a simple change in the code and would most likely require some sort of off-chain encryption/storage of the password. You could encrypt/decrypt the password on-chain and pass it through a load of obfuscation; however, while it will be harder to obtain, the password will likely never truly be secret if everything is done on-chain.

### [H-2] `PasswordStore::setPassword` lacks owner authorization

**Description**:

The `setPassword` function is used to set the password of the contract. However, it lacks any sort of authorization to ensure that only the owner is allowed to call the function.

**Impact**:

All addresses are allowed to change the contract's password.

**Proof**:

1. Add this test to `test/PasswordStore.t.sol`:
	```javascript
	function test_non_owner_cant_set_password() public {
		vm.startPrank(address(1));
		vm.expectRevert(PasswordStore.PasswordStore__NotOwner.selector);
		passwordStore.setPassword("woahWhatAPassword");
	}
	```
2. Run the test w/ `forge test --mt test_non_owner_cant_set_password`

The test will pass when the issue is resolved and fail if the issue persists.

**Patch**:

Authorize the sender's address at the beginning of `setPassword` using the `PasswordStore__NotOwner` error, similar to the `getPassword` function.

```diff
diff --git a/src/PasswordStore.sol b/src/PasswordStore.sol
index b84717f..5000729 100644
--- a/src/PasswordStore.sol
+++ b/src/PasswordStore.sol
@@ -24,6 +24,9 @@ contract PasswordStore {
      * @param newPassword The new password to set.
      */
     function setPassword(string memory newPassword) external {
+        if (msg.sender != s_owner) {
+            revert PasswordStore__NotOwner();
+        }
         s_password = newPassword;
         emit SetNetPassword();
     }
```

Alternatively use the `onlyOwner` modifier from the `OpenZeppelin` library. Read more [here](https://docs.openzeppelin.com/contracts/2.x/api/ownership).


## Medium Severity Findings

## Low Severity Findings

## Informational Findings

### [I-1] `PasswordStore::getPassword` has incorrect NatSpec

**Description**:

The `getPassword` function's NatSpec incorrectly states the function accepts a `newPassword` parameter. The original NatSpec uses block comments like `/* */`, but if changed to the correct syntax of `/** */` the Solidity compiler (`solc`) will refuse to compile because of the incorrect NatSpec parameter.

**Impact**:

Incorrect documentation at best, failure to compile at worst.

**Proof**:

The NatSpec in question:

```javascript
/**
 * @notice This allows only the owner to retrieve the password.
 * @param newPassword The new password to set.
 */
function getPassword() external view returns (string memory) {
```

The `solc` error for the incorrect NatSpec:

```
Error:
Compiler run failed:
Error (3881): Documented parameter "newPassword" not found in the parameter list of the function.
  --> src/PasswordStore.sol:31:5:
```

**Patch**:

Remove the NatSpec line in question.

```diff
diff --git a/src/PasswordStore.sol b/src/PasswordStore.sol
index b84717f..232a7d3 100644
--- a/src/PasswordStore.sol
+++ b/src/PasswordStore.sol
@@ -30,7 +30,6 @@ contract PasswordStore {

     /*
      * @notice This allows only the owner to retrieve the password.
-     * @param newPassword The new password to set.
      */
     function getPassword() external view returns (string memory) {
         if (msg.sender != s_owner) {
```

### [I-2] `PasswordStore::SetNetPassword` event name has a typo

**Description**:

The `SetNetPassword` event should likely be `SetNewPassword` instead.

**Impact**:

An incorrect event name may cause confusion during contract execution.

**Proof**:

The event is emitted in `setPassword`, likely referring to a *new* password being set.

```javascript
function setPassword(string memory newPassword) external {
	s_password = newPassword;
	emit SetNetPassword();
}
```

**Patch**:

Rename the event to `SetNewPassword`

```diff
diff --git a/src/PasswordStore.sol b/src/PasswordStore.sol
index b84717f..300464a 100644
--- a/src/PasswordStore.sol
+++ b/src/PasswordStore.sol
@@ -13,7 +13,7 @@ contract PasswordStore {
     address private s_owner;
     string private s_password;

-    event SetNetPassword();
+    event SetNewPassword();

     constructor() {
         s_owner = msg.sender;
@@ -25,7 +25,7 @@ contract PasswordStore {
      */
     function setPassword(string memory newPassword) external {
         s_password = newPassword;
-        emit SetNetPassword();
+        emit SetNewPassword();
     }

     /*
```
