# Piece Widgets: Building the artwork widget

In addition to placing pieces into you pages, you can also include them on pages as a widget. A piece widget is a special Apostrophe widget that easily gives users the ability to add one or more pieces to the page.

When we used the CLI to [create the artworks piece](link-to-previous-step) we used the `--widget` flag to also generate the files and folders needed for a piece-widget

That command creates a module directory named `artwork-widget` that contains `views/widget.html` and `index.js`

## Piece Widget Default fields

Your piece-widget will also automatically give you a "Select by" field with the following options:

    * All
        * Will display all the pieces up to the number specified in the Maximum Displayed field
    * Individually
        * Gives you an auto-completing search and the ability to manually browse to select pieces to add to the page
    * By Tag
        * Apostrophe gives you the ability tag pieces out of the box. Here's a place where you can reference these tags and select only pieces with those tags to appear on the page.

QUESTION TO ANSWER: What's the default order if you select all of get By Tag?

## Customizing a Piece Widget Schema

As with any widget you can add custom fields to appear within the widgets on the page. For this piece we add a simple `title` string field.

```
module.exports = {
  label: 'Artwork Widget',
  addFields: [
    {
      name: 'title',
      label: 'Widget Title',
      type: 'string'
    }]
}
```

Later we will also [join a page document](#link-to-tutorial-12) with a "Page to Link" field that will give the user the ability to place a button on the widget that can link to any existing page on the site.


## Building the Widget View

In the view template we start by setting the widget object to a variable `w = data.widget` and import a macro to display the artwork called `artworkCards` from the file [`macros/artwork-cards.html`](#link-to-macros-file).

We pull in the our custom widget `title` field (if it exists) as `{{ w.title }}` and then send all the widget's pieces objects found by the user's selections to be rendered by the macro with `w._pieces`.

```
{% set w = data.widget %}
{% import 'macros/artwork-cards.html' as artworkCards %}

<div class="o-container c-artworks-widget">
  <header class="c-artworks-widgets__header">
    {% if w.title %}
      <h3 class="o-headline o-color-brand-primary">{{ w.title }}</h3>
    {% endif %}
  </header>
  <div class="c-artworks-widget__content">
    {{ artworkCards.render(w._pieces) }}
  </div>
</div>
```

Note: After we join the page document to the widget we will add that to the above view, as well.
