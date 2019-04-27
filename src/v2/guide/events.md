---
title: Afhandelen van 'events'
type: guide
order: 9
---

## Luisteren naar 'events'

Het is mogelijk om met de `v-on` 'directive' naar DOM 'events' te luisteren en JavaScript uit te voeren wanneer ze geactiveerd worden.

Bijvoorbeeld:

``` html
<div id="voorbeeld-1">
  <button v-on:click="teller += 1">1 toevoegen</button>
  <p>Bovenstaande knop is {{ teller }} geklikt.</p>
</div>
```
``` js
var voorbeeld1 = new Vue({
  el: '#voorbeeld-1',
  data: {
    teller: 0
  }
})
```

Resultaat:

{% raw %}
<div id="voorbeeld-1" class="demo">
  <button v-on:click="teller += 1">1 toevoegen</button>
  <p>Bovenstaande knop is {{ teller }} geklikt.</p>
</div>
<script>
var voorbeeld1 = new Vue({
  el: '#voorbeeld-1',
  data: {
    teller: 0
  }
})
</script>
{% endraw %}

## Afhandelen van een 'event' met een methode

Vaak zal de logica te complex zijn om de JavaScript in de waarde van het `v-on`-attribuut te houden. Daarom accepteert `v-on` ook de naam van een methode die uitgevoerd moet worden wanneer het 'event' geactiveerd wordt.

Bijvoorbeeld:

``` html
<div id="voorbeeld-2">
  <!-- `groet` is de naam van de methode (hieronder gedefinieerd) -->
  <button v-on:click="groet">Groet</button>
</div>
```

``` js
var voorbeeld2 = new Vue({
  el: '#voorbeeld-2',
  data: {
    naam: 'John Duck'
  },
  // definieer methodes onder het `methods`-object
  methods: {
    groet: function (event) {
      // `this` in 'methods' verwijst naar de Vue-instantie
      alert('Hallo ' + this.naam + '!')
      // `'event'` is het oorspronkelijke DOM 'event'
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})

// het is mogelijk om methodes aan te roepen in JavaScript
voorbeeld2.groet() // => 'Hallo John Duck!'
```

Resultaat:

{% raw %}
<div id="voorbeeld-2" class="demo">
  <button v-on:click="groet">Groet</button>
</div>
<script>
var voorbeeld2 = new Vue({
  el: '#voorbeeld-2',
  data: {
    naam: 'John Duck'
  },
  methods: {
    groet: function (event) {
      alert('Hallo ' + this.naam + '!')
      if (event) {
        alert(event.target.tagName)
      }
    }
  }
})
</script>
{% endraw %}

## Methodes direct afhandelen

In plaats van de methodenaam te koppelen aan het 'event', is het ook mogelijk om de methodes direct te koppelen:

``` html
<div id="voorbeeld-3">
  <button v-on:click="zeg('hallo')">Zeg hallo</button>
  <button v-on:click="zeg('wat')">Zeg wat</button>
</div>
```
``` js
new Vue({
  el: '#voorbeeld-3',
  methods: {
    zeg: function (bericht) {
      alert(bericht)
    }
  }
})
```

Resultaat:
{% raw %}
<div id="voorbeeld-3" class="demo">
  <button v-on:click="zeg('hallo')">Zeg hallo</button>
  <button v-on:click="zeg('wat')">Zeg wat</button>
</div>
<script>
new Vue({
  el: '#voorbeeld-3',
  methods: {
    zeg: function (bericht) {
      alert(bericht)
    }
  }
})
</script>
{% endraw %}

Wanneer het nodig is om toegang te hebben tot het originele DOM 'event' in een directe koppeling, is het mogelijk om de speciale `$event`-variabele door te geven aan een methode:

``` html
<button v-on:click="waarschuwing('Formulier kan nog niet worden ingediend.', $event)">
  Indienen
</button>
```

``` js
// ...
methods: {
  waarschuwing: function (bericht, event) {
    // er is toegang tot het oorspronkelijke 'event'
    if (event) event.preventDefault()
    alert(bericht)
  }
}
```

## 'Event'-modificaties

Het komt vaak voor dat `event.preventDefault()` of `event.stopPropagation()` aangeroepen worden bij het afhandelen van een 'event'. Ook al is het mogelijk om dit makkelijk te doen in methodes, is het beter dat methodes zich alleen bezig houden met de datalogica in plaats van het afhandelen van een DOM 'event'.

