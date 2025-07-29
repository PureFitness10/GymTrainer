<!DOCTYPE html>
<html lang="fi">
<head>
  <meta charset="UTF-8">
  <title>Gym Trainer</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(to right, #f0f4f8, #e0f7fa);
      margin: 0;
      padding: 0;
      color: #333;
    }
    header {
      background: #00796b;
      color: white;
      padding: 20px;
      text-align: center;
      box-shadow: 0 2px 8px rgba(0, 0, 0, 0.2);
    }
    main {
      max-width: 800px;
      margin: 40px auto;
      padding: 20px;
      background: white;
      border-radius: 12px;
      box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
    }
    h2 {
      color: #00796b;
    }
    input[type="text"] {
      padding: 8px;
      width: 60%;
      font-size: 1em;
      margin: 10px 0;
    }
    button {
      padding: 10px 20px;
      background: #00796b;
      color: white;
      border: none;
      border-radius: 6px;
      cursor: pointer;
      font-size: 1em;
    }
    button:hover {
      background: #004d40;
    }
    .section {
      margin-top: 30px;
    }
    .exercise-list label {
      display: inline-block;
      margin-right: 12px;
      margin-bottom: 8px;
    }
    .hidden {
      display: none;
    }
    .record-item, .history-item {
      padding: 8px;
      margin: 4px 0;
      background: #f1f8e9;
      border-radius: 6px;
    }
    .exercise-entry {
      margin-bottom: 10px;
    }
    .exercise-entry input {
      width: 60px;
      margin-right: 5px;
    }
  </style>
</head>
<body>
  <header>
    <h1>🏋️ Gym Trainer</h1>
  </header>
  <main>
    <div id="loginSection">
      <h2>Luo käyttäjä</h2>
      <input type="text" id="usernameInput" placeholder="Käyttäjänimi">
      <button onclick="createUser()">Aloita</button>
    </div>

    <div id="appSection" class="hidden">
      <h2>Tervetuloa, <span id="usernameDisplay"></span>!</h2>

      <div class="section">
        <h3>1. Valitse liikkeet:</h3>
        <div class="exercise-list" id="exerciseList"></div>
        <button onclick="startWorkout()">Aloita treeni</button>
      </div>

      <div class="section hidden" id="workoutSection">
        <h3>2. Kirjaa sarjat:</h3>
        <div id="workoutArea"></div>
        <button onclick="saveWorkout()">💾 Tallenna treeni</button>
      </div>

      <div class="section">
        <h3>📈 Ennätystulokset</h3>
        <div id="recordsArea"></div>
      </div>

      <div class="section">
        <h3>📅 Treenihistoria</h3>
        <div id="historyArea"></div>
      </div>
    </div>
  </main>

  <script>
    let currentUser = null;
    let data = {};
    const allExercises = [
      "Kyykky", "Penkki", "Maastaveto", "Leuanveto", "Pystypunnerrus",
      "Hauiskääntö", "Ojentajapunnerrus", "Pystysoutu", "Jalkaprässi", "Askelkyykky",
      "Vatsarutistus", "Lankku", "Vatsapyörä", "Punnerrus", "Dipit",
      "Kulmasoutu", "Ylätalja", "Alatalja", "Ranskalainen punnerrus", "Hammer curl"
    ];

    function saveData() {
      localStorage.setItem("gymAppData", JSON.stringify(data));
    }

    function loadData() {
      const stored = localStorage.getItem("gymAppData");
      if (stored) data = JSON.parse(stored);
      const user = localStorage.getItem("gymAppCurrentUser");
      if (user && data[user]) {
        currentUser = user;
        showApp();
      }
    }

    function createUser() {
      const username = document.getElementById("usernameInput").value.trim();
      if (!username) return alert("Anna käyttäjänimi.");
      currentUser = username;
      if (!data[username]) {
        data[username] = { records: {}, history: [] };
        saveData();
      }
      localStorage.setItem("gymAppCurrentUser", username);
      showApp();
    }

    function showApp() {
      document.getElementById("loginSection").classList.add("hidden");
      document.getElementById("appSection").classList.remove("hidden");
      document.getElementById("usernameDisplay").textContent = currentUser;
      const list = document.getElementById("exerciseList");
      list.innerHTML = allExercises.map(ex => `
        <label><input type="checkbox" value="${ex}"> ${ex}</label>
      `).join("");
      showRecords();
      showHistory();
    }

    function startWorkout() {
      const selected = Array.from(document.querySelectorAll('input[type="checkbox"]:checked')).map(cb => cb.value);
      if (selected.length === 0) return alert("Valitse vähintään yksi liike!");

      const area = document.getElementById("workoutArea");
      area.innerHTML = selected.map(ex => `
        <div class="exercise-entry">
          <h4>${ex}</h4>
          Sarjat: <input type="number" min="1" placeholder="3">
          Toistot: <input type="number" min="1" placeholder="10">
          Paino (kg): <input type="number" min="0" placeholder="50">
        </div>
      `).join("");
      document.getElementById("workoutSection").classList.remove("hidden");
    }

    function saveWorkout() {
      const entries = document.querySelectorAll(".exercise-entry");
      const workout = [];
      entries.forEach(entry => {
        const name = entry.querySelector("h4").textContent;
        const sets = parseInt(entry.querySelectorAll("input")[0].value) || 0;
        const reps = parseInt(entry.querySelectorAll("input")[1].value) || 0;
        const weight = parseInt(entry.querySelectorAll("input")[2].value) || 0;
        workout.push({ name, sets, reps, weight });
        const record = data[currentUser].records[name];
        if (!record || weight > record) data[currentUser].records[name] = weight;
      });
      data[currentUser].history.push({ date: new Date().toLocaleString(), workout });
      saveData();
      showRecords();
      showHistory();
      alert("Treenit tallennettu!");
    }

    function showRecords() {
      const area = document.getElementById("recordsArea");
      const records = data[currentUser].records;
      area.innerHTML = Object.keys(records).map(name => `
        <div class="record-item">${name}: ${records[name]} kg</div>
      `).join("");
    }

    function showHistory() {
      const area = document.getElementById("historyArea");
      const history = data[currentUser].history;
      area.innerHTML = history.map(entry => `
        <div class="history-item">
          <strong>${entry.date}</strong><br>
          ${entry.workout.map(w => `${w.name}: ${w.sets}x${w.reps} @ ${w.weight}kg`).join("<br>")}
        </div>
      `).join("");
    }

    loadData();
  </script>
</body>
</html>
