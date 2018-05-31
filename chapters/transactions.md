# Transaktionen

- Alle Transaktionen MÜSSEN einen Wert überweisen (value > 0).
- Der Wert wird immer vom Sender (_sender_) zum Empfänger (_recipient_) überwiesen.
- Der Hash der Transaktion enthält nicht die Signatur/den Beweis.
- Alle Transaktionen in Nimiq haben eine maximale Gültigkeitsperiode von 120 Blöcken (ungefähr zwei Stunden).
- Transaktionen in Nimiq verwenden keine Nonce. Wiederkehrende, identische Transaktionen können nur gesendet werden, sobald die vorherige Transaktion für ungültig erklärt oder gemined wurde.

## Einfache Transaktion
Größen-optimiertes Format (138 bytes) für einfache Überweisungen von einfachem zu einfachem Konto.

| Element               | Datentyp     | Bytes | Beschreibung                                    |
|-----------------------|--------------|-------|-------------------------------------------------|
| Type                  | uint8        | 1     | `0`                                             |
| sender                | raw          | 32    | Public key des Senders                          |
| recipient             | Address      | 20    | Addresse des Empfängers                         |
| recipient type        | uint8        | 1     | Account-Typ des Empfängers                      |
| value                 | uint64       | 8     | Betrag in Satoshi                               |
| fee                   | uint64       | 8     | Miner-Gebühr                                    |
| validity start height | uint32       | 4     | Verzögerung in Blöcken, (std. jetzige Höhe + 1) |
| signature             | raw          | 64    | Signatur durch den private key des Senders      |

Um die Gültigkeitsperiode einer Transaktion kürzer als die standardmäßigen 120 Blöcke zu machen, kann die Starthöhe auf einen Wert vor der momentanen Blockchain-Höhe gesetzt werden. Zum Beispiel wird _jetzige Höhe - 60_ in einer verbleibenden Zeitspanne von 60 Blöcken resultieren, was ungefähr einer Stunde entspricht.

## Erweiterte Transaktion
Jede erweiterte Transaktion ist mindestens 68 (3+|data|+65+|proof|) bytes lang.

| Element               | Datentyp     | Bytes        | Beschreibung                                                                  |
|-----------------------|--------------|--------------|------------------------------------------------------------------------------|
| Type                  | uint8        | 1            | `1`                                                                          |
| data length           | uint16       | 2            |                                                                              |
| data                  | raw          | data length  | Für den Empfänger vorgesehen                                                       |
| sender                | Address      | 20           |                                                                              |
| sender type           | uint8        | 1            | Account-Typ des Empfängers                                                       |
| recipient             | Address      | 20           |                                                                              |
| recipient type        | uint8        | 1            | Account-Typ des Senders                                                |
| value                 | uint64       | 8            |                                                                              |
| fee                   | uint64       | 8            | Gebühr für den Miner, die Transaktion in einen Block aufzunehmen                                      |
| validity start height | uint32       | 4            | Verzögerung in Blöcken, standardmäßig jetzige Höhe + 1                             |
| flags                 | uint8        | 1            | Unbekannte Flags müssen 0 sein; Erstellung eines Vertrags (0x1)                             |
| proof length          | uint16       | 2            |                                                                              |
| proof                 | raw          | proof length | Validität hängt vom Account-Typ des Senders, Account-Zustand und momentaner Blöck-Höhe ab |

## Transaktions-Hash
Ein Hash einer Transaktion wird durch Verwendung von Blake2b auf allen Feldern auf einer [erweiterten Transaktion](#erweiterte-transaktion) ohne `type`, `proof length`, und `proof` erstellt.
