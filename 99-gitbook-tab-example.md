{% tabs %}
{% tab title="HTML" %}
{% code-tabs %}
{% code-tabs-item title="/lib/modules/artworks-widgets/views/widget.html" %}
```markup
{% set w = data.widget %}
{% import 'macros/button.html' as button %}
{% import 'macros/artwork-cards.html' as artworkCards %}

<div class="o-container c-artworks-widget">
  <header class="c-artworks-widgets__header">
    {% if w.title %}
      <h3 class="o-headline o-color-brand-primary">{{ w.title }}</h3>
    {% endif %}
    {% if w._page %}
      {{ button.render('View ' + w._page.title, w._page._url) }}
    {% endif %}
  </header>
  <div class="c-artworks-widget__content">
    {{ artworkCards.render(w._pieces) }}
  </div>
</div>
```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="CSS" %}
{% code-tabs %}
{% code-tabs-item title="/lib/modules/apostrophe-assets/public/css/components/\_artworks-widgets.less" %}
```css
.c-artworks-widget {
  border: 1px solid @color-brand-primary;
  padding: 4rem;
  @media @breakpoint-large {
    border: none;
  }
}

.c-artworks-widgets__header {
  display: flex;
  justify-content: space-between;
  margin-bottom: 4rem;
  @media @breakpoint-medium {
    flex-direction: column;
    text-align: center;
    .o-headline {
      margin-bottom: 2rem;
    }
  }
}

```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}

{% tab title="JS" %}
{% code-tabs %}
{% code-tabs-item title="lib/modules/apostrophe-assets/public/js/modules/mobile-menu.js" %}
```javascript
$(function () {
  var $body = $('body');
  var stateClass = 's-show-menu';

  $body.on('click', '[data-mobile-trigger]', function () {
    $body.toggleClass(stateClass);
  });
});

```
{% endcode-tabs-item %}
{% endcode-tabs %}
{% endtab %}
{% endtabs %}
