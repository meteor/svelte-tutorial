---
title: '7: Adding User Accounts'
---

## 7.1: Password Authentication

Meteor already comes with a basic authentication and account management system out-of-the box, so you only need to add the `accounts-password` to enable username and password authentication:

```
meteor add accounts-password
```

> There are many more authentication methods supported. You can read more about the accounts system [here](https://docs.meteor.com/api/accounts.html).

We also recommend you to install `bcrypt` node module, otherwise you are going to see a warning saying that you are using pure-Javascript implementation of it.

```
meteor npm install --save bcrypt
```

> You should always use `meteor npm` instead of only `npm` so you always use the `npm` version pinned by Meteor, this helps you to avoid problems due to different versions of npm installing different modules.

## 7.2: Create User Account

Now you can create a default user for our app, we are going to use `meteorite` as username, we just create a new user on server startup if we didn't find it in the database.

`server/main.js`

```js
import { Meteor } from 'meteor/meteor';
import { Accounts } from 'meteor/accounts-base';
import { TasksCollection } from '/imports/api/TasksCollection';

..

const SEED_USERNAME = 'meteorite';
const SEED_PASSWORD = 'password';

Meteor.startup(() => {
  if (!Accounts.findUserByUsername(SEED_USERNAME)) {
    Accounts.createUser({
      username: SEED_USERNAME,
      password: SEED_PASSWORD,
    });
  }
  ..
});
```

You should not see anything different in your app UI yet.

## 7.3: Login Form

You need to provide a way for the users to input the credentials and authenticate, for that we need a form.

Create a new file called `LoginForm.svelte` and add a form to it. You should use `Meteor.loginWithPassword(username, password);` to authenticate your user with the provided inputs.

`imports/ui/LoginForm.svelte`

```html
<script>
    import { Meteor } from 'meteor/meteor';

    let username = "";
    let password = "";

    const handleSubmit = () => {
        Meteor.loginWithPassword(username, password);
    }
</script>

<form class="login-form" on:submit|preventDefault={handleSubmit}>
    <div>
        <label htmlFor="username">Username</label>

        <input
                type="text"
                placeholder="Username"
                name="username"
                required
                bind:value={username}
        />
    </div>

    <div>
        <label htmlFor="password">Password</label>

        <input
                type="password"
                placeholder="Password"
                name="password"
                required
                bind:value={password}
        />
    </div>
    <div>
        <button type="submit">Log In</button>
    </div>
</form>

```

Ok, now you have a form, let's use it.

## 7.4: Require Authentication

Our app should only allow an authenticated user to access its task management features.

We can accomplish that rendering the `LoginForm` component when we don’t have an authenticated user, otherwise we return the form, filter, and list component.

You can get your authenticated user or null from `Meteor.user()`. Then you can verify if you have a logged user, if yes, render the app, otherwise, your render the `LoginForm`:

`imports/ui/App.svelte`

```html
<script>
    import { TasksCollection } from '../api/TasksCollection';
    import { Meteor } from 'meteor/meteor';
    ..
    let user = null;

    $m: {
        user = Meteor.user();
        ..
    }
</script>

<div class="app">
    ..
    <div class="main">
        {#if user}
            <TaskForm user={user}/>

            <div class="filter">
                <button on:click={() => setHideCompleted(!hideCompleted)}>
                {hideCompleted ? 'Show All' : 'Hide Completed'}
                </button>
            </div>
            <ul class="tasks">
                {#each tasks as task}
                <Task key={task._id} task={task} />
                {/each}
            </ul>
        {:else}
            <LoginForm />
        {/if}
    </div>
</div>
```

## 7.5: Login Form style

Ok, let's style the login form now:

`client/main.css`

```css
.login-form {
  display: flex;
  flex-direction: column;
  height: 100%;

  justify-content: center;
  align-items: center;
}

.login-form > div {
  margin: 8px;
}

.login-form > div > label {
  font-weight: bold;
}

.login-form > div  > input {
  flex-grow: 1;
  box-sizing: border-box;
  padding: 10px 6px;
  background: transparent;
  border: 1px solid #aaa;
  width: 100%;
  font-size: 1em;
  margin-right: 16px;
  margin-top: 4px;
}

.login-form > div > input:focus {
  outline: 0;
}

.login-form > div > button {
  background-color: #62807e;
}
```

Now your login form should be centralized and look beautiful.

## 7.6: Server startup

Every task should have an owner from now on. So go to your database, as you learn before, and remove all the tasks from there:

`db.tasks.remove({});`

Change your `server/main.js` to add the seed tasks using your `meteorite` user as owner.

Make sure you restart the server after this change so `Meteor.startup` block can run again. This is probably going to happen automatically as you make changes in the server side code.

`server/main.js`

```js
import { Meteor } from 'meteor/meteor';
import { Accounts } from 'meteor/accounts-base';
import { TasksCollection } from '/imports/api/TasksCollection';

const insertTask = (taskText, user) =>
  TasksCollection.insert({
    text: taskText,
    userId: user._id,
    createdAt: new Date(),
  });

const SEED_USERNAME = 'meteorite';
const SEED_PASSWORD = 'password';

Meteor.startup(() => {
  if (!Accounts.findUserByUsername(SEED_USERNAME)) {
    Accounts.createUser({
      username: SEED_USERNAME,
      password: SEED_PASSWORD,
    });
  }

  const user = Accounts.findUserByUsername(SEED_USERNAME);

  if (TasksCollection.find().count() === 0) {
    [
      'First Task',
      'Second Task',
      'Third Task',
      'Fourth Task',
      'Fifth Task',
      'Sixth Task',
      'Seventh Task',
    ].forEach(taskText => insertTask(taskText, user));
  }
});
```

See that we are using a new field called `userId` with our user `_id` field, we are also setting `createdAt` field.

## 7.7: Task owner

Now you can filter the tasks in the UI by the authenticated user. Use the user `_id` to add the field `userId` to your Mongo selector when getting the tasks from Mini Mongo.

`imports/ui/App.svelte`

```html
<script>
    ..
    $m: {
        user = Meteor.user();

        const userFilter = user ? { userId: user._id } : {};
        const pendingOnlyFilter = { ...hideCompletedFilter, ...userFilter };


        tasks = user
                ? TasksCollection.find(
                        hideCompleted ? pendingOnlyFilter : userFilter,
                        { sort: { createdAt: -1 } }
                ).fetch()
                : [];

        incompleteCount = user
                ? TasksCollection.find(pendingOnlyFilter).count()
                : 0;

        ..
    }
</script>

<div class="app">
    ..
    <div class="main">
        ..
            <ul class="tasks">
                {#each tasks as task}
                    <Task key={task._id} task={task} />
                {/each}
            </ul>
        ..
    </div>
</div>
```

Also update the `insert` call to include the field `userId` in the `TaskForm`. You should pass the user from the `App` component to the `TaskForm`.

`imports/ui/TaskForm.svelte`
```html
<script>
    ..
    export let user = null;
    ..
    const handleSubmit = () => {
        // Insert a task into the collection
        TasksCollection.insert({
            text: newTask,
            createdAt: new Date(), // current time
            userId: user._id,
        });
        ..
    }
</script>
```

## 7.8: Log out

We also can better organize our tasks by showing the username of the owner below our app bar. Let's add a new `div` where the user can click and log out from the app:


`imports/ui/App.svelte`

```html
..
<div class="app">
    ..
    <div class="main">
        {#if user}
            <div class="user" on:click={logout}>
                {user.username} 🚪
            </div>
..
```

Remember to style your user name as well.

`client/main.css`

```css
.user {
  display: flex;

  align-self: flex-end;

  margin: 8px 16px 0;
  font-weight: bold;
}
```

Phew! You have done quite a lot in this step. Authenticated the user, set the user in the tasks and provided a way for the user to log out.

Your app should now look like this:

<img width="200px" src="/simple-todos/assets/step07-login.png"/>
<img width="200px" src="/simple-todos/assets/step07-logout.png"/>

> Review: you can check how your code should be in the end of this step [here](https://github.com/meteor/svelte-tutorial/tree/master/src/simple-todos/step07) 

In the next step we are going to start using Methods to only change the data after checking some conditions.
