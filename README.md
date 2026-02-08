<!DOCTYPE html>
<html lang="de">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Tante Rosa Tracker</title>
    <style>
        body { font-family: sans-serif; background-color: #fff0f5; color: #4a4a4a; display: flex; justify-content: center; padding: 20px; }
        .card { background: white; padding: 20px; border-radius: 15px; box-shadow: 0 4px 10px rgba(0,0,0,0.1); width: 100%; max-width: 400px; }
        h1 { color: #d81b60; text-align: center; }
        .prediction { background: #ffebee; border: 2px solid #d81b60; padding: 15px; border-radius: 10px; text-align: center; margin-bottom: 20px; }
        .prediction h2 { margin: 0; font-size: 1.2rem; }
        .input-group { margin-bottom: 15px; display: flex; flex-direction: column; }
        label { margin-bottom: 5px; font-weight: bold; }
        input { padding: 8px; border: 1px solid #ccc; border-radius: 5px; }
        button { cursor: pointer; padding: 10px; border: none; border-radius: 5px; transition: 0.3s; margin-top: 5px; }
        .btn-today { background-color: #f06292; color: white; }
        .btn-save { background-color: #d81b60; color: white; font-weight: bold; margin-top: 10px; width: 100%; }
        .history { margin-top: 20px; font-size: 0.9rem; }
        table { width: 100%; border-collapse: collapse; }
        th, td { border-bottom: 1px solid #eee; padding: 8px; text-align: left; }
    </style>
</head>
<body>

<div class="card">
    <h1>Tante Rosa 🌸</h1>

    <div class="prediction">
        <p>Nächste Menstruation am:</p>
        <h2 id="next-date">Noch keine Daten</h2>
    </div>

    <div class="input-group">
        <label>Anfang der Periode:</label>
        <input type="date" id="start-date">
        <button class="btn-today" onclick="setToday('start-date')">Heute</button>
    </div>

    <div class="input-group">
        <label>Ende der Periode:</label>
        <input type="date" id="end-date">
        <button class="btn-today" onclick="setToday('end-date')">Heute</button>
    </div>

    <button class="btn-save" onclick="saveData()">Eintragen & Speichern</button>

    <div class="history">
        <h3>Verlauf</h3>
        <table id="history-table">
            <thead>
                <tr><th>Von</th><th>Bis</th></tr>
            </thead>
            <tbody></tbody>
        </table>
        <button onclick="clearData()" style="font-size: 0.7rem; background: none; color: gray; margin-top: 10px;">Daten löschen</button>
    </div>
</div>

<script>
    // Daten beim Laden der Seite abrufen
    document.addEventListener('DOMContentLoaded', () => {
        renderHistory();
        calculateNextDate();
    });

    function setToday(id) {
        document.getElementById(id).valueAsDate = new Date();
    }

    function saveData() {
        const start = document.getElementById('start-date').value;
        const end = document.getElementById('end-date').value;

        if (!start || !end) {
            alert("Bitte gib beide Daten ein!");
            return;
        }

        let history = JSON.parse(localStorage.getItem('periodHistory')) || [];
        history.push({ start, end });
        
        // Sortieren nach Datum (neueste zuerst)
        history.sort((a, b) => new Date(b.start) - new Date(a.start));
        
        localStorage.setItem('periodHistory', JSON.stringify(history));
        
        renderHistory();
        calculateNextDate();
    }

    function renderHistory() {
        const history = JSON.parse(localStorage.getItem('periodHistory')) || [];
        const tbody = document.querySelector('#history-table tbody');
        tbody.innerHTML = '';

        history.forEach(entry => {
            const row = `<tr><td>${formatDate(entry.start)}</td><td>${formatDate(entry.end)}</td></tr>`;
            tbody.innerHTML += row;
        });
    }

    function formatDate(dateStr) {
        const options = { day: '2-digit', month: '2-digit', year: 'numeric' };
        return new Date(dateStr).toLocaleDateString('de-DE', options);
    }

    function calculateNextDate() {
        const history = JSON.parse(localStorage.getItem('periodHistory')) || [];
        const display = document.getElementById('next-date');

        if (history.length < 2) {
            display.innerText = "Mehr Daten benötigt";
            return;
        }

        // Berechne Abstände zwischen den Startdaten
        let intervals = [];
        for (let i = 0; i < history.length - 1; i++) {
            const d1 = new Date(history[i].start);
            const d2 = new Date(history[i+1].start);
            const diffDays = Math.abs((d1 - d2) / (1000 * 60 * 60 * 24));
            intervals.push(diffDays);
        }

        // Durchschnittlicher Zyklus (oder 28 Tage Standard, falls Berechnung unmöglich)
        const averageCycle = intervals.reduce((a, b) => a + b, 0) / intervals.length;
        
        const lastStart = new Date(history[0].start);
        const nextDate = new Date(lastStart);
        nextDate.setDate(lastStart.getDate() + Math.round(averageCycle));

        display.innerText = formatDate(nextDate);
    }

    function clearData() {
        if(confirm("Alle Daten löschen?")) {
            localStorage.removeItem('periodHistory');
            location.reload();
        }
    }
</script>

</body>
</html>

