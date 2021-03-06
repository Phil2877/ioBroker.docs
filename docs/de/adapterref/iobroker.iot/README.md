---
translatedFrom: en
translatedWarning: Wenn Sie dieses Dokument bearbeiten möchten, löschen Sie bitte das Feld "translationsFrom". Andernfalls wird dieses Dokument automatisch erneut übersetzt
editLink: https://github.com/ioBroker/ioBroker.docs/edit/master/docs/de/adapterref/iobroker.iot/README.md
title: ioBroker IoT Adapter
hash: AyRn68CYR4s+qIW3NFGuHjhm45ipfK0KTILNm9iI+fY=
---
![Logo](../../../en/adapterref/iobroker.iot/admin/iot.png)

![Anzahl der Installationen](http://iobroker.live/badges/iot-stable.svg)
![NPM-Version](http://img.shields.io/npm/v/iobroker.iot.svg)
![Downloads](https://img.shields.io/npm/dm/iobroker.iot.svg)
![NPM](https://nodei.co/npm/iobroker.iot.png?downloads=true)

# IoBroker IoT Adapter
Dieser Adapter ist NUR für die Kommunikation mit Amazon Alexa, Google Home und Nightscout vorgesehen.
Es ist nicht für den Remotezugriff auf Ihre ioBroker-Instanz vorgesehen. Verwenden Sie dazu den ioBroker.cloud-Adapter.

** Dieser Adapter verwendet Sentry-Bibliotheken, um Ausnahmen und Codefehler automatisch an die Entwickler zu melden. ** Weitere Details und Informationen zum Deaktivieren der Fehlerberichterstattung finden Sie unter [Sentry-Plugin-Dokumentation](https://github.com/ioBroker/plugin-sentry#plugin-sentry)! Sentry Reporting wird ab js-controller 3.0 verwendet.

## Die Einstellungen
Um einen Cloud-Adapter zu verwenden, müssen Sie sich zuerst in der ioBroker-Cloud registrieren. [https://iobroker.pro](https://iobroker.pro).

[Verweis auf die Einstellungen des Google API-Typs](https://developers.google.com/actions/smarthome/guides/)

![Intro](../../../en/adapterref/iobroker.iot/img/intro.png)

### Sprache
Wenn Sie "Standard" -Sprache auswählen, werden die intelligenten Namen von Geräten und Aufzählungen nicht übersetzt. Wenn eine Sprache angegeben ist, werden alle bekannten Namen in diese Sprache übersetzt.
Zu Demonstrationszwecken wird schnell zwischen vielen Sprachen gewechselt.

### Platziere die Funktion zuerst in den Namen
Ändern Sie die Reihenfolge der Funktionen und Rollen in selbst generierten Namen:

- falls falsch: "Raumfunktion", z. "Wohnzimmer Dimmer"
- falls zutreffend: "Veranstaltungsraum", z. "Dimmer Wohnzimmer"

### Wörter verketten mit
Sie können das Wort definieren, das zwischen Funktion und Raum platziert wird. Z.B. "in" und von "Dimmer Wohnzimmer" wird "Dimmer im Wohnzimmer" sein.

Dies wird jedoch nicht empfohlen, da die Erkennungs-Engine ein weiteres Wort analysieren muss und dies zu Missverständnissen führen kann.

### AUS-Pegel für Schalter
Einige Gruppen bestehen aus gemischten Geräten: Dimmer und Schalter. Es ist erlaubt, sie mit "EIN" - und "AUS" -Befehlen und mit Prozenten zu steuern.
Wenn der Befehl "Auf 30% eingestellt" und der * AUS-Pegel "" 30% "ist, werden die Schalter eingeschaltet. Mit dem Befehl" Auf 25% einstellen "werden alle Schalter ausgeschaltet.

Wenn der Befehl "AUS" ist, speichert der Adapter den aktuellen Dimmerpegel, wenn der tatsächliche Wert über oder gleich "30%" liegt.
Später, wenn der neue Befehl "EIN" kommt, schaltet der Adapter den Dimmer nicht auf 100%, sondern auf den Pegel im Speicher.

Beispiel:

- Angenommen, der *AUS-Wert* beträgt 30%.
- Das virtuelle Gerät "Light" verfügt über zwei physische Geräte: *switch* und *dimmer*
- Befehl: "Licht auf 40% einstellen". Der Adapter speichert diesen Wert für *Dimmer* stellt ihn auf "Dimmer" und schaltet den *Schalter* ein.
- Befehl: "Licht ausschalten". Der Adapter stellt den *Dimmer* auf 0% und schaltet den *Schalter* aus.
- Befehl: "Licht einschalten". *Dimmer* => 40%, *Schalter* => ON.
- Befehl: "Licht auf 20% einstellen". *Dimmer* => 20%, *Schalter* => AUS. Der Wert für den Dimmer wird nicht gespeichert, da er unter *AUS* liegt.
- Befehl: "Licht einschalten". *Dimmer* => 40%, *Schalter* => ON.

### Von ON
Sie können das Verhalten des EIN-Befehls für den Nummernstatus auswählen. Der spezifische Wert kann ausgewählt werden oder der letzte Wert ungleich Null wird verwendet.

### Antwort schreiben an
Für jeden Befehl wird die Textantwort generiert. Hier können Sie die Objekt-ID definieren, in die dieser Text geschrieben werden muss. Z.B. *sayit.0.tts.text*

### Farben
Momentan unterstützt nur Englisch Alexa die Farbkontrolle.
Der Kanal muss 4 Zustände mit folgenden Rollen haben:

- level.color.saturation (erforderlich für die Erkennung des Kanals),
- level.color.hue,
- level.dimmer,
- Schalter (optional)

```
Alexa, set the "device name" to "color"
Alexa, turn the light fuschia
Alexa, set the bedroom light to red
Alexa, change the kitchen to the color chocolate
```

### Sperren
Um die Sperren sperren zu können, muss der Status die Rolle "switch.lock" und native.LOCK_VALUE haben, um den Sperrstatus zu bestimmen.

```
Alexa, is "lock name" locked/unlocked
Alexa, lock the "lock name"
```

## Wie Namen generiert werden
Der Adapter versucht, virtuelle Geräte für die Smart Home-Steuerung zu generieren (z. B. Amazon Alexa oder Google Home).

Dafür gibt es zwei wichtige Aufzählungen: Räume und Funktionen.

Die Zimmer sind wie: Wohnzimmer, Bad, Schlafzimmer.
Funktionen sind wie: Licht, Blind, Heizung.

Folgende Bedingungen müssen erfüllt sein, um den Status in die automatisch generierte Liste aufzunehmen:

- Der Status muss sich in einer "Funktions" -Aufzählung befinden.
- Der Status muss eine Rolle haben ("Status", "Schalter" oder "Ebene. *", Z. B. Ebene.Dimmer), wenn er nicht direkt in "Funktionen" enthalten ist.

Es kann sein, dass sich der Kanal in den "Funktionen" befindet, sich aber nicht selbst angibt.

- Der Status muss beschreibbar sein: common.write = true
- Der Zustandsdimmer muss common.type als 'number' haben.
- Die Zustandsheizung muss eine gemeinsame Einheit als '°C', '°F' oder '° K' und einen gemeinsamen Typ als 'Nummer' haben.

Befindet sich der Status nur in "Funktionen" und nicht in einem "Raum", wird der Name des Status verwendet.

Die Statusnamen werden aus Funktion und Raum generiert. Z.B. Alle *Lichter* im *Wohnzimmer* werden im virtuellen Gerät *Wohnzimmerlicht* gesammelt.
Der Benutzer kann diesen Namen nicht ändern, da er automatisch generiert wird.
Wenn sich der Aufzählungsname ändert, wird auch dieser Name geändert. (z. B. die Funktion "Licht" wurde in "Lichter" geändert, sodass das *Wohnzimmerlicht* in *Wohnzimmerlichter* geändert wird.)

Alle Regeln werden ignoriert, wenn der Status common.smartName hat. In diesem Fall wird nur der Smart Name verwendet.

Wenn *common.smartName* **false** ist, wird der Status oder die Aufzählung nicht in die Listengenerierung einbezogen.

Im Konfigurationsdialog können Sie die einzelnen Status bequem entfernen und virtuellen Gruppen oder als einzelnes Gerät hinzufügen.
![Aufbau](../../../en/adapterref/iobroker.iot/img/configuration.png)

Wenn die Gruppe nur einen Status hat, kann sie umbenannt werden, da hierfür der SmartName des Status verwendet wird.
Wenn die Gruppe mehr als einen Status hat, muss die Gruppe über die Namen der Aufzählung umbenannt werden.

Um eigene Gruppen zu erstellen, kann der Benutzer den Adapter "Szenen" installieren oder "Skript" im Javascript-Adapter erstellen.

### Ersetzt
Sie können Zeichenfolgen angeben, die automatisch in den Gerätenamen ersetzt werden können. Zum Beispiel, wenn Sie Ersetzungen auf setzen:

```.STATE,.LEVEL```, so all ".STATE" and ".LEVEL" will be deleted from names. Be careful with spaces.
If you will set ```.STATE, .LEVEL```, so ".STATE" and " .LEVEL" will be replaced and not ".LEVEL".

## Helper states
- **smart.lastObjectID**: This state will be set if only one device was controlled by home skill (alexa, google home).
- **smart.lastFunction**: Function name (if exists) for which last command was executed.
- **smart.lastRoom**:     Room name (if exists) for which last command was executed.
- **smart.lastCommand**:  Last executed command. Command can be: true(ON), false(OFF), number(%), -X(decrease at x), +X(increase at X)
- **smart.lastResponse**: Textual response on command. It can be sent to some text2speech (sayit) engine.

## IFTTT
[instructions](doc/ifttt.md)

## Services
There is a possibility to send messages to cloud adapter.
If you call ```[POST]https://service.iobroker.in/v1/iotService?service=custom_<NAME>&key=<XXX>&user=<USER_EMAIL>``` und value as payload.

```

curl --data "myString" https://service.iobroker.in/v1/iotService?service=custom_<NAME>&key=<XXX>&user= <USER_EMAIL>

```

or

```[GET]https://service.iobroker.in/v1/iotService?service=custom_<NAME>&key=<XXX>&user=<USER_EMAIL>&data=myString```

Wenn Sie in den Einstellungen im Feld "Weiße Liste für Dienste" den Namen *custom_test* festlegen und mit "custom_test" als Dienstnamen aufrufen, wird der Status **cloud.0.services.custom_test** auf *myString gesetzt*

Sie können "*" in die weiße Liste schreiben und alle Dienste werden zugelassen.

Hier finden Sie Anweisungen zur Verwendung mit [Tasker](doc/tasker.md).

Der IFTTT-Dienst ist nur zulässig, wenn der IFTTT-Schlüssel festgelegt ist.

Reservierte Namen sind "ifttt", "text2command", "simpleApi", "swagger". Diese müssen ohne das Präfix ```"custom_"``` verwendet werden.

### Text2command
Sie können "text2command" in eine weiße Liste schreiben. Sie können eine POST-Anfrage an ```https://service.iobroker.in/v1/iotService?service=text2command&key=<user-app-key>&user=<USER_EMAIL>``` senden, um Daten in die Variable *text2command.X.text* zu schreiben.

Sie können auch die GET-Methode verwenden ```https://service.iobroker.in/v1/iotService?service=text2command&key=<user-app-key>&user=<USER_EMAIL>&data=<MY COMMAND>```

"X" kann in den Einstellungen mit der Option "text2command-Instanz verwenden" definiert werden.

## Benutzerdefinierte Fertigkeit
Die Antworten für benutzerdefinierte Fertigkeiten können auf zwei Arten verarbeitet werden:

- text2command
- Javascript

### Text2command
Wenn im Konfigurationsdialog die Instanz *text2command* definiert ist, wird die Frage an die Instanz gesendet.

* text2command * muss so konfiguriert sein, dass die erwartete Phrase analysiert und die Antwort zurückgegeben wird.

### Javascript
Es besteht die Möglichkeit, die Frage direkt mit dem Skript zu bearbeiten. Es ist standardmäßig aktiviert, wenn keine *text2command* -Instanz ausgewählt ist.

Wenn die Instanz *text2command* definiert ist, muss diese Instanz die Antwort bereitstellen, und die Antwort von *script* wird ignoriert.

Der Adapter liefert die Details in zwei Zuständen mit unterschiedlicher Detailstufe

* **smart.lastCommand** enthält den empfangenen Text einschließlich einer Information über den Abfragetyp (Absicht). Beispiel: "askDevice Status Rasenmäher"
* ** smart.lastCommandObj *** enthält eine JSON-Zeichenfolge, die auf ein Objekt analysiert werden kann, das die folgenden Informationen enthält
 * **Wörter** enthält die empfangenen Wörter in einem Array
 * **intent** enthält den Abfragetyp. Mögliche Werte sind derzeit "askDevice", "controlDevice", "actionStart", "actionEnd", "askWhen", "askWhere", "askWho".
 * **Geräte-ID** enthält eine Geräte-ID, die angibt, an welches Gerät, an das die Anforderung gesendet wurde und das von Amazon gesendet wurde, eine leere Zeichenfolge ist, wenn sie nicht angegeben wird
 * **sessionId** enthält eine sessionId der Skill-Sitzung. Sollte identisch sein, wenn mehrere von Amazon gelieferte Befehle gesprochen wurden, ist die Zeichenfolge leer, wenn sie nicht angegeben wird
 * **Benutzer-ID** enthält eine Benutzer-ID des Gerätebesitzers (oder möglicherweise später des Benutzers, der mit der Fertigkeit interagiert hat), die von Amazon bereitgestellt wird, ist eine leere Zeichenfolge, wenn sie nicht angegeben wird

 Weitere Informationen darüber, wie die Wörter erkannt werden und welche Art von Abfragen die Alexa Custom Skill unterscheidet, finden Sie unter https://forum.iobroker.net/viewtopic.php?f=37&t=17452.

** Ergebnis über den Status smart.lastResponse zurückgeben **

Die Antwort muss innerhalb von 200 ms im Status "smart.lastResponse" gesendet werden und kann eine einfache Textzeichenfolge oder ein JSON-Objekt sein.
Wenn es sich um eine Textzeichenfolge handelt, wird dieser Text als Antwort auf die Fertigkeit gesendet.
Wenn der Text ein JSON-Objekt ist, können die folgenden Schlüssel verwendet werden:

* **responseText** muss den Text enthalten, um zu Amazon zurückzukehren
* **shouldEndSession** ist ein Boolescher Wert und steuert, ob die Sitzung geschlossen wird, nachdem die Antwort gesprochen wurde, oder offen bleibt, um eine weitere Spracheingabe zu akzeptieren.

** Ergebnis per Nachricht an iot-Instanz zurückgeben **

Die iot-Instanz akzeptiert auch eine Nachricht mit dem Namen "alexaCustomResponse", die den Schlüssel "response" enthält, mit einem Objekt, das die Schlüssel **responseText** und **shouldEndSession** enthalten kann, wie oben beschrieben.
Die iot-Instanz antwortet nicht auf die Nachricht!

** Beispiel eines Skripts, das Texte verwendet **

```
// important, that ack=true
on({id: 'iot.0.smart.lastCommand', ack: true, change: 'any'}, obj => {
    // you have 200ms to prepare the answer and to write it into iot.X.smart.lastResponse
    setState('iot.0.smart.lastResponse', 'Received phrase is: ' + obj.state.val); // important, that ack=false (default)
});
```

** Beispiel eines Skripts, das JSON-Objekte verwendet **

```
// important, that ack=true
on({id: 'iot.0.smart.lastCommandObj', ack: true, change: 'any'}, obj => {
    // you have 200ms to prepare the answer and to write it into iot.X.smart.lastResponse
    const request = JSON.parse(obj.state.val);
    const response = {
        'responseText': 'Received phrase is: ' + request.words.join(' ') + '. Bye',
        'shouldEndSession': true
    };

    // Return response via state
    setState('iot.0.smart.lastResponse', JSON.stringify(response)); // important, that ack=false (default)

    // or alternatively return as message
    sendTo('iot.0', response);
});
```

### Private Wolke
Wenn Sie private Skill / Action / навык für die Kommunikation mit `Alexa/Google Home/Алиса` verwenden, haben Sie die Möglichkeit, die IoT-Instanz zu verwenden, um die Anforderungen daraus zu verarbeiten.

Z.B. für `yandex alice`:

```
const OBJECT_FROM_ALISA_SERVICE = {}; // object from alisa service or empty object
OBJECT_FROM_ALISA_SERVICE.alisa = '/path/v1.0/user/devices'; // called URL, 'path' could be any text, but it must be there
sendTo('iot.0', 'private', {type: 'alisa', request: OBJECT_FROM_ALISA_SERVICE}, response => {
    // Send this response back to alisa service
    console.log(JSON.stringify(response));
});
```

Folgende Typen werden unterstützt:

- `alexa` - mit Amazon Alexa oder Amazon Custom Skill handeln
- `ghome` - Handeln mit Google Actions über Google Home
- `alisa` - mit Yandex Алиса handeln
- `ifttt` - verhält sich wie IFTTT (eigentlich nicht erforderlich, aber zu Testzwecken)

## Changelog
### 1.4.11 (2020-04-26)
* (bluefox) fixed IOBROKER-IOT-REACT-F

### 1.4.10 (2020-04-24)
* (bluefox) Fixed crashes reported by sentry

### 1.4.7 (2020-04-23)
* fix iot crash when timeouts in communications to Google happens (Sentry IOBROKER-IOT-2)
* fix iot crash when google answers without customData (Sentry IOBROKER-IOT-1)

### 1.4.6 (2020-04-18)
* (Apollon77) Add Sentry error reporting to React Frontend

### 1.4.4 (2020-04-14)
* (Apollon77) remove js-controller 3.0 warnings and replace adapter.objects access
* (Apollon77) add linux dependencies for canvas library
* (Apollon77) add sentry configuration

### 1.4.2 (2020-04-08)
* (TA2k) Fix updateState for Google Home

### 1.4.1 (2020-04-04)
* (bluefox) The blood glucose request supported now

### 1.3.4 (2020-02-26)
* (TA2k) Fixed deconz issues in Google Home

### 1.3.3 (2020-02-12)
* (Apollon77) fix alisa error with invalid smartName attributes

### 1.3.2 (2020-02-10)
* (Apollon77) usage with all kinds of admin ports and reverse proxies optimized

### 1.3.1 (2020-02-09)
* (Apollon77) Dependency updates
* (APollon77) Make compatible with Admin > 4.0 because of updated socket.io

### 1.2.1 (2020-01-18)
* (bluefox) Fixed problem if the port of admin is not 8081

### 1.2.0 (2020-01-04)
* (TA2k) Google Home handling and visualization improved.

### 1.1.10 (2020-01-03)
* (bluefox) Now is allowed to selected the temperature values as alexa states
* (bluefox) Allowed to set type immediately after insertion of new state

### 1.1.9 (2019-11-27)
* (bluefox) Fixed: sometimes the configuration could not be loaded

### 1.1.8 (2019-09-12)
* (bluefox) Optimization of googe home communication was done

### 1.1.7 (2019-09-11)
* (bluefox) The sending rate to google home is limited now

### 1.1.6 (2019-09-11)
* (TA2k) Room fix for Google Home and LinkedDevices

### 1.1.4 (2019-09-10)
* (bluefox) decreased keepalive value to fix issue with disconnect

### 1.1.3 (2019-09-09)
* (TA2k) Google Home problem fixed with LinkedDevices

### 1.1.0 (2019-09-06)
* (bluefox) Added support of aliases

### 1.0.8 (2019-09-03)
* (TA2k) Improved support for Google Home
* (TA2k) Added auto detection for RGB, RGBSingle, Hue, CT, MediaDevice, Switch, Info, Socket, Light, Dimmer, Thermostat, WindowTilt, Blinds, Slider
* (TA2k) Added support for manualy adding states as devices
* (TA2k) Fix update state after Sync
* (TA2k) Added typical Google Home devices and traits/actions
* (TA2k) Fix only process update message when Alexa is checked in the options

### 1.0.4 (2019-08-01)
* (bluefox) Fixed password encoding. Please enter password anew!

### 1.0.3 (2019-07-30)
* (bluefox) Fixed language issues for google home and yandex alice

### 1.0.1 (2019-07-26)
* (bluefox) Support of private skills/actions was added.

### 1.0.0 (2019-07-14)
* (TA2k) Google Home list was added

### 0.5.0 (2019-06-29)
* (bluefox) tried to add yandex Alisa

### 0.4.3 (2019-04-14)
* (Apollon77) Change enable/disable of Amazon Alexa and of Google Home from configuration to be really "active if selected".

### 0.4.2 (2019-03-10)
* (bluefox) Allowed the enable and disable of Amazon Alexa and of Google Home from configuration.

### 0.4.1 (2019-02-19)
* (bluefox) Add version check to google home

### 0.3.1 (2019-01-13)
* (bluefox) Blockly was fixed

### 0.3.0 (2018-12-30)
* (bluefox) Detection of google devices was fixed

### 0.2.2 (2018-12-21)
* (bluefox) Generation of new URL key was added

### 0.2.0 (2018-12-18)
* (bluefox) Change the name of adapter

### 0.1.8 (2018-10-21)
* (bluefox) Added extended diagnostics

### 0.1.7 (2018-10-14)
* (bluefox) The configuration dialog was corrected
* (bluefox) The possibility to create the answer with script for the custom skill was implemented.

### 0.1.4 (2018-09-26)
* (bluefox) Initial commit

## License
The MIT License (MIT)

Copyright (c) 2018-2020 bluefox <dogafox@gmail.com>

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.