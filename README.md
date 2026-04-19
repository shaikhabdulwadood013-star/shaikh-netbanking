<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8" />
  <meta name="viewport" content="width=device-width,initial-scale=1" />
  <title>NetBanking by shaikh</title>
  <style>
    :root {
      --bg:#208091;
      --card:#61fffa;
      --accent:#2b6ef6;
      --muted:#6b7280;
      --danger:#e53e3e;
    }
    *{box-sizing:border-box;font-family:Inter,system-ui,Arial,Helvetica,sans-serif}
    body{margin:0;background:var(--bg);padding:32px;display:flex;align-items:flex-start;justify-content:center;min-height:100vh}
    .container{width:960px;max-width:98%;display:flex;gap:20px;flex-wrap:wrap}
    .card{background:var(--card);border-radius:12px;padding:20px;box-shadow:0 6px 22px rgba(20,20,40,0.06);flex:1 1 320px}
    h1,h2{margin:0 0 12px 0}
    .login-grid{display:grid;grid-template-columns:1fr 1fr;gap:12px}
    label{display:block;margin-bottom:6px;font-size:13px;color:var(--muted)}
    input,select{width:100%;padding:10px;border-radius:8px;border:1px solid #000000;background:#fff}
    button{padding:10px 14px;border-radius:8px;border:0;background:var(--accent);color:#ffffff;font-weight:600;cursor:pointer}
    .small{font-size:13px;color:var(--muted)}
    .user-list{display:flex;flex-direction:column;gap:8px}
    .row{display:flex;gap:12px;align-items:center}
    .balance{font-size:28px;font-weight:700;margin-top:8px}
    .muted{color:var(--muted);font-size:13px}
    .tx{border-left:3px solid #eee;padding:8px 12px;border-radius:6px;background:#b3f27b}
    .tx.in{border-left-color:#16a34a}
    .tx.out{border-left-color:#ef4444}
    .right{margin-left:auto}
    .logout{background:#acf8f8;color:var(--accent);border:1px solid #e6e9ef}
    .table{width:100%;border-collapse:collapse;margin-top:12px}
    .table th,.table td{padding:8px;text-align:left;border-bottom:1px solid #f1f5f9}
    .note{font-size:12px;color:#fcfcfc;margin-top:8px}
    .danger{color:var(--danger);font-weight:600}
    footer{width:100%;text-align:center;color:var(--muted);font-size:12px;margin-top:14px}
    @media (max-width:740px){ .login-grid{grid-template-columns:1fr} }
  </style>
</head>
<body>
  <div class="container">
    <!-- Login card -->
    <div class="card" id="loginCard">
      <h1>NetBanking by shaikh</h1>
      <p class="small">Client-side demo only — passwords and data are stored in browser memory/localStorage.</p>

      <div style="margin-top:12px">
        <div class="login-grid">
          <div>
            <label for="username">Username</label>
            <input id="username" placeholder="e.g. user1" />
          </div>
          <div>
            <label for="password">Password</label>
            <input id="password" type="password" placeholder="Password" />
          </div>
        </div>

        <div style="display:flex;gap:8px;margin-top:12px">
          <button id="loginBtn">Log In</button>
          <button id="demoBtn" class="logout">Auto-fill Demo</button>
        </div>

        
      <div class="note">Tip: use <code>user1 / pass1</code> to try transfers.</div>
    </div>

    <!-- Dashboard card -->
    <div class="card" id="dashCard" style="display:none;min-width:380px;">
      <div class="row">
        <div>
          <h2 id="welcome">Welcome, User</h2>
          <div class="muted" id="acct">Account: <span id="acctId"></span></div>
        </div>
        <div class="right">
          <button id="logoutBtn" class="logout">Logout</button>
        </div>
      </div>

      <div style="margin-top:12px">
        <div class="balance">₹ <span id="balance">0.00</span></div>
        <div class="muted">Available balance</div>
      </div>

      <hr style="margin:16px 0;border:none;border-top:1px solid #eef2f7"/>

      <h3 style="margin-bottom:8px">Transfer Funds</h3>
      <div class="row" style="gap:8px">
        <div style="flex:1">
          <label for="toUser">To</label>
          <select id="toUser"></select>
        </div>
        <div style="width:140px">
          <label for="amount">Amount (₹)</label>
          <input id="amount" type="number" min="1" step="1" placeholder="100" />
        </div>
        <div style="align-self:end">
          <button id="sendBtn">Send</button>
        </div>
      </div>
      <div id="transferMsg" class="note" style="margin-top:8px"></div>

      <hr style="margin:18px 0;border:none;border-top:1px solid #eef2f7"/>

      <h3 style="margin-bottom:8px">Transaction History</h3>
      <table class="table" id="txTable">
        <thead><tr><th>Date</th><th>Details</th><th>Amount</th></tr></thead>
        <tbody id="txBody"></tbody>
      </table>
    </div>
  </div>

  <footer>netbanking design by abdul wadood</footer>

  <script>
    
    const DEMO_USERS = [
      { username: "user1", password: "pass1", name: "abdul wadood", balance: 50000, tx: [] },
      { username: "user2", password: "pass2", name: "sufiyan", balance: 35000, tx: [] },
      { username: "user3", password: "pass3", name: "mustafa", balance: 12000, tx: [] },
      { username: "user4", password: "pass4", name: "moinu", balance: 8000, tx: [] },
      { username: "user5", password: "pass5", name: "sidra", balance: 250000, tx: [] },
      { username: "user6", password: "pass6", name: "fatima k", balance: 6000, tx: [] },
      { username: "user7", password: "pass7", name: "harshita", balance: 1500, tx: [] },
    ];

    // Storage keys
    const LS_USERS_KEY = "demo_netbank_users_v1";
    const LS_SESSION_KEY = "demo_netbank_session_v1";

    // Helper: load or initialize users in localStorage so balances persist between reloads
    function loadUsers() {
      const data = localStorage.getItem(LS_USERS_KEY);
      if (!data) {
        localStorage.setItem(LS_USERS_KEY, JSON.stringify(DEMO_USERS));
        return JSON.parse(JSON.stringify(DEMO_USERS));
      }
      try {
        const parsed = JSON.parse(data);
        // migrate if missing tx arrays
        parsed.forEach(u => { if (!u.tx) u.tx = []; });
        return parsed;
      } catch (e) {
        localStorage.setItem(LS_USERS_KEY, JSON.stringify(DEMO_USERS));
        return JSON.parse(JSON.stringify(DEMO_USERS));
      }
    }
    function saveUsers(users) {
      localStorage.setItem(LS_USERS_KEY, JSON.stringify(users));
    }

    // Session helpers
    function setSession(username) { localStorage.setItem(LS_SESSION_KEY, username); }
    function clearSession() { localStorage.removeItem(LS_SESSION_KEY); }
    function getSession() { return localStorage.getItem(LS_SESSION_KEY); }

    // UI elements
    const loginCard = document.getElementById("loginCard");
    const dashCard = document.getElementById("dashCard");
    const usernameInput = document.getElementById("username");
    const passwordInput = document.getElementById("password");
    const loginBtn = document.getElementById("loginBtn");
    const demoBtn = document.getElementById("demoBtn");
    const logoutBtn = document.getElementById("logoutBtn");
    const welcomeEl = document.getElementById("welcome");
    const acctEl = document.getElementById("acctId");
    const balanceEl = document.getElementById("balance");
    const toUserSel = document.getElementById("toUser");
    const amountInput = document.getElementById("amount");
    const sendBtn = document.getElementById("sendBtn");
    const txBody = document.getElementById("txBody");
    const transferMsg = document.getElementById("transferMsg");

    let users = loadUsers();
    let currentUser = null; // object reference to logged-in user

    // Utility: find user by username
    function findUser(username) {
      return users.find(u => u.username === username);
    }

    // Render recipients (exclude current user)
    function populateRecipients() {
      toUserSel.innerHTML = "";
      users.forEach(u => {
        if (!currentUser || u.username !== currentUser.username) {
          const opt = document.createElement("option");
          opt.value = u.username;
          opt.textContent = `${u.username} — ${u.name}`;
          toUserSel.appendChild(opt);
        }
      });
    }

    // Render transaction table for current user
    function renderTx() {
      txBody.innerHTML = "";
      if (!currentUser) return;
      const txs = (currentUser.tx || []).slice().reverse(); // show newest first
      if (txs.length === 0) {
        const tr = document.createElement("tr");
        tr.innerHTML = '<td colspan="3" class="muted">No transactions yet</td>';
        txBody.appendChild(tr);
        return;
      }
      txs.forEach(t => {
        const tr = document.createElement("tr");
        const d = new Date(t.date);
        tr.innerHTML = `<td>${d.toLocaleString()}</td>
                        <td>${t.details}</td>
                        <td>${t.amount >=0 ? '₹ ' + t.amount.toFixed(2) : '- ₹ ' + Math.abs(t.amount).toFixed(2)}</td>`;
        txBody.appendChild(tr);
      });
    }

    // Update dashboard UI
    function updateDashboard() {
      welcomeEl.textContent = `Welcome, ${currentUser.name.split(" ")[0] || currentUser.username}`;
      acctEl.textContent = currentUser.username;
      balanceEl.textContent = currentUser.balance.toFixed(2);
      populateRecipients();
      renderTx();
    }

    // Login flow
    loginBtn.addEventListener("click", () => {
      const uname = usernameInput.value.trim();
      const pwd = passwordInput.value;
      if (!uname || !pwd) {
        alert("Enter username and password.");
        return;
      }
      users = loadUsers(); // reload
      const user = findUser(uname);
      if (!user || user.password !== pwd) {
        alert("Invalid username or password.");
        return;
      }
      currentUser = user;
      setSession(user.username);
      loginCard.style.display = "none";
      dashCard.style.display = "block";
      updateDashboard();
    });

    // Autofill demo credentials
    demoBtn.addEventListener("click", () => {
      usernameInput.value = "user1";
      passwordInput.value = "pass1";
    });

    // Logout
    logoutBtn.addEventListener("click", () => {
      clearSession();
      currentUser = null;
      dashCard.style.display = "none";
      loginCard.style.display = "block";
      usernameInput.value = "";
      passwordInput.value = "";
      transferMsg.textContent = "";
    });

    // Send money
    sendBtn.addEventListener("click", () => {
      transferMsg.textContent = "";
      const to = toUserSel.value;
      const amt = Number(amountInput.value);
      if (!to) { transferMsg.textContent = "Select recipient."; return; }
      if (!amt || amt <= 0) { transferMsg.textContent = "Enter a valid amount."; return; }
      if (!currentUser) { transferMsg.textContent = "Not logged in."; return; }
      if (to === currentUser.username) { transferMsg.textContent = "Cannot transfer to yourself."; return; }
      // Reload users from storage to avoid stale data
      users = loadUsers();
      const sender = findUser(currentUser.username);
      const recipient = findUser(to);
      if (!recipient) { transferMsg.textContent = "Recipient not found."; return; }
      if (sender.balance < amt) { transferMsg.textContent = "Insufficient balance."; return; }

      // Perform transfer
      sender.balance = Number((sender.balance - amt).toFixed(2));
      recipient.balance = Number((recipient.balance + amt).toFixed(2));

      // Add transactions (both sides)
      const now = new Date().toISOString();
      sender.tx = sender.tx || [];
      recipient.tx = recipient.tx || [];

      sender.tx.push({
        date: now,
        details: `Transfer to ${recipient.name} (${recipient.username})`,
        amount: -amt
      });
      recipient.tx.push({
        date: now,
        details: `Received from ${sender.name} (${sender.username})`,
        amount: amt
      });

      // Save and update UI
      saveUsers(users);
      currentUser = findUser(sender.username); // refresh reference
      updateDashboard();
      amountInput.value = "";
      transferMsg.textContent = `Sent ₹ ${amt.toFixed(2)} to ${recipient.username}.`;
      transferMsg.style.color = "#0f5132"; // green-ish (inline)
      setTimeout(()=> transferMsg.textContent = "", 4000);
    });

    // On load: auto-login if session present
    window.addEventListener("load", () => {
      users = loadUsers();
      const session = getSession();
      if (session) {
        const u = findUser(session);
        if (u) {
          currentUser = u;
          loginCard.style.display = "none";
          dashCard.style.display = "block";
          updateDashboard();
          return;
        } else {
          clearSession();
        }
      }
    });

    // For safety: expose a function to reset demo (only in dev)
    // Uncomment to add a 'Reset demo' button if needed.
    // function resetDemo() { localStorage.removeItem(LS_USERS_KEY); location.reload(); }

  </script>
</body>
</html>
