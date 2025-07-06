<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Arbeitszeiterfassung</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/html2canvas/1.4.1/html2canvas.min.js"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jspdf/2.5.1/jspdf.umd.min.js"></script>
    <style>
        body {
            font-family: Arial, sans-serif;
            background: #f2f2f2;
            margin: 0;
            padding: 15px;
            color: #333;
        }
        header {
            text-align: left;
            padding: 5px 0;
        }
        header h2 {
            color: #4CAF50;
            font-size: 18px;
            margin: 0;
        }
        header small {
            color: #777;
            font-size: 12px;
        }
        .info {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-bottom: 10px;
        }
        .info div {
            flex: 1 1 45%;
            min-width: 200px;
        }
        .info label {
            font-weight: bold;
            display: block;
            margin-bottom: 3px;
        }
        .info input {
            border: 1px solid #ccc;
            border-radius: 4px;
            padding: 5px;
            width: 100%;
        }
        h1 {
            text-align: center;
            color: #4CAF50;
            margin-top: 15px;
        }
        table {
            width: 100%;
            border-collapse: collapse;
            margin-top: 10px;
            background: #fff;
        }
        th, td {
            border: 1px solid #ddd;
            padding: 6px;
            text-align: center;
            font-size: 14px;
        }
        th {
            background-color: #4CAF50;
            color: white;
        }
        input[type="text"], input[readonly] {
            width: 100%;
            box-sizing: border-box;
            padding: 4px;
            border: none;
            background: #f9f9f9;
            text-align: center;
        }
        button {
            background-color: #4CAF50;
            color: white;
            border: none;
            padding: 10px;
            margin-top: 10px;
            font-size: 16px;
            border-radius: 5px;
            cursor: pointer;
            width: 100%;
        }
        button:hover {
            background-color: #45a049;
        }
        .signatures {
            margin-top: 20px;
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
        }
        .signature-box {
            flex: 1 1 45%;
            min-width: 200px;
        }
        .signature-box label {
            display: block;
            margin-bottom: 5px;
            font-weight: bold;
        }
        .signature-box canvas {
            border: 1px solid #888;
            width: 100%;
            height: 120px;
            border-radius: 4px;
            background: #fff;
            touch-action: none;
        }
        .signature-box button {
            margin-top: 5px;
            background-color: #f44336;
            font-size: 14px;
            padding: 5px 10px;
        }
        .total {
            margin-top: 15px;
            font-weight: bold;
            font-size: 18px;
            color: #4CAF50;
            text-align: right;
        }
        .actions {
            display: flex;
            flex-wrap: wrap;
            gap: 10px;
            margin-top: 10px;
        }
        .actions button {
            flex: 1 1 45%;
        }
        @media(max-width: 600px) {
            .info div, .signature-box, .actions button {
                flex: 1 1 100%;
            }
        }
    </style>
