<script>
  import Task from './Task.svelte';
  import { TasksCollection } from '../api/TasksCollection';

  let getTasks;
  $m: getTasks = TasksCollection.find({}).fetchAsync()
</script>


<div class="container">
    <header>
        <h1>Todo List</h1>
    </header>

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
