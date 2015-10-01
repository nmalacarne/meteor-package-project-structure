I recently started working on a real-time web application using [Meteor](https://www.meteor.com/). It took a bit to get the hang of exactly how Meteor works compared to other conventional [RESTful](https://en.wikipedia.org/wiki/Representational_state_transfer) frameworks out there, but now that I am somewhat adjusted to Meteor's 'data over the wire' style I am really enjoying working with it.

There are, however, some practices that I have learned about Meteor in my brief time (as of this post) working with it. The most glaring issue I had with my initial experimenting was how to structure my project in a way that was extensible and easily maintainable.

### Default Project Structure

I started my project using the 'default' project structure for Meteor, a structure which looked like the following:

```
.
|-- client/
|-- lib/
|-- public/
|-- server/
```

These folder names have specific meanings in Meteor. For example, all files located in the `clients/` directory will be available on the client while all the files located in the `server/` directory will be available on the server. Files not in either the `client/` or `server/` directories are available on both the client and the server. Any files in the `lib/` directory are not only available on both the client and the server, but they are also loaded before any other files in the same directory. Files prefixed with `main.*` are loaded last, and all other files are loaded in alphabetical order.

As time went on, I found that this structure was becoming increasingly cumbersome to work with (the project was not even that big at this point). So, with some help from Google, I stumbled upon a way to structure your projects via [smart packages](http://www.matb33.me/2013/09/05/meteor-project-structure.html).

### Structure via Smart Packages

Simply put, smart packages are a way to break your application up into testable, easily maintainable modules. In fact, if you are using Meteor, you are already using smart packages (such as the default `insecure` and `autopublish` packages that you should always remove before deploying). Actually, anytime that you use `meteor add <package-name>` to add a dependency to your project, you are using smart packages.

The neat thing about `meteor add <package-name>` is the fact that it will check for a `packages/` directory in your project before looking for the package in the [Atmosphere](https://atmospherejs.com/). This allows us to create packages that are *local* to our project. 

So, structuring a Meteor application via smart packages is cool and all, but exactly how do we do it?

### Creating a simple project

>Note: this walk-through uses Ubuntu 15.04

If you need to install Meteor, check out the [install guide](https://www.meteor.com/install).

The first thing we need to do is create a Meteor project. Go ahead and `$ cd` into whatever directory you want to use to house your Meteor projects and run:

`$ meteor create app`

We will be naming our project `app` for the sake of brevity, so you can substitute your project name throughout this guide whenever you see `app` being used.

If we `$ cd app && ls -la` we will see the current project structure:

```
app/
|-- .meteor/
|-- app.html
|-- app.js
|-- app.css
```  

We are going to get rid of the three default project files via `rm app.*` since we will not be using them for this project.

Next, we will create a main package for our project (e.g. index):

```
meteor create --package app-main
```

The `--package` flag lets Metoer know that we want to create a package and not an application, and since our current directory is already a Meteor application, it will know that we want the package to be *local* to this application. Our project directory should now look like this:

```
app/
|-- packages/
  |-- app-main/
    |-- app-main.js
    |-- app-main-tests.js
    |-- package.js
    |-- README.md
```

Take note of the [`package.js`](http://docs.meteor.com/#/full/packagedefinition) file because it is the heart of our package. Within it, we declare information about the package as well as dependencies, files that our package uses, variables that our package exports, where each file should be run (client/server), etc.

If you open `package.js` in your text editor you will see this:

```
Package.describe({
  name: 'app-main',
  version: '0.0.1',
  // Brief, one-line summary of the package.
  summary: '',
  // URL to the Git repository containing the source code for this package.
  git: '',
  // By default, Meteor will default to using README.md for documentation.
  // To avoid submitting documentation, set this field to null.
  documentation: 'README.md'
});

Package.onUse(function(api) {
  api.versionsFrom('1.2.0.2');
  api.use('ecmascript');
  api.addFiles('app-main.js');
});

Package.onTest(function(api) {
  api.use('ecmascript');
  api.use('tinytest');
  api.use('app-main');
  api.addFiles('app-main-tests.js');
});
```

The [`Package.onUse`](http://docs.meteor.com/#/full/pack_onUse) method is where we declare what our package dependencies are ([`api.use`](http://docs.meteor.com/#/full/pack_use)), and what files (and in what order those files are loaded) our package uses ([`api.addFiles`](http://docs.meteor.com/#/full/pack_addFiles)). [`Package.onTest`](http://docs.meteor.com/#/full/Package-onTest) is where we set up our package for unit testing, but this is out-of-scope for this walk-through. If you would like to read more about unit testing in Meteor, check out [this](https://www.eventedmind.com/feed/meteor-testing-packages-with-tinytest) great video by [Evented Mind](https://www.eventedmind.com/).

Now that we have our `app-main` package, we can create a template via:

`$ cd packages/app-main && touch app-main.html`

Add the following HTML for now:

```
<body>
  <h1>Hello world!</h1>
</body>
```

Next, we need to add our template to our package:

```
Package.describe({
  name: 'app-main',
  version: '0.0.1',
  // Brief, one-line summary of the package.
  summary: '',
  // URL to the Git repository containing the source code for this package.
  git: '',
  // By default, Meteor will default to using README.md for documentation.
  // To avoid submitting documentation, set this field to null.
  documentation: 'README.md'
});

Package.onUse(function(api) {
  api.versionsFrom('1.2.0.2');
  api.use('ecmascript');
  api.use('templating');
  api.addFiles('app-main.js');
  api.addFiles('app-main.html');
});

Package.onTest(function(api) {
  api.use('ecmascript');
  api.use('tinytest');
  api.use('app-main');
  api.addFiles('app-main-tests.js');
});
```

Notice that we have added `app-main.html` to our package (available on both client/server since we did not specify which). We also had to declare a new dependency for our package: `templating`. Without this dependency, our package does not know how to render templates.

If we `$ cd ../..` we will end up in our project's root directory. Go ahead and run `$ meteor` to start your application if it is not already running. We should see `Hello world!` in big bold text, right? Not exactly. Since all of our templates (`body` in `app-main`) are in a package, we need to add that package to our Meteor application in the same way we would add a third-party package to our project:

`$ meteor add app-main`

Again, Meteor will look for a `packages` folder before it looks anywhere else. If it finds a *local* package by the name `app-main` in our `packages` directory, it will use that. Otherwise, it will try to pull a package named `app-name` from [Atmosphere](https://atmospherejs.com/).

If we check our page again (running by default at `localhost:3000`), we can see our beautiful `Hello world!` on the page.

### Summary

As we can see, smart packages are a really neat way to structure a Meteor applications. Not only can our *local* packages depend on *generic* packages such as `templating`, our *local* packages can also depend on other *local* packages! 

Another great benefit to organizing your project in this manner is that if you end up creating a *local* package that you think might help out another Meteor developer, you are only one step away from converting that package into a *generic* package and hosting it on [Atmosphere](https://atmospherejs.com/) for others to use.

### References
[Smart Package Structure](http://www.matb33.me/2013/09/05/meteor-project-structure.html) - matb33

[Meteor Package Testing](https://www.eventedmind.com/feed/meteor-testing-packages-with-tinytest) - Evented Mind

