<script>
  import { TasksCollection } from '../api/TasksCollection';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';

  let getTasks;
  $m: getTasks = TasksCollection.find({}, { sort: { createdAt: -1 } }).fetchAsync()
</script>


<div class="container">
    <header>
      <h1>Todo List</h1>
    </header>

    <TaskForm />


    {#await getTasks}
        <p>Loading...</p>
    {:then tasks}
        <ul>
            {#each tasks as task (task._id)}
                <Task task={task}/>
            {/each}
        </ul>
    {:catch error}
        <p>{error.message}</p>
    {/await}
</div>
