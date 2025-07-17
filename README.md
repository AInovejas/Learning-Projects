# Learning-Projects
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Pagbilao Plant Dispatch Tracker</title>
    <!-- Tailwind CSS for styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts: Inter -->
    <link rel="preconnect" href="https://fonts.googleapis.com">
    <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <!-- Chart.js for data visualization -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <!-- Custom Styles -->
    <style>
        body {
            font-family: 'Inter', sans-serif;
        }
        .driver-card {
            transition: all 0.3s ease-in-out;
        }
        .modal-backdrop, .tab-content {
            transition: opacity 0.3s ease;
        }
        .modal-content {
            transition: transform 0.3s ease;
        }
        .tab-button.active {
            border-bottom-color: #4f46e5; /* indigo-600 */
            color: #111827; /* gray-900 */
        }
        .hidden-tab {
            display: none;
        }
        /* Ensure charts are responsive */
        .chart-container {
            position: relative;
            height: 40vh;
            width: 100%;
        }
    </style>
</head>
<body class="bg-gray-100 text-gray-800">

    <div id="app-container" class="max-w-7xl mx-auto p-4 sm:p-6 lg:p-8">
        
        <header class="mb-6 text-center">
            <h1 class="text-3xl sm:text-4xl font-bold text-gray-900">Pagbilao Plant Dispatch Tracker</h1>
            <p class="mt-2 text-md text-gray-600">Pagbilao Plant Dispatch Tracker</p>
            <div class="mt-4 inline-block bg-white p-3 rounded-lg shadow-md">
                <p id="currentDate" class="text-lg text-indigo-700 font-bold"></p>
            </div>
            <div class="mt-4 text-xs text-gray-500 bg-gray-200 rounded-md p-2 inline-block">
                <span class="font-semibold">Shareable App ID:</span> <span id="appIdDisplay">loading...</span>
            </div>
        </header>

        <!-- Tabs -->
        <div class="mb-6 border-b border-gray-200">
            <nav class="-mb-px flex space-x-6" aria-label="Tabs">
                <button id="tab-live" class="tab-button active whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm">Live Status</button>
                <button id="tab-history" class="tab-button whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300">Historical Log</button>
                <button id="tab-charts" class="tab-button whitespace-nowrap py-4 px-1 border-b-2 font-medium text-sm border-transparent text-gray-500 hover:text-gray-700 hover:border-gray-300">Charts</button>
            </nav>
        </div>

        <!-- Live Status Content -->
        <div id="content-live" class="tab-content">
            <div class="mb-8">
                <h2 class="text-2xl font-bold mb-4">Driver Status</h2>
                <div id="driverList" class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-6">
                    <div id="loading-placeholder" class="text-center text-gray-500 col-span-full py-10"><p>Connecting to the live database...</p></div>
                </div>
            </div>
            <div class="bg-white p-6 rounded-xl shadow-md">
                <h2 class="text-xl font-semibold mb-4">Add New Driver</h2>
                <form id="addDriverForm" class="grid sm:grid-cols-3 gap-4 items-end">
                    <div class="sm:col-span-1"><label for="driverName" class="block text-sm font-medium text-gray-700">Driver Name</label><input type="text" id="driverName" required class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"></div>
                    <div class="sm:col-span-1"><label for="carDetails" class="block text-sm font-medium text-gray-700">Car / Vehicle Details</label><input type="text" id="carDetails" required class="mt-1 block w-full px-3 py-2 bg-white border border-gray-300 rounded-md shadow-sm focus:outline-none focus:ring-indigo-500 focus:border-indigo-500"></div>
                    <button type="submit" class="sm:col-span-1 w-full bg-indigo-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-indigo-500 transition">Add Driver</button>
                </form>
            </div>
        </div>

        <!-- History Log Content -->
        <div id="content-history" class="tab-content hidden-tab">
             <div class="bg-white p-6 rounded-xl shadow-md">
                <div class="sm:flex justify-between items-center mb-4">
                    <h2 class="text-2xl font-bold">Activity History</h2>
                    <div class="mt-4 sm:mt-0 flex flex-col sm:flex-row items-center gap-4">
                        <div><label for="startDate" class="text-sm font-medium">From:</label><input type="date" id="startDate" class="ml-2 border-gray-300 rounded-md shadow-sm"></div>
                        <div><label for="endDate" class="text-sm font-medium">To:</label><input type="date" id="endDate" class="ml-2 border-gray-300 rounded-md shadow-sm"></div>
                        <button id="exportCsvButton" class="w-full sm:w-auto bg-green-600 text-white font-semibold py-2 px-4 rounded-md hover:bg-green-700 transition">Export to CSV</button>
                    </div>
                </div>
                <div class="overflow-x-auto"><table class="min-w-full divide-y divide-gray-200"><thead class="bg-gray-50"><tr><th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Timestamp</th><th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Driver</th><th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Vehicle</th><th scope="col" class="px-6 py-3 text-left text-xs font-medium text-gray-500 uppercase tracking-wider">Action</th></tr></thead><tbody id="historyLogList" class="bg-white divide-y divide-gray-200"></tbody></table></div>
            </div>
        </div>
        
        <!-- Charts/Visualization Content -->
        <div id="content-charts" class="tab-content hidden-tab">
            <div class="grid grid-cols-1 lg:grid-cols-2 gap-8">
                <div class="bg-white p-6 rounded-xl shadow-md lg:col-span-2"><h3 class="text-lg font-semibold mb-4 text-center">Current Fleet Status</h3><div class="chart-container" style="height:30vh;"><canvas id="liveStatusChart"></canvas></div></div>
                <div class="bg-white p-6 rounded-xl shadow-md"><h3 class="text-lg font-semibold mb-4 text-center">Dispatches per Driver (Today)</h3><div class="chart-container"><canvas id="dispatchesByDriverChart"></canvas></div></div>
                <div class="bg-white p-6 rounded-xl shadow-md"><h3 class="text-lg font-semibold mb-4 text-center">Peak Dispatch Hours (Today)</h3><div class="chart-container"><canvas id="peakHoursChart"></canvas></div></div>
                <div class="bg-white p-6 rounded-xl shadow-md lg:col-span-2"><h3 class="text-lg font-semibold mb-4 text-center">Most Visited Locations (Today)</h3><div class="chart-container"><canvas id="locationsChart"></canvas></div></div>
            </div>
        </div>

    </div>

    <!-- Confirmation Modal -->
    <div id="confirmationModal" class="fixed inset-0 bg-gray-900 bg-opacity-50 flex items-center justify-center p-4 modal-backdrop hidden opacity-0 z-50"><div class="bg-white rounded-lg shadow-xl p-6 w-full max-w-sm modal-content transform scale-95"><h3 class="text-lg font-medium leading-6 text-gray-900" id="modalTitle">Confirm Action</h3><div class="mt-2"><p class="text-sm text-gray-500" id="modalMessage">Are you sure?</p></div><div class="mt-5 sm:mt-6 sm:grid sm:grid-cols-2 sm:gap-3 sm:grid-flow-row-dense"><button id="confirmButton" type="button" class="w-full inline-flex justify-center rounded-md border border-transparent shadow-sm px-4 py-2 bg-red-600 text-base font-medium text-white hover:bg-red-700 focus:outline-none sm:col-start-2 sm:text-sm">Confirm</button><button id="cancelButton" type="button" class="mt-3 w-full inline-flex justify-center rounded-md border border-gray-300 shadow-sm px-4 py-2 bg-white text-base font-medium text-gray-700 hover:bg-gray-50 focus:outline-none sm:mt-0 sm:col-start-1 sm:text-sm">Cancel</button></div></div></div>

    <!-- Firebase Libraries -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, onSnapshot, doc, updateDoc, deleteDoc, serverTimestamp, setLogLevel, query, orderBy, where, getDocs } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {};
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'pagbilao-dispatch-tracker';
        const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
        
        let db, auth, userId, driversCollectionRef, historyCollectionRef;
        let fullHistory = [];
        let liveStatusChart, dispatchesChart, peakHoursChart, locationsChart;

        const plantLocations = ["Admin Bldg", "Boiler", "MHI", "Control", "RO", "Open Hatch", "OSH", "Tuklas"];

        const UI = {
            driverList: document.getElementById('driverList'),
            addDriverForm: document.getElementById('addDriverForm'),
            appIdDisplay: document.getElementById('appIdDisplay'),
            historyLogList: document.getElementById('historyLogList'),
            currentDate: document.getElementById('currentDate'),
            exportCsvButton: document.getElementById('exportCsvButton'),
            startDateInput: document.getElementById('startDate'),
            endDateInput: document.getElementById('endDate'),
        };

        async function main() {
            setLogLevel('debug');
            UI.appIdDisplay.textContent = appId;
            displayCurrentDate();

            try {
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                onAuthStateChanged(auth, async (user) => {
                    if (user) {
                        userId = user.uid;
                        driversCollectionRef = collection(db, `/artifacts/${appId}/public/data/drivers`);
                        historyCollectionRef = collection(db, `/artifacts/${appId}/public/data/history`);
                        listenForDriverUpdates();
                        listenForHistoryUpdates();
                    }
                });

                if (initialAuthToken) await signInWithCustomToken(auth, initialAuthToken);
                else await signInAnonymously(auth);
            } catch (error) {
                console.error("Firebase Initialization Error:", error);
                UI.driverList.innerHTML = `<div class="text-center text-red-500 col-span-full py-10"><p>Error connecting to the database.</p></div>`;
            }
        }

        function displayCurrentDate() {
            const options = { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric', timeZone: 'Asia/Manila' };
            UI.currentDate.textContent = new Date().toLocaleDateString('en-US', options);
        }

        function listenForDriverUpdates() {
            onSnapshot(driversCollectionRef, (snapshot) => {
                UI.driverList.innerHTML = snapshot.empty ? `<div class="text-center text-gray-500 col-span-full py-10"><p>No drivers added yet.</p></div>` : '';
                const drivers = snapshot.docs.map(doc => ({ id: doc.id, ...doc.data() }));
                drivers.sort((a, b) => a.name.localeCompare(b.name)).forEach(driver => UI.driverList.appendChild(createDriverCard(driver)));
                updateLiveStatusChart(drivers);
            });
        }

        function listenForHistoryUpdates() {
            const q = query(historyCollectionRef, orderBy("timestamp", "desc"));
            onSnapshot(q, (snapshot) => {
                UI.historyLogList.innerHTML = snapshot.empty ? `<tr><td colspan="4" class="text-center text-gray-500 py-10">No history records found.</td></tr>` : '';
                fullHistory = snapshot.docs.map(doc => doc.data());
                UI.historyLogList.innerHTML = ''; // Clear before re-populating
                fullHistory.forEach(log => UI.historyLogList.appendChild(createHistoryRow(log)));
                updateAnalyticsCharts();
            });
        }
        
        function createDriverCard(driver) {
            const card = document.createElement('div');
            const { status, name, car, location } = driver;
            let statusText, statusTextColor, statusDotColor, bgColor, mainButtonText, mainButtonClass;
            switch (status) {
                case 'on the road': statusText = `ON THE ROAD TO: ${location || '...'}`; statusTextColor = 'text-blue-800'; statusDotColor = 'bg-blue-500 animate-pulse'; bgColor = 'bg-blue-100'; mainButtonText = 'Mark as Available'; mainButtonClass = 'bg-green-500 hover:bg-green-600'; break;
                case 'unavailable': statusText = 'NOT AVAILABLE TODAY'; statusTextColor = 'text-gray-800'; statusDotColor = 'bg-gray-500'; bgColor = 'bg-gray-200'; mainButtonText = 'Set as Available'; mainButtonClass = 'bg-green-500 hover:bg-green-600'; break;
                default: statusText = 'AVAILABLE'; statusTextColor = 'text-green-800'; statusDotColor = 'bg-green-500'; bgColor = 'bg-green-100'; mainButtonText = 'Dispatch Driver'; mainButtonClass = 'bg-yellow-500 hover:bg-yellow-600';
            }
            card.className = `driver-card p-5 rounded-lg shadow-lg flex flex-col justify-between ${bgColor}`;
            const locationDropdownHTML = status === 'available' ? `<select class="location-select mt-2 block w-full pl-3 pr-10 py-2 text-base border-gray-300 focus:outline-none focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm rounded-md"><option disabled selected>Select destination...</option>${plantLocations.map(loc => `<option value="${loc}">${loc}</option>`).join('')}</select>` : '';
            card.innerHTML = `<div><div class="flex justify-between items-start"><h3 class="text-lg font-bold ${statusTextColor}">${name}</h3><button data-id="${driver.id}" data-name="${name}" class="delete-driver-btn text-gray-400 hover:text-red-600"><svg xmlns="http://www.w3.org/2000/svg" width="18" height="18" fill="currentColor" viewBox="0 0 16 16"><path d="M5.5 5.5A.5.5 0 0 1 6 6v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5m2.5 0a.5.5 0 0 1 .5.5v6a.5.5 0 0 1-1 0V6a.5.5 0 0 1 .5-.5m3 .5a.5.5 0 0 0-1 0v6a.5.5 0 0 0 1 0z"/><path d="M14.5 3a1 1 0 0 1-1 1H13v9a2 2 0 0 1-2 2H5a2 2 0 0 1-2-2V4h-.5a1 1 0 0 1-1-1V2a1 1 0 0 1 1-1H6a1 1 0 0 1 1-1h2a1 1 0 0 1 1 1h3.5a1 1 0 0 1 1 1zM4.118 4 4 4.059V13a1 1 0 0 0 1 1h6a1 1 0 0 0 1-1V4.059L11.882 4zM2.5 3h11V2h-11z"/></svg></button></div><p class="text-sm text-gray-600 mt-1">${car}</p><div class="flex items-center space-x-2 mt-3"><div class="w-3 h-3 rounded-full ${statusDotColor}"></div><span class="text-xs font-semibold ${statusTextColor}">${statusText}</span></div></div><div class="mt-4">${locationDropdownHTML}<div class="mt-2 flex space-x-2"><button data-id="${driver.id}" data-status="${status}" data-name="${name}" data-car="${car}" data-location="${location || ''}" class="toggle-status-btn w-full text-white font-semibold py-2 px-4 rounded-md transition ${mainButtonClass}">${mainButtonText}</button>${status !== 'on the road' ? `<button data-id="${driver.id}" data-status="${status}" data-name="${name}" data-car="${car}" class="toggle-unavailable-btn flex-shrink-0 ${status === 'unavailable' ? 'bg-gray-400' : 'bg-gray-300'} hover:bg-gray-400 text-gray-800 font-bold py-2 px-3 rounded-md transition" title="${status === 'unavailable' ? 'Set as Available' : 'Set as Unavailable Today'}"><svg xmlns="http://www.w3.org/2000/svg" width="16" height="16" fill="currentColor" viewBox="0 0 16 16"><path d="M11 1.5a.5.5 0 0 1 .5.5v1.541l1.16.663a.5.5 0 0 1 .24 1.11l-1.399.799 1.065 1.846a.5.5 0 0 1-.24.798l-1.541.514-1.065 1.846a.5.5 0 0 1-.798.24l-1.846-1.065-1.541.514a.5.5 0 0 1-.798-.24L3.46 8.846l-1.541-.514a.5.5 0 0 1-.24-.798L2.745 5.69l-1.399-.8a.5.5 0 0 1 .24-1.109l1.16-.663V2a.5.5 0 0 1 1 0v1.11L5.69 2.745a.5.5 0 0 1 .798.24l.8 1.399 1.846-1.065a.5.5 0 0 1 .798.24l.514 1.541L11 1.5zM8 5.25a2.75 2.75 0 1 1 0 5.5 2.75 2.75 0 0 1 0-5.5"/></svg></button>` : ''}</div></div>`;
            return card;
        }

        function createHistoryRow({ timestamp, name, car, action }) {
            const row = document.createElement('tr');
            const date = timestamp ? timestamp.toDate() : new Date();
            const formattedTime = date.toLocaleString('en-US', { timeZone: 'Asia/Manila', year: 'numeric', month: 'short', day: 'numeric', hour: '2-digit', minute: '2-digit', hour12: true });
            row.innerHTML = `<td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${formattedTime}</td><td class="px-6 py-4 whitespace-nowrap text-sm font-medium text-gray-900">${name}</td><td class="px-6 py-4 whitespace-nowrap text-sm text-gray-500">${car}</td><td class="px-6 py-4 whitespace-nowrap text-sm text-gray-800">${action}</td>`;
            return row;
        }
        
        UI.driverList.addEventListener('click', (e) => {
            const toggleBtn = e.target.closest('.toggle-status-btn');
            const deleteBtn = e.target.closest('.delete-driver-btn');
            const unavailableBtn = e.target.closest('.toggle-unavailable-btn');
            if (toggleBtn) {
                const { id, status, name, car, location } = toggleBtn.dataset;
                if (status === 'available') {
                    const select = toggleBtn.closest('.driver-card').querySelector('.location-select');
                    if (!select.value || select.value === "Select destination...") { alert("Please select a destination before dispatching."); return; }
                    updateDriverStatus(id, { status: 'on the road', location: select.value }, { name, car, action: `Dispatched to ${select.value}` });
                } else if (status === 'on the road') {
                    updateDriverStatus(id, { status: 'available', location: null }, { name, car, action: `Returned from ${location}` });
                } else { updateDriverStatus(id, { status: 'available', location: null }, { name, car, action: `Set to AVAILABLE` }); }
            }
            if (unavailableBtn) {
                const { id, status, name, car } = unavailableBtn.dataset;
                const newStatus = status === 'unavailable' ? 'available' : 'unavailable';
                const action = newStatus === 'available' ? 'Set to AVAILABLE' : 'Set to UNAVAILABLE TODAY';
                updateDriverStatus(id, { status: newStatus, location: null }, { name, car, action });
            }
            if (deleteBtn) {
                const { id, name } = deleteBtn.dataset;
                showConfirmation(`Delete ${name}?`, `Are you sure you want to permanently delete driver ${name}?`, () => deleteDriver(id));
            }
        });

        UI.addDriverForm.addEventListener('submit', async (e) => {
            e.preventDefault();
            const name = document.getElementById('driverName').value.trim();
            const car = document.getElementById('carDetails').value.trim();
            if (name && car) {
                try {
                    await addDoc(driversCollectionRef, { name, car, status: 'available', location: null, createdAt: serverTimestamp() });
                    UI.addDriverForm.reset();
                } catch (error) { console.error("Error adding driver: ", error); }
            }
        });

        async function updateDriverStatus(id, updateData, logData) {
            try {
                await updateDoc(doc(driversCollectionRef, id), { ...updateData, lastUpdated: serverTimestamp() });
                await addDoc(historyCollectionRef, { ...logData, timestamp: serverTimestamp() });
            } catch (error) { console.error("Error updating status: ", error); }
        }

        async function deleteDriver(id) {
            try { await deleteDoc(doc(driversCollectionRef, id)); }
            catch (error) { console.error("Error deleting driver: ", error); }
        }

        function updateLiveStatusChart(drivers) {
            const statusCounts = drivers.reduce((acc, driver) => {
                const status = driver.status || 'unavailable';
                acc[status] = (acc[status] || 0) + 1;
                return acc;
            }, { 'available': 0, 'on the road': 0, 'unavailable': 0 });

            const labels = ['Available', 'On the Road', 'Unavailable'];
            const data = [statusCounts.available, statusCounts['on the road'], statusCounts.unavailable];
            const backgroundColors = ['rgba(75, 192, 192, 0.7)', 'rgba(54, 162, 235, 0.7)', 'rgba(201, 203, 207, 0.7)'];
            
            renderChart('liveStatusChart', 'pie', labels, data, 'Fleet Status', backgroundColors);
        }

        function updateAnalyticsCharts() {
            const todayStart = new Date();
            todayStart.setHours(0, 0, 0, 0);
            const todayHistory = fullHistory.filter(log => log.timestamp && log.timestamp.toDate() >= todayStart);
            
            const driverDispatches = todayHistory.filter(log => log.action.startsWith('Dispatched to')).reduce((acc, log) => { acc[log.name] = (acc[log.name] || 0) + 1; return acc; }, {});
            renderChart('dispatchesByDriverChart', 'bar', Object.keys(driverDispatches), Object.values(driverDispatches), 'Dispatches');

            const peakHours = todayHistory.filter(log => log.action.startsWith('Dispatched to')).reduce((acc, log) => { const hour = log.timestamp.toDate().getHours(); acc[hour] = (acc[hour] || 0) + 1; return acc; }, {});
            const hoursLabels = Array.from({length: 24}, (_, i) => `${i}:00`);
            const hoursData = hoursLabels.map((_, i) => peakHours[i] || 0);
            renderChart('peakHoursChart', 'line', hoursLabels, hoursData, 'Dispatches');

            const locationCounts = todayHistory.filter(log => log.action.startsWith('Dispatched to')).reduce((acc, log) => { const location = log.action.replace('Dispatched to ', ''); acc[location] = (acc[location] || 0) + 1; return acc; }, {});
            renderChart('locationsChart', 'doughnut', Object.keys(locationCounts), Object.values(locationCounts), 'Visits');
        }

        function renderChart(canvasId, type, labels, data, label, backgroundColors) {
            const defaultColors = ['rgba(255, 99, 132, 0.7)', 'rgba(54, 162, 235, 0.7)', 'rgba(255, 206, 86, 0.7)', 'rgba(75, 192, 192, 0.7)', 'rgba(153, 102, 255, 0.7)', 'rgba(255, 159, 64, 0.7)'];
            const ctx = document.getElementById(canvasId).getContext('2d');
            if (window[canvasId]) window[canvasId].destroy();
            window[canvasId] = new Chart(ctx, {
                type: type,
                data: { labels, datasets: [{ label, data, backgroundColor: backgroundColors || defaultColors, borderColor: 'rgba(255, 255, 255, 0.5)', borderWidth: 1 }] },
                options: { responsive: true, maintainAspectRatio: false, scales: type === 'bar' || type === 'line' ? { y: { beginAtZero: true, ticks: { stepSize: 1 } } } : {}, plugins: { legend: { position: 'top' } } }
            });
        }

        UI.exportCsvButton.addEventListener('click', async () => {
            const startDate = UI.startDateInput.value ? new Date(UI.startDateInput.value) : null;
            const endDate = UI.endDateInput.value ? new Date(UI.endDateInput.value) : null;
            if (startDate) startDate.setHours(0, 0, 0, 0);
            if (endDate) endDate.setHours(23, 59, 59, 999);
            let dataToExport = fullHistory.filter(log => {
                if (!log.timestamp) return false;
                const logDate = log.timestamp.toDate();
                const afterStart = startDate ? logDate >= startDate : true;
                const beforeEnd = endDate ? logDate <= endDate : true;
                return afterStart && beforeEnd;
            });
            if (dataToExport.length === 0) { alert("No data found for the selected period."); return; }
            let csvContent = "Timestamp,Driver,Vehicle,Action\n";
            dataToExport.forEach(log => {
                const date = log.timestamp.toDate().toLocaleString('en-US', { timeZone: 'Asia/Manila' });
                csvContent += `"${date}","${log.name}","${log.car}","${log.action}"\n`;
            });
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const link = document.createElement("a");
            link.setAttribute("href", URL.createObjectURL(blob));
            link.setAttribute("download", `logistics_history_${new Date().toISOString().split('T')[0]}.csv`);
            document.body.appendChild(link);
            link.click();
            document.body.removeChild(link);
        });

        // --- Tab and Modal Logic ---
        const tabs = document.querySelectorAll('.tab-button');
        const contents = document.querySelectorAll('.tab-content');
        tabs.forEach(tab => {
            tab.addEventListener('click', () => {
                tabs.forEach(t => t.classList.remove('active', 'text-gray-900') || t.classList.add('text-gray-500', 'border-transparent'));
                contents.forEach(c => c.classList.add('hidden-tab'));
                tab.classList.add('active', 'text-gray-900');
                tab.classList.remove('text-gray-500', 'border-transparent');
                document.getElementById(tab.id.replace('tab-', 'content-')).classList.remove('hidden-tab');
            });
        });
        
        const modal = document.getElementById('confirmationModal');
        let confirmationCallback = null;
        function showConfirmation(title, message, onConfirm) {
            modal.querySelector('#modalTitle').textContent = title;
            modal.querySelector('#modalMessage').textContent = message;
            confirmationCallback = onConfirm;
            modal.classList.remove('hidden');
            setTimeout(() => modal.classList.remove('opacity-0'), 10);
            setTimeout(() => modal.querySelector('.modal-content').classList.remove('scale-95'), 10);
        }
        function hideConfirmation() {
            modal.querySelector('.modal-content').classList.add('scale-95');
            modal.classList.add('opacity-0');
            setTimeout(() => modal.classList.add('hidden'), 300);
            confirmationCallback = null;
        }
        modal.querySelector('#confirmButton').addEventListener('click', () => { if (confirmationCallback) confirmationCallback(); hideConfirmation(); });
        modal.querySelector('#cancelButton').addEventListener('click', hideConfirmation);

        document.addEventListener('DOMContentLoaded', main);
    </script>
</body>
</html>