Om dit probleem op te lossen voorziet Vue **'event'-modificaties** voor `v-on`. Modificaties zijn achtervoegsels op een 'directive' aangeduid met een punt.

- `.stop`
- `.prevent`
- `.capture`
- `.self`
- `.once`
- `.passive`

``` html
<!-- de propagatie van een klik 'event' zal gestopt worden -->
<a v-on:click.stop="doeDit"></a>

<!-- het 'submit event' zal de pagina niet herladen -->
<form v-on:submit.prevent="bijIndienen"></form>

<!-- modificaties kunnen achter elkaar gebruikt worden -->
<a v-on:click.stop.prevent="doeDat"></a>

<!-- enkel de modificatie -->
<form v-on:submit.prevent></form>

<!-- 'capture'-modus gebruiken bij het toevoegen van het luisteren naar een 'event' -->
<!-- dit betekent dat wanneer een 'event' van een element in de 'div' geactiveerd wordt, 
     het eerst hier zal afgehandeld worden voordat het element zelf het 'event' zal afhandelen -->
<div v-on:click.capture="doeDit">...</div>

<!-- handel het 'event' enkel af wanneer 'event.target' het element zelf is -->
<!-- dit betekent dat 'events' van onderliggende elementen niet zorgen voor het uitvoeren van 'doeDat' -->
<div v-on:click.self="doeDat">...</div>
```

<p class="tip">De volgorde van het gebruik van modificaties maakt uit, aangezien de relevante code gegenereerd is in dezelfde volgorde. `v-on:click.prevent.self` zal **alle klikken** voorkomen, `v-on:click.self.prevent` zal alleen de klikken op dat element voorkomen.</p>

> Nieuw in 2.1.4+

``` html
<!-- het klik 'event' zal maximum één keer activeren -->
<a v-on:click.once="doeDit"></a>
```

In tegenstelling tot andere modificaties die exclusief zijn voor oorspronkelijke DOM 'events', kan de `.once`-modificatie ook gebruikt worden op ['events' van een component](components-custom-events.html). Indien er nog geen kennis is over componenten, kan dit genegeerd worden voor nu.

> Nieuw in 2.3.0+

Voe voorziet ook de `.passive`-modificatie, overeenkomend met de [`addEventListener``passive` optie](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters).

