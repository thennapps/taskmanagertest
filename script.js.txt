document.getElementById('addTaskBtn').onclick = () => {
  document.getElementById('taskForm').style.display = 'block';
};

let editIndex = -1; // For tracking editing state

function addTask() {
  const name = document.getElementById('taskName').value;
  const due = document.getElementById('dueDate').value;
  const assignedTo = document.getElementById('assignedTo').value;

  if (!name || !due || !assignedTo) {
    alert("Please fill all fields.");
    return;
  }

  let tasks = JSON.parse(localStorage.getItem("tasks")) || [];

  if (editIndex === -1) {
    tasks.push({ name, due, assignedTo, status: "To Do" });
  } else {
    tasks[editIndex].name = name;
    tasks[editIndex].due = due;
    tasks[editIndex].assignedTo = assignedTo;
    editIndex = -1;
  }

  localStorage.setItem("tasks", JSON.stringify(tasks));
  document.getElementById('taskForm').reset();
  document.getElementById('taskForm').style.display = 'none';
  displayTasks();
}

else {
    alert("Please fill all fields.");
  }
}

function displayTasks() {
  const taskList = document.getElementById('taskList');
  taskList.innerHTML = "";

  const filter = document.getElementById("statusFilter").value;
  let tasks = JSON.parse(localStorage.getItem("tasks")) || [];

  tasks.sort((a, b) => new Date(a.due) - new Date(b.due)); // Sort by due date

  tasks.forEach((task, index) => {
    if (filter && task.status !== filter) return;

    const taskDiv = document.createElement("div");
    taskDiv.className = "task";

    taskDiv.innerHTML = `
      <strong>${task.name}</strong><br>
      Due: ${task.due}<br>
      Assigned to: ${task.assignedTo}<br>
      Status: <select onchange="updateStatus(${index}, this.value)">
        <option ${task.status === "To Do" ? "selected" : ""}>To Do</option>
        <option ${task.status === "In Progress" ? "selected" : ""}>In Progress</option>
        <option ${task.status === "Done" ? "selected" : ""}>Done</option>
      </select>
      <button onclick="deleteTask(${index})" style="margin-left:10px; color: red;">Delete</button>
<button onclick="editTask(${index})" style="margin-left:5px;">Edit</button>

    `;

    taskList.appendChild(taskDiv);
  });
}

function updateStatus(index, newStatus) {
  const tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  tasks[index].status = newStatus;
  localStorage.setItem("tasks", JSON.stringify(tasks));
  displayTasks();
}

function deleteTask(index) {
  const tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  tasks.splice(index, 1);
  localStorage.setItem("tasks", JSON.stringify(tasks));
  displayTasks();
}

window.onload = displayTasks;

function editTask(index) {
  const tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  const task = tasks[index];

  document.getElementById('taskName').value = task.name;
  document.getElementById('dueDate').value = task.due;
  document.getElementById('assignedTo').value = task.assignedTo;
  document.getElementById('taskForm').style.display = 'block';
  editIndex = index;
}

function exportCSV() {
  const tasks = JSON.parse(localStorage.getItem("tasks")) || [];
  const rows = [
    ["Task Name", "Due Date", "Assigned To", "Status"],
    ...tasks.map(t => [t.name, t.due, t.assignedTo, t.status])
  ];

  let csvContent = "data:text/csv;charset=utf-8," + rows.map(e => e.join(",")).join("\\n");

  const encodedUri = encodeURI(csvContent);
  const link = document.createElement("a");
  link.setAttribute("href", encodedUri);
  link.setAttribute("download", "tasks.csv");
  document.body.appendChild(link);
  link.click();
  document.body.removeChild(link);
}
