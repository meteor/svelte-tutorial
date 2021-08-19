<script>
  import { TasksCollection } from '../api/TasksCollection';
  import Task from './Task.svelte';
  import TaskForm from './TaskForm.svelte';

  $m: tasks = TasksCollection.find({}, { sort: { createdAt: -1 } }).fetch()
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

        <ul class="tasks">
          {#each tasks as task (task._id)}
              <Task task={task} />
          {/each}
        </ul>
    </div>
</div>
