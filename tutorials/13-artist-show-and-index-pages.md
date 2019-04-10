# Artist pages \(show and index\)

{% hint style="note" %}
  After creating a few `artists` pieces, you'll want to display them within an index (or directory), in addition to having dedicated detail show pages. This functionality is built into Apostrophe, and easily achieved by extending `apostrophe-pieces-pages`.
{% endhint %}

## Basic configuration

In your `lib/modules` directory, create a new folder titled `artists-pages`. Within that folder, create an `index.js` file and add the following basic configuration:

```javascript
// lib/modules/artists-pages
module.exports = {
  label: 'Artist Page',
  name: 'artist-pages',
  addFields: []
};
```

{% hint style="note" %}
  Alternatively, if you used the `apostrophe-cli` "create-piece" command, and supplied it the `--pages` flag, it will automatically do the above for you. See [Apostrophe-Cli: Create A  Piece](https://github.com/apostrophecms/apostrophe-cli#create-a-piece) for more details.
{% endif %}

In your `app.js` file at the root of the project, add the following to the module configuration:

```javascript
// inside the modules: property in `app.js`
'artists-pages': { extend: 'apostrophe-pieces-pages' },
```

For in-depth coverage of `apostrophe-pieces-pages` options, see [Apostrophe Pieces Pages (Module)](https://docs.apostrophecms.org/apostrophe/modules/apostrophe-pieces-pages)

## Adding Artist Index page to available page template options

After you've added `artists-pages` to the `app.js` module configuration, you'll want to now include it within the Apostrophe pages ecosystem. This will allow you to create an Artist index when creating a new page. In the `apostrophe-pages` configuration, you'll need to add the following to the `types` array:

```javascript
// lib/modules/apostrophe-pages/index.js
types: [
  ...
  {
    label: 'Artworks Index',
    name: 'artwork-pages'
  }
]
```
{% hint style="warning" %}
  In the above example, the `label` is what will display to the user, and the `name` refers to the `name` property within `artists-pages`. To learn more about adding custom page templates, see: [Adding A New Page Template](https://docs.apostrophecms.org/apostrophe/tutorials/getting-started/editing-page-templates#adding-a-new-page-template)
{% endhint %}

## Configuring basic views for Index and Show Pages

{% hint style="note" %}
  Out of the box, `artists-pages` will inherit views from `apostrophe-pieces-pages`. However, you'll want to override this project level and provide your own custom markup.
{% endhint %}

In the root of the `artists-pages` module, create a `views` folder. Within that `views` folder, create both an `index.html` and `show.html` file. Apostrophe will look for these files, and (when available), display them as dedicated index and show page templates.

{% hint style="note" %}
  Within your `index.html` file, you'll have access to all your Artist pieces automatically via `data.pieces`. Similarly, you'll have access to the individual piece via `data.piece` within the `show.html`.
{% endhint %}

In the corresponding `index.html` file, copy the following example markup:

```html
{% extends "layout.html" %}
{% import "macros/masthead.html" as masthead %}
{% import "apostrophe-pager:macros.html" as pager %}
{% import "macros/artist-cards.html" as cards %}

{% set page = data.page %}

{% block main %}
  {{ masthead.render(data.page) }}
  <div class="o-container o-container--medium">
    {{ cards.render(data.pieces) }}
    <div class="c-pager">
      {{ pager.render({ page: data.currentPage, total: data.totalPages }, data.url) }}
    </div>
  </div>
{% endblock %}
```
{% hint style="note" %}
  The above markup will loop through all the pieces, and display them through a `cards` macro. Additionally, there is a standard pagination leveraging the `pager` macro.
{% endhint %}



In the corresponding `show.html`, copy the following markup:
```html
{% extends 'layout.html' %}
{% set piece = data.piece %}
{% set artist = data.piece._artist %}
{% import 'apostrophe-templates:macros/breadcrumbs.html' as breadcrumbs %}
{% import 'macros/artwork-cards.html' as cards %}
{% import 'macros/image.html' as image %}
{% import 'macros/definitionList.html' as dl %}

{% block main %}
  <main class="o-mt-6 o-container o-container--medium c-artwork-page">
    {{ breadcrumbs.render(data.page) }}
    <h1 class="o-color-brand-primary o-title c-artwork-page__title">{{ piece.title }}</h1>
    {% if piece._artist %}
      <h2 class="o-subheadline c-artwork-page__artist">{{ artist.title }}</h2>  
    {% endif %}
    <figure class="c-artist-page__image">
      {{ image.render(piece.thumbnail.items[0]._pieces[0], { description: true }) }}
    </figure>
    <div class="c-meta-columns c-artist-page__content">
      <div class="c-meta-columns__left c-artist-page__meta">
        <dl class="c-defintion-list c-artist-page__meta-list">
          {{ dl.render(piece, [
            { name: 'lifetime' },
            { name: 'nationality' },
            { name: 'movement' }
          ]) }}
        </dl>
      </div>
      <div class="c-meta-columns__right c-artist-page__body">
        {{ apos.area(piece, 'body', { widgets: apos.helpers.narrowWidgets }) }}
      </div>
    </div>
    <div class="u-small-dropdowns c-artist-page__extra">
      {{ apos.area(piece, 'extra', { widgets: apos.helpers.baseWidgets }) }}
    </div>
  </main>
{% endblock %}
```

{% hint style="note" %}
  The above markup will output basic meta information on the piece, as well as columns for areas that include base widgets. *Note that these areas are saved to the `piece` and not the `page`*.
{% endhint %}
