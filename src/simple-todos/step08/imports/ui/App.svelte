<script>
  import { TasksCollection } from '../db/TasksCollection';
  import { Meteor } from 'meteor/meteor';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';
  import LoginForm from './LoginForm.svelte';

  let hideCompleted = false;

  const hideCompletedFilter = { isChecked: { $ne: true } };


  let getTasks;
  let getCount;
  let user = null;

  $m: {
    user = Meteor.user();

    const userFilter = user ? { userId: user._id } : {};
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
  const pendingTitle = (count) => `${count ? ` (${count})` : ''}`;
  const setHideCompleted = value => {
    hideCompleted = value;
  };
  const logout = () => Meteor.logout();
</script>


<div class="app">
    <header>
        <div class="app-bar">
            <div class="app-header">
                {#await getCount}
                    <h1>📝️ To Do List (...)</h1>
                {:then count}
                    <h1>📝️ To Do List {pendingTitle(count)}</h1>
                {/await}
            </div>
        </div>
    </header>

    <div class="main">
        {#if user}
            <div class="user" on:click={logout}>
                {user.username} 🚪
            </div>

            <TaskForm user={user}/>

            <div class="filter">
                <button on:click={() => setHideCompleted(!hideCompleted)}>
                    {hideCompleted ? 'Show All' : 'Hide Completed'}
                </button>
            </div>
            {#await getTasks}
                <p>Loading...</p>
            {:then tasks}
                <ul class="tasks">
                    {#each tasks as task (task._id)}
                        <Task task={task}/>
                    {/each}
                </ul>
            {:catch error}
                <p>{error.message}</p>
            {/await}
        {:else}
            <LoginForm />
        {/if}
    </div>
</div>
