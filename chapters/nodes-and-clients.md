# Node und Client-Arten

Es gibt zwei Arten von Clients im Nimiq Netzwerk: NodeJS-Clients und Browser-Clients. Beide basieren auf demselben JavaScript-Code und können einen der drei Node-Arten ausführen: Full, Light und Nano. Einige Kombinationen von Node/Client-Arten werden auf längere Sicht häufiger sein als andere (z.B. werden Browser-Clients hauptsächlich light/nano Nodes sein und nicht unbedingt als Miner teilnehmen).

## Client-Typen

Je nachdem, welche Plattform (Browser/NodeJS) benutzt wird, um den Client auszuführen wird die Node als Browser- oder NodeJS-Client arbeiten. Eine weiterer großer Unterschied zwischen den zwei ist, dass Browser-Clients die Blockchain-Daten in  [IndexedDB](https://developers.google.com/web/ilt/pwa/working-with-indexeddb#what_is_indexeddb) speichern (auf ~50 MB begrenzt), wohingegen NodeJS-Clients auf [LevelDB](https://github.com/google/leveldb) basieren (keine Größenbeschränkung). Damit weniger Code gewartet werden muss, der für beide Client-Typen wiederverwendet werden kann, haben wir [JungleDB](https://github.com/nimiq-network/jungle-db) entwickelt.

### NodeJS-Client

NodeJS-Clients eignen sich am Besten für Server. Dank ihrer praktisch unbegrenzten Speicherkapazität und öffentlichen IP-Adresse fungieren sie als das Backbone des Netzwerks. Sie kommunizieren miteinander via [WebSockets](https://developer.mozilla.org/en-US/docs/Web/API/WebSocket) und sind Einstiegspunkte und Vermittler für Browser-zu-Browser WebRTC-Verbindungen.

Browser APIs sind beschränkt auf sichere Quellen. Das heißt, dass NodeJS-Clients eine verschlüsselte Verbindung für Browser-Clients bereitstellen müssen, damit sie sich verbinden können. Das setzt eine Domain und ein SSL-Zertifikat voraus.

### Browser-Client

Browser-Clients sind auf Browser-Engines aufgebaut und daher komplett frei von Installationen. Um sich mit dem Netzwerk zu verbinden, stellen sie mindestens eine Verbindung zu einem NodeJS-Client her. Sobald sie ihre erste Verbindung hergestellt haben, beginnen sie Browser-zu-Browser-Verbindungen mit Hilfe von NodeJS-Clients als Vermittler zu eröffnen. Browser-Client können auch Vermittler für weitere Browser-zu-Browser-Verbindungen sein.

Durch den Einsatz von [Babel](https://babeljs.io/) werden auch ältere Browser-Versionen unterstützt. WebRTC ist jedoch eine Voraussetzung, also können sich nur Browser mit WebRTC-Unterstützung mit dem Netzwerk verbinden.

Ein Blick auf [die einfachste Anwendung auf der Nimiq Blockchain](https://demo.nimiq.com/) kann dabei helfen, die verschiedenen Client/Node-Arten zu verstehen.

## Node-Typen

Unterschiedliche Node-Arten werden für verschiedene Einsatzzwecke herangezogen. Sie unterscheiden sich durch die Menge an Daten, die sie benötigen, um mit dem Netzwerk zu synchronisieren und beizutreten, was sich direkt auf die Dauer, Konsens zu erreichen (Fähigkeit Transaktionen zu verarbeiten und überprüfen), auswirkt.

### Full-Nodes

Nodes dieses Typen laden die vollständige Blockchain und brauchen daher mehr Speicher. Wir erwarten, dass diese Art von Node ausschließlich mit NodeJS verwendet wird, obwohl sie theoretisch auch in einem Browser laufen könnte.

### Light-Nodes

Diese Art von Node synchronisiert sich sicher zu nahezu vollständigem Konsens durch Laden von nur ungefähr 100 MB. Um das zu erreichen, benutzen sie eine LightChain, welche durch Verwendung von NiPoPoWs anstelle des vollständigen Blockchain-Verlaufs initialisiert wird. Nach Initialisierung verhält sie sich jedoch wie eine reguläre Full-Blockchain.

### Nano-Nodes

Sie ist die bevorzugte Art von Node für Browser-Clients aufgrund des geringen benötigten Datenvolumens (~1 MB), um Konsens zu erreichen.
