<script>
  import { TasksCollection } from '../api/TasksCollection';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';

  let getTasks;
  $m: getTasks = TasksCollection.find({}, { sort: { createdAt: -1 } }).fetchAsync()
</script>


<div class="app">
    <header>
        <div class="app-bar">
            <div class="app-header">
                <h1>📝️ To Do List</h1>
            </div>
        </div>
    </header>

    <div class="main">
        <TaskForm />
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
