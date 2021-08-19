---
title: "1: Creating the app"
---

## 1.1: Install Meteor
First we need to install Meteor.

If you running on OSX or Linux run this command in your terminal:
```shell
curl https://install.meteor.com/ | sh
```

If you are on Windows:
```shell
npm install --global meteor
```

> You can check more details about Meteor installation [here](https://www.meteor.com/install)

## 1.2: Create Meteor Project

The easiest way to set up Meteor with Svelte is by using the command `meteor create` with the option `--svelte` and your project name:

```
meteor create --svelte simple-todos-svelte
```

After this, Meteor will create all the necessary files for you. 

The files located in the `client` directory are setting up your client side (web), you can see for example `client/main.js` which is where your app begins on the client side.

Also, check the `server` directory where Meteor is setting up the server side (Node.js), you can see the `server/main.js`. If Meteor, you don't need to install MongoDB as Meteor provides an embedded version of it ready for you to use.

You can define which are the main files (client and server) on your `package.json` like this:

```json
{
  ..,
  "meteor": {
    "mainModule": {
      "client": "client/main.js",
      "server": "server/main.js"
    }
  }
} 
```

You can now run your Meteor app using: 

```shell
cd simple-todos-svelte
meteor run
```

Don't worry, Meteor will keep your app in sync with all your changes from now on.

Take a quick look in all the files created by Meteor, you don't need to understand them now, but it's good to know where they are.

Here is a small summary of some files created:

```
client/main.js        # a JavaScript entry point loaded on the client
client/main.html      # an HTML file that defines view templates
client/main.css       # a CSS file to define your app's styles
server/main.js        # a JavaScript entry point loaded on the server
test/main.js          # a JavaScript entry point when running tests
package.json          # a control file for installing npm packages
package-lock.json     # describes the npm dependency tree
node_modules/         # packages installed by npm
.meteor/              # internal Meteor files
.gitignore            # a control file for git
```

## 1.3: Create Task Component

To start working on our todo list app, let's replace the code of the default starter app with the code below. From there, we'll talk about what it does.

First, let's change the `<div/>` inside our file `App.svelte` inside the folder `imports/ui`:

`imports/ui/App.svelte`

```html
..

<div class="container">
    <header>
        <h1>Todo List</h1>
    </header>

    <ul>
        {#each getTasks() as task (task._id)}
            <Task task={task} />
        {/each}
    </ul>
</div>
```

Also, you can create the `<Task />` component. Go ahead and create a new file called `Task.svelte` inside the import folder:

`imports/ui/Task.svelte`

```html
<script>
    export let task;
</script>

<li> { task.text }</li>
```

Now we need the data to render on this page. 

## 1.4: Create Sample Tasks

As you are not connecting to your server and your database yet, let's define some sample data which will be used to render a list of tasks. It will be an array of list items, and you can call it `tasks`. Go ahead and change the `<script/>` tag inside the `App.svelte` file to the following code:

`imports/ui/App.svelte`

```html
<script>
    import Task from './Task.svelte';

    const getTasks = () => ([
        { _id: 'task_1', text: 'This is task 1' },
        { _id: 'task_2', text: 'This is task 2' },
        { _id: 'task_3', text: 'This is task 3' },
    ])
</script>

...
```

You can put anything as your `text` property on each task. Be creative!

In Svelte, single file components are created with the `.svelte` file extension and are comprised of three sections, the script section, the markup (HTML) section, and the style section. Within the script section you will write Javascript that runs when the component instance is created. The Svelte component format is fully explained in the [Svelte Guide](https://svelte.dev/docs#Component_format).

You can read more about how to structure your code in the [Application Structure article](https://guide.meteor.com/structure.html) of the Meteor Guide.

## 1.5 Mobile look

Let's see how your app is looking on Mobile. You can simulate a mobile environment by `right clicking` your app in the browser (we are assuming you are using Google Chrome, as it is the most popular browser) and then `inspect`, this will open a new window inside your browser called `Dev Tools`. In the `Dev Tools` you have a small icon showing a Mobile device and a Tablet:

<img width="500px" src="/simple-todos/assets/step01-dev-tools-mobile-toggle.png"/>

Click on it and then select the phone that you want to simulate and in the top nav bar.

> You can also check your app in your personal cellphone. To do so, connect to your App using your local IP in the navigation browser of your mobile browser.
>
> This command should print your local IP for you on Unix systems
`ifconfig | grep "inet " | grep -Fv 127.0.0.1 | awk '{print $2}'`

You should see the following:

<img width="200px" src="/simple-todos/assets/step01-mobile-without-meta-tags.png"/>

As you can see, everything is small, as we are not adjusting the view port for mobile devices. You can fix this and other similar issues by adding these lines to your `client/main.html` file, inside the `head` tag, after the `title`.

`client/main.html`
```html
  <meta charset="utf-8"/>
  <meta http-equiv="x-ua-compatible" content="ie=edge"/>
  <meta
      name="viewport"
      content="width=device-width, height=device-height, viewport-fit=cover, initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no"
  />
  <meta name="mobile-web-app-capable" content="yes"/>
  <meta name="apple-mobile-web-app-capable" content="yes"/>
```

Now your app should look like this:

<img width="200px" src="/simple-todos/assets/step01-mobile-with-meta-tags.png"/>

## 1.6 Hot Module Replacement

Meteor by default when using Svelte is already adding for you a package called `hot-module-replacement`. This package updates the javascript modules in a running app that were modified during a rebuild. Reduces the feedback cycle while developing, so you can view and test changes quicker (it even updates the app before the build has finished).

You should also add the package `dev-error-overlay` at this point, so you can see the errors in your web browser.

```shell
meteor add dev-error-overlay
```

You can try to make some mistakes and then you are going to see the errors in the browser and not only in the console.

> Review: you can check how your code should look in the end of this step [here](https://github.com/meteor/svelte-tutorial/tree/master/src/simple-todos/step01) 

In the next step we are going to work with our MongoDB database to be able to store our tasks.
