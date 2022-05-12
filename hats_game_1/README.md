CTF: [https://github.com/hats-finance/games](https://github.com/hats-finance/games)

Exploit:
1. Deploy Game
2. Deploy 2 Accomplice instances.
   Args: 
   - address of the game
3. Deploy 1 Attacker istance.
   Args:
   - adddress of the game
   - addresses of the Accomplice instances, as a 2-elements array

Call `attack()` on the Attacker instance. Flag should now be owned by the attacker.
