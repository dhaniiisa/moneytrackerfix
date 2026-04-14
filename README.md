<!DOCTYPE html>
<html lang="id">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Money Tracker Multi Akun</title>
  <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>

  <style>
    * {
      box-sizing: border-box;
    }

    body {
      font-family: 'Segoe UI', sans-serif;
      background: linear-gradient(135deg, #84c88f, #3f4e3e);
      margin: 0px;
      padding: 20px;
      display: flex;
      justify-content: center;
    }

    @media (max-width: 600px) {
    .container {
      width: 100%;
      max-width: 480%;
      padding: 10px;
    }}

    .box {
      background: #ffffffcc;
      padding: 15px;
      margin: 15px;
      border-radius: 15px;
    }

    input, select {
      width: 100%;
      padding: 10px;
      margin: 6px 0;
      border-radius: 10px;
      border: 1px solid #ddd;
    }

    button {
      width: 70%;
      margin: 6px auto;
      display: block;
      padding: 10px;
      border-radius: 20px;
      border: none;
      background: #9ee8ff;
      font-size: 14px;
      text-align: center;
      cursor: pointer;
    }

    .hidden { display: none; }

    li {
      display: flex;
      justify-content: space-between;
      padding: 4px;
      border-bottom: 8px solid #eee;
      font-size: 14px;
    }

    .income { color: green; }
    .expense { color: red; }

    canvas {
      width: 100% !important;
      height: 300px !important;
      max-width: 350px;
    }

    #historyPage {
  width: 100%;
}

#list {
  padding: 0;
  margin-top: 10px;
}
@media (max-width: 600px) {
  body {
    padding: 10px;
  }

  .box {
    margin: 8px;
    padding: 12px;
  }

  button {
    width: 100%;
  }

  li {
    font-size: 13px;
    flex-direction: column;
    align-items: flex-start;
    gap: 4px;
  }
}


  </style>
</head>

<body>

<div class="container">

  <!-- LOGIN -->
  <div class="box" id="loginPage">
    <h3>👤 Account</h3>
    <input type="text" id="username" placeholder="Nama akun">
    <button onclick="login()">login</button>
  </div>

  <!-- MAIN -->
  <div class="box hidden" id="mainPage">
    <h3>💸 Your Money Tracker</h3>
    <p id="userLabel"></p>

    <input type="date" id="date">
    <input type="text" id="desc" placeholder="information">
    <input type="number" id="amount" placeholder="amount (Rp)">

    <select id="type">
      <option value="income">income</option>
      <option value="expense">expense</option>
    </select>

    <button onclick="addTransaction()">input</button>
    <button onclick="showHistory()">History</button>
    <button onclick="logout()">change account</button>

    <h3>balance: <span id="balance">Rp 0</span></h3>
  </div>

  <!-- HISTORY -->
  <div class="box hidden" id="historyPage">
    <h3>📒 History</h3>
    <canvas id="myChart"></canvas>
    <ul id="list"></ul>
    <button onclick="backToMain()">Back</button>
  </div>

</div>

<script>
let currentUser = localStorage.getItem("lastUser") || null;
let data = JSON.parse(localStorage.getItem("moneyData")) || {};
let chart;

function saveData(){
  localStorage.setItem("moneyData", JSON.stringify(data));
  if(currentUser) localStorage.setItem("lastUser", currentUser);
}

function formatRupiah(angka) {
  return 'Rp ' + angka.toString().replace(/\B(?=(\d{3})+(?!\d))/g, '.');
}

function login(){
  let username = document.getElementById("username").value;
  if(!username) return alert("Masukkan nama akun!");

  currentUser = username;

  if(!data[currentUser]){
    data[currentUser] = { balance: 0, transactions: [] };
  }

  saveData();
  showMain();
}

function showMain(){
  document.getElementById("loginPage").classList.add("hidden");
  document.getElementById("mainPage").classList.remove("hidden");
  document.getElementById("historyPage").classList.add("hidden");

  document.getElementById("userLabel").innerText = "Akun: " + currentUser;
  updateUI();
}

function logout(){
  currentUser = null;
  localStorage.removeItem("lastUser");

  document.getElementById("loginPage").classList.remove("hidden");
  document.getElementById("mainPage").classList.add("hidden");
}

function addTransaction() {
  let date = document.getElementById("date").value;
  let desc = document.getElementById("desc").value;
  let amount = parseInt(document.getElementById("amount").value);
  let type = document.getElementById("type").value;

  if (!date || !desc || !amount) return alert("Isi semua data!");

  let userData = data[currentUser];

  if (type === "income") userData.balance += amount;
  else userData.balance -= amount;

  userData.transactions.push({ date, desc, amount, type });

  saveData();
  updateUI();

  document.getElementById("date").value = "";
  document.getElementById("desc").value = "";
  document.getElementById("amount").value = "";
}

function updateUI(){
  let userData = data[currentUser];
  document.getElementById("balance").innerText = formatRupiah(userData.balance);
}

function showHistory() {
  document.getElementById("mainPage").classList.add("hidden");
  document.getElementById("historyPage").classList.remove("hidden");

  let list = document.getElementById("list");
  list.innerHTML = "";

  let trxList = data[currentUser].transactions;

  // tampilkan list
  trxList.forEach(trx => {
    let li = document.createElement("li");
    let sign = trx.type === "income" ? "+" : "-";

    li.innerHTML = `
      <span>${trx.date} - ${trx.desc}</span>
      <span class='${trx.type}'><b>${sign}${formatRupiah(trx.amount)}</b></span>
    `;

    list.appendChild(li);
  });

  // grafik
  let map = {};

  trxList.forEach(trx => {
    if(!map[trx.date]) map[trx.date] = {income:0, expense:0};
    map[trx.date][trx.type] += trx.amount;
  });

  let labels = Object.keys(map).sort();
  let incomeData = labels.map(d => map[d].income);
  let expenseData = labels.map(d => map[d].expense);

  if(chart) chart.destroy();

  chart = new Chart(document.getElementById('myChart'), {
    type: 'bar',
    data: {
      labels: labels,
      datasets: [
        { label: 'Pemasukan', data: incomeData, barThickness: 6 },
        { label: 'Pengeluaran', data: expenseData, barThickness: 6 }
      ]
    },
    options: {
      responsive: true,
      maintainAspectRatio: false
    }
  });
}

function backToMain(){
  showMain();
}

// auto login
if(currentUser && data[currentUser]){
  showMain();
}
</script>

</body>
</html># moneytrackerfix
mtfweb
