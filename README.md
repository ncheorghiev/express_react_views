# express-react-views

This is an [Express][express] view engine which renders [React][react] components on server. It renders static markup and *does not* support mounting those views on the client.

This is intended to be used as a replacement for existing server-side view solutions, like [jade][jade], [ejs][ejs], or [handlebars][hbs].


## Usage

```sh
npm install express-react-views
```

### Add it to your app.

```js
// app.js

var app = express();

app.set('view engine', 'jsx');
app.engine('jsx', require('express-react-views').__express);
```

### Views

Your views should be node modules that export a React component. Let's assume you have this file in `views/index.jsx`:

```js
/** @jsx React.DOM */

var HelloMessage = React.createClass({
  render: function() {
    return <div>Hello {this.props.name}</div>;
  }
});

module.exports = HelloMessage
```

### Routes

Your routes would look identical to the default routes Express gives you out of the box.

```js
// app.js

app.get('/', require('./routes').index);
```

```js
// routes/index.js

exports.index = function(req, res){
  res.render('index', { name: 'John' });
};
```

**That's it!** Layouts follow really naturally from the idea of composition.

### Layouts

Simply pass the relevant props to a layout component.

`views/layouts/default.jsx`:
```js
/** @jsx React.DOM */

var DefaultLayout = React.createClass({
  render: function() {
    return (
      <html>
        <head><title>{this.props.title}</title></head>
        <body>{this.props.children}</body>
      </html>
    );
  }
});

module.exports = DefaultLayout;
```

`views/index.jsx`:
```js
/** @jsx React.DOM */

var DefaultLayout = require('./layouts/default');

var HelloMessage = React.createClass({
  render: function() {
    return (
      <DefaultLayout title={this.props.title}>
        <div>Hello {this.props.name}</div>
      </DefaultLayout>
    );
  }
});

module.exports = HelloMessage;
```


## Questions

### What about partials & includes?

These ideas don't really apply. But since they are familiar ideas to people coming from more traditional "templating" solutions, let's address it. Most of these can be solved by packaging up another component that encapsulates that piece of functionality.

### What about view helpers?

I know you're used to registering helpers with your view helper (`hbs.registerHelper('something', ...))`) and operating on strings. But you don't need to do that here.

* Many helpers can be turned into components. Then you can just require and use them in your view.
* You have access to everything else in JS. If you want to do some date formatting, you can `require('moment')` and use directly in your view. You can bundle up other helpers as you please.

### Where does my data come from?

All "locals" are exposed to your view in `this.props`. These should work identically to other view engines, with the exception of how they are exposed. Using `this.props` follows the pattern of passing data into a React component, which is why we do it that way. Remember, as with other engines, rendering is synchronous. If you have database access or other async operations, they should be done in your routes.


## Caveats

* I'm saying it again to avoid confusion: this does not do anything with React in the browser. This is *only* a solution for server-side rendering.
* This currently uses `require` to access your views. This means that contents are cached for the lifetime of the server process. You need to restart your server when making changes to your views.
* React & JSX have their own rendering caveats. For example, inline `<script>`s and `<style>`s will need to use `dangerouslySetInnerHTML={{__html: 'script content'}}`.
* A doctype is automatically included for you since it's not possible to specify one in JSX. `<!doctype html>` is prepended to your components' output.

## Author

* Nicolai Gheorghiev, gheorghievn@gmail.com

[express]: http://expressjs.com/
[react]: http://facebook.github.io/react/
[jade]: http://jade-lang.com/
[ejs]: http://embeddedjs.com/
[hbs]: https://github.com/barc/express-hbs
