<script>
  import { TasksCollection } from '../db/TasksCollection';
  import { Meteor } from 'meteor/meteor';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';
  import LoginForm from './LoginForm.svelte';

  let hideCompleted = false;

  const hideCompletedFilter = { isChecked: { $ne: true } };


  let incompleteCount;
  let pendingTasksTitle = '';
  let tasks = [];
  let user = null;

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

    pendingTasksTitle = `${
      incompleteCount ? ` (${incompleteCount})` : ''
    }`;
  }

  const setHideCompleted = value => {
    hideCompleted = value;
  };
  const logout = () => Meteor.logout();
</script>


<div class="app">
    <header>
        <div class="app-bar">
            <div class="app-header">
                <h1>📝️ To Do List {pendingTasksTitle}</h1>
            </div>
        </div>
    </header>

    <div class="main">
        {#if user}
            <div class="user" on:click={logout}>
                {user.username} 🚪
            </div>

            <TaskForm />

            <div class="filter">
                <button on:click={() => setHideCompleted(!hideCompleted)}>
                    {hideCompleted ? 'Show All' : 'Hide Completed'}
                </button>
            </div>
            <ul class="tasks">
              {#each tasks as task (task._id)}
                  <Task task={task} />
              {/each}
            </ul>
        {:else}
            <LoginForm />
        {/if}
    </div>
</div>
