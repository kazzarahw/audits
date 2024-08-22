# Security Audit - Notes
*kazzarahw @ 2024-08-18*

1. [PuppyRaffle.sol](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L18)
   1. [`refund` vulnerable to reentrancy](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L96)
   2. [`refund` does not properly remove the player's address](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L96)
   3. [`selectWinner` uses predictable RNG](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L128)
   4. [`selectWinner` typecasts `totalFees` from `uint256` to `uint64`](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L134)
   5. [`selectWinner` can be front-run](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L125)
   6. [Nested loop in `enterRaffle` could lead to DOS from increasing gas fees](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L85)
   7. [Strict balance requirement in `withdrawFees` could prevent withdrawal of fees](../../contracts/2023-10-Puppy-Raffle/src/PuppyRaffle.sol#L158)
   8.
