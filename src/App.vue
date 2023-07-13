<template>
  <div style="margin-bottom: 10px">
    <button style="margin-right: 5px" @click="save()">Save</button>
    <button style="margin-right: 5px" @click="removeCompleted()">
      Remove Completed
    </button>
    <button style="margin-right: 5px" @click="addOwner()">Add Owner</button>
    <button @click="copyAsImage($event)">Copy as Image</button>
  </div>
  <table>
    <tr>
      <td style="border: none"></td>
      <th colspan="8">Daily Status Meeting</th>
      <th style="border: none"></th>
    </tr>
    <tr>
      <td style="border: none"></td>
      <th>Owner</th>
      <th>Location</th>
      <th>Ticket ID</th>
      <th>Start Date</th>
      <th>Estimation <small>(in hrs)</small></th>
      <th>Time Spent <small>(in hrs)</small></th>
      <th>Status</th>
      <th>Remarks</th>
      <th style="border: none"></th>
    </tr>
    <tr v-if="data.length == 0">
      <td style="border: none"></td>
      <td colspan="8">No data stored, add owner to start recording status.</td>
    </tr>
    <template v-else>
      <template v-for="(owner, ownerIndex) in data" :key="ownerIndex">
        <template v-for="(task, taskIndex) in owner.tasks" :key="taskIndex">
          <tr>
            <td v-if="taskIndex == 0" :rowspan="owner.tasks.length + 1">
              <button style="margin-bottom: 3px" @click="moveUp(ownerIndex)">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="black"
                  viewBox="0 0 24 24"
                  style="width: 10px; height: 10px"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M4.5 15.75l7.5-7.5 7.5 7.5"
                  />
                </svg>
              </button>
              <br />
              <button style="margin-bottom: 3px" @click="moveDown(ownerIndex)">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="black"
                  viewBox="0 0 24 24"
                  style="width: 10px; height: 10px"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M19.5 8.25l-7.5 7.5-7.5-7.5"
                  />
                </svg>
              </button>
              <br />
              <button @click="removeOwner(ownerIndex)">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="red"
                  viewBox="0 0 24 24"
                  style="width: 10px; height: 10px"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
                  />
                </svg>
              </button>
            </td>
            <td v-if="taskIndex == 0" :rowspan="owner.tasks.length + 1">
              {{ owner.name }}
            </td>
            <td v-if="taskIndex == 0" :rowspan="owner.tasks.length + 1">
              <select v-model="owner.location">
                <option>Home</option>
                <option>GDC</option>
                <option>HO</option>
              </select>
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="text" v-model="task.name" />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="date" v-model="task.startDate" :max="todaysDate" />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="number" v-model="task.estimation" />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="number" v-model="task.timeSpent" />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <select v-model="task.status">
                <option>In Progress</option>
                <option>Completed</option>
                <option>On Hold</option>
              </select>
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="text" v-model="task.remarks" />
            </td>
            <td>
              <button @click="removeTask(ownerIndex, taskIndex)">
                <svg
                  xmlns="http://www.w3.org/2000/svg"
                  fill="red"
                  viewBox="0 0 24 24"
                  style="width: 10px; height: 10px"
                >
                  <path
                    stroke-linecap="round"
                    stroke-linejoin="round"
                    d="M14.74 9l-.346 9m-4.788 0L9.26 9m9.968-3.21c.342.052.682.107 1.022.166m-1.022-.165L18.16 19.673a2.25 2.25 0 01-2.244 2.077H8.084a2.25 2.25 0 01-2.244-2.077L4.772 5.79m14.456 0a48.108 48.108 0 00-3.478-.397m-12 .562c.34-.059.68-.114 1.022-.165m0 0a48.11 48.11 0 013.478-.397m7.5 0v-.916c0-1.18-.91-2.164-2.09-2.201a51.964 51.964 0 00-3.32 0c-1.18.037-2.09 1.022-2.09 2.201v.916m7.5 0a48.667 48.667 0 00-7.5 0"
                  />
                </svg>
              </button>
            </td>
          </tr>
          <tr v-if="taskIndex == owner.tasks.length - 1">
            <td colspan="6" style="text-align: center">
              <button @click="addTask(ownerIndex)">+</button>
            </td>
          </tr>
        </template>
      </template>
    </template>
  </table>
  <div id="preview">
    <table>
      <tr>
        <th colspan="8">Daily Status Meeting</th>
      </tr>
      <tr>
        <th>Owner</th>
        <th>Location</th>
        <th>Ticket ID</th>
        <th>Start Date</th>
        <th>Estimation <small>(in hrs)</small></th>
        <th>Time Spent <small>(in hrs)</small></th>
        <th>Status</th>
        <th>Remarks</th>
      </tr>
      <template v-for="(owner, ownerIndex) in data" :key="ownerIndex">
        <template v-for="(task, taskIndex) in owner.tasks" :key="taskIndex">
          <tr>
            <td v-if="taskIndex == 0" :rowspan="owner.tasks.length">
              {{ owner.name }}
            </td>
            <td v-if="taskIndex == 0" :rowspan="owner.tasks.length">
              {{ owner.location }}
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              {{ task.name }}
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              {{ task.startDate }}
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              {{ task.estimation }}
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              {{ task.timeSpent }}
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              {{ task.status }}
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              {{ task.remarks }}
            </td>
          </tr>
        </template>
      </template>
    </table>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import domtoimage from "dom-to-image-more";

