<script>
    import { TasksCollection } from '../api/TasksCollection';

    export let user = null;
    let newTask = '';

    const handleSubmit = async () => {
      // Insert a task into the collection
      await TasksCollection.insertAsync({
        text: newTask,
        createdAt: new Date(), // current time
        userId: user._id,
      });

      // Clear form
      newTask = '';
    }
</script>

<form class="task-form" on:submit|preventDefault={handleSubmit}>
    <input
            type="text"
            name="text"
            placeholder="Type to add new tasks"
            bind:value={newTask}
    />
    <button type="submit">Add Task</button>
</form>
