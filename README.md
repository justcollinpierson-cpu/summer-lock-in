<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=yes" />
  <title>Summer Lock‑In Command Center</title>
  <!-- Chart.js for progress graphs -->
  <script src="https://cdn.jsdelivr.net/npm/chart.js@4.4.0/dist/chart.umd.min.js"></script>
  <!-- html5-qrcode for barcode scanning -->
  <script src="https://unpkg.com/html5-qrcode"></script>
  <style>
    * { margin: 0; padding: 0; box-sizing: border-box; }
    body {
      background: #121212;
      color: #e0e0e0;
      font-family: 'Segoe UI', system-ui, -apple-system, sans-serif;
      display: flex;
      flex-direction: column;
      min-height: 100vh;
    }
    .header {
      background: #1e1e1e;
      border-bottom: 2px solid #00b894;
      padding: 1.2rem;
      text-align: center;
    }
    .header h1 { color: #00b894; font-size: 2rem; margin: 0; }
    .tabs {
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
      background: #1a1a1a;
      gap: 0.2rem;
    }
    .tab-btn {
      padding: 0.8rem 1.2rem;
      background: transparent;
      border: none;
      color: #aaa;
      font-weight: 600;
      cursor: pointer;
      transition: 0.2s;
    }
    .tab-btn:hover { color: #fff; }
    .tab-btn.active {
      color: #00b894;
      border-bottom: 3px solid #00b894;
    }
    .tab-content {
      display: none;
      padding: 1.5rem;
      max-width: 1000px;
      margin: 0 auto;
      width: 100%;
      flex: 1;
    }
    .tab-content.active { display: block; }
    .card {
      background: #1e1e1e;
      border-radius: 12px;
      padding: 1.5rem;
      margin-bottom: 1.5rem;
      border-left: 4px solid #00b894;
      box-shadow: 0 4px 12px rgba(0,0,0,0.4);
    }
    input, select, textarea, button {
      background: #2c2c2c;
      border: 1px solid #444;
      color: #fff;
      padding: 0.7rem;
      border-radius: 8px;
      font-size: 1rem;
      outline: none;
      margin: 0.3rem 0;
    }
    button {
      background: #00b894;
      border: none;
      color: #121212;
      font-weight: bold;
      cursor: pointer;
      transition: background 0.2s;
    }
    button:hover { background: #009a7b; }
    .btn-secondary { background: #3a3a3a; color: #fff; }
    .btn-secondary:hover { background: #4a4a4a; }
    .flex-row { display: flex; gap: 0.5rem; flex-wrap: wrap; align-items: center; }
    .macro-bar {
      display: flex;
      gap: 1rem;
      flex-wrap: wrap;
      margin: 1rem 0;
    }
    .macro-item {
      background: #2c2c2c;
      padding: 0.8rem;
      border-radius: 8px;
      flex: 1;
      min-width: 100px;
      text-align: center;
    }
    .macro-item span { display: block; font-size: 0.8rem; color: #aaa; }
    .macro-item strong { font-size: 1.4rem; color: #00b894; }
    .list-item {
      background: #2c2c2c;
      padding: 0.8rem;
      border-radius: 8px;
      margin-bottom: 0.5rem;
      display: flex;
      justify-content: space-between;
      align-items: center;
    }
    .hidden { display: none; }
    canvas { margin: 1rem 0; }
    .recipe-detail {
      background: #2a2a2a;
      padding: 1rem;
      border-radius: 8px;
      margin-top: 0.5rem;
    }
    .recipe-img {
      width: 100%;
      height: 180px;
      background: linear-gradient(135deg, #2d3436, #636e72);
      border-radius: 8px;
      display: flex;
      align-items: center;
      justify-content: center;
      color: #aaa;
      font-size: 0.9rem;
      margin-bottom: 0.8rem;
    }
    .exercise-log { display: flex; gap: 0.3rem; margin: 0.3rem 0; }
    @media (max-width: 600px) {
      .tab-btn { padding: 0.6rem 0.8rem; font-size: 0.9rem; }
    }
  </style>
</head>
<body>
  <div class="header">
    <h1>💀 SUMMER LOCK-IN COMMAND CENTER 💀</h1>
  </div>

  <div class="tabs" id="tab-bar">
    <button class="tab-btn active" data-tab="dashboard">📊 Dashboard</button>
    <button class="tab-btn" data-tab="workout">🏋️ Workout</button>
    <button class="tab-btn" data-tab="diet">🥗 Diet</button>
    <button class="tab-btn" data-tab="recipes">📖 Recipes</button>
    <button class="tab-btn" data-tab="finance">💰 Finance</button>
    <button class="tab-btn" data-tab="progress">📈 Progress</button>
    <button class="tab-btn" data-tab="settings">⚙️ Settings</button>
  </div>

  <script>
    /* ============================================
       GLOBAL APP STATE
       ============================================ */
    const APP = {
      currentProfile: null,
      profiles: {},
      today: new Date().toISOString().split('T')[0],
      recipeDB: [
        { name: "Protein Pancakes", macros: { cal: 420, p: 42, c: 40, f: 12 }, ingredients: "1 scoop whey, 1 egg, 1/2 cup oats, 1/2 banana, baking powder", instructions: "1. Blend all ingredients until smooth.\n2. Pour onto a medium‑hot skillet.\n3. Cook until bubbles form, then flip.\n4. Serve with Greek yogurt or sugar‑free syrup." },
        { name: "Chicken & Rice Bowl", macros: { cal: 650, p: 55, c: 55, f: 15 }, ingredients: "8oz chicken breast, 1 cup cooked white rice, 1 cup steamed broccoli", instructions: "1. Season chicken with salt, pepper, garlic powder.\n2. Grill or pan‑sear until cooked through.\n3. Serve over rice with broccoli." },
        { name: "Tuna Avocado Wrap", macros: { cal: 500, p: 35, c: 30, f: 22 }, ingredients: "1 can tuna, 1 ripe avocado, 1 whole wheat wrap", instructions: "1. Drain tuna and mash with avocado.\n2. Spread onto wrap and roll tightly." },
        { name: "Salmon & Asparagus", macros: { cal: 580, p: 48, c: 10, f: 30 }, ingredients: "6oz salmon fillet, 10 asparagus spears, olive oil, lemon", instructions: "1. Preheat oven to 400°F.\n2. Place salmon and asparagus on a baking sheet.\n3. Drizzle with olive oil, salt, pepper.\n4. Bake 18‑20 minutes." },
        { name: "Beef Stir Fry", macros: { cal: 620, p: 50, c: 45, f: 20 }, ingredients: "6oz lean beef strips, mixed bell peppers, broccoli, soy sauce", instructions: "1. Sear beef strips in a hot pan.\n2. Add vegetables and stir‑fry.\n3. Add soy sauce and cook until veggies are tender." },
        { name: "Egg White Omelet", macros: { cal: 350, p: 40, c: 10, f: 14 }, ingredients: "6 egg whites, handful spinach, 1/4 cup feta", instructions: "1. Whisk egg whites and pour into a non‑stick pan.\n2. Add spinach and feta, fold and cook through." },
        { name: "Protein Oatmeal", macros: { cal: 500, p: 38, c: 55, f: 14 }, ingredients: "1 cup oats, 1 scoop protein powder, 1 tbsp peanut butter", instructions: "1. Cook oats with water or milk.\n2. Stir in protein powder and peanut butter." },
        { name: "Turkey Meatballs", macros: { cal: 480, p: 45, c: 15, f: 22 }, ingredients: "1lb ground turkey, 1/4 cup breadcrumbs, 1 egg, seasoning", instructions: "1. Mix all ingredients and form balls.\n2. Bake at 375°F for 25 mins." },
        { name: "Shrimp Quinoa Salad", macros: { cal: 540, p: 40, c: 48, f: 18 }, ingredients: "6oz shrimp, 1 cup cooked quinoa, cucumber, tomatoes", instructions: "1. Grill shrimp with salt and pepper.\n2. Toss with quinoa and chopped veggies." },
        { name: "Cottage Cheese Bowl", macros: { cal: 400, p: 42, c: 28, f: 12 }, ingredients: "1 cup cottage cheese, mixed berries, almonds", instructions: "1. Combine all ingredients in a bowl." },
        { name: "Steak & Sweet Potato", macros: { cal: 670, p: 55, c: 50, f: 22 }, ingredients: "6oz sirloin, 1 medium sweet potato, green beans", instructions: "1. Season steak and grill to desired doneness.\n2. Bake sweet potato at 400°F for 45 mins.\n3. Steam green beans." },
        { name: "Greek Yogurt Parfait", macros: { cal: 380, p: 32, c: 45, f: 10 }, ingredients: "1 cup Greek yogurt, 1/4 cup granola, berries", instructions: "1. Layer yogurt, granola, berries in a glass." },
        { name: "Bison Burger", macros: { cal: 550, p: 48, c: 30, f: 22 }, ingredients: "6oz bison patty, whole grain bun, lettuce, tomato", instructions: "1. Grill patty.\n2. Assemble burger." },
        { name: "Lentil & Chicken Soup", macros: { cal: 450, p: 40, c: 38, f: 14 }, ingredients: "1 chicken breast, 1/2 cup lentils, carrots, celery", instructions: "1. Simmer all ingredients in broth for 30 mins." },
        { name: "Tofu Scramble", macros: { cal: 400, p: 30, c: 25, f: 18 }, ingredients: "1 block firm tofu, turmeric, bell peppers", instructions: "1. Crumble tofu and sauté with turmeric and veggies." },
        { name: "Pork Tenderloin", macros: { cal: 520, p: 50, c: 15, f: 25 }, ingredients: "6oz pork tenderloin, Brussels sprouts", instructions: "1. Sear pork, then roast at 400°F with sprouts." },
        { name: "Cod & Veggies", macros: { cal: 490, p: 45, c: 20, f: 22 }, ingredients: "6oz cod fillet, zucchini, lemon", instructions: "1. Bake cod and sliced zucchini with olive oil." },
        { name: "Egg Muffins", macros: { cal: 380, p: 30, c: 8, f: 24 }, ingredients: "4 eggs, diced ham, cheese, bell pepper", instructions: "1. Whisk eggs, pour into muffin tin, add fillings.\n2. Bake at 350°F for 20 mins." },
        { name: "Peanut Butter Banana Shake", macros: { cal: 450, p: 35, c: 40, f: 18 }, ingredients: "2 scoops protein, 1 banana, 2 tbsp PB, milk", instructions: "1. Blend until smooth." },
        { name: "Chicken Fajita Bowl", macros: { cal: 600, p: 50, c: 45, f: 18 }, ingredients: "chicken, bell peppers, onions, black beans, rice", instructions: "1. Sauté chicken and veggies.\n2. Serve over rice and beans." }
      ]
    };

    function getDailyTop5() {
      let seed = new Date().toISOString().slice(0,10);
      let hash = 0;
      for (let i=0; i<seed.length; i++) hash = ((hash<<5)-hash)+seed.charCodeAt(i);
      let idx = Math.abs(hash) % APP.recipeDB.length;
      let picks = [];
      for (let i=0; i<5; i++) picks.push(APP.recipeDB[(idx+i) % APP.recipeDB.length]);
      return picks;
    }

    function switchTab(tabName) {
      document.querySelectorAll('.tab-content').forEach(el => el.classList.remove('active'));
      document.getElementById('tab-'+tabName).classList.add('active');
      document.querySelectorAll('.tab-btn').forEach(btn => btn.classList.remove('active'));
      document.querySelector(`.tab-btn[data-tab="${tabName}"]`).classList.add('active');
    }

    function loadProfile(name) {
      let stored = localStorage.getItem('lockin_profile_'+name);
      if (stored) {
        APP.currentProfile = name;
        APP.profiles[name] = JSON.parse(stored);
        return true;
      }
      return false;
    }

    function saveCurrentProfile() {
      if (!APP.currentProfile) return;
      localStorage.setItem('lockin_profile_'+APP.currentProfile, JSON.stringify(APP.profiles[APP.currentProfile]));
    }

    function updateAllUI() {
      if (!APP.currentProfile) return;
      let p = APP.profiles[APP.currentProfile];
      document.getElementById('dash-name').textContent = p.name;
      document.getElementById('dash-goal').textContent = p.goal;
      let todayLog = p.foodLogs[APP.today] || [];
      let consumed = { cal:0, protein:0, carbs:0, fat:0 };
      todayLog.forEach(f => {
        consumed.cal += f.macros.cal || 0;
        consumed.protein += f.macros.protein || 0;
        consumed.carbs += f.macros.carbs || 0;
        consumed.fat += f.macros.fat || 0;
      });
      document.getElementById('consumed-cal').textContent = consumed.cal;
      document.getElementById('remaining-cal').textContent = p.macroTargets.cal - consumed.cal;
      if (p.weightLog.length>0) {
        let last = p.weightLog[p.weightLog.length-1].weight;
        document.getElementById('current-weight').textContent = last;
      }
      document.getElementById('finance-bal').textContent = p.financeBalance || 0;
    }

    // ========== INITIALIZATION ==========
    window.addEventListener('load', () => {
      document.body.insertAdjacentHTML('beforeend', `
        <div class="tab-content active" id="tab-dashboard"></div>
        <div class="tab-content" id="tab-workout"></div>
        <div class="tab-content" id="tab-diet"></div>
        <div class="tab-content" id="tab-recipes"></div>
        <div class="tab-content" id="tab-finance"></div>
        <div class="tab-content" id="tab-progress"></div>
        <div class="tab-content" id="tab-settings"></div>
      `);
      let existingProfiles = [];
      for (let i=0; i<localStorage.length; i++) {
        let key = localStorage.key(i);
        if (key.startsWith('lockin_profile_')) existingProfiles.push(key.replace('lockin_profile_',''));
      }
      if (existingProfiles.length > 0) {
        let selHtml = existingProfiles.map(n => `<option value="${n}">${n}</option>`).join('');
        document.getElementById('tab-settings').innerHTML = `
          <div class="card">
            <h2>Select Profile</h2>
            <select id="profile-select">${selHtml}</select>
            <button id="load-profile-btn">Load</button>
            <button class="btn-secondary" id="new-profile-btn">Create New</button>
            <br/><br/>
            <button id="export-btn">⬇ Export All Data</button>
            <input type="file" id="import-file" accept=".json" hidden />
            <button class="btn-secondary" id="import-btn">⬆ Import Data</button>
          </div>`;
        document.getElementById('load-profile-btn').onclick = () => {
          let sel = document.getElementById('profile-select').value;
          if (loadProfile(sel)) buildAllTabs();
        };
        document.getElementById('new-profile-btn').onclick = showNewProfileForm;
      } else {
        showNewProfileForm();
      }
      document.querySelectorAll('.tab-btn').forEach(btn => {
        btn.addEventListener('click', (e) => switchTab(e.target.dataset.tab));
      });
    });

    function showNewProfileForm() {
      document.getElementById('tab-settings').innerHTML = `
        <div class="card">
          <h2>Create Your Profile</h2>
          <label>Profile Name:</label>
          <input id="new-name" placeholder="Your name" required />
          <label>Age:</label>
          <input type="number" id="new-age" required />
          <label>Weight (lbs):</label>
          <input type="number" id="new-weight" required />
          <label>Height (inches):</label>
          <input type="number" id="new-height" required />
          <label>Sex:</label>
          <select id="new-sex"><option value="male">Male</option><option value="female">Female</option></select>
          <label>Goal:</label>
          <select id="new-goal"><option value="bulk">Bulk (+300 cal)</option><option value="cut">Cut (-500 cal)</option><option value="maintain">Maintain</option></select>
          <br/><br/>
          <button id="create-profile-btn">Create Profile</button>
        </div>`;
      document.getElementById('create-profile-btn').onclick = () => {
        let name = document.getElementById('new-name').value.trim();
        if (!name) return alert('Enter a name');
        let age = parseInt(document.getElementById('new-age').value);
        let weight = parseFloat(document.getElementById('new-weight').value);
        let height = parseFloat(document.getElementById('new-height').value);
        let sex = document.getElementById('new-sex').value;
        let goal = document.getElementById('new-goal').value;
        if (isNaN(age) || isNaN(weight) || isNaN(height)) return alert('Enter valid numbers');
        // Mifflin-St Jeor BMR
        let bmr;
        if (sex==='male') {
          bmr = 10 * weight * 0.453592 + 6.25 * height * 2.54 - 5 * age + 5;
        } else {
          bmr = 10 * weight * 0.453592 + 6.25 * height * 2.54 - 5 * age - 161;
        }
        let tdee = Math.round(bmr * 1.55);
        let calTarget;
        if (goal==='bulk') calTarget = tdee + 300;
        else if (goal==='cut') calTarget = tdee - 500;
        else calTarget = tdee;
        let protein, carbs, fat;
        if (goal==='bulk') {
          protein = Math.round(0.3 * calTarget / 4);
          carbs = Math.round(0.4 * calTarget / 4);
          fat = Math.round(0.3 * calTarget / 9);
        } else if (goal==='cut') {
          protein = Math.round(0.4 * calTarget / 4);
          carbs = Math.round(0.3 * calTarget / 4);
          fat = Math.round(0.3 * calTarget / 9);
        } else {
          protein = Math.round(0.3 * calTarget / 4);
          carbs = Math.round(0.4 * calTarget / 4);
          fat = Math.round(0.3 * calTarget / 9);
        }
        APP.profiles[name] = {
          name, age, weight, height, sex, goal,
          startWeight: weight,
          startDate: new Date().toISOString().slice(0,10),
          macroTargets: { cal: calTarget, protein, carbs, fat },
          foodLogs: {},
          transactions: [],
          weightLog: [],
          liftPRs: [],
          workoutSchedule: ['Push','Pull','Legs','Push','Pull','Legs','Rest'],
          workoutStartDate: new Date().toISOString().slice(0,10),
          customFoods: [],
          customRecipes: [],
          financeBalance: 0
        };
        localStorage.setItem('lockin_profile_'+name, JSON.stringify(APP.profiles[name]));
        APP.currentProfile = name;
        buildAllTabs();
      };
    }

    function buildAllTabs() {
      if (!APP.currentProfile) return;
      // Dashboard
      document.getElementById('tab-dashboard').innerHTML = `
        <div class="card">
          <h2>Welcome, <span id="dash-name"></span> | Goal: <span id="dash-goal"></span></h2>
          <div class="macro-bar">
            <div class="macro-item"><span>Calories Consumed</span><strong id="consumed-cal">0</strong></div>
            <div class="macro-item"><span>Remaining</span><strong id="remaining-cal">0</strong></div>
          </div>
        </div>
        <div class="card">
          <h3>Current Weight: <span id="current-weight">--</span> lbs</h3>
          <h3>Balance: $<span id="finance-bal">0</span></h3>
        </div>`;
      updateAllUI();
      buildWorkoutTab();
      buildDietTab();
      buildRecipesTab();
      buildFinanceTab();
      buildProgressTab();
      buildSettingsTab();
      switchTab('dashboard');
    }

    // -------- WORKOUT TAB (UPDATED) ----------
    function buildWorkoutTab() {
      let p = APP.profiles[APP.currentProfile];
      let schedule = p.workoutSchedule;
      let start = new Date(p.workoutStartDate + 'T00:00:00');
      let todayDate = new Date(APP.today + 'T00:00:00');
      let diffDays = Math.floor((todayDate - start) / (1000*60*60*24));
      let dayIndex = ((diffDays % schedule.length) + schedule.length) % schedule.length;
      let workoutName = schedule[dayIndex];
      let dayNumber = diffDays + 1;
      let html = `
        <div class="card">
          <h2>Today: ${todayDate.toDateString()} | Day ${dayNumber}</h2>
          <h3>Scheduled: <span style="color:#00b894">${workoutName === 'Rest' ? 'Rest Day' : workoutName}</span></h3>
          <div id="workout-exercises"></div>
          <button id="log-workout-btn" ${workoutName==='Rest'?'style="display:none"':''}>Log Exercises</button>
          <div id="workout-log-area" class="hidden"></div>
        </div>`;
      document.getElementById('tab-workout').innerHTML = html;
      let exDiv = document.getElementById('workout-exercises');
      if (workoutName !== 'Rest') {
        let exercises = getExercisesForWorkout(workoutName);
        let list = exercises.map(e => `<li>${e.name}: ${e.sets} sets × ${e.reps}</li>`).join('');
        exDiv.innerHTML = `<ul>${list}</ul>`;
        document.getElementById('log-workout-btn').onclick = () => {
          document.getElementById('workout-log-area').classList.toggle('hidden');
          document.getElementById('workout-log-area').innerHTML = exercises.map(e => `
            <div class="exercise-log">
              <span>${e.name}: </span>
              <input type="number" placeholder="Weight" id="weight-${e.name.replace(/\s/g,'')}" />
              <input type="number" placeholder="Sets" id="sets-${e.name.replace(/\s/g,'')}" />
              <input type="number" placeholder="Reps" id="reps-${e.name.replace(/\s/g,'')}" />
            </div>`).join('') + `<button id="save-logs-btn">Save Today's Workout</button>`;
          document.getElementById('save-logs-btn').onclick = () => {
            alert('Workout logged! (PRs can be added in Progress tab)');
            document.getElementById('workout-log-area').classList.add('hidden');
          };
        };
      } else {
        exDiv.innerHTML = '<p>Enjoy your rest day — stretch, foam roll, or work on stickhandling!</p>';
      }
    }

    function getExercisesForWorkout(type) {
      const exercises = {
        Push: [
          {name:'Barbell Bench Press', sets:4, reps:'6-8'},
          {name:'Overhead Dumbbell Press', sets:3, reps:'8-10'},
          {name:'Dips (or assisted)', sets:3, reps:'8-12'},
          {name:'Lateral Raises', sets:3, reps:'12-15'},
          {name:'Tricep Pushdowns', sets:3, reps:'10-12'},
          {name:'Hanging Leg Raises', sets:3, reps:'to failure'}
        ],
        Pull: [
          {name:'Deadlift', sets:3, reps:'5'},
          {name:'Pull‑Ups (or lat pulldown)', sets:3, reps:'8-10'},
          {name:'Barbell Row', sets:3, reps:'8-10'},
          {name:'Face Pulls', sets:3, reps:'12-15'},
          {name:'Barbell Curl', sets:3, reps:'10-12'},
          {name:'Hammer Curls', sets:2, reps:'12-15'}
        ],
        Legs: [
          {name:'Barbell Back Squat', sets:4, reps:'5-6'},
          {name:'Romanian Deadlift', sets:3, reps:'8-10'},
          {name:'Walking Lunges', sets:3, reps:'10 per leg'},
          {name:'Leg Press', sets:3, reps:'10-12'},
          {name:'Calf Raises', sets:4, reps:'15-20'},
          {name:'Plank', sets:3, reps:'60 sec'}
        ]
      };
      return exercises[type] || [];
    }

    // -------- DIET TAB (unchanged) ----------
    function buildDietTab() {
      document.getElementById('tab-diet').innerHTML = `
        <div class="card">
          <h2>Daily Food Log (${APP.today})</h2>
          <div class="flex-row">
            <input type="text" id="food-search" placeholder="Search food (e.g., banana)" />
            <button id="search-food-btn">Search</button>
          </div>
          <div class="flex-row">
            <input type="text" id="barcode-input" placeholder="Barcode number" />
            <button id="scan-barcode-btn">📷 Scan Barcode</button>
          </div>
          <div id="food-results"></div>
          <button class="btn-secondary" id="add-custom-food-btn">+ Add Custom Food</button>
          <div id="custom-food-form" class="hidden">
            <input type="text" id="cf-name" placeholder="Food name" />
            <input type="number" id="cf-cal" placeholder="Calories" />
            <input type="number" id="cf-protein" placeholder="Protein (g)" />
            <input type="number" id="cf-carbs" placeholder="Carbs (g)" />
            <input type="number" id="cf-fat" placeholder="Fat (g)" />
            <button id="save-custom-food">Save</button>
          </div>
          <h3>Today's Log:</h3>
          <div id="food-log-list"></div>
          <div class="macro-bar">
            <div class="macro-item"><span>Total Cal</span><strong id="today-total-cal">0</strong></div>
            <div class="macro-item"><span>Protein</span><strong id="today-total-p">0</strong></div>
            <div class="macro-item"><span>Carbs</span><strong id="today-total-c">0</strong></div>
            <div class="macro-item"><span>Fat</span><strong id="today-total-f">0</strong></div>
          </div>
        </div>`;
      renderFoodLog();
      document.getElementById('search-food-btn').onclick = searchOpenFoodFacts;
      document.getElementById('scan-barcode-btn').onclick = startBarcodeScanner;
      document.getElementById('add-custom-food-btn').onclick = () => document.getElementById('custom-food-form').classList.toggle('hidden');
      document.getElementById('save-custom-food').onclick = saveCustomFood;
      document.getElementById('barcode-input').addEventListener('keypress', (e) => {
        if (e.key === 'Enter') lookupBarcode(document.getElementById('barcode-input').value);
      });
    }

    function searchOpenFoodFacts() {
      let query = document.getElementById('food-search').value.trim();
      if (!query) return;
      fetch(`https://world.openfoodfacts.org/cgi/search.pl?search_terms=${encodeURIComponent(query)}&json=1&page_size=5`)
        .then(r=>r.json())
        .then(data => {
          let results = data.products.filter(p=>p.nutriments && p.product_name);
          let html = results.map(p => {
            let cal = p.nutriments['energy-kcal_100g'] ? Math.round(p.nutriments['energy-kcal_100g'] * (p.serving_size ? p.serving_size/100 : 1)) : '?';
            return `<div class="list-item"><span>${p.product_name} (${cal} kcal)</span><button data-barcode="${p.code}">Add</button></div>`;
          }).join('');
          document.getElementById('food-results').innerHTML = html;
          document.querySelectorAll('#food-results button').forEach(btn => btn.onclick = () => addFoodByBarcode(btn.dataset.barcode));
        });
    }

    function startBarcodeScanner() {
      const html5QrCode = new Html5Qrcode("reader");
      html5QrCode.start({ facingMode: "environment" }, { fps: 10, qrbox: 250 }, 
        (decodedText) => {
          document.getElementById('barcode-input').value = decodedText;
          html5QrCode.stop();
          document.getElementById('reader').remove();
          lookupBarcode(decodedText);
        },
        (error) => { /* ignore */ }
      ).catch(err => alert('Camera access failed. Enter barcode manually.'));
      let scannerDiv = document.createElement('div');
      scannerDiv.id = 'reader';
      document.getElementById('food-results').appendChild(scannerDiv);
    }

    function lookupBarcode(code) {
      fetch(`https://world.openfoodfacts.org/api/v0/product/${code}.json`)
        .then(r=>r.json())
        .then(data => {
          if (data.status===1) addFoodFromProduct(data.product);
          else alert('Product not found');
        });
    }

    function addFoodByBarcode(code) { lookupBarcode(code); }

    function addFoodFromProduct(product) {
      let serving = product.serving_size ? product.serving_size : 100;
      let factor = serving / 100;
      let cal = Math.round((product.nutriments['energy-kcal_100g'] || 0) * factor);
      let protein = Math.round((product.nutriments.proteins_100g || 0) * factor);
      let carbs = Math.round((product.nutriments.carbohydrates_100g || 0) * factor);
      let fat = Math.round((product.nutriments.fat_100g || 0) * factor);
      let foodItem = { name: product.product_name, barcode: product.code, serving: serving + 'g', macros: { cal, protein, carbs, fat } };
      addFoodToLog(foodItem);
    }

    function addFoodToLog(food) {
      let p = APP.profiles[APP.currentProfile];
      if (!p.foodLogs[APP.today]) p.foodLogs[APP.today] = [];
      p.foodLogs[APP.today].push(food);
      saveCurrentProfile();
      renderFoodLog();
      updateAllUI();
    }

    function saveCustomFood() {
      let name = document.getElementById('cf-name').value.trim();
      if (!name) return;
      let cal = parseInt(document.getElementById('cf-cal').value) || 0;
      let protein = parseInt(document.getElementById('cf-protein').value) || 0;
      let carbs = parseInt(document.getElementById('cf-carbs').value) || 0;
      let fat = parseInt(document.getElementById('cf-fat').value) || 0;
      let food = { name, macros: { cal, protein, carbs, fat }, custom: true };
      addFoodToLog(food);
      APP.profiles[APP.currentProfile].customFoods.push(food);
      saveCurrentProfile();
      document.getElementById('custom-food-form').classList.add('hidden');
    }

    function renderFoodLog() {
      let p = APP.profiles[APP.currentProfile];
      let log = p.foodLogs[APP.today] || [];
      let listHtml = log.map((f,i) => `
        <div class="list-item"><span>${f.name} (${f.macros.cal}kcal)</span><button data-index="${i}" class="remove-food">×</button></div>`).join('');
      document.getElementById('food-log-list').innerHTML = listHtml;
      document.querySelectorAll('.remove-food').forEach(btn => btn.onclick = function() {
        let idx = parseInt(this.dataset.index);
        p.foodLogs[APP.today].splice(idx,1);
        saveCurrentProfile();
        renderFoodLog();
        updateAllUI();
      });
      let totals = { cal:0, protein:0, carbs:0, fat:0 };
      log.forEach(f => { totals.cal += f.macros.cal; totals.protein += f.macros.protein; totals.carbs += f.macros.carbs; totals.fat += f.macros.fat; });
      document.getElementById('today-total-cal').textContent = totals.cal;
      document.getElementById('today-total-p').textContent = totals.protein+'g';
      document.getElementById('today-total-c').textContent = totals.carbs+'g';
      document.getElementById('today-total-f').textContent = totals.fat+'g';
    }

    // -------- RECIPES TAB (UPDATED WITH DETAIL VIEW) ----------
    function buildRecipesTab() {
      let top5 = getDailyTop5();
      let html = `
        <div class="card">
          <h2>🍳 Daily Top 5 Recipes</h2>
          ${top5.map(r => `<div class="list-item"><strong>${r.name}</strong> (${r.macros.cal}kcal) <button class="view-recipe-btn" data-name="${r.name}">View</button></div>`).join('')}
        </div>
        <div class="card">
          <h2>🔍 Search Recipes</h2>
          <input type="text" id="recipe-search" placeholder="e.g., pancakes, chicken" />
          <div id="recipe-results"></div>
          <div id="recipe-detail-container"></div>
        </div>
        <div class="card">
          <h3>➕ Add Your Own Recipe</h3>
          <input type="text" id="rec-name" placeholder="Recipe name" />
          <input type="text" id="rec-ingredients" placeholder="Ingredients" />
          <textarea id="rec-instructions" placeholder="Instructions"></textarea>
          <div class="flex-row">
            <input type="number" id="rec-cal" placeholder="Cal" />
            <input type="number" id="rec-protein" placeholder="Prot" />
            <input type="number" id="rec-carbs" placeholder="Carbs" />
            <input type="number" id="rec-fat" placeholder="Fat" />
          </div>
          <button id="save-recipe">Save Recipe</button>
        </div>`;
      document.getElementById('tab-recipes').innerHTML = html;
      document.getElementById('recipe-search').addEventListener('input', filterRecipes);
      document.getElementById('save-recipe').onclick = addCustomRecipe;
      // attach view buttons for top5
      document.querySelectorAll('.view-recipe-btn').forEach(btn => btn.onclick = () => showRecipeDetail(btn.dataset.name));
    }

    function filterRecipes() {
      let q = document.getElementById('recipe-search').value.toLowerCase();
      let allRecipes = APP.recipeDB.concat(APP.profiles[APP.currentProfile].customRecipes);
      let filtered = allRecipes.filter(r => r.name.toLowerCase().includes(q) || r.ingredients.toLowerCase().includes(q));
      document.getElementById('recipe-results').innerHTML = filtered.map(r => `
        <div class="list-item"><strong>${r.name}</strong> (${r.macros.cal}kcal) <button class="view-recipe-btn" data-name="${r.name}">View</button></div>`).join('');
      document.querySelectorAll('#recipe-results .view-recipe-btn').forEach(btn => btn.onclick = () => showRecipeDetail(btn.dataset.name));
    }

    function showRecipeDetail(name) {
      let allRecipes = APP.recipeDB.concat(APP.profiles[APP.currentProfile].customRecipes);
      let recipe = allRecipes.find(r => r.name === name);
      if (!recipe) return;
      document.getElementById('recipe-detail-container').innerHTML = `
        <div class="recipe-detail">
          <div class="recipe-img">🍽️ ${recipe.name}</div>
          <h3>${recipe.name}</h3>
          <p><strong>Macros:</strong> ${recipe.macros.cal} kcal | P:${recipe.macros.protein}g C:${recipe.macros.carbs}g F:${recipe.macros.fat}g</p>
          <p><strong>Ingredients:</strong> ${recipe.ingredients}</p>
          <p><strong>Instructions:</strong><br>${recipe.instructions.replace(/\n/g,'<br>')}</p>
          <button class="btn-secondary" onclick="document.getElementById('recipe-detail-container').innerHTML=''">Close</button>
        </div>
      `;
    }

    function addCustomRecipe() {
      let name = document.getElementById('rec-name').value.trim();
      if (!name) return;
      let ingredients = document.getElementById('rec-ingredients').value;
      let instructions = document.getElementById('rec-instructions').value;
      let cal = parseInt(document.getElementById('rec-cal').value)||0;
      let protein = parseInt(document.getElementById('rec-protein').value)||0;
      let carbs = parseInt(document.getElementById('rec-carbs').value)||0;
      let fat = parseInt(document.getElementById('rec-fat').value)||0;
      let recipe = { name, ingredients, instructions, macros: { cal, protein, carbs, fat } };
      APP.profiles[APP.currentProfile].customRecipes.push(recipe);
      saveCurrentProfile();
      alert('Recipe saved!');
      buildRecipesTab();
    }

    // -------- FINANCE TAB ----------
    function buildFinanceTab() {
      let p = APP.profiles[APP.currentProfile];
      document.getElementById('tab-finance').innerHTML = `
        <div class="card">
          <h2>💰 Balance: $<span id="fin-bal">${p.financeBalance || 0}</span></h2>
          <div class="flex-row">
            <input type="text" id="desc-input" placeholder="Description" />
            <input type="number" id="amount-input" placeholder="Amount (+ in, - out)" />
            <button id="add-transaction">Add</button>
          </div>
          <h3>Transaction History</h3>
          <div id="trans-list"></div>
        </div>`;
      renderTransactions();
      document.getElementById('add-transaction').onclick = () => {
        let desc = document.getElementById('desc-input').value.trim();
        let amt = parseFloat(document.getElementById('amount-input').value);
        if (!desc || isNaN(amt)) return;
        p.transactions.push({ date: APP.today, desc, amount: amt });
        p.financeBalance = (p.financeBalance || 0) + amt;
        saveCurrentProfile();
        renderTransactions();
        document.getElementById('fin-bal').textContent = p.financeBalance;
        updateAllUI();
      };
    }

    function renderTransactions() {
      let p = APP.profiles[APP.currentProfile];
      let list = p.transactions.slice().reverse();
      document.getElementById('trans-list').innerHTML = list.map(t => `
        <div class="list-item"><span>${t.date} - ${t.desc}</span> <span style="color:${t.amount>=0?'lightgreen':'#ff6b6b'}">${t.amount>=0?'+':''}$${t.amount}</span></div>`).join('');
    }

    // -------- PROGRESS TAB ----------
    function buildProgressTab() {
      document.getElementById('tab-progress').innerHTML = `
        <div class="card">
          <h2>Weight Progress</h2>
          <canvas id="weightChart" width="400" height="200"></canvas>
          <div class="flex-row">
            <input type="number" id="weight-entry" placeholder="Today's weight" />
            <button id="log-weight">Log Weight</button>
          </div>
        </div>
        <div class="card">
          <h2>🏋️ Lift PRs</h2>
          <div id="lift-list"></div>
          <div class="flex-row">
            <input type="text" id="lift-name" placeholder="Lift (e.g., Deadlift)" />
            <input type="number" id="lift-weight" placeholder="Weight (lbs)" />
            <button id="add-lift">Add PR</button>
          </div>
        </div>`;
      renderWeightChart();
      document.getElementById('log-weight').onclick = () => {
        let w = parseFloat(document.getElementById('weight-entry').value);
        if (!w) return;
        APP.profiles[APP.currentProfile].weightLog.push({ date: APP.today, weight: w });
        saveCurrentProfile();
        renderWeightChart();
        updateAllUI();
      };
      renderLifts();
      document.getElementById('add-lift').onclick = () => {
        let name = document.getElementById('lift-name').value.trim();
        let weight = parseFloat(document.getElementById('lift-weight').value);
        if (!name || !weight) return;
        APP.profiles[APP.currentProfile].liftPRs.push({ lift: name, weight, date: APP.today });
        saveCurrentProfile();
        renderLifts();
      };
    }

    let weightChartInstance;
    function renderWeightChart() {
      let p = APP.profiles[APP.currentProfile];
      let ctx = document.getElementById('weightChart').getContext('2d');
      if (weightChartInstance) weightChartInstance.destroy();
      let labels = p.weightLog.map(e => e.date);
      let data = p.weightLog.map(e => e.weight);
      weightChartInstance = new Chart(ctx, {
        type: 'line',
        data: { labels, datasets: [{ label: 'Weight (lbs)', data, borderColor: '#00b894', tension: 0.1 }] },
        options: { scales: { y: { beginAtZero: false } } }
      });
    }

    function renderLifts() {
      let p = APP.profiles[APP.currentProfile];
      let html = p.liftPRs.map(l => `<div class="list-item"><span>${l.lift}: ${l.weight} lbs</span><span>${l.date}</span></div>`).join('');
      document.getElementById('lift-list').innerHTML = html || '<p>No PRs logged yet.</p>';
    }

    // -------- SETTINGS / EXPORT IMPORT ----------
    function buildSettingsTab() {
      document.getElementById('tab-settings').innerHTML = `
        <div class="card">
          <h2>Profile: ${APP.currentProfile}</h2>
          <button id="export-btn2">⬇ Export All Data</button>
          <button class="btn-secondary" id="import-btn2">⬆ Import Data</button>
          <input type="file" id="import-file2" accept=".json" hidden />
          <button class="btn-secondary" id="switch-profile-btn">🔄 Switch Profile</button>
        </div>`;
      document.getElementById('export-btn2').onclick = exportData;
      document.getElementById('import-btn2').onclick = () => document.getElementById('import-file2').click();
      document.getElementById('import-file2').onchange = importData;
      document.getElementById('switch-profile-btn').onclick = () => {
        let existingProfiles = [];
        for (let i=0; i<localStorage.length; i++) {
          let key = localStorage.key(i);
          if (key.startsWith('lockin_profile_')) existingProfiles.push(key.replace('lockin_profile_',''));
        }
        let selHtml = existingProfiles.map(n => `<option value="${n}">${n}</option>`).join('');
        document.getElementById('tab-settings').innerHTML = `
          <div class="card">
            <h2>Select Profile</h2>
            <select id="profile-select">${selHtml}</select>
            <button id="load-profile-btn2">Load</button>
            <button class="btn-secondary" id="new-profile-btn2">Create New</button>
          </div>`;
        document.getElementById('load-profile-btn2').onclick = () => {
          if (loadProfile(document.getElementById('profile-select').value)) buildAllTabs();
        };
        document.getElementById('new-profile-btn2').onclick = showNewProfileForm;
      };
    }

    function exportData() {
      let p = APP.profiles[APP.currentProfile];
      let blob = new Blob([JSON.stringify(p, null, 2)], {type: 'application/json'});
      let a = document.createElement('a');
      a.href = URL.createObjectURL(blob);
      a.download = `lockin_${p.name}_backup.json`;
      a.click();
    }

    function importData(e) {
      let file = e.target.files[0];
      if (!file) return;
      let reader = new FileReader();
      reader.onload = function(ev) {
        let data = JSON.parse(ev.target.result);
        if (data.name) {
          APP.profiles[data.name] = data;
          localStorage.setItem('lockin_profile_'+data.name, JSON.stringify(data));
          APP.currentProfile = data.name;
          buildAllTabs();
          alert('Data imported successfully!');
        }
      };
      reader.readAsText(file);
    }
  </script>
</body>
</html>
