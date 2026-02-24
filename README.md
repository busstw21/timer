<!DOCTYPE html>
<html lang="th">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pro Timer - Custom Labels</title>
    <style>
        :root {
            --primary: #4f46e5;
            --success: #10b981;
            --danger: #ef4444;
            --warning: #f59e0b;
            --dark: #1f2937;
        }
        body {
            font-family: 'Inter', 'Segoe UI', sans-serif;
            background-color: #f3f4f6;
            display: flex;
            flex-direction: column;
            align-items: center;
            padding: 20px;
        }
        .timer-card {
            background: white;
            padding: 25px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
            text-align: center;
            width: 420px;
        }
        #display {
            font-size: 4.5rem;
            font-weight: 800;
            font-family: 'Courier New', monospace;
            margin: 10px 0;
            color: var(--dark);
            letter-spacing: -2px;
        }
        .input-group {
            margin-bottom: 10px;
            display: flex;
            flex-direction: column;
            gap: 8px;
        }
        input[type="text"] {
            width: 100%;
            padding: 12px;
            border: 2px solid #e5e7eb;
            border-radius: 10px;
            box-sizing: border-box;
            font-size: 0.9rem;
        }
        input#excelName { border-color: var(--primary); background-color: #f5f7ff; font-weight: bold; }
        input#saveNote { border-color: #10b981; }

        .controls {
            display: grid;
            grid-template-columns: repeat(3, 1fr);
            gap: 10px;
        }
        button {
            padding: 12px;
            border: none;
            border-radius: 10px;
            color: white;
            font-weight: bold;
            cursor: pointer;
            transition: 0.2s;
        }
        .btn-start { background: var(--success); grid-column: span 3; font-size: 1.4rem; padding: 18px; }
        .btn-pause { background: var(--warning); grid-column: span 3; font-size: 1.4rem; padding: 18px; }
        .btn-lap { background: var(--primary); }
        .btn-save { background: #6366f1; }
        .btn-reset { background: #6b7280; }
        .btn-excel { background: #157347; grid-column: span 2; }
        .btn-clear { background: #374151; }

        .list-container {
            margin-top: 20px;
            width: 420px;
            display: grid;
            grid-template-columns: 1fr 1fr;
            gap: 15px;
        }
        .list-box {
            background: white;
            padding: 15px;
            border-radius: 15px;
            height: 380px;
            overflow-y: auto;
            box-shadow: 0 4px 6px rgba(0,0,0,0.05);
        }
        h3 { font-size: 0.85rem; margin: 0 0 10px 0; border-bottom: 2px solid #f3f4f6; padding-bottom: 8px; color: #6b7280; text-transform: uppercase; }
        
        .item { padding: 12px 0; border-bottom: 1px solid #f1f5f9; }
        .item-header { display: flex; justify-content: space-between; align-items: flex-start; margin-bottom: 2px; }
        
        /* หัวข้อ: ชื่อไฟล์ + หมายเหตุ */
        .title-group { display: flex; flex-direction: column; max-width: 60%; }
        .file-label { font-size: 0.7rem; color: #6366f1; font-weight: bold; text-transform: uppercase; }
        .note-title { font-size: 1rem; font-weight: 800; color: var(--dark); line-height: 1.2; word-break: break-word; }
        
        .time-val {
            font-family: 'Courier New', monospace;
            font-weight: 700;
            color: var(--primary);
            font-size: 1.1rem;
            white-space: nowrap;
        }
        .date-sub { font-size: 0.65rem; color: #94a3b8; margin-top: 4px; display: block; }
    </style>
</head>
<body>

    <div class="timer-card">
        <div class="input-group">
            <input type="text" id="excelName" placeholder="📁 ชื่อโปรเจกต์ (เช่น ทดสอบวิ่ง)">
            <input type="text" id="saveNote" placeholder="✍️ ระบุชื่อรายการ/หมายเหตุ (เช่น รอบที่ 1)">
        </div>
        
        <div id="display">00:00:00.00</div>
        
        <div class="controls">
            <button id="mainBtn" class="btn-start" onclick="toggleTimer()">START</button>
            <button class="btn-lap" onclick="lap()">LAP</button>
            <button class="btn-save" onclick="save()">SAVE</button>
            <button class="btn-reset" onclick="reset()">RESET</button>
            <button class="btn-excel" onclick="exportToExcel()">DOWNLOAD EXCEL</button>
            <button class="btn-clear" onclick="clearLogs()">ล้างข้อมูล</button>
        </div>
    </div>

    <div class="list-container">
        <div class="list-box">
            <h3>🚩 LAP TIME</h3>
            <div id="lapList"></div>
        </div>
        <div class="list-box">
            <h3>💾 SAVED HISTORY</h3>
            <div id="saveList"></div>
        </div>
    </div>

<script>
    let startTime, elapsedTime = 0, timerInterval;
    let isRunning = false;
    let lapData = JSON.parse(localStorage.getItem('lapData')) || [];
    let saveData = JSON.parse(localStorage.getItem('saveData')) || [];

    window.onload = () => { renderLaps(); renderSaves(); };

    function timeToString(time) {
        let h = Math.floor(time / 3600000);
        let m = Math.floor((time % 3600000) / 60000);
        let s = Math.floor((time % 60000) / 1000);
        let ms = Math.floor((time % 1000) / 10);
        return `${h.toString().padStart(2, "0")}:${m.toString().padStart(2, "0")}:${s.toString().padStart(2, "0")}.${ms.toString().padStart(2, "0")}`;
    }

    function toggleTimer() { isRunning ? pause() : start(); }

    function start() {
        isRunning = true;
        startTime = Date.now() - elapsedTime;
        timerInterval = setInterval(() => {
            elapsedTime = Date.now() - startTime;
            document.getElementById("display").innerHTML = timeToString(elapsedTime);
        }, 10);
        updateBtn();
    }

    function pause() {
        isRunning = false;
        clearInterval(timerInterval);
        updateBtn();
    }

    function updateBtn() {
        const btn = document.getElementById("mainBtn");
        btn.innerHTML = isRunning ? "PAUSE" : "START";
        btn.className = isRunning ? "btn-pause" : "btn-start";
    }

    function reset() {
        pause();
        elapsedTime = 0;
        document.getElementById("display").innerHTML = "00:00:00.00";
    }

    function lap() {
        if (elapsedTime > 0) {
            lapData.push(timeToString(elapsedTime));
            localStorage.setItem('lapData', JSON.stringify(lapData));
            renderLaps();
        }
    }

    function save() {
        if (elapsedTime > 0) {
            const fileInput = document.getElementById("excelName");
            const noteInput = document.getElementById("saveNote");
            
            const currentFileName = fileInput.value.trim() || "GENERAL";
            const note = noteInput.value.trim() || "ไม่มีหมายเหตุ";
            const now = new Date();
            
            saveData.push({
                fileName: currentFileName,
                note: note, // ตอนนี้เป็นชื่อรายการหลัก
                time: timeToString(elapsedTime),
                date: now.toLocaleDateString() + " " + now.toLocaleTimeString()
            });
            
            localStorage.setItem('saveData', JSON.stringify(saveData));
            renderSaves();
            noteInput.value = ""; 
        }
    }

    function renderLaps() {
        document.getElementById("lapList").innerHTML = lapData.map((t, i) => `
            <div class="item">
                <div class="item-header">
                    <span class="note-title" style="font-size:0.85rem">LAP ${i+1}</span>
                    <span class="time-val" style="color:#94a3b8">${t}</span>
                </div>
            </div>
        `).reverse().join('');
    }

    function renderSaves() {
        document.getElementById("saveList").innerHTML = saveData.map((d) => `
            <div class="item">
                <div class="item-header">
                    <div class="title-group">
                        <span class="file-label">${d.fileName}</span>
                        <span class="note-title">${d.note}</span>
                    </div>
                    <span class="time-val">${d.time}</span>
                </div>
                <span class="date-sub">🕒 ${d.date}</span>
            </div>
        `).reverse().join('');
    }

    function clearLogs() {
        if(confirm("ล้างบันทึกทั้งหมดหรือไม่?")) {
            lapData = []; saveData = [];
            localStorage.clear();
            renderLaps(); renderSaves();
            reset();
        }
    }

    function exportToExcel() {
        let fileName = document.getElementById("excelName").value || "timer_report";
        let csv = "\uFEFFโปรเจกต์,หมายเหตุ/ชื่อรายการ,เวลาที่บันทึก,วันที่/เวลา\n";
        
        saveData.forEach((d) => {
            csv += `"${d.fileName}","${d.note}","${d.time}","${d.date}"\n`;
        });

        const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' });
        const link = document.createElement("a");
        link.href = URL.createObjectURL(blob);
        link.download = fileName + ".csv";
        link.click();
    }
</script>
</body>
</html>
