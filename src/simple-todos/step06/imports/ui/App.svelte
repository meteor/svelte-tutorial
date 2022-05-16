<script>
  import { TasksCollection } from '../api/TasksCollection';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';

  let hideCompleted = false;

  const hideCompletedFilter = { isChecked: { $ne: true } };

  let incompleteCount;
  let pendingTasksTitle = '';
  let tasks = [];

  $m: {
    tasks = TasksCollection.find(hideCompleted ? hideCompletedFilter : {}, {
      sort: { createdAt: -1 },
    }).fetch();

    incompleteCount = TasksCollection.find(hideCompletedFilter).count();

    pendingTasksTitle = `${incompleteCount ? ` (${incompleteCount})` : ''}`;
  }

  const setHideCompleted = (value) => {
    hideCompleted = value;
  };
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
  </div>
</div>
