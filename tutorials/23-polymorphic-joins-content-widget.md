# Polymorphic joins: "content-widgets" that can pick more than one type

We've made widgets that let users pick something to link to. But sometimes we want to let the
user pick documents of many different types. This is where polymorphic joins come in handy.

Let's take a peek at `content-widgets`. First we declare them in `app.js`:

```javascript
// inside the modules: property in `app.js`
'content-widgets': {
  extend: 'apostrophe-widgets'
}
```

Now let's set them up:

{% code-tabs %}
{% code-tabs-item title="lib/modules/content-widgets/index.js" %}
```javascript
module.exports = {
  label: 'Mixed Content',
  addFields: [
    {
      name: 'headline',
      label: 'Headline',
      type: 'string'
    },
    {
      name: '_content',
      label: 'Content to Link',
      type: 'joinByArray',
      withType: ['artwork', 'artist', 'article', 'event'],
      filters: {
        projection: {
          _url: 1,
          title: 1,
          thumbnail: 1,
          year: 1
        }
      }
    }
  ]
};
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## `withJoins` can take an array of type names

We're used to seeing just one type in a join, but we can pass an array. Each value must match the `name` property of a
page type or piece type. We can also use the special value `apostrophe-page` to allow the user to
pick any page (as opposed to piece) on the site. The user seems a nifty tabbed interface that lets them browse each
type separately, then put their combined choices in the order they want.

We also use a `projection` filter to keep the site fast by fetching the properties we care about.
Here we're interested in the `_url` property, but also a `thumbnail` singleton as well a `year` property that some of our pieces have.

## Rendering the widget

The implementation on the site uses some [fancy Nunjucks macros](https://github.com/apostrophecms/apostrophe-open-museum/blob/master/views/macros/generic-cards.html) to [put each piece on a card](https://github.com/apostrophecms/apostrophe-open-museum/blob/master/lib/modules/content-widgets/views/widget.html), but all you really need to know is that it works like rendering any other widget with a join. Here's a simple example:

{% code-tabs %}
{% code-tabs-item title="lib/modules/content-widgets/views/widget.html" %}
```markup
<h3>{{ data.widget.headline }}</h3>
{% for piece in data.widget_content %}
  <h4 class="type-{{ piece.type }}"><a href="{{ piece._url }}">{{ piece.title }}</a></h4>
{% endfor %}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

Here we pop a CSS class on each piece based on its type to help the designer distinguish them visually.
