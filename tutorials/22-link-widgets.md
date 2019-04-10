# Creating a link widget: classy links to your content

Users can already create links in the rich text editor, but we'd like to do something more
sustainable for linking to content that might move around on the site. Or, we just want
a special visual treatment that goes beyond a simple anchor tag. Here's how to create a link widget.

First declare it in `app.js`:

```javascript
// inside the modules: property in `app.js`
'link-widgets': {
  extend: 'apostrophe-widgets'
}
```

Now let's configure it:

{% code-tabs %}
{% code-tabs-item title="lib/modules/link-widgets/index.js" %}
```javascript
const schema = require('./lib/schema.js');
const _ = require('lodash');

module.exports = {
  label: 'Link',
  addFields: [
    {
      name: 'links',
      label: 'Links',
      type: 'array',
      titleField: 'linkText',
      schema: _.clone(schema)
    }
  ]
};
```
{% endcode-tabs-item %}
{% code-tabs-item title="lib/modules/link-widgets/lib/schema.js" %}
```javascript
module.exports = [
  {
    name: 'linkText',
    label: 'Link Text',
    type: 'string',
    required: true
  },
  {
    name: 'linkType',
    label: 'Link Type',
    type: 'select',
    required: true,
    choices: [{
      label: 'Page',
      value: 'page',
      showFields: [
        '_linkPage'
      ]
    },
    {
      label: 'Custom',
      value: 'custom',
      showFields: [
        'linkUrl'
      ]
    }
    ]
  },
  {
    name: '_linkPage',
    label: 'Link Page',
    type: 'joinByOne',
    withType: 'apostrophe-page',
    idField: 'pageId',
    required: true,
    filters: {
      projection: {
        title: 1,
        _url: 1
      }
    }
  },
  {
    name: 'linkUrl',
    label: 'Link URL',
    type: 'url',
    required: true
  },
  {
    name: 'linkStyle',
    label: 'Link Style',
    type: 'select',
    choices: [
      { label: 'Underlined', value: 'o-link' },
      { label: 'Button (Normal)', value: 'o-button' },
      { label: 'Button (Large)', value: 'o-button o-button--large' },
      { label: 'Button (Ghost)', value: 'o-button o-button--ghost' }
    ]
  },
  {
    name: 'linkTarget',
    label: 'Will the link open a new browser tab?',
    type: 'boolean'
  }
];
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## `array` fields let us have more than one

In `index.js`  we're using a field of type `array`, which lets the user add *one or more* links. We broke
out the fields for each link into their own file, `lib/schema.js`.

Later when we write the template the `links` property will be an array, and we'll iterate over it
with `for` in `widget.html`.

## `showFields` lets us decide which fields are visible

The user can decide whether to link to a pasted URL, or to a page on the site. The schema contains both
a join to a page and a field for the pasted URL. In `lib/schema.js` we use a `select` field with the
`showFields` option to decide which field is visible:
 
```javascript
  {
    name: 'linkType',
    label: 'Link Type',
    type: 'select',
    required: true,
    choices: [{
      label: 'Page',
      value: 'page',
      showFields: [
        '_linkPage'
      ]
    },
    {
      label: 'Custom',
      value: 'custom',
      showFields: [
        'linkUrl'
      ]
    }
    ]
  }
```

## Joining to a page

This field lets the user pick a page:

```javascript
  {
    name: '_linkPage',
    label: 'Link Page',
    type: 'joinByOne',
    withType: 'apostrophe-page',
    idField: 'pageId',
    required: true,
    filters: {
      projection: {
        title: 1,
        _url: 1
      }
    }
  }
```

As a refresher, `joinByOne` is a field type that lets you select another piece type, or a page as in this case, using the special `withType` value `apostrophe-page`.

We set `idField` so Apostrophe knows where to store the `_id` of the page in our widget. We could also skip it and `linkPageId` would be used.

Under `filters`, we configure a `projection` so that Apostrophe doesn't fetch too much information and slow the site down. This is a MongoDB projection, extended with a cool extra feature: projecting `_url` gives us *everything we need to be sure the page object will have the right `_url` property when it is fetched.*

## Rendering the widget

{% code-tabs %}
{% code-tabs-item title="lib/modules/link-widgets/views/widget.html" %}
```markup
{% set w = data.widget %}

<div class="c-link__items">
  {% for item in w.links %}
    {% if item.linkType === 'page' %}
    {% set url = item._linkPage._url %}
    {% elif item.linkType === 'custom' %}
      {% set url = item.linkUrl %}
    {% else %}
      {% set url = '#' %}
    {% endif %}
    <div class="c-link__item">
      <a class="{{ item.linkStyle }} c-link__link" href="{{ url }}">{{ item.linkText }}</a>
    </div>
  {% endfor %}
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}

If the user chose `page` for `linkType`, we use the `_url` property of `_linkPage`, the page that they
selected. If not, we use the `linkUrl` that they manually pasted in.

## Putting it on the page

As a reminder, actually including the link widget in a template is simple. Here's an example of an area with a mix of rich text widgets and link widgets:

```markup
{{ apos.area(data.page, 'body', {
  widgets: {
    'apostrophe-rich-text': { ... },
    'link': {}
  }
) }}
```

