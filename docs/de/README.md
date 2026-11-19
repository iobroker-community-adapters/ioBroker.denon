![Logo](media/denon.png)

# ioBroker.denon – Denon/Marantz AV-Receiver

Mit diesem Adapter steuerst du deinen Denon- oder Marantz-AV-Receiver aus ioBroker: ein- und ausschalten, Lautstärke,
Eingang, Surround-Modus, Zone 2/3, Tuner und viele Klang- und Bildeinstellungen. Änderungen am Receiver (z. B. über die
Fernbedienung) erscheinen sofort in ioBroker.

## Haftungsausschluss

DENON und Marantz sind Marken der D&M Holdings Inc.
Die Entwickler dieses Moduls stehen in keiner Verbindung zur D&M Holdings Inc. oder zu deren Tochtergesellschaften,
Logos oder Marken und werden von diesen nicht unterstützt.

## Schnellstart

1. Am Receiver im Menü „Netzwerk“ → „Netzwerk-Strg.“ auf **„Immer ein“** stellen (Details [unten](#netzwerkeinstellungen-am-av-receiver)).
2. In ioBroker im Reiter „Adapter“ nach **denon** suchen und auf „+“ klicken.
3. In der Instanz die IP-Adresse des Receivers eintragen (oder aus der Liste der gefundenen Geräte wählen) und speichern.
4. Nach wenigen Sekunden ist `info.connection` grün und unter `denon.0` erscheinen die Datenpunkte deines Receivers.

Mehr ist nicht nötig. Welche Datenpunkte angelegt werden, hängt vom Modell ab: Der Adapter legt nur an, was dein Receiver auch kann.

Voraussetzungen: Denon- oder Marantz-AV-Receiver im Netzwerk (LAN oder WLAN), Node.js >= 22.19, js-controller >= 7.2.5, Admin >= 8.3.5.

## Die wichtigsten Datenpunkte

Für VIS, Blockly oder Skripte reichen meistens diese (statt `main` gibt es `zone2` und `zone3` genauso):

| Datenpunkt | Wozu | Beispiel |
|---|---|---|
| `denon.0.system.power` | ganzen Receiver ein / Standby | `true` = an, `false` = alles aus |
| `denon.0.main.power` | nur die Hauptzone ein/aus | `true` |
| `denon.0.main.volume` | Lautstärke (0 ... 98, wie im Display) | `40` |
| `denon.0.main.mute` | stumm | `true` |
| `denon.0.main.input` | Eingang wählen | `TV` oder der Name aus dem Receiver-Menü, z. B. `TV Audio` |
| `denon.0.main.quickSelect` | Quick Select (Marantz: Smart Select) abrufen | `1` |
| `denon.0.main.surroundMode` | Surround-Modus | `STEREO` |
| `denon.0.info.connection` | Verbindung zum Receiver steht | `true` |

Tasten wie `main.volumeUp` oder `remote.up` werden mit `true` ausgelöst. Alle weiteren Datenpunkte: [Alle Datenpunkte](states.md).

## Konfiguration

Normalerweise muss nur die IP-Adresse eingetragen werden.

| Einstellung | Bedeutung |
|---|---|
| IP-Adresse des Receivers | z. B. `192.168.1.50` (IPv6 und Hostname gehen auch). Solange die Instanz läuft, werden gefundene Receiver vorgeschlagen. Die Suche findet nur Geräte im selben Netz wie ioBroker, sonst die Adresse von Hand eintragen. |
| Aktualisierungsintervall (ms) | Wie oft Werte gelesen werden, die der Receiver nicht selbst meldet (z. B. Sleep-Timer). Standard 7000 = alle 7 Sekunden |
| Pause zwischen Befehlen (ms) | Nur erhöhen, wenn der Receiver Befehle verpasst. Standard 100, mindestens 50 |
| Port | Immer 23. Nur für einen Seriell-Netzwerk-Wandler ändern |
| Marke, Modell und Zonen automatisch erkennen | Empfohlen. Liest beim Start Marke, Modell und Anzahl der Zonen aus dem Receiver. Gesteuert wird trotzdem immer über Port 23 |

## Netzwerkeinstellungen am AV-Receiver

1. Taste SETUP auf der Fernbedienung drücken, das Menü erscheint im Gerätedisplay und auf dem Fernseher
2. „Netzwerk“ → „Einstellungen“ wählen und einstellen
    * DHCP: „Ein“, wenn ein DHCP-Server (z. B. Router) vorhanden ist, sonst IP-Adresse, Subnetzmaske und Gateway manuell setzen
    * Proxy: „Aus“
3. „Netzwerk“ → „Netzwerk-Strg.“ (bei manchen Modellen „IP-Steuerung“) auf **„Immer ein“** stellen.
   Mit „Aus im Standby“ ignoriert der Receiver im Standby alle Netzwerkbefehle und lässt sich von ioBroker nicht einschalten.

Tipp: Im Router dem Receiver eine feste IP-Adresse geben, damit sich die Adresse nicht ändert.

## Häufige Fragen

**`info.connection` wird nicht grün**
* Stimmt die IP-Adresse? Sie steht im Receiver-Menü unter „Netzwerk“ → „Information“ oder in der Geräteliste des Routers.
* Der Receiver erlaubt nur **eine** Steuerverbindung. Läuft eine andere App, die ihn steuert (Home Assistant, ein zweiter ioBroker, ein Skript), bekommt der Adapter keine Verbindung.
* Steht „Netzwerk-Strg.“ auf „Immer ein“?

**Der Receiver lässt sich nicht einschalten**
* „Netzwerk-Strg.“ steht wahrscheinlich auf „Aus im Standby“. Auf „Immer ein“ stellen.

**Ein Datenpunkt fehlt**
* Der Adapter legt nur Datenpunkte an, auf die der Receiver antwortet. Fehlt einer, kann dein Modell diese Funktion nicht (z. B. Bass/Höhen in Zone 2 beim AVR-X2200W).
* Manche Werte meldet der Receiver erst, wenn die Zone eingeschaltet ist.

**Ein Eingang fehlt in der Auswahl oder hat einen anderen Namen**
* Die Liste kommt aus dem Receiver: Eingänge, die im Menü unter „Eingänge“ ausgeblendet sind, fehlen auch hier. Umbenannte Eingänge erscheinen mit dem neuen Namen. Nach einer Änderung im Receiver die Instanz einmal neu starten.

**Display, Cover oder Wiedergabetasten fehlen**
* Modelle mit HEOS (ab etwa 2016) geben diese Infos nicht über die Steuerverbindung heraus.

## Update von 1.x

Mit Version 2.0 haben alle Datenpunkte neue Namen (z. B. `zoneMain.powerZone` → `main.power`), der alte Objektbaum wird beim
ersten Start gelöscht. Skripte, VIS und Aliase müssen angepasst werden, History-Einstellungen an den Datenpunkten gehen verloren.
Bitte vorher ein Backup machen. Die komplette Tabelle alt → neu: [Update von 1.x](upgrade-1x.md)

## Alle Datenpunkte und Details

* [Alle Datenpunkte](states.md) – jeder Datenpunkt mit Beschreibung
* [Mehrzonen-Verstärker (DENON POA-3012CI)](states.md#mehrzonen-verstärker-denon-poa-3012ci)
* [Technische Details](states.md#technische-details) – Verbindung, Befehlsabstand, Unterschiede Denon/Marantz

## Fehlende Funktionen & Fehler

Fehlende Funktionen oder Fehler bitte als [Issue](https://github.com/iobroker-community-adapters/ioBroker.denon/issues) melden,
mit Debug-Log und Modell des Receivers.

Version 2.0 ist mit einem Denon AVR-X2200W getestet. Tester haben sie erfolgreich mit einem Marantz CINEMA 60 und einem Marantz SR6011 genutzt.
