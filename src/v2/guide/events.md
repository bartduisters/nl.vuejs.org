---
title: Afhandelen van 'events'
type: guide
order: 9
---

## Luisteren naar 'events'

Het is mogelijk om met de `v-on` 'directive' naar DOM-'events' te luisteren en JavaScript uit te voeren wanneer ze geactiveerd worden.

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
      alert('Hello ' + this.naam + '!')
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

Wanneer het het nodig is om toegang te hebben tot het originele DOM 'event' in een directe koppeling, is het mogelijk om de speciale `$event`-variabele door te geven aan een methode:

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

Het komt vaak voor dat `event.preventDefault()` of `event.stopPropagation()` aangeroepen worden bij het afhandelen van een 'event'. Ook al is het mogelijk om dit makkelijk te doen in methodes, is het beter dat methodes zich enkel bezig houden met de datalogica in plaats van het afhandelen van een DOM 'event'.

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

<p class="tip">De volgorde van het gebruik van modificaties maakt uit, aangezien de relevante code gegenereerd is in dezelfde volgorde. `v-on:click.prevent.self` zal **alle klikken** voorkomen, `v-on:click.self.prevent` zal enkel de klikken op dat element voorkomen.</p>

> Nieuw in 2.1.4+

``` html
<!-- het klik 'event' zal maximum één keer activeren -->
<a v-on:click.once="doeDit"></a>
```

Unlike the other modifiers, which are exclusive to native DOM events, the `.once` modifier can also be used on [component events](components-custom-events.html). If you haven't read about components yet, don't worry about this for now.

> New in 2.3.0+

Vue also offers the `.passive` modifier, corresponding to [`addEventListener`'s `passive` option](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/addEventListener#Parameters).

``` html
<!-- the scroll event's default behavior (scrolling) will happen -->
<!-- immediately, instead of waiting for `onScroll` to complete  -->
<!-- in case it contains `event.preventDefault()`                -->
<div v-on:scroll.passive="onScroll">...</div>
```

The `.passive` modifier is especially useful for improving performance on mobile devices.

<p class="tip">Don't use `.passive` and `.prevent` together, because `.prevent` will be ignored and your browser will probably show you a warning. Remember, `.passive` communicates to the browser that you _don't_ want to prevent the event's default behavior.</p>

## Key Modifiers

When listening for keyboard events, we often need to check for specific keys. Vue allows adding key modifiers for `v-on` when listening for key events:

``` html
<!-- only call `vm.submit()` when the `key` is `Enter` -->
<input v-on:keyup.enter="submit">
```

You can directly use any valid key names exposed via [`KeyboardEvent.key`](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) as modifiers by converting them to kebab-case.

``` html
<input v-on:keyup.page-down="onPageDown">
```

In the above example, the handler will only be called if `$event.key` is equal to `'PageDown'`.

### Key Codes

<p class="tip">The use of `keyCode` events [is deprecated](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/keyCode) and may not be supported in new browsers.</p>

Using `keyCode` attributes is also permitted:

``` html
<input v-on:keyup.13="submit">
```

Vue provides aliases for the most commonly used key codes when necessary for legacy browser support:

- `.enter`
- `.tab`
- `.delete` (captures both "Delete" and "Backspace" keys)
- `.esc`
- `.space`
- `.up`
- `.down`
- `.left`
- `.right`

<p class="tip">A few keys (`.esc` and all arrow keys) have inconsistent `key` values in IE9, so these built-in aliases should be preferred if you need to support IE9.</p>

You can also [define custom key modifier aliases](../api/#keyCodes) via the global `config.keyCodes` object:

``` js
// enable `v-on:keyup.f1`
Vue.config.keyCodes.f1 = 112
```

## System Modifier Keys

> New in 2.1.0+

You can use the following modifiers to trigger mouse or keyboard event listeners only when the corresponding modifier key is pressed:

- `.ctrl`
- `.alt`
- `.shift`
- `.meta`

> Note: On Macintosh keyboards, meta is the command key (⌘). On Windows keyboards, meta is the windows key (⊞). On Sun Microsystems keyboards, meta is marked as a solid diamond (◆). On certain keyboards, specifically MIT and Lisp machine keyboards and successors, such as the Knight keyboard, space-cadet keyboard, meta is labeled “META”. On Symbolics keyboards, meta is labeled “META” or “Meta”.

For example:

```html
<!-- Alt + C -->
<input @keyup.alt.67="clear">

<!-- Ctrl + Click -->
<div @click.ctrl="doSomething">Do something</div>
```

<p class="tip">Note that modifier keys are different from regular keys and when used with `keyup` events, they have to be pressed when the event is emitted. In other words, `keyup.ctrl` will only trigger if you release a key while holding down `ctrl`. It won't trigger if you release the `ctrl` key alone. If you do want such behaviour, use the `keyCode` for `ctrl` instead: `keyup.17`.</p>

### `.exact` Modifier

> New in 2.5.0+

The `.exact` modifier allows control of the exact combination of system modifiers needed to trigger an event.

``` html
<!-- this will fire even if Alt or Shift is also pressed -->
<button @click.ctrl="onClick">A</button>

<!-- this will only fire when Ctrl and no other keys are pressed -->
<button @click.ctrl.exact="onCtrlClick">A</button>

<!-- this will only fire when no system modifiers are pressed -->
<button @click.exact="onClick">A</button>
```

### Mouse Button Modifiers

> New in 2.2.0+

- `.left`
- `.right`
- `.middle`

These modifiers restrict the handler to events triggered by a specific mouse button.

## Why Listeners in HTML?

You might be concerned that this whole event listening approach violates the good old rules about "separation of concerns". Rest assured - since all Vue handler functions and expressions are strictly bound to the ViewModel that's handling the current view, it won't cause any maintenance difficulty. In fact, there are several benefits in using `v-on`:

1. It's easier to locate the handler function implementations within your JS code by skimming the HTML template.

2. Since you don't have to manually attach event listeners in JS, your ViewModel code can be pure logic and DOM-free. This makes it easier to test.

3. When a ViewModel is destroyed, all event listeners are automatically removed. You don't need to worry about cleaning it up yourself.
