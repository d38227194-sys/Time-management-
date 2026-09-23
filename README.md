<!DOCTYPE html>
<html lang="ar" dir="rtl">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>منظم وقت الدراسة</title>

  <style>
    * {
      box-sizing: border-box;
      font-family: Arial, sans-serif;
    }

    body {
      margin: 0;
      background: linear-gradient(135deg, #eff6ff, #dbeafe);
      color: #172554;
    }

    .container {
      max-width: 850px;
      margin: 25px auto;
      padding: 15px;
    }

    .card {
      background: white;
      padding: 22px;
      margin-bottom: 18px;
      border-radius: 18px;
      box-shadow: 0 5px 18px #0002;
    }

    h1, h2 {
      text-align: center;
    }

    h1 {
      color: #2563eb;
    }

    label {
      display: block;
      margin: 12px 0 6px;
      font-weight: bold;
    }

    input, select, button {
      width: 100%;
      padding: 12px;
      border-radius: 9px;
      font-size: 15px;
    }

    input, select {
      border: 1px solid #aaa;
    }

    button {
      border: none;
      margin-top: 12px;
      background: #2563eb;
      color: white;
      font-weight: bold;
      cursor: pointer;
    }

    button:hover {
      background: #1d4ed8;
    }

    .task {
      border: 1px solid #dbeafe;
      background: #f8fafc;
      padding: 15px;
      margin-top: 12px;
      border-radius: 12px;
    }

    .task.done {
      opacity: 0.6;
      text-decoration: line-through;
    }

    .task-title {
      font-weight: bold;
      font-size: 18px;
    }

    .task-info {
      color: #475569;
      margin-top: 7px;
    }

    .task-buttons {
      display: flex;
      gap: 8px;
    }

    .task-buttons button {
      flex: 1;
    }

    .start {
      background: #16a34a;
    }

    .finish {
      background: #7c3aed;
    }

    .remove {
      background: #dc2626;
    }

    .timer {
      text-align: center;
      font-size: 45px;
      font-weight: bold;
      color: #dc2626;
      margin: 15px;
    }

    .empty {
      text-align: center;
      color: #64748b;
    }

    @media (max-width: 600px) {
      .task-buttons {
        flex-direction: column;
      }
    }
  </style>
</head>

<body>

  <div class="container">

    <div class="card">
      <h1>📚 منظم وقت الدراسة</h1>
      <p style="text-align:center">
        أضف المواد والواجبات وحدد الوقت المطلوب لكل مهمة
      </p>

      <label>اسم المادة أو المهمة</label>
      <input id="taskName" placeholder="مثلاً: رياضيات - حل الواجب">

      <label>نوع المهمة</label>
      <select id="taskType">
        <option>مذاكرة</option>
        <option>واجب</option>
        <option>مراجعة</option>
        <option>اختبار</option>
        <option>قراءة</option>
      </select>

      <label>المدة بالدقائق</label>
      <input id="taskTime" type="number" min="1" placeholder="مثلاً: 30">

      <button onclick="addTask()">➕ إضافة المهمة</button>
    </div>

    <div class="card">
      <h2>⏱️ المؤقت</h2>

      <div class="timer" id="timer">00:00</div>

      <button onclick="pauseTimer()">إيقاف مؤقت</button>
      <button onclick="resetTimer()" class="remove">إلغاء المؤقت</button>
    </div>

    <div class="card">
      <h2>📝 مهامي الدراسية</h2>
      <div id="tasks">
        <p class="empty">لم تضف أي مهام حتى الآن</p>
      </div>
    </div>

  </div>

  <script>
    let tasks = JSON.parse(localStorage.getItem("studyTasks")) || [];
    let timerInterval = null;
    let remainingSeconds = 0;
    let currentTaskIndex = null;

    function saveTasks() {
      localStorage.setItem("studyTasks", JSON.stringify(tasks));
    }

    function addTask() {
      const name = document.getElementById("taskName").value.trim();
      const type = document.getElementById("taskType").value;
      const time = Number(document.getElementById("taskTime").value);

      if (!name || !time || time <= 0) {
        alert("اكتب اسم المهمة والمدة المطلوبة");
        return;
      }

      tasks.push({
        name: name,
        type: type,
        time: time,
        done: false
      });

      saveTasks();
      showTasks();

      document.getElementById("taskName").value = "";
      document.getElementById("taskTime").value = "";
    }

    function showTasks() {
      const box = document.getElementById("tasks");

      if (tasks.length === 0) {
        box.innerHTML = `<p class="empty">لم تضف أي مهام حتى الآن</p>`;
        return;
      }

      box.innerHTML = "";

      tasks.forEach((task, index) => {
        const div = document.createElement("div");
        div.className = task.done ? "task done" : "task";

        div.innerHTML = `
          <div class="task-title">${task.name}</div>
          <div class="task-info">
            النوع: ${task.type} | المدة: ${task.time} دقيقة
          </div>

          <div class="task-buttons">
            <button class="start" onclick="startTimer(${index})">
              ▶️ ابدأ
            </button>

            <button class="finish" onclick="finishTask(${index})">
              ✅ ${task.done ? "تم الإنجاز" : "إنهاء"}
            </button>

            <button class="remove" onclick="deleteTask(${index})">
              🗑️ حذف
            </button>
          </div>
        `;

        box.appendChild(div);
      });
    }

    function startTimer(index) {
      clearInterval(timerInterval);

      currentTaskIndex = index;
      remainingSeconds = tasks[index].time * 60;

      updateTimer();

      timerInterval = setInterval(() => {
        remainingSeconds--;
        updateTimer();

        if (remainingSeconds <= 0) {
          clearInterval(timerInterval);
          alert("🎉 انتهى وقت المهمة: " + tasks[index].name);
          finishTask(index);
        }
      }, 1000);
    }

    function updateTimer() {
      const minutes = Math.floor(remainingSeconds / 60);
      const seconds = remainingSeconds % 60;

      document.getElementById("timer").textContent =
        String(minutes).padStart(2, "0") + ":" +
        String(seconds).padStart(2, "0");
    }

    function pauseTimer() {
      clearInterval(timerInterval);
    }

    function resetTimer() {
      clearInterval(timerInterval);
      remainingSeconds = 0;
      currentTaskIndex = null;
      document.getElementById("timer").textContent = "00:00";
    }

    function finishTask(index) {
      tasks[index].done = true;
      saveTasks();
      showTasks();
    }

    function deleteTask(index) {
      if (confirm("هل تريد حذف هذه المهمة؟")) {
        tasks.splice(index, 1);
        saveTasks();
        showTasks();
      }
    }

    showTasks();
  </script>

</body>
</html>
