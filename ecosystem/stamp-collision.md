# @stamp/collision

_Controls collision behavior: forbid or defer_

This stamp \(aka behavior\) will check if there are any conflicts on every `compose` call. Throws an `Error` in case of a forbidden collision or ambiguous setup.

## Usage

```javascript
import Collision from '@stamp/collision';

const ForbidRedrawCollision = Collision.collisionSetup({forbid: ['redraw']});
```

Or if you don't want to import the stamp you can import only the method:

```javascript
import {collisionSetup} from '@stamp/collision';
const ForbidRedrawCollision = collisionSetup({forbid: ['redraw']});
```

The `defer` collects same named methods and wraps them into a single method.

```javascript
import Collision from '@stamp/collision';

import {Border, Button, Graph} from './drawable/primitives';

const UiComponent = Collision.collisionSetup({defer: ['draw']})
.compose(Border, Button, Graph);

const component = UiComponent();
const results = component.draw(); // will draw() all three primitives
console.log(results.length); // prints "3" to the console.
```

The deferred method returns the array of the values your methods returned. In the example above the `results` array will have the values the `.draw()` methods returned.

## API

### Static methods

#### collisionSetup

Forbid or Defer an exclusive method `stamp.collisionSetup({forbid: ['methodName1'], defer: ['methodName2']}) -> Stamp`

#### collisionProtectAnyMethod

Forbid any collisions, excluding those allowed `stamp.collisionProtectAnyMethod({allow: ['methoName']}) -> Stamp`

#### collisionSettingsReset

Remove any Collision settings from the stamp `stamp.collisionSettingsReset() -> Stamp`

## Example

See the comments in the code:

```javascript
import compose from '@stamp/compose';
import Collision from '@stamp/collision';
import Privatize from '@stamp/privatize';

// General purpose behavior to defer "draw()" method collisions
const DeferDraw = Collision.collisionSetup({defer: ['draw']});

const Border = compose(DeferDraw, {
  methods: {
    draw: jest.fn() // Spy function
  }
});
const Button = compose(DeferDraw, {
  methods: {
    draw: jest.fn() // Spy function
  }
});

// General purpose behavior to privatize the "draw()" method
const PrivateDraw = Privatize.privatizeMethods('draw');

// General purpose behavior to forbid the "redraw()" method collision
const ForbidRedrawCollision = Collision.collisionSetup({forbid: ['redraw']});

// The aggregating component
const ModalDialog = compose(PrivateDraw) // privatize the "draw()" method
  .compose(Border, Button) // modal will consist of Border and Button
  .compose(ForbidRedrawCollision) // there can be only one "redraw()" method
  .compose({
    methods: {
      // the public method which calls the deferred private methods
      redraw(int) {
        this.draw(int);
      }
    }
  });

// Creating an instance of the stamp
const dialog = ModalDialog();

// Check if the "draw()" method is actually privatized
expect(dialog.draw).toBeUndefined();

// Calling the public method, which calls the deferred private methods
dialog.redraw(42);

// Check if the spy "draw()" deferred functions were actually invoked
expect(Border.compose.methods.draw).toBeCalledWith(42);
expect(Button.compose.methods.draw).toBeCalledWith(42);

// Make sure the ModalDialog throws every time on the "redraw()" collisions
const HaveRedraw = compose({methods: {redraw() {}}})
expect(() => compose(ModalDialog, HaveRedraw)).toThrow();
```

