<script>
  import Task from './Task.svelte';
  import { TasksCollection } from '../api/TasksCollection';

  $m: tasks = TasksCollection.find({}).fetch()
</script>


<div class="container">
    <header>
      <h1>Todo List</h1>
    </header>

    <ul>
      {#each tasks as task (task._id)}
          <Task task={task} />
      {/each}
    </ul>
</div>
