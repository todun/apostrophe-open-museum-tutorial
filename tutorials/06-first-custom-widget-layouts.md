# The First Custom Widget - Layout Widgets

<!--
Called the columns-widget in the code.
Minimal rewrite here (hopefully).
Remember to add this new widget to the default page.
-->

We've already added some core Apostrophe widgets to the page templates, and there are even more that are available "out of the box." But we'll need some new widgets that are specific to our project. The first of those will be a "Columns" custom widget to provide content layout on the page as-desired.

## Layout / Nested widgets

Layout widgets aren't really special from a technical standpoint, but are a particular pattern used to create complex layouts of widgets. This pattern allows us to enforce particular widgets for particular page layout, but lets the editor decide where it's appropriate to create these complex arrangements and how to arrange them with other widget types. In short, it is a layout of inner widgets.

## Creating the widget

One way to create a new widget is using the [`apostrophe-cli`](https://github.com/punkave/apostrophe-cli). If you're using that, use the following command to create the structure of the new widget:

```bash
apostrophe create-widget columns
```

If not using the CLI tool, then create a folder, `columns-widgets` in `lib/modules` and add an `index.js` file there with the following:

```javascript
module.exports = {
  label: 'Columns',
  addFields: []
};
```

To finish the initial setup, also create a `views` folder in `columns-widgets`, and then an empty `views/widget.html` file. We'll come back to that later, but at this point we have everything the CLI tool would create for us.

{% hint style="info" %}
`label` is the name of the widget as CMS users will see it. `addFields` is an array of fields that users can update in this widget. Some module types, such as `apostrophe-pages`, include initial fields that you'll add to, but `apostrophe-widgets` is going to be a blank slate.
{% endhint %}

### Initialize the widget

Finally, we need to initialize the module on app start. Every new module, whether a piece type, widget, page type, or otherwise, will need to do this next step.

Open up `app.js`. You'll see some start widgets there already. Inside the `modules` object, add:

```javascript
modules: {
  // ...,
  'columns-widgets': { extend: 'apostrophe-widgets' },
}
```
{% hint style="info" %}
What does `extend` mean here? Our module is extending the `apostrophe-widgets` module, which provides almost all the code we need. Apostrophe uses [moog](https://npmjs.org/packages/moog) to handle extending or "subclassing" other modules.

If you used the CLI, the `extend` property would have been included. Stylistically, we prefer this pattern to have the `extend` properties for modules all in `app.js` to make it easy to see all the widgets and what types they are together. If the property ends up in both places it's fine (as long as they have the same value), it's better to choose one pattern and be consistent.
{% endhint %}

## Adding widget fields and functionality

Right now the widget could be added to a page template, but it wouldn't do anything. We need to add fields and some other properties.

First, add the preoprty `skipInitialModal: true` to the module:

```javascript
module.exports = {
  label: 'Columns',
  skipInitialModal: true,
  addFields: [
  //...
}
```

{% hint style="info" %}
In short, since the "Columns" widget is mainly for layout, this disables an initial modal when the widget is added, while allowing for widget settings later on. [More on this in the Apostrophe documentation.](https://docs.apostrophecms.org/apostrophe/tutorials/getting-started/layout-widgets#conveniences-for-layout-widgets-contextualonly-and-skipinitialmodal)
{% endhint %}

Before we start adding fields, for our particular use case we'll need a couple of helpers. Add the following to the top, _before_ `module.exports`:

```javascript
const _ = require('lodash');
const {baseWidgets} = require('../helpers/lib/areas.js');

const columnWidgets = _.cloneDeep(baseWidgets);
delete columnWidgets.columns;
```

The `baseWidgets` object is referenced from a helpers file and is a collection of widgets that will be available inside each column. Then to prevent infintely recursive columns, we remove that widget option from the list with the help of Lodash.

### Adding the fields

Now that we have all of our helpers in place, let's add the fields. We'll walk through the options, but first here's the full array of fields to add:

```javascript
module.exports = {
  // ...,
  addFields: [
    {
      name: 'config',
      label: 'Column Configuration',
      type: 'select',
      choices: [
        { label: 'Three Columns (3 / 3 / 2)', value: 'three' },
        { label: 'Two Columns (5 / 3)', value: 'two' },
        { label: 'Two Columns (3 / 5)', value: 'two-reverse' }
      ]
    },
    {
      name: 'background',
      label: 'Background',
      type: 'select',
      choices: [
        { label: 'None', value: 'none' },
        { label: 'Cream', value: 'o-background-brand-secondary' },
        { label: 'Light Purple', value: 'o-background-light' }
      ]
    },
    {
      name: 'column1',
      label: 'Column One',
      contextual: true,
      type: 'area',
      options: {
        widgets: columnWidgets
      }
    },
    {
      name: 'column2',
      label: 'Column Two',
      contextual: true,
      type: 'area',
      options: {
        widgets: columnWidgets
      }
    },
    {
      name: 'column3',
      label: 'Column Three',
      contextual: true,
      type: 'area',
      options: {
        widgets: columnWidgets
      }
    }
  ]
};
```

We're using two types of field schema here, `select` and `area`. A [`select`](https://docs.apostrophecms.org/apostrophe/tutorials/getting-started/schema-guide#select) is a traditional select, or drop-down, field. The `config` field will let users choose what configuration of columns they want

### Conveniences for layout widgets \(contextualOnly and skipInitialModal\)

> **contextualOnly: true** If your widget contains _only_ other areas and singletons that you want to edit contextually on the page then you don't need the typical manager modal UI popping up when you create the widget. Nor do you need an Edit button UI to edit non-existing configuration. `contextualOnly` will shortcut these and instantly plop your empty widget on the page.

![](../../.gitbook/assets/ezgif.com-video-to-gif-1.gif)

> **skipInitialModal: true** An alternative to `contextualOnly`, `skipInitialModal` lets you skip the widget manager modal when the widget is created \(like `contextualOnly`\) but preserves the Edit UI for later use. This is useful for widgets that have secondary configuration, like setting a background color.

## Putting it in the page

Now, like any other widget, you need to have a `widget.html` template. In this case we'll use that template to call `apos.area` once for each area and introduce nested widgets.

In `lib/modules/two-column-widgets/views/widget.html`

```markup
<div class="two-column">
    <div class="column-left">
        {{ apos.area(data.widget, 'areaLeft', {
            widgets: {
                'apostrophe-images': {}
            }
        }) }}
    </div>
    <div class="column-right">
        {{ apos.area(data.widget, 'areaRight', {
            widgets: {
                'apostrophe-images': {}
            }
        }) }}
    </div>
</div>
```

> "Why are the two columns stacked on top of each other?" You need to write your own CSS to position the `column-left` and `column-right` divs. However, you can find a [complete, working example with CSS here in the apostrophe-samples project](https://github.com/apostrophecms/apostrophe-samples).

