# Canvas Interceptor
`canvas-interceptor.js` is a snippet to aid with debugging canvas issues, and
designed to create reduced test cases.

Load `canvas-interceptor.js` before the web page uses the HTML5 canvas API.
Then the all method calls and property invocations on canvas objects will be
logged and stored in the `__proxyLogs` property on the canvas context. The final
code to recreate the canvas can be generated by calling `getCanvasReplay(canvas)`.

Example:

```html
<script src="canvas-interceptor.js"></script>

<canvas id="mycanvas"></canvas>
<!-- ... a lot of canvas magic on #mycanvas... -->

<script>
var canvas = document.getElementById('mycanvas');
var code = getCanvasReplay(canvas);
// code can be copied to a new file to recreate the canvas,
// and used to e.g. create reduced test cases for canvas bugs.
</script>
```

See test.html for more examples.

## Browser compatibility

Run test.html to see whether the tool works as expected.

- Chrome 43+ (not 42- because accessor descriptors were not prototypical, and
  data descriptors were not configurable)
- Opera 31+ (same reason as Chrome).
- Firefox 17+ (not 16- because accessor descriptors were inaccessible)
- IE 10+ and Edge 0.11+ (IE9 and earlier doesn't support 
- Safari (not supported because accessor descriptors are not prototypical)

## API documentation

When `canvas-interceptor.js` is loaded, it modifies the prototype of
`CanvasRenderingContext2D` to intercept calls. The following APIs are exposed:

* `getCanvasReplay(HTMLCanvasElement|CanvasRenderingContext2D)` - Get the code
  to recreate the state of the canvas from the logs.

* `getCanvasName(CanvasRenderingContext2D)` - Get the identifier of the context.
  If the context was not known yet, a new name will be generated and stored in
  the `__proxyName` property of the given parameter. To get the name for a given
  canvas, use `getCanvasName(canvas.getContext('2d'))`.

* `CanvasRenderingContext2D.prototype.__proxyUnwrap` - Disable canvas logging.
  To re-enable canvas logging, the page has to be refreshed.
  Existing logs are still preserved and `getCanvasReplay` will continue to work.

* `serializeArg(obj)` - Get the string representation for the given object.
  Calling `eval` on the return value ought to reconstruct the original object.
  Not all objects can be serialized; if serialization is not supported, object
  literals with an inline comment is returned (`{ /* ... */ }`).

* `constructImageData(width, height, data)` - Create a new `ImageData` object
  with a given width and height, initialized with `data`.

* `wrapObject(...)` implements the logic of intercepting APIs. Do not use this
  method unless you know what you're doing (see the code for documentation).

If you need to use a canvas method without logging, retrieve the original
property from `.__proxyOriginal`, and call the method (value) or setter/getter.
Example: `ctx.__proxyOriginal.scale.value.call(ctx, 1, 1)` or
`ctx.__proxyOriginal.lineWidth.set.call(ctx, 1)`.