</head>
<body>
    <div id="captureArea">
        <header>
            <h2>Wackler_groupe.de</h2>
            <small>Arbeitszeiten nach §19 Abs.1 AEntG</small>
        </header>

        <div class="info">
            <div>
                <label for="monat">Monat:</label>
                <input type="text" id="monat" placeholder="z.B. Juli">
            </div>
            <div>
                <label for="jahr">Jahr:</label>
                <input type="number" id="jahr" placeholder="z.B. 2025">
            </div>
            <div>
                <label for="arbeitnehmer">Arbeitnehmer:</label>
                <input type="text" id="arbeitnehmer" placeholder="Name des Arbeitnehmers">
            </div>
            <div>
                <label for="objekt">Objekt:</label>
                <input type="text" id="objekt" placeholder="Name des Objekts">
            </div>
        </div>

        <h1>🕒 Tägliche Arbeitszeiterfassung</h1>

        <table>
            <tr>
                <th>Tag</th>
                <th>Beginn</th>
                <th>Ende</th>
                <th>Pausen (Min)</th>
                <th>Dauer</th>
            </tr>
            <script>
                for (let i = 1; i <= 31; i++) {
                    document.write(`
                    <tr>
                        <td>${i}</td>
                        <td><input type="text" id="start-${i}"></td>
                        <td><input type="text" id="end-${i}"></td>
                        <td><input type="text" id="pause-${i}"></td>
                        <td><input type="text" id="duration-${i}" readonly></td>
                    </tr>`);
                }
            </script>
        </table>

        <div class="total" id="totalTime">Gesamtstunden: 0h 0m</div>

        <div class="signatures">
            <div class="signature-box">
                <label>✍ Unterschrift Arbeitnehmer</label>
                <canvas id="signEmployee"></canvas>
                <button onclick="clearSignature('signEmployee')">🗑 Löschen</button>
            </div>
            <div class="signature-box">
                <label>✍ Unterschrift Vorgesetzter</label>
                <canvas id="signSupervisor"></canvas>
                <button onclick="clearSignature('signSupervisor')">🗑 Löschen</button>
            </div>
        </div>
    </div>

    <div class="actions">
        <button onclick="saveData()">💾 Speichern</button>
        <button onclick="downloadImage()">🖼 Als Bild speichern</button>
        <button onclick="downloadPDF()">📄 Als PDF speichern</button>
        <button onclick="shareViaWhatsApp()">📲 WhatsApp teilen</button>
        <button onclick="shareViaEmail()">✉️ E-Mail senden</button>
    </div>

    <script>
        document.querySelectorAll('input').forEach(input => {
            input.addEventListener('input', () => {
                calculateDurations();
                calculateTotal();
            });
        });

        function calculateDurations() {
            for (let i = 1; i <= 31; i++) {
                let start = document.getElementById(`start-${i}`).value.trim();
                let end = document.getElementById(`end-${i}`).value.trim();
                let pause = document.getElementById(`pause-${i}`).value.trim();

                if (isTimeFormat(start) && isTimeFormat(end) && !isNaN(pause)) {
                    let startTime = new Date(`1970-01-01T${start}:00`);
                    let endTime = new Date(`1970-01-01T${end}:00`);
                    let diffMs = endTime - startTime - (parseInt(pause) * 60000);
                    if (diffMs >= 0) {
                        let hours = Math.floor(diffMs / (1000 * 60 * 60));
                        let minutes = Math.floor((diffMs % (1000 * 60 * 60)) / (1000 * 60));
                        document.getElementById(`duration-${i}`).value = `${hours}h ${minutes}m`;
                    } else {
                        document.getElementById(`duration-${i}`).value = "";
                    }
                } else {
                    document.getElementById(`duration-${i}`).value = start || end || pause;
                }
            }
        }

        function calculateTotal() {
            let totalMinutes = 0;
            for (let i = 1; i <= 31; i++) {
                let duration = document.getElementById(`duration-${i}`).value;
                if (duration && duration.match(/^\d+h \d+m$/)) {
                    let [h, m] = duration.replace('h', '').replace('m', '').split(' ');
                    totalMinutes += parseInt(h) * 60 + parseInt(m);
                }
            }
            let totalHours = Math.floor(totalMinutes / 60);
            let totalRemainingMinutes = totalMinutes % 60;
            document.getElementById('totalTime').innerText = `Gesamtstunden: ${totalHours}h ${totalRemainingMinutes}m`;
        }

        function isTimeFormat(str) {
            return /^\d{1,2}:\d{2}$/.test(str);
        }

        function initCanvas(id) {
            const canvas = document.getElementById(id);
            const ctx = canvas.getContext('2d');
            let drawing = false;

            function getPos(e) {
                if (e.touches) {
                    return {
                        x: e.touches[0].clientX - canvas.getBoundingClientRect().left,
                        y: e.touches[0].clientY - canvas.getBoundingClientRect().top
                    };
                } else {
                    return {
                        x: e.offsetX,
                        y: e.offsetY
                    };
                }
            }

            canvas.addEventListener('mousedown', e => { drawing = true; ctx.beginPath(); });
            canvas.addEventListener('mouseup', () => drawing = false);
            canvas.addEventListener('mousemove', e => { if (drawing) { let p = getPos(e); ctx.lineTo(p.x, p.y); ctx.stroke(); } });

            canvas.addEventListener('touchstart', e => { drawing = true; ctx.beginPath(); e.preventDefault(); });
            canvas.addEventListener('touchend', () => drawing = false);
            canvas.addEventListener('touchmove', e => { if (drawing) { let p = getPos(e); ctx.lineTo(p.x, p.y); ctx.stroke(); e.preventDefault(); } });
        }

        initCanvas('signEmployee');
        initCanvas('signSupervisor');

        function clearSignature(id) {
            const canvas = document.getElementById(id);
            const ctx = canvas.getContext('2d');
            ctx.clearRect(0, 0, canvas.width, canvas.height);
        }

        // مشاركة عبر WhatsApp أو E-Mail
        function getShareText() {
            const arbeitnehmer = document.getElementById('arbeitnehmer').value || "Arbeitnehmer";
            const monat = document.getElementById('monat').value || "Monat";
            const jahr = document.getElementById('jahr').value || "Jahr";
            const total = document.getElementById('totalTime').innerText || "";

            return `📝 Arbeitszeiterfassung\n👤 ${arbeitnehmer}\n📅 ${monat} ${jahr}\n${total}`;
        }

        function shareViaWhatsApp() {
            const text = encodeURIComponent(getShareText());
            const url = `https://wa.me/?text=${text}`;
            window.open(url, '_blank');
        }

        function shareViaEmail() {
            const subject = encodeURIComponent("Arbeitszeiterfassung");
            const body = encodeURIComponent(getShareText());
            const mailto = `mailto:?subject=${subject}&body=${body}`;
            window.location.href = mailto;
        }
    </script>
</body>
</html>
