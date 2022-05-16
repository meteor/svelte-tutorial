---
title: "3: Forms and Events"
---

All apps need to allow the user to perform some types of interaction with the data that is stored. In our case, the first type of interaction we'll need is a new tasks. Without it, our To-Do app wouldn't be very helpful.  

One of the main ways in which a user can insert or edit data in a website is through forms. In most cases it is a good idea to use the `<form>` tag since it gives semantic meaning to the elements inside it.

## 3.1: Create Task Form

First we need to create a simple form component to encapsulate our logic.

Create a `form` element inside a new file called `TaskForm.svelte` file, and add an input field and a button:

`imports/ui/TaskForm.svelte`
```html
<form class="task-form">
    <input type="text" name="text" placeholder="Type to add new tasks" />
    <button type="submit">Add Task</button>
</form>

```

## 3.2: Update the App

Then we can simply add this new form to our `App.svelte` component by first importing it inside the `<script/>` tag and then using it inside our `<div />`:

`imports/ui/App.svelte`

```html
<script>
    import { TasksCollection } from '../api/TasksCollection';
    import Task from './Task.svelte';
    import TaskForm from './TaskForm.svelte';
    ..
</script>


<div class="container">
    <header>
        <h1>Todo List</h1>
    </header>

    <TaskForm />
    
    ..
    
</div>


```

## 3.3: Update the Stylesheet

You also can style it as you wish. For now, we only need some margin at the top so the form doesn't seem off the mark. Add the CSS class `.task-form`, this needs to be the same name in your `class` attribute in the form component.

`client/main.css`
```css
.task-form {
  margin-top: 1rem;
}
```

## 3.4: Add Submit Listener

Now we need to add a listener to the `submit` event on the form and create the `handleSubmit` function that will, in fact, insert our task:

`imports/ui/TaskForm.svelte`

```html
<script>
    import { TasksCollection } from '../api/TasksCollection';

    let newTask = '';

    const handleSubmit = () => {
        // Insert a task into the collection
        TasksCollection.insert({
            text: newTask,
            createdAt: new Date(), // current time
        });

        // Clear form
        newTask = '';
    }
</script>

<form class="task-form" on:submit|preventDefault={handleSubmit}>
    <input
            type="text"
            name="text"
            placeholder="Type to add new tasks"
            bind:value={newTask}
    />
    <button type="submit">Add Task</button>
</form>
```

This form's input tag will have a `bind:value` attribute added to it and this will bind the input's value to the `newTask` property.

Next, the `handleSubmit` method will be added to the `App` component's script section. In order to execute the `handleSubmit` function on our form's submit event we will add the [on:submit|preventDefault](https://svelte.dev/docs#on_element_event) attribute to the form tag:

Also, insert a date `createdAt` in your `task` document, so you know when each task was created.

## 3.5: Show Newest Tasks First

All that is left now is to make one final change: we need to show the newest tasks first. We can accomplish this quite quickly by sorting our [Mongo](https://guide.meteor.com/collections.html#mongo-collections) query.

`imports/ui/App.svelte`

```html
<script>
  ..
  $m: tasks = TasksCollection.find({}, { sort: { createdAt: -1 } }).fetch()
</script>
..
```

Now your app should look like this:

<img width="200px" src="/simple-todos/assets/step03-form-new-task.png"/>

<img width="200px" src="/simple-todos/assets/step03-new-task-on-list.png"/>

> Review: you can check how your code should be at the end of this step [here](https://github.com/meteor/svelte-tutorial/tree/master/src/simple-todos/step03) 

In the next step we are going to update your tasks state and provide a way for users to remove tasks.
