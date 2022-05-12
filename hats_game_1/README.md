CTF: [https://github.com/hats-finance/games](https://github.com/hats-finance/games)

## Exploit

1. Deploy Game
2. Deploy 2 Accomplice instances.
   Args: 
   - address of the game
3. Deploy 1 Attacker istance.
   Args:
   - adddress of the game
   - addresses of the Accomplice instances, as a 2-elements array

Call `attack()` on the Attacker instance. Flag should now be owned by the attacker.

## Quick Explanation

Since the outcome of a fight is decided by the respectivce `_balance` of the
opponents after the defeated mons have been burned, we could win by entering a
fight with a `_balance` of 7. It would indeed leave us with a `_balance` of 4
after our 3 mons have lost their fights, and this wins over the flag holder's
`_balance` of 3.

How can we end up with a `_balance` of 7? Well, The `swap` function uses
OpenZeppelin's `_safeTransfer`. When the target is a contract, it calls its
`onERC721Received` method synchronously, which potentially opens the way to
reentrancy attacks.

So all we have to do is have 2 accomplices. We'll ask one of them to swap their
first mon with us. When the `onERC721Received` callback is called right after
the first transfer, 3 things have been done:

1. The sender's `_balance` has been decremented
2. The receiver's `_balance` has been incremented
3. The token has been maked as owned by the receiver

So there you go, at this point our `_balance` is magically 4. Before it
returns, the `onERC721Received` can request another swap with an accomplice,
and our balance will be increased again. When Our balance reaches 7, we can
finally call `fight` for a guaranteed win against the flag holder.

After that, all those nested `onERC721Received` calls will return one after the
other, and the other transfer that makes up a swap (from attacker to
accomplice) will occur. These transfers need to succeed, or the whole
transaction would be reverted. An easy way to make those transfers succeed is
to requests swaps with `_mon1` and `_mon2` both set to the same mon id (the id
of the accomplice's mon). Since this mon is still marked for sale, it will just
be assigned back to the accomplice.