const todaysDate = new Date().toJSON().split("T")[0];

const data = ref([]);

onMounted(async () => {
  // const json = await (await fetch("http://localhost:8080/data.json")).json();
  if (localStorage.getItem("data")) {
    const local = localStorage.getItem("data");
    data.value = JSON.parse(local);
  }
});

function moveUp(index) {
  let u = data.value;
  if (index > 0) {
    let el = u[index];
    u[index] = u[index - 1];
    u[index - 1] = el;
  }
  data.value = u;
}

function moveDown(index) {
  let d = data.value;
  if (index !== -1 && index < d.length - 1) {
    let el = d[index];
    d[index] = d[index + 1];
    d[index + 1] = el;
  }
  data.value = d;
}

function addTask(index) {
  data.value[index].tasks.push({
    name: "",
    startDate: todaysDate,
    estimation: 0,
    timeSpent: 0,
    status: "In Progress",
    remarks: "",
  });
}

function removeTask(ownerIndex, taskIndex) {
  if (data.value[ownerIndex].tasks.length > 1) {
    data.value[ownerIndex].tasks.splice(taskIndex, 1);
  }
}

function removeCompleted() {
  data.value.forEach(function (owner, index) {
    let tasks = data.value[index].tasks;
    let completedTasks = tasks.filter((task) => task.status == "Completed");
    let incompleteTasks = tasks.filter((task) => task.status != "Completed");

    if (completedTasks.length != tasks.length) {
      data.value[index].tasks = incompleteTasks;
    }
  });
}

function addOwner() {
  const name = prompt("Enter owner name: ");
  data.value.push({
    name: name,
    location: "GDC",
    tasks: [
      {
        name: "",
        startDate: todaysDate,
        estimation: 0,
        timeSpent: 0,
        status: "In Progress",
        remarks: "",
      },
    ],
  });
}

function removeOwner(index) {
  if (data.value.length > 1) {
    data.value.splice(index, 1);
  }
}

function copyAsImage(event) {
  var node = document.getElementById("preview");

  event.target.setAttribute("disabled", true);
  event.target.innerText = "Copying..";
  domtoimage.toBlob(node).then(function (blob) {
    navigator.clipboard
      .write([
        new ClipboardItem({
          [blob.type]: blob,
        }),
      ])
      .then(() => {
        console.log("Copied");
      });

    event.target.innerText = "Copied!";

    setTimeout(function () {
      event.target.innerText = "Copy as Image";
      event.target.removeAttribute("disabled");
    }, 1000);
  });
}

function save() {
  localStorage.setItem("data", JSON.stringify(data.value));
}
</script>

<style>
* {
  font-family: Arial, Helvetica, sans-serif;
}

table {
  border-collapse: collapse;
}

th,
td {
  border: 1px solid black;
}

th,
td {
  padding: 5px;
}

td.completed {
  background-color: #66cc66;
}

#preview {
  background-color: white;
  position: absolute;
  top: -99em;
  width: 100%;
  font-size: 24px;
}

#preview table {
  width: 100%;
}
</style>
