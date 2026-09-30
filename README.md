# Winch

**Solo mining in a browser tab.** The block is built and checked by the tab's own node from its own validated mempool, hashed in the tab, and handed to the chain. Testnet's twenty-minute rule makes it real: twenty minutes after the last block, the next one may be mined at the lowest difficulty, and a laptop finds one of those in about a minute.

Live: https://bitcoin-blake.github.io/winch/

## How it works

One page over [blaketestnode](https://github.com/bitcoin-blake/blaketestnode)'s browser node, through the same loader [Reef](https://github.com/bitcoin-blake/reef) and [Bight](https://github.com/bitcoin-blake/bight) use (`browser/tabnode.js`, pinned by commit). The tab fetches the fork-point UTXO snapshot of txbt4 (the BLAKE2b testnet4), checks its hashes, validates every block since the fork, follows the chain tip and keeps its own mempool.

- **The block.** The node worker builds the next block from the tab's chain tip and mempool (datstr SPEC 6.3, without a pool): coinbase paying this tab's script, witness commitment, merkle root, a v2 header with the difficulty the chain's rules expect for the block's time. Every block rule the worker knows is run on it before it is handed out.
- **The window.** The header's bits say whether the twenty-minute window is open. Outside it the tab waits, with a countdown read from the last block's time in its own chain; the moment it opens the tab asks for fresh work and hashes. A setting allows hashing at full difficulty outside the window, for tests.
- **Hashing.** The datstr browser miner's loop, BLAKE2b in WebAssembly (about 40× the JavaScript loop), one worker per core, rolling the nonce over the 80-byte work. A nonce that meets the target goes back to the node worker, which assembles the full block and checks the hash against the network target.
- **Handing it over.** A tab has no peer-to-peer connection, so the block is published as a kind 23405 event (content the block hex, tagged with the chain). A node that follows that kind runs `submitblock`; the estate's mempool publisher does. The tab does not take anyone's word for the result: it sees its block when its own chain tip moves to it, or sees which block won instead.
- **Rewards** pay Reef's wallet when this browser has one (same origin, same storage), else a key Winch makes and keeps; a coinbase on this chain matures after 6,705 blocks.

Measured on the estate's workstation: 12 workers, about 85 MH/s, about 50 s per block on average at difficulty 1.

## Name

A winch hauls a line in. Reef is the knot, Bight the slack, Winch the pull.

## Licence

AGPL-3.0-or-later.
