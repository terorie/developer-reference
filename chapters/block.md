# Blockchain

## Block
Ein Nimiq Block kann in zwei Arten existieren: Full und Light. Ein Light-Block ist ein Full-Block ohne Block-Body.
Ein Block kann maximal 100 kB (10^5 bytes) groß sein und ist folgendermaßen aufgebaut (Body optional):

| Element             | Bytes   | Beschreibung
|---------------------|---------|--------
| Header              | 146     |
| Interlink           | <= 8193 |
| Full/Light Schalter | 1       | `0` or `1`
| Body                | >= 117  | Falls der Schalter `1` ist


## Header
Der Header hat eine Gesamtgröße von 146 bytes und besteht aus:

| Element       | Datentyp  | Bytes | Beschreibung                                                      |
|---------------|-----------|-------|-------------------------------------------------------------------|
| version       | uint16    | 2     | Protokoll-Version                                                 |
| previous hash | Hash      | 32    | Hash des Vorgänger-Blocks                                         |
| interlink     | Hash      | 32    | Siehe [interlink](#interlink)                                     |
| body hash     | Hash      | 32    | Siehe [body hash](#body-hash) und [body](#body)                   |
| accounts hash | Hash      | 32    | Root-Hash des PM-Baums, der den Zustand aller Accounts speichert. Siehe. [account tree](accounts-tree.md) |
| nBits         | bits      | 4     | Minimale Difficulty für Proof-of-Work                             |
| height        | uint32    | 4     | Blockchain height zum Zeitpunkt des Erstellens                    |
| timestamp     | uint32    | 4     | Wann der Block erstellt wurde                                     |
| nonce         | uint32    | 4     | Verwendet, um die benötigte Proof-of-Work Difficulty zu erfüllen  |

Die version ist `1` seit Mainnet. Alle Hashes im Header sind [Blake2b](#hash).

### Previous Hash

Die Ausdrücke "Block Hash" und "Block Header Hash" beziehen sich auf denselben Hash.
Der Hash wird benutzt, um den vorherigen Block in der Blockchain zu referenzieren.
Er wird durch Anwenden von [Blake2b](#hash) auf dem Block Header des Vorgänger-Blocks erstellt.

## Interlink
Der Interlink implementiert die [Non Interactive Proofs of Proof of Work (NiPoPow)](https://eprint.iacr.org/2017/963.pdf) und enthält Links zu vorherigen Blöcken.

Ein Interlink besteht aus:

| Element     | Datentyp     | Bytes         | Beschreibung                             |
|-------------|--------------|---------------|------------------------------------------|
| count       | uint8        | 1             | Bis zu 255 Blöcke können referenziert werden |
| repeat bits | bit map      | ceil(count/8) | Für das Übersprigen von doppelten Links. Siehe unten. |
| hashes      | [Hash]       | <= count * 32 | Hashes der referenzierten Blöcke.        |

Repeat bits ist eine Bitmaske, die sich auf die Liste von Hashes bezieht.
Eine `1` gibt an, dass der Hash an diesem Index (Position des Bits in der Maske) derselbe ist wie der vorherige,
also wiederholt (repeated).
Eine `0` zeigt einen neuen Hash an. Für jede `0` wird ein Hash aus der Hashes-Liste geladen.
Da jeder Hash in der Bitmaske von einem Bit repräsentiert wird ist die Gesamtgröße der Maske `ceil(count/8)`.

`hashes` ist eine Liste von bis zu 255 Hashes von jeweils 32 Bytes.

Ein Interlink hat also eine maximale Größe von 1 + ceil(255/8) + 255*32 = 8193 Bytes.

### Interlink Construction
The concept of [Non-Interactive Proofs of Proof-of-Work](https://eprint.iacr.org/2017/963.pdf) are used to create the interlink.

An interlink contains hashes to previous blocks with a higher-than-target difficulty. The position of a hash in the interlink correlates to how much higher the difficulty was compared to the target difficulty.

An interlink is created based on the interlink of the previous block plus putting the current hash into place.

1. The block's hash will be placed into the beginning of the new interlink as many times as it is more difficult than the required difficulty.
2. The remaining places of the new interlink will be filled by the hashes of the previous interlink, keeping the position.

Finally, the interlink is compressed by counting and skipping repeating hashes.

## Body
The body part is 25 bytes plus data, transactions, and prunded accounts:

| Element               | Data type                     | Bytes             | Description                                         |
|-----------------------|-------------------------------|-------------------|-----------------------------------------------------|
| miner address         | Address                       | 20                | Recipient of the mining reward                      |
| extra data length     | uint8                         | 1                 |                                                     |
| extra data            | raw                           | extra data length | For future use                                      |
| transaction count     | uint16                        | 2                 |                                                     |
| transactions          | [Transaction]                 | ~150 each         | Transactions mined in this block                    |
| pruned accounts count | uint16                        | 2                 |                                                     |
| pruned accounts       | [Pruned Account](accounts.md) | each >= 20+8      | Accounts with balence `0`; so they will be dropped  |

[Transactions](./transactions) can be basic or extended.
Basic uses 138 bytes, extended more than 68 bytes.
Transactions need to be in block order (first recipient, then validityStartHeight, fee, value, sender).
Pruned accounts by their address.

### Body Hash
The body hash is generated by calculating the root of a [Merkle Tree](https://en.wikipedia.org/wiki/Merkle_tree) with following leaves: `miner address`, `extra data`, each `Transaction`, and each `Pruned Account` (in this order).

### Pruned Account
A pruned account is composed of an account of any type with an address:

| Element | Data type | Bytes | Description                                       |
|---------|-----------|-------|---------------------------------------------------|
| address | Address   | 20    | Address of account                                |
| account | Account   | >9    | Can be a basic account, vesting contract, or HTLC |

This type is used in the body of a block to communicate the accounts to be pruned with this block.


