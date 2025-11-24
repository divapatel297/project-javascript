<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Assignment Reminder | ITM(SLS) Baroda University</title>
  <link href="https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;500;600;700&display=swap" rel="stylesheet">
  <style>
    body {
      font-family: "Poppins", sans-serif;
      background-color: #eef2f7;
      margin: 0;
      padding: 0;
      display: flex;
      flex-direction: column;
      align-items: center;
      min-height: 100vh;
    }

    header {
      background-color: #003366;
      color: white;
      padding: 15px 0;
      text-align: center;
      width: 100%;
      font-size: 1.4em;
      letter-spacing: 1px;
      box-shadow: 0 3px 6px rgba(0,0,0,0.2);
    }

    #container {
      margin-top: 30px;
      background-color: white;
      border-radius: 10px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
      width: 90%;
      max-width: 550px;
      padding: 25px;
      transition: all 0.3s ease;
    }

    h2 {
      text-align: center;
      color: #003366;
      margin-bottom: 15px;
    }

    input, button {
      width: 100%;
      padding: 10px;
      margin: 8px 0;
      border-radius: 6px;
      border: 1px solid #ccc;
      font-size: 1em;
    }

    input:focus {
      outline: none;
      border-color: #003366;
      box-shadow: 0 0 5px #003366;
    }

    button {
      background-color: #003366;
      color: white;
      cursor: pointer;
      transition: 0.3s;
      font-weight: 600;
    }

    button:hover {
      background-color: #0056b3;
    }

    .task {
      display: flex;
      justify-content: space-between;
      align-items: center;
      padding: 10px;
      background-color: #f7f9fc;
      border-left: 5px solid #003366;
      border-radius: 5px;
      margin: 10px 0;
      transition: 0.3s;
    }

    .task.done {
      border-left-color: #2ecc71;
      background-color: #e9fbe9;
      text-decoration: line-through;
      opacity: 0.7;
    }

    .task.overdue {
      border-left-color: #e74c3c;
      background-color: #ffe6e6;
    }

    .task.soon {
      border-left-color: #f1c40f;
      background-color: #fff8e1;
    }

    .task strong {
      color: #003366;
    }

    .days-left {
      font-size: 0.9em;
      color: #555;
    }

    .task button {
      margin-left: 8px;
      border: none;
      padding: 6px 10px;
      border-radius: 4px;
      cursor: pointer;
    }

    .delete-btn {
      background-color: #e74c3c;
      color: white;
    }

    .done-btn {
      background-color: #2ecc71;
      color: white;
    }

    footer {
      text-align: center;
      color: #666;
      font-size: 0.9em;
      margin-top: 20px;
      padding: 10px;
    }

    #searchBar {
      width: 100%;
      margin-bottom: 15px;
      padding: 8px;
      border: 1px solid #bbb;
      border-radius: 5px;
    }
    
    .notification {
      position: fixed;
      top: 20px;
      right: 20px;
      padding: 15px 20px;
      border-radius: 5px;
      color: white;
      font-weight: bold;
      box-shadow: 0 4px 12px rgba(0,0,0,0.15);
      z-index: 1000;
      transform: translateX(200%);
      transition: transform 0.3s ease;
    }
    
    .notification.show {
      transform: translateX(0);
    }
    
    .notification.success {
      background-color: #2ecc71;
    }
    
    .notification.warning {
      background-color: #f1c40f;
    }
    
    .notification.error {
      background-color: #e74c3c;
    }
  </style>
