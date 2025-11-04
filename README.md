<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Zell Education | NPS Dashboard</title>
    <!-- 1. Load Tailwind CSS for "Apple-like" UI -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- 2. Load a charting library (Chart.js) for visualizations -->
    <script src="https://cdn.jsdelivr.net/npm/chart.js"></script>
    <style>
        /* 3. Use the 'Inter' font family, which is clean and modern */
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap');
        body {
            font-family: 'Inter', sans-serif;
        }
        /* Custom scrollbar for a cleaner look */
        ::-webkit-scrollbar {
            width: 6px;
            height: 6px;
        }
        ::-webkit-scrollbar-thumb {
            background: #cbd5e1; /* slate-300 */
            border-radius: 10px;
        }
        ::-webkit-scrollbar-thumb:hover {
            background: #94a3b8; /* slate-400 */
        }
    </style>
</head>
<body class="bg-zinc-100 text-zinc-900 antialiased">

    <!-- Header -->
    <header class="bg-white/80 backdrop-blur-lg sticky top-0 z-10 shadow-sm">
        <div class="max-w-7xl mx-auto px-4 sm:px-6 lg:px-8">
            <div class="flex justify-between items-center h-20">
                <h1 class="text-3xl font-bold text-zinc-900">NPS Dashboard</h1>
                <p class="text-sm text-zinc-500">Zell Education Feedback</p>
            </div>
        </div>
    </header>

    <!-- Main Content -->
    <main class="max-w-7xl mx-auto p-4 sm:p-6 lg:px-8">

        <!-- Filter Section -->
        <div class="bg-white p-6 rounded-3xl shadow-lg border border-zinc-200/50 mb-8">
            <h3 class="text-xl font-semibold text-zinc-900 mb-4">Filter by Date</h3>
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 items-end">
                <div>
                    <label for="start-date" class="block text-sm font-medium text-zinc-700">Start Date</label>
                    <input type="date" id="start-date" name="start-date" class="mt-1 block w-full rounded-xl border-zinc-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm p-3 h-12">
                </div>
                <div>
                    <label for="end-date" class="block text-sm font-medium text-zinc-700">End Date</label>
                    <input type="date" id="end-date" name="end-date" class="mt-1 block w-full rounded-xl border-zinc-300 shadow-sm focus:border-blue-500 focus:ring-blue-500 sm:text-sm p-3 h-12">
                </div>
                <div class="flex space-x-2">
                    <button id="filter-button" class="w-full h-12 flex justify-center items-center px-6 py-3 border border-transparent rounded-xl shadow-sm text-base font-medium text-white bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">
                        Apply
                    </button>
                    <button id="clear-filter-button" class="w-full h-12 flex justify-center items-center px-6 py-3 border border-zinc-300 rounded-xl shadow-sm text-base font-medium text-zinc-700 bg-white hover:bg-zinc-50 focus:outline-none focus:ring-2 focus:ring-offset-2 focus:ring-blue-500">
                        Clear
                    </button>
                </div>
            </div>
        </div>

        <!-- Loading State -->
        <div id="loading-state" class="flex flex-col items-center justify-center h-64">
            <svg class="animate-spin h-8 w-8 text-blue-600" xmlns="http://www.w3.org/2000/svg" fill="none" viewBox="0 0 24 24">
                <circle class="opacity-25" cx="12" cy="12" r="10" stroke="currentColor" stroke-width="4"></circle>
                <path class="opacity-75" fill="currentColor" d="M4 12a8 8 0 018-8V0C5.373 0 0 5.373 0 12h4zm2 5.291A7.962 7.962 0 014 12H0c0 3.042 1.135 5.824 3 7.938l3-2.647z"></path>
            </svg>
            <p class="mt-4 text-lg font-medium text-zinc-600">Fetching live data...</p>
            <p class="text-sm text-zinc-500">This may take a moment.</p>
        </div>

        <!-- Error State -->
        <div id="error-state" class="hidden bg-red-100 border-l-4 border-red-500 text-red-700 p-6 rounded-2xl shadow-lg" role="alert">
            <h3 class="font-bold text-2xl mb-2">Error Fetching Data</h3>
            <p class="mb-4">Could not load dashboard data from the Google Sheet. Please ensure the deployment URL is correct and the script has permissions.</p>
            <code id="error-message" class="text-sm bg-red-200 p-2 rounded"></code>
        </div>

        <!-- Dashboard Grid (hidden until data is loaded) -->
        <div id="dashboard-content" class="hidden">
            
            <!-- Overall NPS Card -->
            <div class="bg-white p-6 sm:p-8 rounded-3xl shadow-xl border border-zinc-200/50 mb-8 transition-all hover:shadow-2xl">
                <h2 class="text-sm font-medium text-zinc-500 uppercase tracking-wide">Overall Net Promoter Score</h2>
                <div class="flex items-baseline space-x-4 mt-2">
                    <p id="overall-nps" class="text-7xl font-bold text-zinc-900">--</p>
                    <p id="overall-trend" class="text-2xl font-medium text-green-600"></p>
                </div>
                <div class="mt-4 flex flex-wrap gap-x-6 gap-y-4">
                    <div>
                        <p class="text-sm text-zinc-500">Total Responses</p>
                        <p id="overall-total" class="text-2xl font-semibold">--</p>
                    </div>
                    <div>
                        <p class="text-sm text-green-600">Promoters (9-10)</p>
                        <p id="overall-promoters" class="text-2xl font-semibold">--</p>
                    </div>
                    <div>
                        <p class="text-sm text-zinc-500">Passives (7-8)</p>
                        <p id="overall-passives" class="text-2xl font-semibold">--</p>
                    </div>
                    <div>
                        <p class="text-sm text-red-600">Detractors (0-6)</p>
                        <p id="overall-detractors" class="text-2xl font-semibold">--</p>
                    </div>
                </div>
            </div>

            <!-- Main Grid: Charts and Lists -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-8">
                
                <!-- Left Column: Month & Course -->
                <div class="lg:col-span-2 space-y-8">
                    
                    <!-- NPS by Month Chart -->
                    <div class="bg-white p-6 rounded-3xl shadow-xl border border-zinc-200/50 transition-all hover:shadow-2xl">
                        <h3 class="text-xl font-semibold text-zinc-900 mb-4">NPS Trend by Month</h3>
                        <div class="h-80">
                            <canvas id="month-chart"></canvas>
                        </div>
                    </div>

                    <!-- NPS by Course List -->
                    <div class="bg-white p-6 rounded-3xl shadow-xl border border-zinc-200/50 transition-all hover:shadow-2xl">
                        <h3 class="text-xl font-semibold text-zinc-900 mb-4">NPS by Course</h3>
                        <div id="course-wise" class="max-h-96 overflow-y-auto space-y-4 pr-2">
                            <!-- Content generated by JS -->
                        </div>
                    </div>

                </div>
                
                <!-- Right Column: Mentors -->
                <div class="lg:col-span-1">
                    <div class="bg-white p-6 rounded-3xl shadow-xl border border-zinc-200/50 transition-all hover:shadow-2xl">
                        <h3 class="text-xl font-semibold text-zinc-900 mb-4">NPS by Mentor</h3>
                        <div id="mentor-wise" class="max-h-[44rem] overflow-y-auto space-y-4 pr-2">
                            <!-- Content generated by JS -->
                        </div>
                    </div>
                </div>

            </div>
        </div>
    </main>

    <footer class="text-center p-8 text-zinc-500 text-sm">
        NPS Dashboard v1.1
    </footer>

    <script>
        // --- CONFIGURATION ---
        // ** UPDATED with your new Deployment ID **
        const WEB_APP_URL = "https://script.google.com/macros/s/AKfycbxDNGpqf1D5Nwy2DrUZmhF9mwPAqRTEaH1CZcwdNZfy7JV-MiZNzYY864-ZcMIwtsIU/exec";
        
        // These keys MUST match the "clean" header keys from your WebApp.gs script
        const NPS_SCORE_KEY = "On_a_scale_of_0-10,_how_likely_are_you_to_recommend_Zell_Education_to_a_friend_or_a_colleague?";
        const MENTOR_KEY = "Mentor_Name";
        const COURSE_KEY = "Course_Enrolled";
        const MONTH_KEY = "Month";
        const TIMESTAMP_KEY = "Timestamp";

        // --- DOM Elements ---
        const loadingState = document.getElementById('loading-state');
        const errorState = document.getElementById('error-state');
        const errorMessage = document.getElementById('error-message');
        const dashboardContent = document.getElementById('dashboard-content');
        
        // --- Filter Elements ---
        const startDateInput = document.getElementById('start-date');
        const endDateInput = document.getElementById('end-date');
        const filterButton = document.getElementById('filter-button');
        const clearFilterButton = document.getElementById('clear-filter-button');

        // --- Data Store ---
        let allDataStore = []; // To hold the original, complete dataset
        let monthChartInstance = null; // To hold the chart object for destroying

        // --- NPS Calculation Logic ---
        
        /**
         * Calculates NPS details from an array of data.
         * @param {Array<Object>} data - Array of response objects.
         * @returns {Object} { nps, total, promoters, passives, detractors }
         */
        function calculateNPS(data) {
            let promoters = 0;
            let passives = 0;
            let detractors = 0;
            const total = data.length;

            if (total === 0) {
                return { nps: 0, total: 0, promoters: 0, passives: 0, detractors: 0 };
            }

            data.forEach(row => {
                // Check if the score key exists and is a number
                const scoreValue = row[NPS_SCORE_KEY];
                if (scoreValue !== null && scoreValue !== undefined && scoreValue !== "") {
                    const score = parseInt(scoreValue, 10);
                    if (!isNaN(score)) {
                        if (score >= 9) {
                            promoters++;
                        } else if (score >= 7) {
                            passives++;
                        } else if (score >= 0) {
                            detractors++;
                        }
                    }
                }
            });
            
            // Recalculate total based on valid responses
            const validTotal = promoters + passives + detractors;
             if (validTotal === 0) {
                return { nps: 0, total: 0, promoters: 0, passives: 0, detractors: 0 };
            }

            const promoterPercent = (promoters / validTotal) * 100;
            const detractorPercent = (detractors / validTotal) * 100;
            const nps = Math.round(promoterPercent - detractorPercent);

            return { nps, total: validTotal, promoters, passives, detractors };
        }

        /**
         * Groups data by a specific key (e.g., "Mentor_Name").
         * @param {Array<Object>} data - The full dataset.
         * @param {string} key - The key to group by (e.g., MENTOR_KEY).
         * @returns {Object} { "Group Name": [Array of data], ... }
         */
        function groupDataByKey(data, key) {
            return data.reduce((acc, row) => {
                // Check if the key exists in the row
                if (row.hasOwnProperty(key)) {
                    let groupName = row[key] ? row[key].toString().trim() : "N/A";
                    
                    // Consolidate empty or placeholder names
                    if (groupName === "" || groupName === "-" || groupName === "N/A") {
                        groupName = "N/A";
                    }

                    if (!acc[groupName]) {
                        acc[groupName] = [];
                    }
                    acc[groupName].push(row);
                }
                return acc;
            }, {});
        }

        /**
         * Gets the CSS color class for an NPS score.
         * @param {number} nps - The NPS score.
         * @returns {string} Tailwind CSS text color class.
         */
        function getNPSColor(nps) {
            if (nps > 50) return 'text-green-600';
            if (nps > 0) return 'text-yellow-600';
            if (nps <= 0) return 'text-red-600';
            return 'text-zinc-900';
        }

        // --- Rendering Functions ---

        /**
         * Renders the main "Overall NPS" card.
         * @param {Object} npsData - The result from calculateNPS().
         */
        function renderOverallNPS(npsData) {
            document.getElementById('overall-nps').textContent = npsData.nps;
            document.getElementById('overall-nps').className = `text-7xl font-bold ${getNPSColor(npsData.nps)}`;
            document.getElementById('overall-total').textContent = npsData.total;
            document.getElementById('overall-promoters').textContent = npsData.promoters;
            document.getElementById('overall-passives').textContent = npsData.passives;
            document.getElementById('overall-detractors').textContent = npsData.detractors;
        }

        /**
         * Renders the "NPS by Month" chart.
         * @param {Object} monthGroups - Grouped data by month.
         */
        function renderMonthChart(monthGroups) {
            const ctx = document.getElementById('month-chart').getContext('2d');
            
            // Destroy the old chart instance if it exists
            if (monthChartInstance) {
                monthChartInstance.destroy();
            }

            // Sort months by the first timestamp in each group to ensure chronological order
            const sortedMonthData = Object.entries(monthGroups)
                .map(([key, data]) => {
                    // Find the earliest timestamp in the group
                    const earliestTimestamp = data.reduce((earliest, row) => {
                        // Ensure row[TIMESTAMP_KEY] is valid before creating a Date
                        const currentTimestamp = row[TIMESTAMP_KEY] ? new Date(row[TIMESTAMP_KEY]) : new Date(0);
                        const earliestValid = earliest || currentTimestamp; // Handle initial case
                        return currentTimestamp < earliestValid ? currentTimestamp : earliestValid;
                    }, null); // Start with null
                    
                    // Use a very early date if no valid timestamp was found
                    const finalTimestamp = earliestTimestamp || new Date(0); 

                    return { key, data, earliestTimestamp: finalTimestamp };
                })
                .sort((a, b) => a.earliestTimestamp - b.earliestTimestamp);

            // Handle empty data for the chart
            if (sortedMonthData.length === 0 || sortedMonthData.every(item => item.data.length === 0)) {
                ctx.clearRect(0, 0, ctx.canvas.width, ctx.canvas.height); // Clear canvas
                ctx.font = "16px Inter";
                ctx.fillStyle = "#6b7280"; // text-zinc-500
                ctx.textAlign = "center";
                ctx.fillText("No data for this period", ctx.canvas.width / 2, ctx.canvas.height / 2);
                return; // Don't try to create the chart
            }

            const labels = sortedMonthData.map(entry => entry.key); // entry.key is the month name
            const npsValues = sortedMonthData.map(entry => calculateNPS(entry.data).nps); // entry.data is the data array

            // Store the new chart instance
            monthChartInstance = new Chart(ctx, {
                type: 'line',
                data: {
                    labels: labels,
                    datasets: [{
                        label: 'NPS',
                        data: npsValues,
                        borderColor: 'rgb(59, 130, 246)', // blue-500
                        backgroundColor: 'rgba(59, 130, 246, 0.1)',
                        fill: true,
                        tension: 0.3,
                        borderWidth: 3,
                        pointBackgroundColor: 'rgb(59, 130, 246)',
                        pointRadius: 5,
                        pointHoverRadius: 7
                    }]
                },
                options: {
                    responsive: true,
                    maintainAspectRatio: false,
                    scales: {
                        y: {
                            beginAtZero: false,
                            min: -100,
                            max: 100,
                            grid: {
                                color: '#e2e8f0' // slate-200
                            }
                        },
                        x: {
                             grid: {
                                display: false
                            }
                        }
                    },
                    plugins: {
                        legend: {
                            display: false
                        },
                        tooltip: {
                            mode: 'index',
                            intersect: false,
                            backgroundColor: '#18181b', // zinc-900
                            titleFont: { size: 16, weight: 'bold' },
                            bodyFont: { size: 14 },
                            padding: 12,
                            cornerRadius: 8,
                            callbacks: {
                                label: function(context) {
                                    return `NPS: ${context.raw}`;
                                }
                            }
                        }
                    }
                }
            });
        }
        
        /**
         * Creates and renders the lists for mentors and courses.
         * @param {string} containerId - The ID of the container to render into.
         */
        function renderGroupList(groupedData, containerId) {
            const container = document.getElementById(containerId);
            container.innerHTML = ''; // Clear existing

            // Calculate NPS for all groups and sort by total responses (descending)
            const sortedData = Object.keys(groupedData)
                .map(groupName => {
                    const data = groupedData[groupName];
                    const npsData = calculateNPS(data);
                    return { name: groupName, ...npsData };
                })
                .filter(item => item.name !== "N/A" && item.total > 0) // Filter out "N/A" and groups with no valid scores
                .sort((a, b) => b.total - a.total); // Sort by total responses

            if (sortedData.length === 0) {
                container.innerHTML = `<p class="text-zinc-500">No data available for this category.</p>`;
                return;
            }

            // Create HTML for each item
            sortedData.forEach(item => {
                const itemHtml = `
                    <div class="flex items-center justify-between p-4 bg-zinc-50 rounded-2xl border border-zinc-200/80">
                        <div>
                            <p class="text-base font-semibold text-zinc-800">${item.name}</p>
                            <p class="text-sm text-zinc-500">
                                ${item.total} Responses
                                <span class="mx-1.5">|</span>
                                <span class="text-green-600">${item.promoters} P</span> / 
                                <span class="text-zinc-500">${item.passives} A</span> / 
                                <span class="text-red-600">${item.detractors} D</span>
                            </p>
                        </div>
                        <p class="text-3xl font-bold ${getNPSColor(item.nps)}">${item.nps}</p>
                    </div>
                `;
                container.innerHTML += itemHtml;
            });
        }
        
        // --- Main Function ---
        
        /**
         * Main function to fetch, process, and render all data.
         */
        async function loadDashboard() {
            try {
                // 1. Fetch Data
                const response = await fetch(WEB_APP_URL);
                if (!response.ok) {
                    throw new Error(`Network response was not ok (Status: ${response.status})`);
                }
                const result = await response.json();

                if (result.status !== 'success') {
                    throw new Error(result.message || "API returned an error status.");
                }

                allDataStore = result.data; // Store the original data
                if (!allDataStore || allDataStore.length === 0) {
                     throw new Error("No data found in the sheet.");
                }

                // 2. Initial Render
                updateDashboard(allDataStore); // Render the dashboard with all data
                
                // 3. Add Filter Event Listeners
                filterButton.addEventListener('click', () => {
                    const startDateVal = startDateInput.value;
                    const endDateVal = endDateInput.value;

                    // If no dates, do nothing (or show all, but 'Clear' button is for that)
                    if (!startDateVal && !endDateVal) {
                        updateDashboard(allDataStore); // Show all if fields are empty
                        return;
                    }
                    
                    // Set defaults if one is missing
                    // new Date(startDateVal + "T00:00:00") parses in local time
                    const start = startDateVal ? new Date(startDateVal + "T00:00:00") : new Date(0); // Epoch start
                    const end = endDateVal ? new Date(endDateVal + "T23:59:59") : new Date(); // Now

                    const filteredData = allDataStore.filter(row => {
                        const rowDate = row[TIMESTAMP_KEY] ? new Date(row[TIMESTAMP_KEY]) : null;
                        if (!rowDate) return false; // Exclude rows with invalid/missing timestamps
                        return rowDate >= start && rowDate <= end;
                    });
                    
                    updateDashboard(filteredData);
                });

                clearFilterButton.addEventListener('click', () => {
                    startDateInput.value = '';
                    endDateInput.value = '';
                    updateDashboard(allDataStore); // Reset to all data
                });

                // 4. Show Content
                loadingState.classList.add('hidden');
                dashboardContent.classList.remove('hidden');

            } catch (error) {
                console.error("Failed to load dashboard:", error);
                errorMessage.textContent = error.message;
                loadingState.classList.add('hidden');
                errorState.classList.remove('hidden');
            }
        }

        /**
         * New function to update all dashboard components with given data.
         * @param {Array<Object>} data - The data to render (can be filtered or full dataset).
         */
        function updateDashboard(data) {
            // 1. Process Data
            
            // Calculate Overall NPS
            const overallNPS = calculateNPS(data);

            // Group data by keys
            const monthGroups = groupDataByKey(data, MONTH_KEY);
            const mentorGroups = groupDataByKey(data, MENTOR_KEY);
            const courseGroups = groupDataByKey(data, COURSE_KEY);

            // 2. Render Dashboard
            renderOverallNPS(overallNPS);
            renderMonthChart(monthGroups);
            renderGroupList(mentorGroups, 'mentor-wise');
            renderGroupList(courseGroups, 'course-wise');
        }


        // --- INITIATE ---
        document.addEventListener('DOMContentLoaded', loadDashboard);
    </script>
</body>
</html>

