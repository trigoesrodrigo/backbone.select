# Backbone.Select

Selecting Backbone models; handling selections in Backbone collections.

Backbone.Select is [compatible with Backbone.Picky][picky-compatibility]. For a superset of Backbone.Select with even more features, have a look at [Backbone.Cycle][].

## Source Code And Downloads

The component is available with Bower: `bower install backbone.select`.

Alternatively, you can download the raw source code from the "src" folder above, or grab one of the builds from the "dist" folder.

To get the latest stable release, use these links which point to the 'master' branch's builds:

### Standard Builds

Development: [backbone.select.js](https://raw.github.com/hashchange/backbone.select/master/dist/backbone.select.js)

Production: [backbone.select.min.js](https://raw.github.com/hashchange/backbone.select/master/dist/backbone.select.min.js)

### AMD/RequireJS Builds

Development: [backbone.select.js](https://raw.github.com/hashchange/backbone.select/master/dist/amd/backbone.select.js)

Production: [backbone.select.min.js](https://raw.github.com/hashchange/backbone.select/master/dist/amd/backbone.select.min.js)

## Requirements

Backbone.Backbone.Select requires Backbone 0.9.9 or later.

## Model and Collection Interactions

If you implement a `Backbone.Select.Me` model, the methods on the models and the `Backbone.Select.Many` collection will keep each other in sync. That is, if you call `model.select()` on a model, the collection will be notified of the model being selected and it will correctly update the `selectedLength` and fire the correct events.

Therefore, the following are functionally the same:

```js
model = new MyModel();
col = new MyCollection([model]);

model.select();
```

```js
model = new MyModel();
col = new MyCollection([model]);

col.select(model);
```

### Model Requirements For Backbone.Select Collections

Your model for a Backbone.Select collection must extend `Backbone.Select.Me`.

## Components of Backbone.Select:

* **Backbone.Select.Me:** Creates select / deselect capabilities for a model
* **Backbone.Select.Many:** Allows a collection to know about the selection of multiple models, including select all / deselect all
* **Backbone.Select.One:** Allow a collection to have an exclusively selected model

## Backbone.Select.Me: Making models selectable

Creates selectable capabilities for a model, including tracking whether or not the model is selected, and raising events when selection changes.

### Basic Usage

To create a selectable model type, apply the mixin in its `initialize` method. Assuming your base type is `Backbone.Model`, augment it with

```js
SelectableModel = Backbone.Model.extend({
  initialize: function(){
    Backbone.Select.Me.applyTo(this);
  }
});
```

Replace `Backbone.Model` in the example above with whatever base type you work with.

### Backbone.Select.Me Methods

The following methods are included in the `Select.Me` object.

#### Select.Me#select([options])

Select a model, setting the model's `selected` property to true and triggering a "select" event.

```js
var myModel = new SelectableModel();

myModel.on("select", function(){
  console.log("I'm selected!");
});

myModel.select(); //=> logs "I'm selected!"
myModel.selected; //=> true
```

The `select` method can be called with the `{silent: true}` option to prevent selection-related events from firing. See the [events section](#selectable-events) below.

#### Select.Me#deselect([options])

Deselect a model, setting the model's `selected` property to false and triggering a "deselected" event.

```js
var myModel = new SelectableModel();

myModel.on("deselected", function(){
  console.log("I'm no longer selected!");
});

// must select it before it can be deselected
myModel.select();

myModel.deselect(); //=> logs "I'm no longer selected!";
myModel.selected; //=> false
```

The `deselect` method supports the `silent` option.

#### Select.Me#toggleSelected([options])

Toggles the selection state between selected and deselected by calling the `select` or `deselect` method appropriately.

```js
var myModel = new SelectableModel();

myModel.on("select", function(){
  console.log("I'm selected!");
});

myModel.on("deselected", function(){
  console.log("I'm no longer selected!");
});

// toggle selection
myModel.toggleSelected(); //=> "I'm selected!"
myModel.toggleSelected(); //=> "I'm no longer selected!"
```

The `toggleSelected` method supports the `silent` option.

### Backbone.Select.Me Properties

The following properties are manipulated by the Select.Me object.

#### Select.Me#selected

Returns a boolean value indicating whether or not the model is currently selected.

### Backbone.Select.Me Events

The events listed below are are triggered from Select.Me models. Events can be prevented from firing when Backbone.Select methods are called with the `silent` option, as in `myModel.select({silent: true})`.

Event handlers with standard names are invoked automatically. Standard names are `onSelect`, `onDeselect`, and `onReselect`. If these methods exist on the model, they are run without having to be wired up with the event manually.

Custom options can be used when invoking any method. See the [section on custom options](#custom-options), below.

#### "selected"

Triggers when a model has been selected. Provides the selected model as the first parameter. Runs the `onSelect` event handler if the method exists on the model.

#### "deselected"

Triggers when a model has been deselected. Provides the selected model as the first parameter. Runs the `onDeselect` event handler if the method exists on the model.

#### "reselected"

Triggers when a model, which is already selected, is selected again. Provides the re-selected model as the first parameter. Runs the `onReselect` event handler if the method exists on the model.

## Backbone.Select.One: a single-select collection

Creates single-select capabilities for a `Backbone.Collection`, allowing a single model to be exclusively selected within the collection. Selecting another model will cause the first one to be deselected.

### Basic Usage

To create a single-select collection type, apply the mixin in the `initialize` method. Assuming your base type is `Backbone.Collection`, augment it with

```js
SelectableModel = Backbone.Model.extend({
  initialize: function(){
    Backbone.Select.Me.applyTo(this);
  }
});

SingleCollection = Backbone.Collection.extend({
  model: SelectableModel,

  initialize: function(models){
    Backbone.Select.One.applyTo(this, models);
  }
});
```

Replace `Backbone.Collection` in the example above with whatever base type you work with.

If you share models among multiple collections, Backbone.Select will handle the interaction for you. You must turn on model-sharing mode explicitly, with the `enableModelSharing` option:

```js
SingleCollection = Backbone.Collection.extend({
  model: SelectableModel,

  initialize: function(models){
    Backbone.Select.One.applyTo(this, models, { enableModelSharing: true });
  }
});
```

See the [section on model sharing][sharing], below, for more.

### Backbone.Select.One Methods

The following methods are provided by the `Select.One` object.

#### Select.One#select(model, [options])

Select a model. This method will store the selected model in the collection's `selected` property, and call the model's `select` method to ensure the model knows it has been selected.

```js
myModel = new SelectableModel();
myCol = new SingleCollection([myModel]);
myCol.select(myModel);
```

Or

```js
myModel = new SelectableModel();
myCol = new SingleCollection([myModel]);
myModel.select();
```

If the model is already selected, this is a no-op. If a previous model is already selected, the previous model will be deselected.

The `select` method supports the `silent` option.

_Compatibility:_ Backbone collections have a `select` method out of the box, an alias of `filter`. It continues to be accessible, based on its different signature. See [below][select-compatibility].

#### Select.One#deselect([model], [options])

Deselect the currently selected model. This method will remove the  model from the collection's `selected` property, and call the model's `deselect` method to ensure the model knows it has been deselected.

```js
myModel = new SelectableModel();
myCol = new SingleCollection([myModel]);
myCol.deselect(myModel);
```

Or

```js
myModel = new SelectableModel();
myCol = new SingleCollection([myModel]);
myModel.deselect();
```

If the model is not currently selected, this is a no-op. If you try to deselect a model that is not the currently selected model, the actual selected model will not be deselected.

You can call `deselect` without a model argument. The currently selected model, if any, will be deselected in that case.

The `deselect` method supports the `silent` option.

### Backbone.Select.One Properties

The following property is set by the multi-select automatically.

### Select.One#selected

Returns the one selected model for this collection

```js
myCol = new SingleCollection();
myCol.select(model);

myCol.selected; //=> model
```

### Backbone.Select.One Events

The events listed below are triggered by Backbone.Select.One based on changes in selection. Events can be prevented from firing when Backbone.Select methods are called with the `silent` option, as in `myCol.select(myModel, {silent: true})`.

Event handlers with standard names are invoked automatically. Standard names are `onSelect`, `onDeselect`, and `onReselect`. If these methods exist on the collection, they are run without having to be wired up with the event manually.

Custom options can be used when invoking any method. See the [section on custom options](#custom-options), below.

#### "select:one"

Triggered when a model has been selected. Provides the selected model as the first parameter, and the collection as the second. Runs the `onSelect` event handler if the method exists on the collection.

#### "deselect:one"

Triggered when a model has been deselected. Provides the deselected model as the first parameter, and the collection as the second.. Runs the `onDeselect` event handler if the method exists on the collection.

The event fires when `deselect` has been called explicitly, and also when the selection is being replaced through another call to `select`.

Combined select-deselect actions are treated as atomic, so "deselect:one" fires after both the deselection and the selection have already taken place. That is also the case for any other events involved in the transaction, such as the  "deselected" and "selected" events on the models involved.

#### "reselect:one"

Triggered when a model, which is already selected, is selected again. Provides the selected model as the first parameter, and the collection as the second. Runs the `onReselect` event handler if the method exists on the collection.

## Backbone.Select.Many: a multi-select collection

Creates multi-select capabilities for a `Backbone.Collection`, allowing one or more models to be selected in a collection. Backbone.Select.Many also provides "select all", "select none" and "select some" features.

### Basic Usage

To create a multi-select collection type, apply the mixin in the `initialize` method. Assuming your base type is `Backbone.Collection`, augment it with

```js
SelectableModel = Backbone.Model.extend({
  initialize: function(){
    Backbone.Select.Me.applyTo(this);
  }
});

MultiCollection = Backbone.Collection.extend({
  model: SelectableModel,

  initialize: function(models){
    Backbone.Select.Many.applyTo(this, models);
  }
});
```

Replace `Backbone.Collection` in the example above with whatever base type you work with.

If you share models among different collections, Backbone.Select will handle the interaction for you. You must turn on model-sharing mode explicitly, with the `enableModelSharing` option:

```js
MultiCollection = Backbone.Collection.extend({
  model: SelectableModel,

  initialize: function(models){
    Backbone.Select.Many.applyTo(this, models, { enableModelSharing: true });
  }
});
```

See the [section on model sharing][sharing], below, for more.

### Backbone.Select.Many Methods

The following methods are provided by the `Select.Many` object.

#### Select.Many#select(model, [options])

Select a model. This method will store the selected model in the collection's `selected` list, and call the model's `select` method to ensure the model knows it has been selected.

```js
myCol = new MultiCollection([myModel]);

myCol.select(myModel);
```

If the model is already selected, this is a no-op.

The `select` method supports the `silent` option.

_Compatibility:_ Backbone collections have a `select` method out of the box, an alias of `filter`. It continues to be accessible, based on its different signature. See [below][select-compatibility].

#### Select.Many#deselect(model, [options])

Deselect a model. This method will remove the  model from the collection's `selected` list, and call the model's `deselect` method to ensure the model knows it has been deselected.

```js
myCol = new MultiCollection([myModel]);

myCol.deselect(myModel);
```

If the model is not currently selected, this is a no-op.

The `deselect` method supports the `silent` option.

#### Select.Many#selectAll([options])

Select all models in the collection.

```js
myCol = new MultiCollection();

myCol.selectAll();
```

Models that are already selected will not be re-selected. Models that are not currently selected will be selected. The end result will be all models in the collection are selected.

The `selectAll` method supports the `silent` option.

#### Select.Many#deselectAll([options])

Deselect all models in the collection.

```js
myCol = new MultiCollection();

myCol.deselectAll();
```

Models that are selected will be deselected. Models that are not selected will not be deselected again. The end result will be no models in the collection are selected.

The `deselectAll` method supports the `silent` option.

#### Select.Many#toggleSelectAll([options])

Toggle selection of all models in the collection:

```js
myCol = new MultiCollection();

myCol.toggleSelectAll(); // select all models in the collection

myCol.toggleSelectAll(); // de-select all models in the collection
```

The following rules are used when toggling:

* If no models are selected, select them all
* If 1 or more models, but less than all models are selected, select them all
* If all models are selected, deselect them all

The `toggleSelectAll` method supports the `silent` option.

### Backbone.Select.Many Properties

The following property is set by the multi-select automatically.

### Select.Many#selected

Returns a hash of selected models, keyed from the model `cid`.

```js
myCol = new MultiCollection();
myCol.select(model);

myCol.selected;

//=> produces
// {
//   "c1": (model object here)
// }
```

#### Select.Many#selectedLength

Returns the number of items in the collection that are selected.

```js
myCol = new MultiCollection();
myCol.select(model);

myCol.selectedLength; //=> 1
```

### Backbone.Select.Many Events

The events below are triggered by Backbone.Select.Many based on changes in selection. Events can be prevented from firing when Backbone.Select methods are called with the `silent` option, as in `myCol.select(myModel, {silent: true})`.

Select.Many events, with the exception of `reselect:any`, pass a "diff" hash to event handlers as the first parameter: `{ selected: [...], deselected: [...] }`. The `selected` array holds models which have been newly selected by the action triggering the event. Likewise, models in the `deselected` array have changed their status from selected to deselected.

_(Note that up to version 0.2, the first parameter passed to event handlers had been the collection.)_

Event handlers with standard names are invoked automatically. Standard names are `onSelectNone`, `onSelectSome`, `onSelectAll` and `onReselect`. If these methods exist on the collection, they are run without having to be wired up with the event manually.

Custom options can be used when invoking any method. See the [section on custom options](#custom-options), below.

#### "select:all"

Triggered when all models have been selected. Provides the ["diff" hash](#multiselect-events) as the first parameter, and the collection as the second. Runs the `onSelectAll` event handler if the method exists on the collection.

#### "select:none"

Triggered when all models have been deselected. Provides the ["diff" hash](#multiselect-events) as the first parameter, and the collection as the second. Runs the `onSelectNone` event handler if the method exists on the collection.

#### "select:some"

Triggered when at least 1 model is selected, but less than all models have been selected. Provides the ["diff" hash](#multiselect-events) as the first parameter, and the collection as the second. Runs the `onSelectSome` event handler if the method exists on the collection.

#### "reselect:any"

Triggered when at least one model, which is already selected, is selected again. Provides an array of the re-selected models as the first parameter. Runs the `onReselect` event handler if the method exists on the collection.

In contrast to the other events, this event fires even if there isn't any change in the resulting selection at all. Note that there is no separate reselect:all event; the re-selection of all items in the collection is also covered by `reselect:any`.

## Sharing models among collections

Models can be part of more than one collection, and Backbone.Select still manages selections correctly. You must enable model sharing explicitly, though, and play by a few rules.

### Features

When sharing models among collections, the collections don't have to be of the same type. A model can be part of single-select and multi-select collections at the same time. Backbone.Select handles all aspects of it:

- Suppose you have selected a model (or models) in one collection, and then you create another one with these models. The new collection will pick up the selections you have already made. That also works when adding models to existing collections.

- The selections in a collection are updated as needed when models are removed. A model loses its `selected` status when it is removed from the last collection holding it. Resetting collections is governed by the same rules.

- When a selection, or deselection, is made with the `silent` option enabled, selection-related events will be silenced in all of the collections sharing the model.

- Selections and deselections involving a number of collections and models are treated as atomic. Events are delayed until all collections and models have been updated, without select or deselect actions pending.

- Edge cases are covered as well. Suppose a number of models are selected in a multi-select collection. You then proceed to add them to a single-select collection. Only one model can be selected there, so Backbone.Select will deselect all but one of them. The last model added to the single-select collection "wins", ie its `selected` status survives.

### Enabling model sharing

To enable model sharing, use the option `enableModelSharing`. Create the mixin with

```js
Backbone.Select.One.applyTo(this, models, { enableModelSharing: true })
```

and likewise for Backbone.Select.Many. See the Basic Usage sections of Backbone.Select.One and Backbone.Select.Many.

### Restrictions when sharing models

There are a few things you must and mustn't do in order to make sharing work, and keep yourself out of trouble.

- Don't use the `silent` option when adding models, removing them, or resetting a collection. If you change the contents of a collection silently, the `selected`/`deselected` status of the shared models won't be synced across collections reliably.

- When a collection is no longer in use, call `close()` on it to avoid memory leaks.

  So don't just replace a collection like this:

        var myCol = new MySelectableCollection([myModel]);
        // ... do stuff
        myCol = new MySelectableCollection([myModel]);  // WRONG!

  Instead, call `close()` before you let an obsolete collection fade away into oblivion:

        var myCol = new MySelectableCollection([myModel]);
        // ... do stuff
        myCol.close();
        myCol = new MySelectableCollection([myModel]);

  Note that you don't need to call `close()` if you use Backbone.Select in "single-collection mode", without sharing models among collections.

### Events

If you are working with events a lot, there are a few details which may help.

- As stated earlier, selections and deselections involving a number of collections and models are treated as atomic. Events are delayed until all collections and models have been updated, without select or deselect actions pending.

- When selected models are added or removed, the collection is updated and the corresponding `select:*` event is fired. In its options, the name of the initial Backbone `add` or `remove` event is available as `options._externalEvent`. The `select:*` options also contain the event options of the initial Backbone event.

- By contrast, a `reset` does not trigger a `select:*` or `deselect:one` event on the collection which is reset. The command is meant to suppress individual notifications, just like it does for `add` and `remove` events, and only fires a `reset` event in the end.

## Custom options

With custom options, you can send additional information to event handlers. Just pass an arbitrary, custom option (or a whole bunch of them) to any method. The option doesn't affect the operation of Backbone.Select, but it is passed on to the event handlers as the last argument.

```js
myCol = new SingleCollection([myModel]);
myCol.on("select:one", function (model, collection, options) {
  if (options) console.log("Selected while foo=" + options.foo);
});

myCol.select(myModel, {foo: "bar"});    // prints "Selected while foo=bar"
```

Options get passed around to all event handlers which are running. In the example above, the event handler is set up for the collection. But it will also pick up an option passed to the `select` method of the model, for instance.

```js
myModel.select({foo: "baz"});    // prints "Selected while foo=baz"
```

## Compatibility with Backbone's own select method

Out of the box, Backbone collections have their own `select` method - an alias of `filter`. In your own code, that should not be an issue: just use `filter`.

The original `select` method of Backbone collections is still available, though, and can be called just as before. Even implementations overriding Backbone's `select` remain accessible. That's because the `select` method is overloaded. If the first argument in a `select` call is a model, the Backbone.Select mixin will handle it. If not, the call is passed up the prototype chain.

That kind of compatibility is crucial for third-party plugins or legacy code. They may rely on Backbone's select, or on their own implementation. Even so, they will continue to work - no modification required.

## Compatibility with Backbone.Picky

This component started off as a series of PRs for [Backbone.Picky][] and eventually turned into an independent fork.

### Picky vs Select: When to choose which

Backbone.Picky has a smaller feature set than Backbone.Select and doesn't handle edge cases nearly as well. Picky provides a bare-bones implementation which leaves the details up to you. But guess what? This is exactly why you might want to choose it over Backbone.Select.

- You can read the source of Backbone.Picky in minutes and understand every aspect of it.
- Because it is easy to understand, it is also easy to tweak. If you adapt the code and make it fit your needs, you are unlikely to accidentally mess up the component.

Simplicity is a virtue. If that is what you want, Picky is the better choice. Conversely, you might want to go with Backbone.Select because there is less need to hack it.

- If you share models among multiple collections, by all means, choose Backbone.Select. It takes care of all the quirks, and there are many. Backbone.Picky doesn't support model sharing.
- Additions, resets, and models passed in during instantiation are taken care of.
- The original `select` method of Backbone collections can still be called. Third-party code relying on it will continue to work.
- You get a richer set of events, more helpful data emitted by these events, a `silent` option, custom options pass-through, and predefined event handlers like `onSelect`.
- Better events make it less likely you ever need to touch the component itself. Your adaptations can go into event handlers, allowing for a clean design.
- Backbone.Select is extremely well tested. Even though the code is (a little) more complex than Backbone.Picky, you can tweak it without hesitation. If you mess up, a unit test will tell you.

### Compatibility

Backbone.Select is fully compatible to Backbone.Picky once you have instantiated it. You only have to change the way the mixin is created and applied to an object.

Picky.Selectable was applied like this:

    initialize: function () {
        var selectable = new Backbone.Picky.Selectable(this);
        _.extend(this, selectable);
    }

In Backbone.Select, it has become

    initialize: function () {
        Backbone.Select.Me.applyTo(this);
    }

Similarly, the initialization of Picky.SingleSelect

    initialize: function () {
        var singleSelect = new Backbone.Picky.SingleSelect(this);
        _.extend(this, singleSelect);
    }

is replaced by

    initialize: function (models) {
        Backbone.Select.One.applyTo(this, models);
    }

Picky.MultiSelect is treated the same way. Use `Backbone.Select.Many.applyTo(this, models)`.

If you want to [enable model sharing][sharing] in a Select.One or Select.Many collection, you need to pass in an options hash as the third argument: `{ enableModelSharing: true }`. [See above][sharing].

## Build process and tests

If you'd like to fix, customize or otherwise improve the project: here are your tools.

### Setup

[npm][] and [Bower][] set up the environment for you.

- The only thing you've got to have on your machine is [Node.js]. Download the installer [here][Node.js].
- Open a command prompt in the project directory.
- Run `npm install`. (Creates the environment.)
- Run `bower install`. (Fetches the dependencies of the script.)

Your test and build environment is ready now. If you want to test against specific versions of Backbone, edit `bower.json` first.

### Running tests, creating a new build

The test tool chain: [Grunt][] (task runner), [Karma][] (test runner), [Jasmine][] (test framework). But you don't really need to worry about any of this.

A handful of commands manage everything for you:

- Run the tests in a terminal with `grunt test`.
- Run the tests in a browser interactively, live-reloading the page when the source or the tests change: `grunt interactive`.
- Build the dist files (also running tests and linter) with `grunt build`, or just `grunt`.
- Build continuously on every save with `grunt ci`.
- Change the version number throughout the project with `grunt setver --to=1.2.3`. Or just increment the revision with `grunt setver --inc`. (Remember to rebuild the project with `grunt` afterwards.)
- `grunt getver` will quickly tell you which version you are at.

Finally, if need be, you can set up a quick demo page to play with the code. First, edit the files in the `demo` directory. Then display `demo/index.html`, live-reloading your changes to the code or the page, with `grunt demo`. Libraries needed for the demo/playground should go into the Bower dev dependencies, in the project-wide `bower.json`, or else be managed by the dedicated `bower.json` in the demo directory.

_The `grunt interactive` and `grunt demo` commands spin up a web server, opening up the **whole project** to access via http. By default, that access is restricted to localhost. You can relax the restriction in `Gruntfile.js`, but be aware of the security implications._

### Changing the tool chain configuration

In case anything about the test and build process needs to be changed, have a look at the following config files:

- `karma.conf.js` (changes to dependencies, additional test frameworks)
- `Gruntfile.js`  (changes to the whole process)
- `web-mocha/_index.html` (changes to dependencies, additional test frameworks)

New test files in the `spec` directory are picked up automatically, no need to edit the configuration for that.

## Release Notes

### v1.2.5

* Restored access to the `select` method of Backbone.Collection by overloading the `select` method.

### v1.2.4

* Added arguments validation to `applyTo` factory methods.
* Various minor bugs fixed.

### v1.2.0

* Related selections and deselections are treated as a single, atomic transaction. Firing of events is delayed until select and deselect actions have spread across all affected models and collections, without any actions still pending.

### v1.1.2

* Fixed bug when models were added or removed with `reset` (collection was not correctly registered with models)

### v1.1.1

* Relaxed dependency requirements in `bower.json`

### v1.1.0

* Moved build to `dist` directory
* Added `_pickyType` property to identify mixins in a model or collection
* Switched development stack to Bower, Karma, Node web server
* Added demo app, memory-leak test environment in `demo` directory

### v1.0.1

* Removed obsolete Backbone.Picky files from build

### v1.0.0

* Forked Backbone.Picky, renaming the project to Backbone.Select
* Renamed components to Select.Me (former Selectable), Select.One (former SingleSelect), Select.Many (former MultiSelect)
* Added `options._externalEvent`, available when the selection in a collection is altered during an `add` or `remove` action
* Added `applyTo` class methods for setup
* Removed support for creating new Backbone.Select objects solely with the constructor
* Event handlers with standard names are invoked automatically if they exist (`onSelect`, `onDeselect`, `onReselect`, `onSelectNone`, `onSelectSome`, `onSelectAll`)
* Options - including arbitrary, custom ones - are passed on to event handlers
* The collection is also passed to event handlers (single-select collection)
* A "diff" hash is passed to select:* event handlers (multi-select collection)
* New events capture when models are re-selected: `reselected` (model), `reselect:one` (single-select collection), `reselect:any` (multi-select collection)
* Multi-select events no longer fire when `selectAll`, `deselectAll` actions are a no-op (change in spec)
* Added support for sharing models among collections
* Added a `silent` option
* Improved events, now firing when model and collection are in a consistent state (Picky issue #18)
* Added `deselectAll`, while keeping `selectNone` around as an alias
* More comprehensive testing
* Added config file for the Karma test runner

### v0.2.0

* Renamed `SingleSelect` events from "select" and "deselect" to "select:one" and "deselect:one"
* Pass model as argument in select:one / deselect:one events
* Updated the build to use latest grunt and related tools
* Removed reliance on ruby for any part of this project

### v0.1.0

* Added Picky.SingleSelect
* Fleshed out the specs more

### v0.0.1

* Initial release of untested code
* Basic "Selectable" mixin for models
* Basic "MultiSelect" mixin for collections

## Credits, Copyright, MIT License

Special credits go to [Derick Bailey][muted-solutions], who created the original version of this component, [Backbone.Picky][]. It is still around; see the [Backbone.Picky Compatibility section][picky-compatibility] for the differences.

Copyright (c) 2014 Michael Heim<br>
Copyright (c) 2013 Derick Bailey, Muted Solutions, LLC

Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.

[Backbone]: http://backbonejs.org/ "Backbone.js"
[Node.js]: http://nodejs.org/ "Node.js"
[Bower]: http://bower.io/ "Bower: a package manager for the web"
[npm]: https://npmjs.org/ "npm: Node Packaged Modules"
[Grunt]: http://gruntjs.com/ "Grunt: The JavaScript Task Runner"
[Karma]: http://karma-runner.github.io/ "Karma - Spectacular Test Runner for Javascript"
[Jasmine]: http://pivotal.github.io/jasmine/ "Jasmine, a behavior-driven development framework"
[JSHint]: http://www.jshint.com/ "JSHint, a JavaScript Code Quality Tool"

[sharing]: #sharing-models-among-collections
[select-compatibility]: #compatibility-with-backbones-own-select-method
[picky-compatibility]: #compatibility-with-backbonepicky
[muted-solutions]: http://mutedsolutions.com/ "Muted Solutions, LLC"
[Backbone.Picky]: https://github.com/derickbailey/backbone.picky#readme "Backbone.Picky"
[Backbone.Cycle]: https://github.com/hashchange/backbone.cycle#readme "Backbone.Cycle"