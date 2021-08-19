<script>
  import Task from './Task.svelte';

  const getTasks = () => ([
    { _id: 'task_1', text: 'This is task 1' },
    { _id: 'task_2', text: 'This is task 2' },
    { _id: 'task_3', text: 'This is task 3' },
  ])
</script>


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
