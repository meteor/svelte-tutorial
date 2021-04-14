<script>
  import { TasksCollection } from '../api/TasksCollection';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';

  $m: tasks = TasksCollection.find({}).fetch()
</script>


<div class="container">
    <header>
      <h1>Todo List</h1>
    </header>

    <TaskForm />

    <ul>
      {#each tasks as task}
          <Task key={task._id} task={task} />
      {/each}
    </ul>
</div>
