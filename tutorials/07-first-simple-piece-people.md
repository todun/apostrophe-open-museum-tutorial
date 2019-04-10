# Simple piece - People

Let's create a directory of people who work at the museum. People should be "global content" so they can be displayed across the site, so we'll need to create our own custom `people` module. To do this, we'll extend the core `apostrophe-pieces` module, which can be extended as many times as we need to create custom pieces throughout the project.

Using the apostrophe cli, run `apostrophe create-piece people` to generate the boilerplate for your `people` module. In `lib/modules` you should see a new folder called `peoples` with an `index.js` file where you'll configure the schema for your new piece type. Note that apostrophe cli pluralizes pieces by adding an `s` when using `create-piece`. Let's edit this to use the appropriate pluralization of `people`.

To do this, change the name of your new folder from `peoples` to `people` then  in `lib/modules/people/index.js` update the `name`, and `pluralLabel`. We'll `extend: 'apostrophe-pieces'` later in `app.js` so we can also remove that property from `lib/modules/people/index.js`, which should now look like this:

```javascript
module.exports = {
  name: 'person',
  label: 'People',
  pluralLabel: 'People',
  addFields: []
};
```

IMPORTANT: note the name property. This identifies ONE piece in the database, so it is always singular. Remember: Modules Are Plural (MAP), but the things they manage may not be.

In Apostrophe, each piece comes with some default fields, such as title (the full name of the piece), slug (used when the piece appears as part of a URL), and published (which determines whether the public can see the piece).In our museum project, we also want each person to have a thumbnail, role, and description field so let's add those custom fields to our schema.

```javascript
module.exports = {
  name: 'person',
  label: 'People',
  pluralLabel: 'People',
  addFields: [
    {
      name: 'thumbnail',
      label: 'Headshot',
      type: 'singleton',
      widgetType: 'apostrophe-images',
      options: {
        aspectRatio: [ 1, 1 ],
        minSize: [ 300, 300 ],
        limit: 1
      }
    },
    {
      name: 'role',
      label: 'Role',
      type: 'string'
    },
    {
      name: 'description',
      label: 'Description',
      type: 'string',
      textarea: true
    }
  ]
};
```

Each person will have a single thumbnail image, so we want to add an `apostrophe-images` singleton then set some options to specify a square `aspectRatio`, `minSize`, and `limit`.

The `role` and `description` fields are each strings but we can add `textarea: true` to change description to a `textarea`.

The last step is adding our new `people` module to `app.js` and extending the core `apostrophe-pieces` module:

```javascript
modules: {

  // Other Modules Go Here
  'people': { extend: 'apostrophe-pieces' },

}
```

We can now log into to the site and see a new "People" menu in the admin bar. Select "Add Person" and you'll see the default fields in the "Basics" tab and our custom fields in the "Info" tab.
