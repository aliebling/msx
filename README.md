# MSX [![Build Status](https://secure.travis-ci.org/insin/msx.png?branch=master)](http://travis-ci.org/insin/msx)

*MSX is based on version 0.13.2 of React's JSX Transformer*

MSX tweaks [React](http://facebook.github.io/react/)'s JSX Transformer to output
contents compatible with [Mithril](http://lhorie.github.io/mithril/)'s
`m.render()` function, allowing you to use HTML-like syntax in your Mithril
view code, like this:

```html
var todos = ctrl.list.map(function(task, index) {
  return <li className={task.completed() && 'completed'}>
    <div className="view">
      <input
        className="toggle"
        type="checkbox"
        onclick={m.withAttr('checked', task.completed)}
        checked={task.completed()}
      />
      <label>{task.title()}</label>
      <button className="destroy" onclick={ctrl.remove.bind(ctrl, index)}/>
    </div>
    <input className="edit"/>
  </li>
})
```

## HTML tags and custom elements

For tag names which look like HTML elements or custom elements (lowercase,
optionally containing hyphens), raw virtual DOM objects - matching the
[`VirtualElement` signature](http://lhorie.github.io/mithril/mithril.render.html#signature)
accepted by `m.render()` - will be generated by default.

_Input:_

```html
<div id="example">
  <h1>Test</h1>
  <my-element name="test"/>
</div>
```

_Output:_

```javascript
{tag: "div", attrs: {id:"example"}, children: [
  {tag: "h1", attrs: {}, children: ["Test"]},
  {tag: "my-element", attrs: {name:"test"}}
]}
```

This effectively [precompiles](http://lhorie.github.io/mithril/optimizing-performance.html)
your view code for a slight performance tweak.

## Mithril components

Otherwise, it's assumed a tag name is a reference to an in-scope variable which
is a [Mithril component](http://lhorie.github.io/mithril/components.html).

Passing attributes or children to a component will generate a call to Mithril's
[`m.component()`](http://lhorie.github.io/mithril/mithril.component.html)
function, with children always being passed as an Array:

_Input:_

```html
<form>
  {/* Bare component */}
  <Uploader/>
  {/* Component with attributes */}
  <Uploader onchange={ctrl.files}/>
  {/* Component with attributes and children */}
  <Uploader onchange={ctrl.files}>
    {ctrl.files().map(file => <File {...file}/>)}
  </Uploader>
  <button type="button" onclick={ctrl.save}>Upload</button>
</form>
```

_Output:_

```javascript
{tag: "form", attrs: {}, children: [
  /* Bare component */
  Uploader,
  /* Component with attributes */
  m.component(Uploader, {onchange:ctrl.files}),
  /* Component with attributes and children */
  m.component(Uploader, {onchange:ctrl.files}, [
    ctrl.files().map(function(file)  {return m.component(File, Object.assign({},  file));})
  ]),
  {tag: "button", attrs: {type:"button", onclick:ctrl.save}, children: ["Upload"]}
]}
```

MSX assumes your component's (optional) `controller()` and (required) `view()`
functions have the following signatures, where `attributes` is an `Object` and
`children` is an `Array`:

```javascript
controller([attributes[, children]])
view(ctrl[, attributes[, children]])
```

As such, if a component has children but no attributes, an empty attributes
object will still be passed:

_Input:_

```html
<Field>
  <input onchange={m.withAttr('value', ctrl.description)} value={ctrl.description()}/>
</Field>
```

_Output:_

```javascript
m.component(Field, {}, [
  {tag: "input", attrs: {onchange:m.withAttr('value', ctrl.description), value:ctrl.description()}}
])
```

## JSX spread attributes and `Object.assign()`

If you make use of [JSX Spread Attributes](http://facebook.github.io/react/docs/jsx-spread.html),
the resulting code will make use of `Object.assign()` to merge attributes - if
your code needs to run in environments which don't implement `Object.assign()`
natively, you're responsible for ensuring it's available via a
[shim](https://github.com/ljharb/object.assign), or otherwise.

Other than that, the rest of React's JSX documentation should still apply:

* [JSX in Depth](http://facebook.github.io/react/docs/jsx-in-depth.html)
* [JSX Spread Attributes](http://facebook.github.io/react/docs/jsx-spread.html)
* [JSX Gotchas](http://facebook.github.io/react/docs/jsx-gotchas.html) - with
  the exception of `dangerouslySetInnerHTML`: use
  [`m.trust()`](http://lhorie.github.io/mithril/mithril.trust.html) on contents
  instead.
* [If-Else in JSX](http://facebook.github.io/react/tips/if-else-in-JSX.html)

## In-browser JSX Transform

For development and quick prototyping, an in-browser JSX transform can be
downloaded from the [dist/](https://github.com/insin/msx/blob/master/dist)
directory. Simply include a `<script type="text/msx">` tag to engage the JSX
transformer.

To enable ES6 transforms, use `<script type="text/msx;harmony=true">`. Check out
the [source](https://github.com/insin/msx/blob/master/demo/index.html) of the
[example of using in-browser JSX + ES6 transforms](http://insin.github.io/msx/).

## Command Line Usage

```
npm install -g msx
```

```
msx --watch src/ build/
```

To disable precompilation from the command line, pass a `--no-precompile` flag.

Run `msx --help` for more information.

## Module Usage

```
npm install msx
```

```javascript
var msx = require('msx')
```

### Module API

#### `msx.transform(source: String[, options: Object])`

Transforms XML-like syntax in the given source into object literals compatible
with Mithril's `m.render()` function, or to function calls using Mithril's
`m()` function, returning the transformed source.

To enable [ES6 transforms supported by JSX Transformer](http://kangax.github.io/compat-table/es6/#jsx),
pass a `harmony` option:

```javascript
msx.transform(source, {harmony: true})
```

To disable default precompilation and always output `m()` calls, pass a
`precompile` option:

```javascript
msx.transform(source, {precompile: false})
```

## Examples

Example inputs (using some ES6 features) and outputs are in
[test/jsx](https://github.com/insin/msx/tree/master/test/jsx) and
[test/js](https://github.com/insin/msx/tree/master/test/js), respectively.

An example [gulpfile.js](https://github.com/insin/msx/blob/master/gulpfile.js)
is provided, which implements an `msxTransform()` step using `msx.transform()`.

## Related Modules

* [gulp-msx](https://github.com/insin/gulp-msx) - gulp plugin.
* [grunt-msx](https://github.com/hung-phan/grunt-msx) - grunt plugin.
* [mithrilify](https://github.com/sectore/mithrilify) - browserify transform.
* [msx-loader](https://github.com/sdemjanenko/msx-loader) - webpack loader.

## MIT Licensed