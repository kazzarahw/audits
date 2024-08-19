# Security Audit - Notes
*kazzarahw @ 2024-08-18*

1. [PasswordStore.sol](../../contracts/2023-10-PasswordStore/src/PasswordStore.sol#L10)
	<!-- 1. [`s_password` is readable by everyone](../../contracts/2023-10-PasswordStore/src/PasswordStore.sol#L14) -->
	<!-- 2. [`setPassword` doesn't auth address](../../contracts/2023-10-PasswordStore/src/PasswordStore.sol#L26) -->
	<!-- 3. [`getPassword` has invalid NatSpec](../../contracts/2023-10-PasswordStore/src/PasswordStore.sol#L33) -->
	<!-- 4. [`SetNetPassword` should be `SetNewPassword`](../../contracts/2023-10-PasswordStore/src/PasswordStore.sol#L16) -->