``` html
<!-- het standaard gedrag van het 'scroll event' (scrollen) zal   -->
<!-- meteen plaatsvinden, in plaats van te wachten tot `onScroll` -->
<!-- klaar is, in het geval dat het `event.preventDefault() bevat -->
<div v-on:scroll.passive="onScroll">...</div>
```

De `.passive`-modificatie is vooral nuttig om de prestaties op mobiele toestellen te verbeteren.

<p class="tip"> Gebruik `.passive` en `.prevent` niet tegelijkertijd. `.prevent` zal genegeerd worden en de browser zal waarschijnlijk een waarschuwing tonen. Herinner, `.passive` communiceert naar de browser dat het standaard gedrag _niet_ voorkomen moet worden.</p>

## Toetsmodificaties

Wanneer er naar 'events' van het toetsenbord geluisterd wordt, komt het vaak voor dat er naar een specifieke toets geluisterd moet worden. Vue laat toe om toetsmodificaties toe te voegen aan `v-on` wanneer er geluisterd wordt naar toestenbord 'events':

``` html
<!-- roep `vm.enterIsGeklikt()` alleen aan wanneer de `toets` (in het Engels `key`, vandaar `keyup.enter`) `enter` is -->
<input v-on:keyup.enter="enterIsGeklikt">
```

Het is mogelijk om alle geldige namen van toetsen die ter beschikking gesteld worden via [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) te gebruiken als modificaties door ze te converteren naar 'kebab-case'.

``` html
<input v-on:keyup.page-down="PageDownIsGeklikt">
```

In bovenstaand voorbeeld zal de methode 'PageDownIsGeklikt' alleen aangeroepen worden wanneer `$event.key` gelijk is aan `'PageDown'`.

### Toetscodes

<p class="tip">Het gebruik van toetscode (`keyCode`) 'events' [is verouderd](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/keyCode) en wordt mogelijk niet ondersteund in nieuwe browsers.</p>

Het gebruik van een `keyCode`-attribuut is toegestaan (maar verouderd):

``` html
<input v-on:keyup.13="indienen">
```

Vue voorziet aliassen voor de meest voorkomende toetscodes indien nodig voor het ondersteunen van oudere browsers:

- `.enter`
- `.tab`
- `.delete` (vangt zowel "Delete"- als "Backspace"-toetsen op)
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

<p class="tip">Een paar toetsen (`.esc` en alle pijltoetsen) hebben inconsistente `'key'`-waarden in IE9, het is aangeraden de aangeboden aliassen te gebruiken indien IE9 ondersteunt moet worden.</p>

Het is mogelijk om [aliassen te definiëren](../api/#keyCodes) via het globale `config.keyCodes`-object:

``` js
// `v-on:keyup.f1` kan gebruikt worden nadat onderstaande regel is uitgevoerd
Vue.config.keyCodes.f1 = 112
```

## Systeemtoetsmodificaties

> Nieuw in 2.1.0+

Dit zijn de modificaties die gebruikt kunnen worden om te luisteren naar een 'event' van een muisknop of toetsenbordtoets te activeren wanneer de bijhorende systeemtoets ingedrukt is:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

> Merk op: Op Macintosh-toetsenborden is de 'meta'-toets, de 'command'-toets (⌘). Op Windows-toetsenborden is de 'meta'-toets, de Windows-toets (⊞). Op Sun Microsystems-toetsenborden is de 'meta'-toets, de toets met een volle diamant (◆). Op sommige toetsenborden is de 'meta'-toets aangeduid met “META” of “Meta”.

Bijvoorbeeld:

```html
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doIets">Doe iets</div>
```

<p class="tip">Merk op dat systeemtoetsen verschillen van normale toetsen wanneer ze gebruikt worden met het `keyup` 'event', ze moeten ingedrukt zijn wanneer het 'event' uitgestuurd wordt. In andere woorden, `keyup.ctrl` zal alleen geactiveerd worden wanneer een toets losgelaten wordt terwijl `ctrl` ingedrukt blijft. Het zal niet geactiveerd worden wanneer alleen de `ctrl`-toets losgelaten wordt. Als zo een gedrag wel gewenst is, gebruik dan `keyCode` in plaats van `ctrl`: `keyup.17`.</p>

### `.exact`-modificatie

> Nieuw in 2.5.0+

De `.exact`-modificatie laat toe om de exacte combinatie van systeemtoetsen te bepalen die nodig zijn om een 'event' te activeren.

``` html
<!-- dit zal geactiveerd worden, ook als Alt of Shift ingedrukt is -->
<button @click.ctrl="bijKlik">A</button>

<!-- dit zal alleen geactiveerd worden wanneer Ctrl ingedrukt is, niet bij andere toetsen -->
<button @click.ctrl.exact="bijCtrlKlik">A</button>

<!-- dit zal alleen geactiveerd worden wanneer geen systeemtoets ingedrukt is -->
<button @click.exact="bijKlik">A</button>
```

### Muisknopmodificaties

> Nieuw in 2.2.0+

- `.left`
- `.right`
- `.middle`

Deze modificaties zorgen ervoor dat de gekoppelde functie alleen aangeroepen wordt wanneer een specifiek muisknop ingedrukt wordt.

## Waarom luisteren in HTML?

Worden de oude regels over 'scheiding van belangen' niet overtreden door te luisteren naar 'events'? Hierover hoeven er geen zorgen te zijn, de functies en expressies voor het afhandelen van een 'event' zijn strikt gekoppeld aan het ViewModel dat de huidige 'view' afhandelt. Het zal niet voor moeilijkheden zorgen tijdens het onderhouden van de code. Er zijn zelfs voordelen aan het gebruik van `v-on`:

1. Het is eenvoudiger om de implementaties van de afhandelfunctie te vinden door te kijken naar de HTML 'template'.

2. Omdat er niet manueel geluisterd moet worden naar een 'event' in JavaScript, kan de ViewModel-code puur logisch en DOM-vrij zijn. Dit zorgt voor makkelijker te testen code.

3. Wanneer een ViewModel vernietigd wordt, worden alle 'events' waarnaar geluisterd wordt automatisch verwijdert.
