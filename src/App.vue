<template>
  <div style="margin-bottom: 10px">
    <button style="margin-right: 5px" @click="save()" id="saveButton">
      Save
    </button>
    <button
      style="margin-right: 5px"
      @click="fetchFromGist()"
      id="downloadButton"
      :disabled="Object.keys(gist).length == 0"
    >
      Fetch
    </button>
    <button style="margin-right: 5px" @click="removeCompleted()">
      Remove Completed
    </button>
    <button style="margin-right: 5px" @click="addOwner()">Add Owner</button>
    <button style="margin-right: 5px" @click="exportData($event)">
      Export
    </button>
    <button style="margin-right: 5px" @click="showImportDialog()">
      Import
    </button>
    <button @click="copyAsImage($event)">Copy as Image</button>
  </div>
  <table>
    <tr>
      <td style="border: none"></td>
      <th colspan="8">Daily Status Meeting</th>
      <th style="border: none; background-color: transparent"></th>
    </tr>
    <tr>
      <td style="border: none"></td>
      <th>Owner</th>
      <th>Location</th>
      <th>Ticket ID</th>
      <th>Start Date</th>
      <th>Estimation <br /><small>(in hrs)</small></th>
      <th>Time Spent <br /><small>(in hrs)</small></th>
      <th>Status</th>
      <th>Remarks</th>
      <th style="border: none; background-color: transparent"></th>
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
              <input type="text" v-model="owner.name" />
            </td>
            <td v-if="taskIndex == 0" :rowspan="owner.tasks.length + 1">
              <select v-model="owner.location">
                <option></option>
                <option>Home</option>
                <option>GDC</option>
                <option>HO</option>
                <option>On Leave</option>
                <option>On Sick Leave</option>
                <option>On Maternity Leave</option>
                <option>On Paternity Leave</option>
              </select>
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="text" v-model="task.name" />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="date" v-model="task.startDate" :max="todaysDate" />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input
                type="number"
                v-model="task.estimation"
                style="width: 50px"
              />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input
                type="number"
                v-model="task.timeSpent"
                style="width: 50px"
                @keydown.space="addTimeSpent(ownerIndex, taskIndex)"
                @keydown.s="subtractTimeSpent(ownerIndex, taskIndex)"
              />
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <select v-model="task.status">
                <option></option>
                <option>In Progress</option>
                <option>Completed</option>
                <option>On Hold</option>
              </select>
            </td>
            <td :class="{ completed: task.status == 'Completed' }">
              <input type="text" v-model="task.remarks" />
            </td>
            <td style="border-right: none; padding-right: 0">
              <button
                style="margin-bottom: 3px"
                @click="moveUp(ownerIndex, taskIndex)"
              >
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
              <button
                style="margin-bottom: 3px"
                @click="moveDown(ownerIndex, taskIndex)"
              >
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
            </td>
            <td style="border-left: none">
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
  <div id="preview" style="border-bottom: 1px solid black">
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
              {{ task.startDate.split("-").reverse().join("-") }}
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
  <dialog id="export">
    <button
      style="position: absolute; right: 10px; color: red; font-weight: bold"
      @click="$event.target.parentElement.close()"
    >
      x
    </button>
    <button @click="copyExportData">Copy</button><br />
    <textarea
      style="margin-top: 10px"
      cols="150"
      rows="10"
      v-model="compressedExport"
      id="exportTextarea"
    ></textarea>
  </dialog>
  <dialog id="import">
    <button
      style="position: absolute; right: 10px; color: red; font-weight: bold"
      @click="$event.target.parentElement.close()"
    >
      x
    </button>
    <button @click="importExportedData($event)">Import</button>
    <br />
    <textarea
      style="margin-top: 10px"
      cols="150"
      rows="10"
      id="importTextArea"
    ></textarea>
  </dialog>
</template>

<script setup>
import { onMounted, ref } from "vue";
import domtoimage from "dom-to-image-more";
import lzjs from "lzjs";
import axios from "axios";

var token = null;
var gist = ref({});

const todaysDate = new Date().toJSON().split("T")[0];

const data = ref([]);

onMounted(async () => {
  const key = new URLSearchParams(window.location.search).get("key");
  token = "ghp_" + atob(new URLSearchParams(window.location.search).get("key"));

  if (key) {
    axios.defaults.headers.common["Authorization"] = "Bearer " + token;

    try {
      const content = (
        await axios.get("https://api.github.com/gists")
      ).data.filter((g) => "statusmaker" in g.files);

      if (content.length) {
        gist.value = content[0];
      }
    } catch (e) {
      console.error(e);
    }
  }

  if (localStorage.getItem("data")) {
    const local = localStorage.getItem("data");
    data.value = JSON.parse(local);
  } else {
    if (Object.keys(gist.value).length) {
      fetchFromGist();
    }
  }
});

function moveUp(ownerIndex, taskIndex = null) {
  let u = data.value;
  if (taskIndex !== null) {
    if (ownerIndex > 0 && taskIndex > 0) {
      let el = u[ownerIndex].tasks[taskIndex];
      u[ownerIndex].tasks[taskIndex] = u[ownerIndex].tasks[taskIndex - 1];
      u[ownerIndex].tasks[taskIndex - 1] = el;
    }
  } else {
    if (ownerIndex > 0) {
      let el = u[ownerIndex];
      u[ownerIndex] = u[ownerIndex - 1];
      u[ownerIndex - 1] = el;
    }
  }
  data.value = u;
}

