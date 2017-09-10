Contributing to Image Sequencer
===

Happily accepting pull requests; to edit the core library, modify files in `./src/`. To build, run `npm install` followed by `grunt build`.

Most contribution (we imagine) would be in the form of API-compatible modules, which need not be directly included.

## Jump To

* [README.md](https://github.com/publiclab/image-sequencer)
* [Contributing Modules](#contributing-modules)
* [Info File](#info-file)
* [Ideas](#ideas)

## Contributing modules

Any module must look like this :

```js
module.exports = function ModuleName(options,UI) {
  options = options || {};
  options.title = "Title of the Module";
  UI.onSetup(options.step);
  var output;

  function draw(input,callback) {
    UI.onDraw(options.step);

    var output = /*do something with the input*/ ;

    this.output = output;
    options.step.output = output.src;
    callback();
    UI.onComplete(options.step);
  }

  return {
    options: options,
    draw: draw,
    output: output,
    UI: UI
  }
}
```

### options

The object `options` stores some important information. This is how you can accept
input from users. If you require a variable "x" from the user and the user passes
it in, you will be able to access it as `options.x`.

Options also has some in-built properties. The `options.inBrowser` boolean denotes
whether your module is being run on a browser.

### draw()

The draw method is run every time the step is `run` using `sequencer.run()`.
So any calculations must go **into** the `draw()` method's definition.

What is not in the draw method, but is in the `module.exports` is executed only
when the step is added. So whatever external npm modules are to be loaded, or
constant definitions must be done **outside** the `draw()` method's definition.

`draw()` receives two arguments - `input` and  `callback` :
* `input` is an object which is essentially the output of the previous step.
    ```js
    input = {
      src: "<$DataURL>",
      format: "<png|jpeg|gif>"
    }
    ```
* `callback` is a function which is responsible to tell the sequencer that the
step has been "drawn".

When you have done your calculations and produced an image output, you are required
to set `this.output` to an object similar to what the input object was, call
`callback()`, and set `options.step.output` equal to the output DataURL

### UI Methods

The module is responsible to emit various events for the UI to capture. There are
four events in all:

* `UI.onSetup(options.step)` must be emitted when the module is added. So it must be emitted outside the draw method's definition as shown above.
* `UI.onDraw(options.step)` must be emitted whenever the `draw()` method is called. So it should ideally be the first line of the definition of the `draw` method.
* `UI.onComplete(options.step)` must be emitted whenever the output of a draw call
is ready. An argument, that is the DataURL of the output image must be passed in.
* `UI.onRemove(options.step)` is emitted automatically and the module should not emit it.

To add a module to Image Sequencer, it must have the following method; you can wrap an existing module to add them:

* `module.draw()`

The `draw(input,callback)` method should accept an `input` parameter, which will be an object of the form:

```js
input = {
  src: "datauri here",
  format: "jpeg/png/etc"
}
```

### options.title

For display in the web-based UI, each module may also have a title `options.title`.

## Info file

All module folders must have an `info.json` file which looks like the following:
```json
{
  "name": "Name of Module to be displayed",
  "inputs": {
    "var1": {
      "type": "text",
      "default": "default value"
    }
  },
  "outputs": {
    "out1": {
      "type": "text"
    }
  }
}
```

Types may be one of "text", "integer", "float", "select".
Integer and Float types should also specify minimum and maximum values like this:

```json
"var1": {
  "type": "integer",
  "min": 0,
  "max": 4,
  "default": 1
}
```

Similarly, "Select" type inputs should have a `values` array.

Also, A module may have output values. These must be defined as shown above.

### Module example

See existing module `green-channel` for an example: https://github.com/publiclab/image-sequencer/tree/master/src/modules/GreenChannel/Module.js

For help integrating, please open an issue.

## Ideas

Notes on development next steps:

### UI

* [ ] spinners before panels are complete
* [ ] click to expand for all images

### Modules

* [ ] remotely includable modules, not compiled in -- see plugin structures in other libs
* [ ] `sequencer.addModule('path/to/module.js')` style module addition -- also to avoid browserifying all of Plotly :-P

### Core Functionality

* [ ] allow passing data as data-uri or Image object, or stream, or ndarray or ImageData array, if both of neighboring pair has ability?
* [ ] (for node-and-line style UIs) non-linear sequences with Y-splitters

### Testing

* [ ] comparisons with diff
* [ ] testing a module's promised functionality: each module could offer before/after images as part of their API; by running the module on the before image, you should get exactly the after image, comparing with an image diff

### Use cases

* [ ] make an Infragram module that accepts a math expression

### Bugs

If you find a bug please list it here, and help us develop Image Sequencer by opening an issue!

## Module Candidates, related Ideas

* https://github.com/linuxenko/rextract.js
* https://www.npmjs.com/package/histogram
* https://github.com/hughsk/flood-fill
* https://github.com/yahoo/pngjs-image has lots of useful general-purpose image getters like `image.getLuminosityAtIndex(idx)`
* some way to add in a new image (respecting alpha) -- `add-image` (with blend mode, default `normal`?)
* https://github.com/yuta1984/CannyJS - edge detection
* http://codepen.io/taylorcoffelt/pen/EsCcr - more edge detection
* https://github.com/jywarren/webgl-distort/ - an image distortion module
* https://github.com/jadnco/whirl - scrubbable image sequence player
* non graphics card GL functions could be shimmed with https://github.com/Overv/JSGL
* or this: https://github.com/stackgl/headless-gl
* https://github.com/mattdesl/fontpath-simple-renderer

### Referencing earlier states

Complex sequences with masking could require accessing previous states (or nonlinearity):

* flood-fill an area
* select only the flooded area
  * roundabout: lighten everything to <50%, then flood-fill with black? Not 100% reliable.
  * roundabout 2: `flood fill`, then `blink-diff` with original
* then add step which recovers original image, repeat `flood-fill`/`blink-diff` for second region
* reference above masked states in a `mask` module, with `maskModule.draw(image, { getMask: function() { return maskImg } })`

**Notes:**

`pattern-fill` module to use patterns in JS canvas:

```js
var c=document.getElementById("myCanvas");
var ctx=c.getContext("2d");
var img=document.getElementById("lamp");
var pat=ctx.createPattern(img,"repeat");
ctx.rect(0,0,150,100);
ctx.fillStyle=pat;
ctx.fill();
```

Masking:

```js
ctx.save();
ctx.beginPath();
ctx.moveTo(0, 0);
ctx.lineTo(160, 600);
ctx.rect(0, 0, 160, 600);
ctx.closePath();
ctx.clip();
ctx.drawImage(img, 0, 0);
ctx.restore();
```

****

## UI notes:

* visual nodes-and-lines UI: https://github.com/flowhub/the-graph
  * https://flowhub.github.io/the-graph/examples/demo-simple.html

```js

settings: {
  'threshold': {
    type: 'slider',
    label: 'Threshold',
    default: 50,
    min: 0,
    max: 100
  },
  'colors': {
    type: 'select',
    label: 'Colors',
    options: [
      { name: '0', value: '0', default: true },
      { name: '1', value: '1' },
      { name: '2', value: '2' }
    ]
  }
}

```

Possible web-based commandline interface: https://hyper.is/?