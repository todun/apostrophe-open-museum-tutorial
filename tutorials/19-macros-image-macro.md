# Creating and Using Macros

Apostrophe uses the [Nunjucks Templating engine](https://mozilla.github.io/nunjucks/). We can create reusable chunks of content called [macros](https://mozilla.github.io/nunjucks/templating.html#macro) that we can call in templates and widgets throughout our project.

We're going to create an index page for our `People` piece that will use an `image` macro to render images for each person. There will be some logic in this macro to help parse out the image data inside the `thumbnail` field (an `apostrophe-images` singleton) on each person. See [apostrophe-images](/add/link/here) for more info on this module.

First, we'll create our macro file at `/views/macros/image.html`

Next, we'll define a function in our macro called `render` with some parameters:

```markup
// views/macros/image.html

{% macro render(imageObj, options= {size: 'full', alt: false, description: false} ) %}

{% endmacro %}
```

Our first parameter `imageObj` will be the `thumbnail` field of our person.

The second paramter will be an `options` object where we can pass in some extra config, like the `size` of the image, custom alt text for the `<img>`, and a custom `description` that will populate a `<figcaption>` for this `<img>`.

Next, let's start building out the macro:

```markup
// views/macros/image.html

{% macro render(imageObj, options= {size: 'full', alt: false, description: false} ) %}

  {% set image = apos.images.first(imageObj) %}

  <div class="o-image__wrapper{% if options.class %} {{ options.class }}{% endif %}">

  </div>
  
{% endmacro %}
```

We're going to pass our `imageObj`into the Apostrophe helper [apos.images.first()](https://docs.apostrophecms.org/apostrophe/modules/apostrophe-attachments#first-within-options-api). This helper takes an `object` with attachments as well as an `options` object. When we pass in our `apostrophe-images` singleton, the helper will return the first image in its attachment items array. We'll save this into a variable called `image` and create a div with a base class as well as some template logic to render any additional class passed into the `class` property of our `options` object.

Now let's flesh out the inside of the macro a bit more:
```markup
// views/macros/image.html

{% macro render(imageObj, options= {size: 'full', alt: false, description: false} ) %}

  {% set image = apos.images.first(imageObj) %}

  <div class="o-image__wrapper{% if options.class %} {{ options.class }}{% endif %}">

    {% if image === undefined %}
      <img class="o-image" src="https://via.placeholder.com/300x200" alt="">
    {% else %}
      {% set url = apos.attachments.url(image, { size: options.size, crop: image._crop }) %}
    {% endif %}

  </div>

{% endmacro %}
```
Inside our div we'll check if our `image` is `undefined`. If it is, we'll render a [placeholder image](https://via.placeholder.com) with some empty alt text. Otherwise, we'll pass our image into another Apostrophe helper called [apos.attachments.url()](https://docs.apostrophecms.org/apostrophe/modules/apostrophe-attachments#url-attachment-options-api). This takes an `attachment` object and an `options` object. In our `options` object, we'll pass in a `size` (which we grab from the `options` object of our macro), as well as a `crop` value, which is a dynamically generated property on our `image` object. See [here](/add/link/here) for more info on image options. We'll save this URL into a variable called `url`.

Now let's take our `url` and some of our `options` and render the image with some description/caption information:

```markup
// views/macros/image.html

{% macro render(imageObj, options= {size: 'full', alt: false, description: false} ) %}

    ...

      {% set url = apos.attachments.url(image, { size: options.size, crop: image._crop }) %}
      <img class="o-image" src="{{ url }}" alt="{{ options.alt or image.title or image._description }}">
      {% if options.description === true and image._description %}
        <figcaption class="o-meta o-image__caption">{{ image._description }}</figcaption>
      {% endif %}
      {% if apos.helpers._isString(options.description) %}
        <figcaption class="o-meta o-image__caption">{{ options.description }}</figcaption>
      {% endif %}

    ...

{% endmacro %}
```

We'll pop our `url` into an `<img>` and add some alt text using the data passed into our macro. If no alt data was passed in, we'll default to the `title` or the `_description` properties of the image. If our macro's `options.description` property is set and there is a `_description` set on the image, we'll render that data in a `<figcaption>` below the image. If `options.description` is a string (using another handy helper called `apos.helpers._isString`), we'll render that string in the `<figcaption>`. This is useful if we want to render a custom caption instead of using the `_description` that was set on the image itself.


Here's the final markup:

```markup
// views/macros/image.html

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

In the next section, we'll take a look at calling this macro to build out our `People` index page.
