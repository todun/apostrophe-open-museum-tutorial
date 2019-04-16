# Making an Artist piece

In addition to artworks, we'll need to store data about artist. Again, create the artist piece with the command:

```
apostrophe create-piece artist --pages
```

Note: Our website does not have an artist widget, so we omit the `--widget` flag.

# Artist piece configuration

First in the configuration we'll require some area helpers and the lodash library.

Then we'll set `contextual` to `true` like when setting up the [Artworks piece](#link-to-artworks-piece) and set the default sorting to use the `title`.

```
const areas = require('../helpers/lib/areas.js');
const _ = require('lodash');

module.exports = {
  name: 'artist',
  label: 'Artist',
  pluralLabel: 'Artists',
  contextual: true,
  sort: {
    title: 1
  }
  ...
}
```

Next we'll add a bunch of string fields...

```
addFields: [
  {
    name: 'description',
    label: 'Short Description',
    help: 'This is displayed at the bottom of an Artwork show page that this artist is associated with',
    type: 'string',
    textarea: true
  },
  {
    name: 'lifetime',
    label: 'Lifetime',
    help: 'Year range like 1840–1926',
    type: 'string'
  },
  {
    name: 'nationality',
    label: 'Nationality',
    type: 'string'
  },
  {
    name: 'movement',
    label: 'Movement',
    help: 'Art movement this artist is most closely associated with',
    type: 'string'
  },
  ...
]
```

We'll give the Artist piece an image with a `singleton` data type and a `'apostrophe-images'` `widgetType`.

```
{
  name: 'thumbnail',
  label: 'Thumbnail',
  type: 'singleton',
  widgetType: 'apostrophe-images',
  options: {
    limit: 1,
    aspectRatio: [ 1, 1 ]
  }
```

Artist pages will also have areas to drop in widgets to build out more extensive content.

```
{
  name: 'body',
  label: 'Body',
  contextual: true,
  type: 'area',
  options: {
    widgets: _.clone(areas.narrowWidgets)
  }
},
{
  name: 'extra',
  label: 'Extra',
  contextual: true,
  type: 'area',
  options: {
    widgets: _.clone(areas.baseWidgets)
  }
},
```

Now with the artist schema established we can return to our Artworks piece and create the relationship between the two.


# Joining the Artworks piece to the Artist piece

Apostrophe makes creating relationships between different object types very easy with joins. After Apostrophe loads the original object, it will fetch the "joined" object and attach it to the original via the specified name property. You can check out this [join reference](https://docs.apostrophecms.org/apostrophe/tutorials/getting-started/schema-guide#joinbyone) to read in detail about this great feature.

In our project we want our Artworks pieces to have Artists associated with them, so in our `artworks/index.js` file we define the join.

```
// lib/modules/artworks/index.js

module.exports = {
    ...
    {
      name: '_artist',
      label: 'Artist',
      idField: 'artistId',
      type: 'joinByOne',
      withType: 'artist',
      withJoins: [ '_artworks' ],
      filters: {
        projection: {
          _url: 1,
          title: 1,
          description: 1,
          thumbnail: 1
        }
      }
    }
    ...
}
```

TODO: Go through the fields

TODO: Talk about projections

TODO: Talk about the joinByOneReverse on the artist piece.

## TODO: Adding the link to the View

```
{% macro render(label, url, options) %}
<a href="{{ url }}" class="o-button{% if options.class %} {{ options.class }}{% endif %}">{{ label }}</a>

{% if w._page %}
  {{ button.render('View ' + w._page.title, w._page._url) }}
{% endif %}
```