</head>
<body>
  <header>📘 ITM(SLS) Baroda University - Assignment Reminder</header>

  <div id="container">
    <h2>Add New Assignment</h2>
    <input type="text" id="taskName" placeholder="Enter Assignment Title" />
    <input type="date" id="dueDate" />
    <input type="time" id="dueTime" />
    <button onclick="addTask()">➕ Add Assignment</button>

    <input type="text" id="searchBar" placeholder="🔍 Search assignments..." onkeyup="displayTasks()" />

    <h2>Your Tasks</h2>
    <div id="taskList"></div>
  </div>

  <footer>Made with ❤ for ITM(SLS) Baroda University Students</footer>

  <audio id="alarmSound" src="https://www.soundjay.com/buttons/sounds/beep-07.mp3"></audio>
  <div id="notification" class="notification"></div>

  <script>
    let tasks = JSON.parse(localStorage.getItem("tasks")) || [];

    function saveTasks() {
      localStorage.setItem("tasks", JSON.stringify(tasks));
    }

    function addTask() {
      const name = document.getElementById("taskName").value.trim();
      const date = document.getElementById("dueDate").value;
      const time = document.getElementById("dueTime").value;

      if (!name || !date || !time) {
        showNotification("⚠️ Please enter assignment title, date, and time!", "error");
        return;
      }

      const dueDateTime = new Date(`${date}T${time}`);

      tasks.push({
        name,
        dueDate: date,
        dueTime: time,
        dueTimestamp: dueDateTime.getTime(),
        alerted: false,
        done: false
      });

      saveTasks();
      displayTasks();

      document.getElementById("taskName").value = "";
      document.getElementById("dueDate").value = "";
      document.getElementById("dueTime").value = "";
      
      showNotification("✅ Assignment added successfully!", "success");
    }

    function deleteTask(index) {
      if (confirm("Delete this assignment?")) {
        tasks.splice(index, 1);
        saveTasks();
        displayTasks();
        showNotification("🗑️ Assignment deleted!", "warning");
      }
    }

    function markDone(index) {
      tasks[index].done = !tasks[index].done;
      saveTasks();
      displayTasks();
      showNotification(tasks[index].done ? "✅ Task marked as done!" : "🔄 Task undone!", "success");
    }

    function daysLeft(dueDate) {
      const today = new Date();
      const due = new Date(dueDate);
      const diff = Math.ceil((due - today) / (1000 * 60 * 60 * 24));
      return diff;
    }

    function displayTasks() {
      const searchValue = document.getElementById("searchBar").value.toLowerCase();
      const taskList = document.getElementById("taskList");
      taskList.innerHTML = "";

      if (tasks.length === 0) {
        taskList.innerHTML = "<p>No assignments added yet ✅</p>";
        return;
      }

      // Sort tasks by nearest due date
      tasks.sort((a, b) => a.dueTimestamp - b.dueTimestamp);

      tasks.forEach((task, index) => {
        if (!task.name.toLowerCase().includes(searchValue)) return;

        const diff = daysLeft(task.dueDate);
        const div = document.createElement("div");
        div.classList.add("task");

        if (task.done) div.classList.add("done");
        else if (diff < 0) div.classList.add("overdue");
        else if (diff <= 3) div.classList.add("soon");

        // live countdown
        const timeLeft = getCountdown(task.dueTimestamp);

        div.innerHTML = `
          <div>
            <strong>${task.name}</strong><br>
            <span class="days-left">
              📅 ${task.dueDate} ${task.dueTime} <br>
              ${diff < 0 ? "❌ Overdue" : "⏳ " + timeLeft}
            </span>
          </div>
          <div>
            <button class="done-btn" onclick="markDone(${index})">${task.done ? "Undo" : "Done"}</button>
            <button class="delete-btn" onclick="deleteTask(${index})">Delete</button>
          </div>
        `;
        taskList.appendChild(div);
      });
    }

    function getCountdown(timestamp) {
      const now = new Date().getTime();
      let diff = timestamp - now;
      if (diff <= 0) return "Due now!";
      let hours = Math.floor(diff / (1000 * 60 * 60));
      let mins = Math.floor((diff % (1000 * 60 * 60)) / (1000 * 60));
      return `${hours}h ${mins}m left`;
    }

    // Alarm checker
    function checkAlarms() {
      const now = new Date().getTime();
      const sound = document.getElementById("alarmSound");

      tasks.forEach((task) => {
        if (!task.alerted && now >= task.dueTimestamp && !task.done) {
          sound.play();
          alert(`⏰ Reminder: ${task.name} is due now!`);
          task.alerted = true;
          saveTasks();
          displayTasks();
        }
      });
    }

    function showNotification(message, type) {
      const notification = document.getElementById("notification");
      notification.textContent = message;
      notification.className = `notification ${type} show`;
      
      setTimeout(() => {
        notification.classList.remove("show");
      }, 3000);
    }

    setInterval(checkAlarms, 60000); // every minute
    setInterval(displayTasks, 60000); // update countdown every minute
    displayTasks();
  </script>
</body>
</html>
