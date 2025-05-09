# Zoom-DeeDee
แอปพลิเคชันเว็บที่ช่วยสุ่มกลุ่มอย่างรวดเร็วแบบไม่ซ้ำ พร้อมแสดงผลเรียลไทม์ เหมาะสำหรับกิจกรรมในห้องเรียน ห้องประชุม หรือกิจกรรมออนไลน์ที่ต้องแบ่งกลุ่มอย่างสนุกสนาน
<!DOCTYPE html>
<html lang="th">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Zoom Dee Dee</title>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/xlsx/0.18.5/xlsx.full.min.js"></script>
  <style>
    body {
      font-family: 'Segoe UI', sans-serif;
      background-color: white;
      text-align: center;
      padding: 40px;
    }
    h1 {
      color: #FF6600;
      font-size: 2.5em;
    }
    select, textarea, input[type="file"] {
      padding: 8px;
      font-size: 1.2em;
      margin-bottom: 20px;
      width: auto;
      max-width: 100px;
    }
    .button {
      padding: 16px 32px;
      font-size: 1.2em;
      margin: 10px;
      border: none;
      border-radius: 12px;
      color: white;
      cursor: pointer;
    }
    #startBtn { background-color: green; }
    #stopBtn { background-color: red; }
    #resetBtn { background-color: orange; }
    .group-list {
      margin-top: 30px;
      display: flex;
      flex-wrap: wrap;
      justify-content: center;
    }
    .group-item {
      background-color: #00aaff;
      color: white;
      padding: 8px 16px;
      margin: 5px;
      border-radius: 8px;
      font-size: 1em;
    }
    .group-done {
      background-color: #ccc;
      text-decoration: line-through;
    }
    .history {
      margin-top: 30px;
      text-align: left;
      display: inline-block;
    }
    .history-item {
      font-size: 1em;
      margin: 5px 0;
      color: #333;
    }
    .history-item .order {
      color: red;
      font-weight: bold;
    }
    #countdown {
      font-size: 2em;
      color: #006600;
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <h1>Zoom Dee Dee</h1>

  <label for="groupCount">เลือกจำนวนกลุ่ม:</label>
  <select id="groupCount"></select><br>

  <label for="nameList">รายชื่อ (คั่นด้วยบรรทัดใหม่):</label><br>
  <textarea id="nameList" rows="6" placeholder="กรอกรายชื่อ เช่น\nสมชาย\nสมหญิง\n...\n"></textarea><br>

  <label for="excelFile">หรืออัปโหลดไฟล์ Excel (.xlsx):</label><br>
  <input type="file" id="excelFile" accept=".xlsx" /><br>

  <label for="countdownTime">เวลานับถอยหลัง (วินาที):</label>
  <input type="number" id="countdownTime" value="3" min="1" style="width: 60px;"><br>

  <button id="startBtn" class="button">เริ่ม</button>
  <button id="stopBtn" class="button">หยุด</button>
  <button id="resetBtn" class="button">รีเซ็ต</button>

  <div id="countdown"></div>
  <div class="group-list" id="groupList"></div>
  <h2 style="font-size: 1.5em; margin-top: 30px;">ประวัติการสุ่ม</h2>
  <div class="history" id="history"></div>

  <script>
    let groupCount = 0;
    let groups = [];
    let shuffled = [];
    let interval = null;
    let countdownTimer = null;
    let currentIndex = 0;

    const groupList = document.getElementById("groupList");
    const history = document.getElementById("history");
    const countdownDisplay = document.getElementById("countdown");
    let groupMap = {};

    function createGroupsFromNames(names, count) {
      groups = [];
      for (let i = 0; i < count; i++) {
        groups.push(`กลุ่ม ${i + 1}`);
      }

      groupMap = {};
      groups.forEach(g => groupMap[g] = []);
      names.forEach((name, index) => {
        const group = groups[index % count];
        groupMap[group].push(name);
      });

      groupList.innerHTML = groups.map(g => {
        const names = groupMap[g].join(", ");
        return `<div class='group-item' id='${g}'>${g}: ${names}</div>`;
      }).join('');

      shuffled = [...groups];
      shuffleArray(shuffled);
      currentIndex = 0;
      history.innerHTML = "";
    }

    function shuffleArray(array) {
      for (let i = array.length - 1; i > 0; i--) {
        const j = Math.floor(Math.random() * (i + 1));
        [array[i], array[j]] = [array[j], array[i]];
      }
    }

    document.addEventListener("DOMContentLoaded", () => {
      const select = document.getElementById("groupCount");
      for (let i = 1; i <= 50; i++) {
        const option = document.createElement("option");
        option.value = i;
        option.textContent = i;
        select.appendChild(option);
      }
      select.value = 10;
    });

    function startRandomizing() {
      if (!shuffled.length || currentIndex >= shuffled.length) return;
      if (interval) clearInterval(interval);

      interval = setInterval(() => {
        if (currentIndex >= shuffled.length) {
          clearInterval(interval);
          return;
        }
        const selected = shuffled[currentIndex];
        document.getElementById(selected).classList.add('group-done');

        const record = document.createElement('div');
        record.className = 'history-item';
        const names = groupMap[selected].join(', ');
        record.innerHTML = `<span class='order'>ลำดับที่ ${currentIndex + 1}</span>: ${selected} - ${names}`;

        history.appendChild(record);
        currentIndex++;
      }, 1000);
    }

    document.getElementById("startBtn").onclick = () => {
      const countdownSeconds = parseInt(document.getElementById("countdownTime").value);
      let timeLeft = countdownSeconds;
      countdownDisplay.textContent = `เริ่มใน ${timeLeft} วินาที`;
      clearInterval(countdownTimer);
      countdownTimer = setInterval(() => {
        timeLeft--;
        if (timeLeft > 0) {
          countdownDisplay.textContent = `เริ่มใน ${timeLeft} วินาที`;
        } else {
          clearInterval(countdownTimer);
          countdownDisplay.textContent = "เริ่มสุ่ม!";
          startRandomizing();
        }
      }, 1000);
    };

    document.getElementById("stopBtn").onclick = () => {
      clearInterval(interval);
      clearInterval(countdownTimer);
      countdownDisplay.textContent = "";
    };

    document.getElementById("resetBtn").onclick = () => {
      clearInterval(interval);
      clearInterval(countdownTimer);
      countdownDisplay.textContent = "";
      const count = parseInt(document.getElementById("groupCount").value);
      const names = document.getElementById("nameList").value.trim().split("\n").filter(n => n.trim() !== "");
      if (names.length === 0) {
        alert("กรุณากรอกรายชื่อก่อน หรืออัปโหลดไฟล์ Excel");
        return;
      }
      createGroupsFromNames(names, count);
    };

    document.getElementById("excelFile").addEventListener("change", function (e) {
      const file = e.target.files[0];
      if (!file) return;

      const reader = new FileReader();
      reader.onload = function (e) {
        const data = new Uint8Array(e.target.result);
        const workbook = XLSX.read(data, { type: "array" });
        const sheet = workbook.Sheets[workbook.SheetNames[0]];
        const json = XLSX.utils.sheet_to_json(sheet, { header: 1 });
        const names = json.flat().filter(n => typeof n === 'string' && n.trim() !== '');
        document.getElementById("nameList").value = names.join("\n");
      };
      reader.readAsArrayBuffer(file);
    });

    document.getElementById("nameList").addEventListener("keydown", function(e) {
      if (e.key === "Enter" && e.ctrlKey) {
        document.getElementById("resetBtn").click();
        document.getElementById("startBtn").click();
      }
    });
  </script>
</body>
</html>
