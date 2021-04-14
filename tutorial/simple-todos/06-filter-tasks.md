---
title: "6: Filter tasks"
---

In this step, you will filter your tasks by status and show the quantity of pending tasks.

## 6.1: Adding button to hide tasks

First you are going to add a button to show or hide the completed tasks from the list.

With Svelte this will be a pretty simple task as we don't have to do much to keep state to the button. Let's add our button:

`imports/ui/App.svelte`

```html
<script>
    ..
    let hideCompleted = false;
    ..
    const setHideCompleted = value =>  {
        hideCompleted = value;
    }
</script>

<div class="app">
..
    <div class="main">
        <TaskForm />
        <div class="filter">
            <button on:click={() => setHideCompleted(!hideCompleted)}>
                {hideCompleted ? 'Show All' : 'Hide Completed'}
            </button>
        </div>
        ..
    </div>
</div>
```

## 6.2: Button style

You should add some style to your button so it doesn't look gray and without good contrast. You can use the styles below as a reference:

`client/main.css`

```css
.filter {
  display: flex;
  justify-content: center;
}

.filter > button {
  background-color: #62807e;
}
```

## 6.3: Filter Tasks

Now, if the user wants to see only pending tasks you can add a filter to your selector in the Mini Mongo query, you want to get all the tasks that are not `isChecked` true.

`imports/ui/App.svelte`

```html
<script>
    ..
    const hideCompletedFilter = { isChecked: { $ne: true } };

    $m:tasks = TasksCollection.find(
      hideCompleted ? hideCompletedFilter : {}, { sort: { createdAt: -1 } }
    ).fetch()
    ..
</script>
..
```

## 6.4: Meteor Dev Tools Extension

You can install an extension to visualize the data in your Mini Mongo.

[Meteor DevTools Evolved](https://chrome.google.com/webstore/detail/meteor-devtools-evolved/ibniinmoafhgbifjojidlagmggecmpgf) will help you to debug your app as you can see what data is on Mini Mongo. 

<img width="800px" src="/simple-todos/assets/step06-extension.png"/>

You can also see all the messages that Meteor is sending and receiving from the server, this is useful for you to learn more about how Meteor works.

<img width="800px" src="/simple-todos/assets/step06-ddp-messages.png"/>

Install it in your Google Chrome browser using this [link](https://chrome.google.com/webstore/detail/meteor-devtools-evolved/ibniinmoafhgbifjojidlagmggecmpgf).

## 6.5: Pending tasks

Update the App component in order to show the number of pending tasks in the app bar.

You should avoid adding zero to your app bar when there are not pending tasks.

`imports/ui/App.svelte`
```html
<script>
    import { TasksCollection } from '../api/TasksCollection';
    import { useTracker } from 'meteor/rdb:svelte-meteor-data';
    ..
    let incompleteCount;
    let pendingTasksTitle = '';

    $: {
        incompleteCount = useTracker(() => TasksCollection.find(hideCompletedFilter).count());
        pendingTasksTitle = `${
                $incompleteCount ? ` (${$incompleteCount})` : ''
        }`;
    }
    ..
</script>

<div class="app">
    <header>
        <div class="app-bar">
            <div class="app-header">
                <h1>📝️ To Do List {pendingTasksTitle}</h1>
            </div>
        </div>
    </header>
    ..
</div>
```

Your app should look like this:

<img width="200px" src="/simple-todos/assets/step06-all.png"/>
<img width="200px" src="/simple-todos/assets/step06-filtered.png"/>

> Review: you can check how your code should be in the end of this step [here](https://github.com/meteor/svelte-tutorial/tree/master/src/simple-todos/step06) 

In the next step we are going to include user access in your app.
