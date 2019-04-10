# Set Up the Default Page

Apostrophe requires that you have a `home.html` template, but when editors are creating new pages it could get confusing if they are all of the "home" page type. We should set up a "default" page type for pages other than the home page.

Fortunately on Open Museum the home page and default page templates are identical. In the same directory where we edited the `home.html`, `lib/modules/apostrophe-pages/views/pages`, copy `home.html` to a new file, `default.html`. (If you're immediately seeing this as duplicative, just keep reading.)

Our template is all set up, but if you restart the app and create a new page, you'll see that "Home" is still the only page type available. We need to tell `apostrophe-pages` that we have a new page type in the app. Open the `index.js` file for `apostrophe-pages`. The only configuration you should see is a single object in the `types` array, which looks like:

```js
{
  name: 'home',
  label: 'Home'
}
```

All we need to do to make the "default" page type available is to add it to the `types` array:

```js
types: [
  {
    name: 'home',
    label: 'Home'
  },
  {
    name: 'default',
    label: 'Default'
  }
]
```

Excellent! Now if you restart, you can create "Default"-type pages.

As suggested earlier, we're duplicating an entire template in our project right now. This sets us up for problems later where someone edits one but forgets to edit the other. Fortunately, Nunjucks makes reducing duplication pretty simple.

Since home pages do tend to end up being unique eventually, let's make the "default" page the canonical source. Since `default.html` is already in place, open up `home.html` in the page views directory and delete everything inside. In that now-empty file, enter `{% extends "pages/default.html" %}`.

As discussed in the [Apostrophe documentation](https://docs.apostrophecms.org/apostrophe/tutorials/getting-started/editing-page-templates#creating-and-extending-page-templates), Nunjucks lets you extend other template files this way. Any code, including template blocks, are inherited and you can override only as needed. The `home.html` file that came from the starter files was already doing this, extending `layout.html` and using the `{% block main %}{% endblock %}` tags to only override the `main` block of that layout file.

Now you have two page types, but only one template that you need to update!