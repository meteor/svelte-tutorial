<script>
  import { TasksCollection } from '../api/TasksCollection';

  export let task;

  const toggleChecked = async () => {
    // Set the checked property to the opposite of its current value
    await TasksCollection.updateAsync(task._id, {
      $set: { isChecked: !task.isChecked }
    });
  };

  const deleteThisTask = async () => {
    await TasksCollection.removeAsync(task._id);
  };
</script>

<li>
    <input
            type="checkbox"
           readonly
           checked={!!task.isChecked}
           on:click={toggleChecked}
    />
    <span>{ task.text }</span>
    <button class="delete" on:click={deleteThisTask}>&times;</button>
</li>
