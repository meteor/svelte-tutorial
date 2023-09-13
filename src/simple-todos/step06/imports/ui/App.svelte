<script>
  import { TasksCollection } from '../api/TasksCollection';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';

  let hideCompleted = false;

  const hideCompletedFilter = { isChecked: { $ne: true } };
  let getTasks;
  let getCount;

  $m: {
    getTasks = TasksCollection.find(hideCompleted ? hideCompletedFilter : {}, {
      sort: { createdAt: -1 },
    }).fetchAsync();

    getCount = TasksCollection.find(hideCompletedFilter).countAsync();
  }

  const pendingTitle = (count) => `${count ? ` (${count})` : ''}`;

  const setHideCompleted = (value) => {
    hideCompleted = value;
  };
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
    <TaskForm />
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
  </div>
</div>
