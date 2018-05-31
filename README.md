# Nimiq Blockchain
[Nimiq](https://nimiq.com/) ist ein reibungsloses Bezahlungsprotokoll für das Web. Für eine High-Level Einführung können Sie das [Nimiq White Paper](https://medium.com/nimiq-network/nimiq-a-peer-to-peer-payment-protocol-native-to-the-web-ffd324bb084) (Englisch) lesen. Der Quelltext ist auf [GitHub](https://github.com/nimiq-network/core) verfügbar.

Diese Entwickler-Referenz enthält:

* [Datenschemas](#datenschemas)
* [High-Level-Konzepte und Architektur](#high-level-konzepte)

## Datenschemas

Datenschemas der Übertragung, mit Fokus auf Feldern, Byte Größen und kurzen Erklärungen.

* [Transaktionen](chapters/transactions.md): basic und erweitert
* [Blockchain](chapters/block.md): Block, Header, Interlink, Body
* [Accounts and Verträge](chapters/accounts-and-contracts.md): Einfacher Account, Vesting and Hashed Time-Locked Contracts
* [Account-Baum](chapters/account-tree.md): Details zum Patricia Merkle-Baum, der benutzt wird, um Accounts zu speichern
* [Primitives](chapters/primitives.md): Nicht zusammengesetzte Elemente, z.B. Hashes, Adressen
* [Nachrichten](chapters/messages.md): Alle p2p-artige Nachrichten zwischen Nodes

## High-Level-Konzepte

High-Level-Konzepte in Nimiq.

* [Konstanten](chapters/constants.md): Konfiguration des Nimiq core
* [Nodes and Clients](chapters/nodes-and-clients.md): Node.js und Browser, full, small, und nano
* [Versorgung und Belohnung](chapters/supply-and-reward.md): Gesamte Versorgung (total supply), Belohnungen (rewards) und Anpassen der Schwierigkeit (difficulty)
* [Verifikation](chapters/verify.md): Validierungsregeln für die Nimiq Blockchain

## Weitere Infos
Weitere Infos für einen Überblick über das Nimiq-Projekt und System:
* [API Dokumentation](https://github.com/nimiq-network/core/blob/master/dist/API_DOCUMENTATION.md)
* [Browser Miner](https://miner.nimiq.com) und [Browser Wallet](https://safe.nimiq.com)
* [Contributing Guidelines](https://github.com/nimiq-network/core/blob/master/.github/CONTRIBUTING.md) und [Code of Conduct](https://github.com/nimiq-network/core/blob/master/.github/CODE_OF_CONDUCT.md)
* Dieses Projekt ist unter der [Apache License 2.0](https://github.com/nimiq-network/core/blob/master/LICENSE.md) lizensiert.

Wir wollen unsere Konzepte und Herangehensweisen so klar wie möglich herüberbringen. Je einfacher es ist, in die Details zu gehen, desto mehr werden es tun, was wiederum in einen tieferen Peer-Reviewing-Prozess resultiert. Dieser ist essentiell für das Härten unseres Protokolls.
Verbesserungen dieser Referenz durch Pull Requests sind sehr willkommen.
