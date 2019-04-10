# Locations content types and rendering via MapQuest

Your Open Museum project will integrate with MapQuest to display location information on a map. This chapter will walk you through the process of configuring that content type, setting up the intial MapQuestion integration, and then creating the widget with some additional front-end JavaScript used to render the map.

## Creating the modules

We'll start by the basic configuration of the piece type via the CLI. We don't need a special page type for this piece type, but we do want to create widget to show off our locations. We'll do that by simply using the `--widgets` flag when creating the piece type.

```bash
apostrophe create-piece location --widgets
```

Our new modules now exist at `/lib/modules/locations` and `lib/modules/locations-widgets`. We'll configure `/app.js` to include these at startup by adding the following inside the `modules` configuration option:

{% code-tabs %}
{% code-tabs-item title="/app.js" %}
```javascript
'locations': { extend: 'apostrophe-pieces' },
'locations-widgets': { extend: 'apostrophe-widgets' },
```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Configuring the Locations piece

Next, we need to extend our new `location` piece type to add all of the fields we need to store our data for this content type.

First up, we need a simple string field to store the full address of the location:

```javascript
    {
      name: 'address',
      label: 'Address',
      type: 'string',
      required: true
    },
```

As part of this module, we're going to be using MapQuest to geocode the address and turn it into more structured data, so we'll need some fields to store that in as well:

```javascript
    {
      name: 'latlng',
      contextual: true,
      label: 'Latitude / Longitude',
      type: 'array'
    },
    {
      name: 'stateCode',
      label: 'stateCode',
      contextual: true,
      type: 'string'
    },
    {
      name: 'zipCode',
      label: 'zipCode',
      contextual: true,
      type: 'string'
    },
    {
      name: 'streetName',
      label: 'streetName',
      contextual: true,
      type: 'string'
    },
    {
      name: 'city',
      label: 'city',
      contextual: true,
      type: 'string'
    }
```

All of these will be defined inside the `addFields` configuration at `/lib/modules/locations/index.js`:

```javascript
  addFields: [
    {
      name: 'address',
      label: 'Address',
      type: 'string',
      required: true
    },
    {
      name: 'latlng',
      contextual: true,
      label: 'Latitude / Longitude',
      type: 'array'
    },
    {
      name: 'stateCode',
      label: 'stateCode',
      contextual: true,
      type: 'string'
    },
    {
      name: 'zipCode',
      label: 'zipCode',
      contextual: true,
      type: 'string'
    },
    {
      name: 'streetName',
      label: 'streetName',
      contextual: true,
      type: 'string'
    },
    {
      name: 'city',
      label: 'city',
      contextual: true,
      type: 'string'
    }
  ],

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Geocoding the Locations data on save

We have our address field where the user will be entering in the full address of the location, but how do we populate the rest of the fields of this piece? We're going to use the `node-geocoder` module (see node-geocoder on npm) and the MapQuest API to do this.

First, we need to install `node-geocoder` into our project with npm:

```bash
npm install node-geocoder --save
```

Now, we can reference this module in `/lib/modules/locations/index.js`:

```javascript
const Geo = require('node-geocoder');
```

With this at the top of our `index.js` file, we can begin to add in the logic to access the MapQuest API. We do this by leveraging the `construct` and `afterConstruct` configuration options.

{% code-tabs %}
{% code-tabs-item title="/lib/modules/locations/index.js" %}
```javascript
  construct: function (self, options) {
    // Instantiate the geocoding library and set the provider and apiKey options so it
    // can be used when `location` pieces are saved.
    self.enableGeo = function () {
      if (self.options.mapQuest) {
        self.geo = Geo({
          provider: 'mapquest',
          apiKey: self.options.mapQuest.key
        });
      } else {
        console.warn('WARNING: No MapQuest API credentials found, expected as part of the `locations` piece type\'s options. See the README for information about where to put these options and see https://developer.mapquest.com/documentation/open/ for generating a MapQuest key/secret');
      }
    };
    // Before the `location` piece is saved, use the geocoding library to call the 
    // geodata API (MapQuest) and update the additional fields with the geocoded location data.
    self.beforeSave = function (req, piece, options, callback) {
      if (!self.geo) {
        console.warn('WARNING: Can\'t geocode `location` piece\'s address field, maps will not be rendered');
        return callback(null);
      }
      return self.geo.geocode(piece.address, function (err, res) {
        if (err) {
          return callback(err);
        }
        let l = res[0];

        piece.latlng = [l.latitude, l.longitude];
        piece.stateCode = l.stateCode;
        piece.city = l.city;
        piece.streetName = l.streetName;
        piece.zipCode = l.zipcode;
        return callback(null);
      });
    };
  },
  afterConstruct: function (self) {
    self.enableGeo();
  }
```
{% endcode-tabs-item %}
{% endcode-tabs %}

[You should take a moment to learn more about `construct` and `afterConstruct` here](https://docs.apostrophecms.org/apostrophe/technical-overviews/how-apostrophe-starts-up#initialization-of-an-individual-module), but what is basically happening above is we're adding a new method to instiate the geocoding library and then using that method to geocode the data via the MapQuest API whenever a `locations` piece is saved.

The last bit of configuration we need to set up here is the MapQuest API key. You can see that we're referencing that above via `self.options.mapQuest.key`. The correct practice for storing things like API keys is to save them in environment specific module settings, and to never commit them to public repositories. [This guide illustrates creating environment-specific module settings](https://apostrophecms.org/docs/tutorials/getting-started/settings.html#changing-the-value-for-a-specific-server-only) without adding them to your repository.

Following the above guide, here's an example of how to pass those API credentials into the `locations` module via its declaration in `/app.js`:

```javascript
'locations': {
  extend: 'apostrophe-pieces',
  'mapQuest': {
    key: process.env.MAPQUEST_KEY,
    secret: process.env.MAPQUEST_SECRET
  }
},
```

With the above methods in place, we now have everything we need to manage our locations pieces, and have the geodata get automatically populated via MapQuest whenever the records are saved.

## Configuring the Locations widget

Now that we have our `location` piece type configured, we can move onto building out the `locations-widget` that we'll use to render the content.

## Rendering the Locations piece data on the widget via MapQuest

