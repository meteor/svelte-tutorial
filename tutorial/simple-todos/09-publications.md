---
title: '9: Publications'
---

Now that we have moved all of our app's sensitive code into methods, we need to learn about the other half of Meteor's security story. Until now, we have worked assuming the entire database is present on the client, meaning if we call `TasksCollection.find()` we will get every task in the collection. That’s not good if our application users want to store privacy-sensitive data. We need to control which data Meteor sends to the client-side database.

## 9.1: autopublish

Just like with `insecure` in the last step, all new Meteor apps start with the `autopublish` package, which automatically synchronizes all the database contents to the client. Remove it by using the command line below:

```
meteor remove autopublish
```

When the app refreshes, the task list will be empty. Without the `autopublish` package, we will have to specify explicitly what the server sends to the client. The functions in Meteor that do this are `Meteor.publish` and `Meteor.subscribe`:

- `Meteor.publish`: allows the data to be published from the server to the client;
- `Meteor.subscribe`: allows the client code to ask for data to the client.

## 9.2: Tasks Publication

You need to add first a publication to your server, this publication should publish all the tasks from the authenticated user. As in the `Methods` you can also use `this.userId` in publication functions to get the authenticated `userId`.

Create a new file called `tasksPublications.js` in the `api` folder.

`imports/api/tasksPublications.js`

```js
import { Meteor } from 'meteor/meteor';
import { TasksCollection } from '/imports/db/TasksCollection';

Meteor.publish('tasks', function publishTasks() {
  return TasksCollection.find({ userId: this.userId });
});
```

As you are using `this` inside this function you should not use `arrow function` (`=>`) as the arrow function does not provide a context for `this`, you need to use the function in the traditional way, using the `function` keyword.

The last part is to make sure your server is registering this publication, you can do this by importing the file, to force the evaluation in the `server/main.js`.

`server/main.js`

```js
import { Meteor } from 'meteor/meteor';
import { Accounts } from 'meteor/accounts-base';
import { TasksCollection } from '/imports/db/TasksCollection';
import '/imports/api/tasksMethods';
import '/imports/api/tasksPublications';
```

## 9.3: Tasks Subscription

From here, we can subscribe to that publication in the client.

As we want to receive changes from this publication we are going to `subscribe` to it inside the `$m`. Normally you could do this using a `useTracker`, but for us it's not really necessary as we're using the `zodern:melte` compiler.

`imports/ui/App.svelte`

```sveltehtml
<script>
    ..
    let isLoading = true;
    const handler = Meteor.subscribe('tasks');

    $m: {
        user = Meteor.user();

        if (user) {

            isLoading = !handler.ready();

            const userFilter = { userId: user._id };
            const pendingOnlyFilter = { ...hideCompletedFilter, ...userFilter };


            getTasks = user
                    ? TasksCollection.find(
                            hideCompleted ? pendingOnlyFilter : userFilter,
                            { sort: { createdAt: -1 } }
                    ).fetchAsync()
                    : [];

            getCount = user
                    ? TasksCollection.find(pendingOnlyFilter).countAsync()
                    : 0;
        }
    }
    const pendingTitle = (count) => `${count ? ` (${count})` : ''}`;
    ..
</script>
..
```

## 9.4: Loading state

You should also add a loading state for your app, that means, while the subscription data is not ready you should inform this to your user. To discover if the subscription is ready or not you should get the return of the `subscribe` call, it is an object with the subscription state including the `ready` function that will return a `boolean`.

`imports/ui/App.svelte`

```html
..
        <div class="filter">
            <button on:click={() => setHideCompleted(!hideCompleted)}>
            {hideCompleted ? 'Show All' : 'Hide Completed'}
            </button>
        </div>
        
        {#if isLoading}
            <div class="loading">loading...</div>
        {/if}
..
```

Let's style this loading a little as well:

`client/main.css`

```css
.loading {
  display: flex;
  flex-direction: column;
  height: 100%;

  justify-content: center;
  align-items: center;

  font-weight: bold;
}
```

Once you have done this, all the tasks will reappear.

Calling `Meteor.publish` on the server registers a publication named `tasks`. When `Meteor.subscribe` is called on the client with the publication name, the client subscribes to all the data from that publication, which in this case is all the tasks in the database for the authenticated user. 

## 9.5: Check User Permission

Only the owner of a task should be able to change certain things. You should change your methods to check if the user that is authenticated is the same user that created the tasks.

`imports/api/tasksMethods.js`

```js
..
    async 'tasks.remove'(taskId) {
      check(taskId, String);
    
      if (!this.userId) {
        throw new Meteor.Error('Not authorized.');
      }
    
      const task = await TasksCollection.findOneAsync({ _id: taskId, userId: this.userId });
    
      if (!task) {
        throw new Meteor.Error('Access denied.');
      }
    
      await TasksCollection.remove(taskId);
    },
    
    async 'tasks.setIsChecked'(taskId, isChecked) {
      check(taskId, String);
      check(isChecked, Boolean);
    
      if (!this.userId) {
        throw new Meteor.Error('Not authorized.');
      }
    
      const task = await TasksCollection.findOneAsync({ _id: taskId, userId: this.userId });
    
      if (!task) {
        throw new Meteor.Error('Access denied.');
      }
    
      await TasksCollection.updateAsync(taskId, {
        $set: {
          isChecked,
        },
      });
    },
..
```

Why this is important if we are not returning tasks from other users in the client?

This is important because anyone can call Meteor `Methods` using the browser `console`. You can test this using your DevTools console tab and then type and hit enter: `Meteor.call('tasks.remove', 'xtPTsNECC3KPuMnDu');`. If you remove the validation from your remove Method and you pass one valid task `_id` from your database you will be able to remove it.

> Review: you can check how your code should be at the end of this step [here](https://github.com/meteor/svelte-tutorial/tree/master/src/simple-todos/step09) 

In the next step we are going to run the app on mobile environment as a Native app.
