# Mining Pool-Protokoll

## Allgemein

### Client-Arten

Das Nimiq Mining Pool-Protokoll beschreibt zur Zeit zwei Client-Arten: Smart und Nano.

#### Smart

Im Smart-Modus ist der Client verantwortlich für die Erstellung von Blöcken.
Der Client verhält wie ein Solo-Miner (und kann auch gegebenenfalls auf Solo-Mining ohne Fehler zurückfall
Der Server schreibt nur die Werte für die Miner-Adresse und im `extraData`-Feld eines Blocks vor.

Der Smart-Modus Pool-Miner setzt eine Nimiq Light- oder Full-Node voraus.

#### Nano

Im Nano-Modus überprüft der Client nur, ob der Pool Arbeit für die richtige Chain verteilt.
Der Server stellt dem Client alle nötigen Details zum Erstellen eines Blocks zur Verfügung solange er auf einer Chain arbeitet, die dem Client bekannt ist.
Dieser Modus ist ähnelt den `getblocktemplate`-basierten Protokollen auf Bitcoin.

Der Nano-Modus Pool-Miner setzt eine Nimiq-Node jeglicher Art voraus, eine Nano-Node reicht dabei aus.

### Netzwerk-Kommunikation

Eine verschlüsselte WebSocket-Verbindung wird als Transportschicht für Client-Server-Kommunikation verwendet. Alle Nachrichten werden als einzelnes WebSocket-Frame ausgetauscht.

## Nachrichten

Alle Nachrichten sind gleich strukturiert: Sie bestehen aus einem JSON-kodierten Objekt mit einer Eigenschaft `message` und Nachricht-spezifischen Eigenschaften.

#### `register`

Wird vom Client direct nach Verbinden an den Server geschickt.

##### Parameter

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `mode`    | string, `nano` oder `smart` | Der Modus des Clients |
| `address` | string | Die Adresse (in IBAN-Format), an die der Lohn geschickt werden soll |
| `deviceId` | number, uint32 | Die Device-ID des Clients. Diese ID sollte für jedes Gerät einzigartig sein und gespeichert werden, sodass sie bei Neustarts gleich bleibt  |
| `deviceData` | object, optional | Ein JSON-Objekt, das Eigenschaften über das Gerät enthält. Das Format dieses Objekts sollte vom Pool-Betreiber definiert werden |
| `genesisHash` | string | Base64-kodierter Hash des Genesis-Blocks des Clients |

##### Beispiel

```json
{
    "message": "register",
    "mode": "nano",
    "address": "NQ07 0000 0000 0000 0000 0000 0000 0000 0000",
    "deviceId": 12345678,
    "deviceData": {
      "groupLabel": "AWS Skylakes"
    },
    "genesisHash": "Jkqvik+YKKdsVQY12geOtGYwahifzANxC+6fZJyGnRI="
}
```

#### `registered`
 
Wird vom Server nach einer erfolgreichen Anmeldung gesendet. Enthält keine Parameter.

##### Beispiel
 
 ```json
 {
     "message": "registered"
 }
 ```

#### `settings`

Wird vom Server gesendet, um neue Mining-Einstellungen bekannt zu geben.

##### Parameter

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `address` | string | Die Adresse (in IBAN-Format), die der Client als Miner-Adresse für zukünftige Shares benutzen sollte |
| `extraData` | string | Der Base64-kodierte Buffer, den der client im `extraData`-Feld für zukünftige Shares benutzen sollte |
| `targetCompact`  | number, uint32 | Der höchste erlaubte Hash-Wert für zukünftige Shares in Kompaktform |
| `nonce`   |number, uint64 | Eine einmalig-verwendete Nummer, die mit dieser Verbindung asoziiert wird |

##### Beispiel
 
```json
{
    "message": "settings",
    "address": "NQ07 0000 0000 0000 0000 0000 0000 0000 0000",
    "extraData": "J8riQiN1lgXT6QYlsdWMKA+qEWxCPYycjZ19zsk1...",
    "targetCompact": 520159232,
    "nonce": 1030036789885656
}
 ```

#### `new-block` (nano)

Wird vom Server gesendet, um dem Nano-Modus-Client mitzuteilen, dass ein neuer Block benutzt werden soll.

##### Parameter

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `bodyHash` | string | Base64-kodierter Hash des Block-Body |
| `accountsHash` | string | Base64-kodierter Hash des Account-Trees nach Anwendung des Block-Body |
| `previousBlock` | string | Base64-kodierter Light-Block, der der Vorgänger des zu  minenden Blocks ist |

##### Beispiel
 
```json
{
    "message": "new-block",
    "bodyHash": "rCBGsSJGOOo82af+MhRXb7Dj+lb0oRzrN0EXBg7Jh4g=",
    "accountsHash": "9GgRxMDSVUpePCyUDKuAw3hmHeMEj7Y77GsWEssd2qU=",
    "previousBlock": "AAHL9kSzaU/uVRSsBVRSlYrx7Dw7NUSFDCWIDhY/..."
}
```

#### `share`

Wird vom Client gesendet, sobald ein gültiger Share gefunden wurde.

##### Parameter (nano)

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `block`   | string | Base64-kodierter Light-Block |

##### Parameter (smart)
--
| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `blockHeader` | string | Base64-kodierter Block-Header |
| `minerAddrProof` | string | Base64-kodierter inclusion proof für das `minerAddr`-Feld im Block-Body |
| `extraDataProof` | string | Base64-kodierter inclusion proof for the `extraData`Feld im Block-Bod |
| `block`   | string, optional | Base64-kodierter Full-Block. Darf nur gesendet werden, falls der Block gültig ist, sodass der Pool-Server ihn schneller verarbeiten kann |

##### Beispiel (nano)
 
```json
{
    "message": "share",
    "block": "AAHL9kSzaU/uVRSsBVRSlYrx7Dw7NUSFDCWIDhY/..."
}
```

##### Beispiel (smart)
 
```json
{
    "message": "share",
    "blockHeader": "AAHL9kSzaU/uVRSsBVRSlYrx7Dw7NUSFDCWIDhY/...",
    "minerAddrProof": "IXd9Hbms/FWTUl5YEYa5ztf6N9eIz+nIfswDOqGk...",
    "extraDataProof": "aZiypZ9S4qvSYbG4dBX4ww9TwOdKeLB+KRwNc0eP..."
}
```

#### `error`

Wird vom Server gesendet, falls der Client einen ungültigen Share gesendet hat.
Der Server darf diese Nachricht höchstens einmal per `share` senden.

##### Parameters

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `reason`  | string | Ein benutzerfreundlicher String, der erklärt, wieso der Server den Share abgelehnt hat |

##### Beispiel
 
```json
{
    "message": "error",
    "reason": "Invalid PoW"
}
```

#### `balance`

Wird vom Server gesendet, um den Betrag des Accounts mitzuteilen, der momentan vom Pool gehalten wird. Die Adresse ist die, die der Benutzer bei `register` gesendet hat.

Der Server darf diese Nachricht zu einem beliebigen Zeitpunkt nach der Registrierung senden. Eine neue `balance`-Nachricht impliziert nicht eine Veränderung des Betrags.

##### Parameter

| Parameter | Typ | Beschreibung |
|-----------|-----|-------------|
| `balance` | number | Der momentane Pool-Kontostand des Benutzers in der kleinstmöglichen Angabe. Beinhaltet Beträge, die noch nicht auf der Blockchain bestätigt wurden |
| `confirmedBalance` | number | Der momentane Pool-Kontostand des Benutzers in der kleinstmöglichen Angabe. Beinhaltet nur Beträge, die vom Betreiber als bestätigt aufgefasst werden und zur Auszahlung verfügbar sind |
| `payoutRequestActive` | boolean | `true`, falls es eine aktive Auszahlungs-Anforderung gibt, ansonsten`false` |

##### Beispiel
 
```json
{
    "message": "balance",
    "balance": 123010221,
    "confirmedBalance": 122010202,
    "payoutRequestActive": false
}
```

#### `payout`

Wird vom Client gesendet, um eine Auszahlung anzufordern. Je nach Server-Konfiguration ist dies mit einer zusätzlichen Gebühr verbunden.

##### Parameter

| Parameter | Typ | Beschreibung |
|-----------|-----|--------------|
| `proof`   | string | Base64-kodierter string `POOL_PAYOUT`, mit anschließender Byte-Repräsentation der connection nonce |

##### Beispiel
 
```json
{
    "message": "payout",
    "proof": "J4VEpFwKNoklkyeUCvzBvixate3yPbDyYfWSusF/..."
}
```
