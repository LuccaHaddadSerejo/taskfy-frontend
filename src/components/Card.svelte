<script lang="ts">
  import arrowdown from "../assets/arrowdown.svg";
  import arrowup from "../assets/arrowup.svg";
  import pluspurple from "../assets/plusPurple.svg";
  import trash from "../assets/trash.svg";
  import editTask from "../assets/edit.svg";
  import SubtaskCard from "./SubtaskCard.svelte";
  import Button from "./Button.svelte";
  import Input from "./Input.svelte";
  import { createSubtask, deleteTask, updateTask } from "../api";
  import type {
    iSubtask,
    tCreateSubtask,
    tPartialTask,
  } from "../interfaces/tasks.";

  export let completed: boolean;
  export let title: string;
  export let key: number;
  export let subtasks: iSubtask[];

  let isEditing: boolean = false;
  let newSubtask: boolean = false;
  let direction: boolean = false;
  let value: string = "";
  let newValue: string = title;

  $: arrow = direction ? arrowup : arrowdown;

  const toogleDirection = () => (direction = !direction);
  const toogleAddSubtask = () => (newSubtask = !newSubtask);
  const toogleEditTask = () => (isEditing = !isEditing);

  /*
    Função para criar um novo objeto, atualizar uma task e mudar a condicional de edição de task.
  */

  const handleEditTask = (): void => {
    const updatedObj: tPartialTask = {
      title: newValue,
    };

    updateTask(updatedObj, key);
    toogleEditTask();
  };

  /*
    Função para mudar a task de pending para done e colapsar o item.
  */

  const handleCompleteTask = (): void => {
    const updatedObj: tPartialTask = {
      done: completed ? true : false,
    };

    updateTask(updatedObj, key);
    direction = false;
  };

  /*
    Função para criar uma nova subtask no item específico, com base no id.
  */

  const handleCreateSubtaskClick = async (): Promise<void> => {
    const newSubtask: tCreateSubtask = {
      title: value,
    };
    createSubtask(newSubtask, key);
    resetSubtaskValue();
  };

  /*
    Função para resetar o valor do title, caso tenha sido alterado no input de edição de tasks.
  */

  const resetTitleValue = () => {
    toogleEditTask();
    newValue = title;
  };

  /*
    Função para resetar o valor do input, caso o usuário cancele a criação da subtask.
  */

  const resetSubtaskValue = () => {
    toogleAddSubtask();
    value = "";
  };
</script>

<li class:completed>
  <div class="contentdiv">
    {#if !isEditing}
      <div class="tasktitle">
        <input
          type="checkbox"
          bind:checked={completed}
          on:change={handleCompleteTask}
        />
        <h2 class:completed>{title}</h2>
      </div>
      <div class="taskbuttons">
        <div class="taskbuttonsone">
          <Button
            img
            src={editTask}
            editAndArrowButton
            alt={"Imagem de uma caneta e uma prancheta"}
            disabled={completed}
            on:click={toogleEditTask}
          />
          <Button
            on:click={() => deleteTask(key)}
            img
            src={trash}
            trash
            alt={"Imagem de um lixo"}
          />
        </div>
        <Button
          on:click={toogleDirection}
          img
          src={arrow}
          editAndArrowButton
          alt={"Imagem de uma seta"}
          disabled={completed}
        />
      </div>
    {:else}
      <Input
        bind:value={newValue}
        inputedittitle
        placeholder={"Digite um novo título"}
      />
      <div class="editbuttonsdiv">
        <Button
          on:click={handleEditTask}
          buttonedittask
          text
          content={"Confirmar"}
          disabled={newValue.length === 0}
        />
        <Button
          on:click={resetTitleValue}
          buttonedittask
          text
          content={"Cancelar"}
        />
      </div>
    {/if}
  </div>
  {#if arrow === arrowup && !completed}
    <ul>
      {#each subtasks as subtask (subtask.id)}
        <SubtaskCard
          subtitle={subtask.title}
          subkey={subtask.id}
          completedSubtask={subtask.done}
        />
      {/each}
    </ul>
    <div>
      {#if !newSubtask}
        <div class="addsubtaskdiv">
          <Button
            on:click={toogleAddSubtask}
            img
            text
            buttonaddnewsubtask
            content={"Adicionar subtask"}
            src={pluspurple}
            alt="Imagem de um símbolo de mais"
          />
        </div>
      {:else}
        <div class="newsubtaskdiv">
          <Input bind:value inputnewtask placeholder={"Adicione uma subtask"} />
          <Button
            on:click={handleCreateSubtaskClick}
            text
            disabled={value.length > 0 ? false : true}
            buttonsnewsubstaskdiv
            content={"Criar"}
          />
          <Button
            on:click={resetSubtaskValue}
            text
            buttonsnewsubstaskdiv
            content={"Voltar"}
          />
        </div>
      {/if}
    </div>
  {/if}
</li>

<style lang="scss">
  li {
    display: flex;
    flex-direction: column;
    gap: var(--gap-1);
    justify-content: space-between;
    background-color: var(--color-gray-2);
    border-radius: var(--radius-1);
    padding: 2rem 1rem 2rem 1rem;
  }

  h2 {
    color: var(--color-gray-4);
    font-size: clamp(1.5rem, 2vw, 2rem);
  }

  .completed {
    opacity: 0.5;
    text-decoration: line-through;
  }

  .tasktitle {
    display: flex;
    gap: 20px;
  }

  .taskbuttons {
    display: flex;
    justify-content: space-between;
    gap: 30px;
  }

  .taskbuttonsone {
    display: flex;
    gap: var(--gap-3);
  }

  .contentdiv {
    display: flex;
    flex-direction: column;
    justify-content: space-between;
    gap: var(--gap-3);

    @media (min-width: 800px) {
      flex-direction: row;
    }
  }

  .addsubtaskdiv {
    display: flex;
    align-items: center;
    justify-content: center;
  }

  .newsubtaskdiv {
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    align-items: center;
    gap: var(--gap-1);
  }

  .editbuttonsdiv {
    display: flex;
    justify-content: center;
    align-items: center;
    gap: var(--gap-3);
  }
</style>
