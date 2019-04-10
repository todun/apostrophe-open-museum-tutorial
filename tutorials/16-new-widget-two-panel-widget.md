# Creating a custom two-panel widget

Occasionally, we'll want to create a widget that has a self-contained layout with a specific configuration of sub-widgets within it.

You'll notice that we're using some helpers throughout this widget to aid in configuration of our sub-widgets and areas. For a more in-depth look at how to use helpers see [Using Helpers](https://stuartromanek.gitbook.io/open-museum/tutorials/33-using-helpers)

## Initializing a new widget

[Apostrophe CLI](https://github.com/apostrophecms/apostrophe-cli) lets you spin up boilerplate for new widgets fairly easily. Here we'll create a new widget using `apostrophe create-widget two-column`. This will give us all the files we need to get started.

We'll want to create a reference to our new widget in `app.js`:

```javascript
modules: {
  ...
  'two-panel-widgets': { extend: 'apostrophe-widgets' },
  ...
}
```

## Configuring the schema for the widget

Let's start by taking a look at our widget configuration (`index.js`):

{% code-tabs %}
{% code-tabs-item title="lib/modules/two-panel-widgets/index.js" %}
```javascript
const areas = require('../helpers/lib/areas.js');
const _ = require('lodash');

module.exports = {
  label: 'Two Panel',
  skipInitialModal: true,
  addFields: [
    {
      name: 'config',
      label: 'Configuration',
      type: 'select',
      def: 'c-two-panel--content-left',
      choices: [
        { label: 'Content Left / Media Right', value: 'c-two-panel--content-left' },
        { label: 'Content Right / Media Left', value: 'c-two-panel--content-right' }
      ]
    },
    {
      name: 'image',
      label: 'Image',
      contextual: true,
      type: 'singleton',
      widgetType: 'apostrophe-images',
      options: {
        limit: 1,
        template: 'background'
      }
    },
    {
      name: 'body',
      label: 'Body',
      contextual: true,
      type: 'area',
      options: {
        widgets: {
          'apostrophe-rich-text': {
            toolbar: _.clone(areas.baseToolbar),
            styles: _.clone(areas.baseStyles)
          },
          'link': {}
        }
      }
    }
  ]
};
```
{% endcode-tabs-item %}
{% endcode-tabs %}

One of the first thing you'll notice is that we're using `skipInitialModal: true`. This allows us to skip the widget manager modal when the widget is created and begin adding sub-widgets contextually right away, but preserves the Edit UI for later use. This is useful for secondary configuration, in this case our layout or `configuration` select. This field will allow us to change the orientation of content within our widget, but gives us a default to get started with.

The non-configuration fields (`image` and `body`) within our `two-column` widget are both set to be edited contextually with `contextual: true`, so they will not appear in the manager modal for `two-column-widgets`. The `image` field gives us an `apostrophe-images` singleton (covered in more depth in another chapter) for placing a single image in the widget. The `body` field gives us an area in which we can place any number of `apostrophe-rich-text` or `link` widgets (both covered in other chapters).

## Creating the view for the widget

Within our `two-panel-widgets` directory we have a `views` folder that contains the view for our widget in `widget.html`. This is where our presentation logic lives and where we'll write our template for rendering our previously configured schema. The file looks like this:

{% code-tabs %}
{% code-tabs-item title="lib/modules/two-panel-widgets/views/widget.html" %}
```markup
{% set w = data.widget %}
{% if w.config %}
  {% set config = w.config %}
{% endif %}

{% set areaSchema = apos.helpers._find(data.manager.schema, { name: 'body' }) %}
{% set imageSchema = apos.helpers._find(data.manager.schema, { name: 'image' }) %}

<div class="o-background-brand-secondary c-two-panel {{ config }}">
  <div class="c-two-panel__content c-two-panel__body">
    {{ apos.area(w, 'body', areaSchema.options) }}
  </div>
  <div class="c-two-panel__content c-two-panel__media">
    {{ apos.singleton(w, 'image', 'apostrophe-images', imageSchema.options) }}
  </div>
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

It's often useful to store dot paths in Nunjucks variables to save us some key strokes. At the top of our file, we're setting `data.widget` to `w` and `data.widget.config` to `config` for use throughout the template.

We're also running parts of our schema through our `_find` helper (covered in another chapter) to keep parity between our options configured in our schema and template. `areaSchema.options` gives us the options (in this case our configured widgets) for our `body` field for use in our body area, contained within `c-two-panel__body`. `imageSchema.options` gives us our image options for our `image` image field for use in our image singleton, contained within `c-two-panel__media`.

{% hint style="info" %}
Using `data.manager.schema` in the template (`widget.html`) will give you access to the schema that is configured for the widget (`index.js`). This is very useful for configuring areas and singletons within your template and avoiding copy/paste errors.
{% endhint %}

## Widget styles

We're choosing to name-space our classes and author our styles in the `apostrophe-assets` module (`lib/modules/apostrophe-assets/public/css/`). Here, we import styles for individual components into `site.less` and tell the asset pipeline to compile them all to a single css file. We could also push our assets on a per module basis if we wanted to. See [Pushing assets to the browser](https://docs.apostrophecms.org/apostrophe/tutorials/getting-started/pushing-assets) for more information on this.

{% code-tabs %}
{% code-tabs-item title="lib/modules/apostrophe-assets/public/css/components/\_two-panel.less" %}
```css
.c-two-panel {
  display: flex;
}

.c-two-panel--content-left {
  @media @breakpoint-medium {
    flex-direction: column;
  }
}

.c-two-panel--content-right {
  flex-direction: row-reverse;
  @media @breakpoint-medium {
    flex-direction: column-reverse;
  }
}

.c-two-panel__body {
  padding: 12rem 8%
}

.c-two-panel__content {
  width: 50%;
  @media @breakpoint-medium {
    width: 100%;
  }
}

.c-two-panel__media {
  @media @breakpoint-medium {
    height: 40vw;
  }
}

.c-two-panel__media {
 .apos-area,
 .apos-area-widgets,
 .apos-area-widget-wrapper,
 .apos-area-widget,
 .o-background-image
 {
   height: 100%;
   width: 100%;
 }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}
