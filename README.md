<!DOCTYPE html>
<html lang="fi">
<head>
  <meta charset="UTF-8">
  <title>Gym Trainer</title>
  <style>
    body {
      font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
      background: linear-gradient(to bottom right, #e0eafc, #cfdef3);
      margin: 0;
      padding: 0;
      color: #333;
    }
    header {
      background-color: #0a2540;
      color: white;
      padding: 20px;
      text-align: center;
      background-image: url('https://images.unsplash.com/photo-1571019613454-1cb2f99b2d8b');
      background-size: cover;
      background-position: center;
    }
    header h1 {
      margin: 0;
      font-size: 2.5em;
      text-shadow: 2px 2px 5px rgba(0,0,0,0.5);
    }
    main {
      max-width: 900px;
      margin: auto;
      padding: 20px;
    }
    .hidden {
      display: none;
    }
    input[type="text"], input[type="number"] {
      padding: 8px;
      border-radius: 5px;
      border: 1px solid #ccc;
      margin: 5px 0;
      width: 200px;
    }
    button {
      background-color: #0a2540;
      color: white;
      border: none;
      padding: 10px 20px;
      border-radius: 8px;
      cursor: pointer;
      margin-top: 10px;
    }
    button:hover {
      background-color: #103d68;
    }
    .section {
      background: white;
      padding: 20px;
      border-radius: 12px;
      box-shadow: 0 2px 6px rgba(0, 0, 0, 0.1);
      margin-top: 20px;
    }
    .exercise-img {
      width: 100px;
      height: auto;
      border-radius: 8px;
      margin-right: 10px;
    }
    .exercise-row {
      display: flex;
      align-items: center;
      margin: 5px 0;
    }
    nav {
      text-align: center;
      margin-bottom: 20px;
    }
    nav button {
      margin: 0 5px;
    }
  </style>
</head>
<body>
  <header>
    <h1>🏋️ Gym Trainer</h1>
  </header>
  <main>
    <nav>
      <button onclick="showSection('login')">Aloitus</button>
      <button onclick="showSection('workout')">Treeni</button>
      <button onclick="showSection('records')">Ennätykset</button>
      <button onclick="showSection('history')">Historia</button>
    </nav>

    <div id="loginSection" class="section">
      <h2>Luo käyttäjä</h2>
      <input type="text" id="usernameInput" placeholder="Käyttäjänimi">
      <button onclick="createUser()">Aloita</button>
    </div>

    <div id="appSection" class="hidden">
      <h2>Tervetuloa, <span id="usernameDisplay"></span>!</h2>

      <div class="section" id="exerciseSelect">
        <h3>Valitse liikkeet:</h3>
        <div id="exerciseList"></div>
        <button onclick="startWorkout()">Aloita treeni</button>
      </div>

      <div class="section hidden" id="workoutSection">
        <h3>Kirjaa sarjat:</h3>
        <div id="workoutArea"></div>
        <button onclick="saveWorkout()">Tallenna treeni</button>
      </div>

      <div class="section hidden" id="recordsSection">
        <h3>📈 Ennätystulokset</h3>
        <div id="recordsArea"></div>
      </div>

      <div class="section hidden" id="historySection">
        <h3>📅 Treenihistoria</h3>
        <div id="historyArea"></div>
      </div>
    </div>
  </main>

  <script>
    let currentUser = null;
    let data = {};
    const allExercises = [
      { name: "Kyykky", img: "https://images.unsplash.com/photo-1605296867304-46d5465a13f1" },
      { name: "Penkki", img: "https://images.unsplash.com/photo-1583454110551-21aa4fd94286" },
      { name: "Maastaveto", img: "https://images.unsplash.com/photo-1605296867304-46d5465a13f1" },
      { name: "Pystypunnerrus", img: "https://images.unsplash.com/photo-1612320241894-3b92f8450008" },
      { name: "Leuanveto", img: "https://images.unsplash.com/photo-1622331773696-2482a43877d1" },
      { name: "Kulmasoutu", img: "https://images.unsplash.com/photo-1608460807752-6eeefcfd7a88" },
      { name: "Jalkaprässi", img: "https://images.unsplash.com/photo-1583454158990-9cd7a9b93db9" }
    ];

    function saveData() {
      localStorage.setItem("gymAppData", JSON.stringify(data));
    }

    function loadData() {
      const stored = localStorage.getItem("gymAppData");
      if (stored) data = JSON.parse(stored);

      const savedUser = localStorage.getItem("gymAppCurrentUser");
      if (savedUser) {
        currentUser = savedUser;
        showApp();
      }
    }

    function createUser() {
      const username = document.getElementById("usernameInput").value.trim();
      if (!username) return alert("Anna käyttäjänimi.");
      currentUser = username;
      if (!data[username]) data[username] = { records: {}, history: [] };
      localStorage.setItem("gymAppCurrentUser", username);
      saveData();
      showApp();
    }

    function showApp() {
      document.getElementById("loginSection").classList.add("hidden");
      document.getElementById("appSection").classList.remove("hidden");
      document.getElementById("usernameDisplay").textContent = currentUser;
      populateExercises();
    }

    function populateExercises() {
      const list = document.getElementById("exerciseList");
      list.innerHTML = "";
      allExercises.forEach(ex => {
        list.innerHTML += `
          <div class="exercise-row">
            <img class="exercise-img" src="${ex.img}" alt="${ex.name}">
            <label><input type="checkbox" value="${ex.name}"> ${ex.name}</label>
          </div>
        `;
      });
    }

    function startWorkout() {
      const selected = Array.from(document.querySelectorAll('input[type="checkbox"]:checked')).map(cb => cb.value);
      if (selected.length === 0) return alert("Valitse vähintään yksi liike!");

      const area = document.getElementById("workoutArea");
      area.innerHTML = "";

      selected.forEach(ex => {
        area.innerHTML += `<div><h4>${ex}</h4><input type='number' placeholder='kg'><input type='number' placeholder='toistot'></div>`;
      });

      showSection('workout');
    }

    function saveWorkout() {
      const inputs = document.querySelectorAll('#workoutArea div');
      const historyEntry = [];
      inputs.forEach(div => {
        const [kg, reps] = div.querySelectorAll('input');
        const exercise = div.querySelector('h4').textContent;
        const weight = parseInt(kg.value);
        const repetitions = parseInt(reps.value);
        if (!isNaN(weight) && !isNaN(repetitions)) {
          historyEntry.push({ exercise, weight, repetitions });
          const record = data[currentUser].records[exercise] || 0;
          if (weight > record) data[currentUser].records[exercise] = weight;
        }
      });

      data[currentUser].history.push({ date: new Date().toLocaleDateString(), entries: historyEntry });
      saveData();
      showSection('records');
      showRecords();
      showHistory();
    }

    function showRecords() {
      const area = document.getElementById("recordsArea");
      area.innerHTML = "";
      for (const [ex, val] of Object.entries(data[currentUser].records)) {
        area.innerHTML += `<p><strong>${ex}</strong>: ${val} kg</p>`;
      }
    }

    function showHistory() {
      const area = document.getElementById("historyArea");
      area.innerHTML = "";
      data[currentUser].history.slice().reverse().forEach(h => {
        area.innerHTML += `<div><strong>${h.date}</strong><ul>${h.entries.map(e => `<li>${e.exercise}: ${e.weight} kg × ${e.repetitions}</li>`).join('')}</ul></div>`;
      });
    }

    function showSection(section) {
      document.getElementById("exerciseSelect").classList.add("hidden");
      document.getElementById("workoutSection").classList.add("hidden");
      document.getElementById("recordsSection").classList.add("hidden");
      document.getElementById("historySection").classList.add("hidden");

      if (section === 'login') {
        document.getElementById("loginSection").classList.remove("hidden");
        document.getElementById("appSection").classList.add("hidden");
      } else {
        document.getElementById("loginSection").classList.add("hidden");
        document.getElementById("appSection").classList.remove("hidden");
        document.getElementById(`${section}Section`).classList.remove("hidden");
      }
    }

    loadData();
  </script>
</body>
</html>
