# Creating and Using Macros

Apostrophe uses the [Nunjucks Templating engine](https://mozilla.github.io/nunjucks/). We can create reusable chunks of content called [macros](https://mozilla.github.io/nunjucks/templating.html#macro) that we can call in templates and widgets throughout our project.

We're going to create an index page for our `People` piece that will use an `image` macro to render images for each person. There will be some logic in this macro to help parse out the image data inside the `apostrophe-images` singleton on each person.

First, we'll create our macro file at `/views/macros.image.html`

Next, let's define a function function called `render` with some parameters:

```markup
{% macro render(imageObj, options= {size: 'full', alt: false, description: false} ) %}

{% endmacro %}
```

Our first parameter is `imageObj` will be the `apostrophe-images` we will pass in. In our immediate case, this will a thumbnail field on a particular person.

The second paramter will be an `options` object where we can pass in some extra config, like the `size` of the image, custom `alt` text for the `<img>`, and a custom `description` that will populate a `<figcaption>` for this `<img>`.

Next, let's flesh out the macro:



```markup
{% macro render(imageObj, options= {size: 'full', alt: false, description: false} ) %}
  {% set image = apos.images.first(imageObj) %}
  <div class="o-image__wrapper{% if options.class %} {{ options.class }}{% endif %}">
    {% if image === undefined %}
      <img class="o-image" src="https://via.placeholder.com/300x200" alt="">
    {% else %}
      {% set url = apos.attachments.url(image, { size: options.size, crop: image._crop }) %}
      <img class="o-image" src="{{ url }}" alt="{{ options.alt or image.title or image._description }}">
      {% if options.description === true and image._description %}
        <figcaption class="o-meta o-image__caption">{{ image._description }}</figcaption>
      {% endif %}
      {% if apos.helpers._isString(options.description) %}
        <figcaption class="o-meta o-image__caption">{{ options.description }}</figcaption>
      {% endif %}
    {% endif %}
  </div>
{% endmacro %}

```
