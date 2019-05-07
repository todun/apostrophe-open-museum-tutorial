# Default Page

- Add the default page type to the pages config and make it available to the page type selector.
- Create the default.html template file, fully copying `home.html`.
- Clean it up by `extend`ing the default template in `home.html`
- We're going to add fields to the default page settings, so make it extend `apos-custom-pages` in app.js.
- What's a "custom page"?
  - Add `masthead` and `peerNav` fields to `custom-pages/index.js`
- What's a template macro? Note that more will be discussed later.
  - Add masthead macro to the template with the conditional.
  - Add the peer nav macro to the page. Note and link to info about how pages come with _ancestors and _children.