function moveDown(ownerIndex, taskIndex = null) {
  let d = data.value;
  if (taskIndex !== null) {
    if (taskIndex !== -1 && taskIndex < d[ownerIndex].tasks.length - 1) {
      let el = d[ownerIndex].tasks[taskIndex];
      d[ownerIndex].tasks[taskIndex] = d[ownerIndex].tasks[taskIndex + 1];
      d[ownerIndex].tasks[taskIndex + 1] = el;
    }
  } else {
    if (ownerIndex !== -1 && ownerIndex < d.length - 1) {
      let el = d[ownerIndex];
      d[ownerIndex] = d[ownerIndex + 1];
      d[ownerIndex + 1] = el;
    }
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
  if (name) {
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
    // document
    //   .querySelector("#preview table")
    //   .setAttribute(
    //     "margin-bottom",
    //     window.getComputedStyle(document.querySelector("#preview table")).height
    //   );
    navigator.clipboard
      .write([
        new ClipboardItem({
          [blob.type]: blob,
        }),
      ])
      .then(() => {
        // console.log("Copied");
        event.target.innerText = "Copied!";

        setTimeout(function () {
          event.target.innerText = "Copy as Image";
          event.target.removeAttribute("disabled");
        }, 1000);
      })
      .catch(function () {
        // console.error("oops, something went wrong!", error);
        event.target.innerText = "Error!";

        setTimeout(function () {
          event.target.innerText = "Copy as Image";
          event.target.removeAttribute("disabled");
        }, 1000);
      });
  });
}

async function save() {
  const button = document.getElementById("saveButton");
  button.setAttribute("disabled", true);
  button.innerText = "Saving..";

  localStorage.setItem("data", JSON.stringify(data.value));

  if (Object.keys(gist.value).length) {
    await axios.patch("https://api.github.com/gists/" + gist.value.id, {
      files: {
        statusmaker: {
          content: lzjs.compressToBase64(JSON.stringify(data.value)),
        },
      },
    });
  }

  button.innerText = "Saved";

  setTimeout(function () {
    button.removeAttribute("disabled");
    button.innerText = "Save";
  }, 1000);
}

async function fetchFromGist() {
  const content = await (
    await fetch(gist.value["files"]["statusmaker"]["raw_url"])
  ).text();
  const decompressed = lzjs.decompressFromBase64(content);
  data.value = JSON.parse(decompressed);
  localStorage.setItem("data", decompressed);
}

const compressedExport = ref("");

function exportData() {
  const modal = document.getElementById("export");

  save();

  compressedExport.value = lzjs.compressToBase64(JSON.stringify(data.value));

  modal.showModal();

  setTimeout(function () {
    document.getElementById("exportTextarea").select();
  }, 1);
}

function copyExportData(event) {
  let exportData = document.getElementById("exportTextarea");

  event.target.setAttribute("disabled", true);
  event.target.innerText = "Copying..";

  exportData.select();

  navigator.clipboard.writeText(exportData.value);

  event.target.innerText = "Copied!";

  setTimeout(function () {
    event.target.removeAttribute("disabled");
    event.target.innerText = "Copy";
  }, 500);
}

function showImportDialog() {
  const modal = document.getElementById("import");
  modal.showModal();
}

function importExportedData(event) {
  const modal = document.getElementById("import");
  const exportedData = document.getElementById("importTextArea").value;
  event.target.setAttribute("disabled", true);
  event.target.innerText = "Importing..";

  data.value = JSON.parse(lzjs.decompressFromBase64(exportedData));

  event.target.innerText = "Imported!";

  setTimeout(function () {
    event.target.removeAttribute("disabled");
    event.target.innerText = "Import";
  }, 500);

  setTimeout(function () {
    document.getElementById("importTextArea").value = "";
    modal.close();
  }, 500);
}

function addTimeSpent(ownerIndex, taskIndex) {
  // alert(event.target.value);
  const originalValue = parseInt(
    data.value[ownerIndex].tasks[taskIndex].timeSpent
  );
  const valueToAdd = parseInt(prompt(""));

  if (valueToAdd) {
    data.value[ownerIndex].tasks[taskIndex].timeSpent =
      originalValue + valueToAdd;
  }
}

function subtractTimeSpent(ownerIndex, taskIndex) {
  const originalValue = parseInt(
    data.value[ownerIndex].tasks[taskIndex].timeSpent
  );
  const valueToSubtract = parseInt(prompt(""));

  if (valueToSubtract) {
    let answerValue = originalValue - valueToSubtract;
    data.value[ownerIndex].tasks[taskIndex].timeSpent =
      answerValue < 0 ? 0 : answerValue;
  }
}
</script>

<style>
* {
  font-family: Arial, Helvetica, sans-serif;
}

table {
  border-collapse: collapse;
  width: 100%;
}

th,
td {
  border: 1px solid black;
}

th,
td {
  padding: 5px;
  text-align: center;
}

th {
  background-color: #336699;
  color: white;
}

td.completed {
  background-color: #66cc66;
}

input,
select {
  height: 30px;
}

#preview {
  background-color: white;
  position: absolute;
  top: -99em;
  width: 100%;
  font-size: 24px;
}

#preview td {
  height: 20px;
}
</style>
