# NaYtx58.github.io
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <meta name="apple-mobile-web-app-capable" content="yes">
    <meta name="apple-mobile-web-app-status-bar-style" content="black-translucent">
    <title>Cycle Sync</title>
    
    <!-- Tailwind CSS -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- Font Awesome -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <style>
        body {
            font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, Helvetica, Arial, sans-serif;
            background-color: #fdf2f8; /* Pink-50 */
            -webkit-tap-highlight-color: transparent;
        }
        
        /* Smooth transitions */
        .transition-all {
            transition-property: all;
            transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
            transition-duration: 300ms;
        }

        /* Custom phase colors */
        .phase-menstrual { background: linear-gradient(135deg, #ef4444, #b91c1c); }
        .phase-follicular { background: linear-gradient(135deg, #f472b6, #db2777); }
        .phase-ovulation { background: linear-gradient(135deg, #fbbf24, #d97706); }
        .phase-luteal { background: linear-gradient(135deg, #a78bfa, #7c3aed); }
    </style>
</head>
<body class="h-screen flex flex-col items-center justify-center p-4 text-gray-800">

    <!-- SETUP SCREEN (Hidden if data exists) -->
    <div id="setupScreen" class="w-full max-w-md bg-white rounded-3xl shadow-xl p-8 hidden">
        <div class="text-center mb-6">
            <i class="fa-solid fa-heart-pulse text-4xl text-pink-500 mb-4"></i>
            <h1 class="text-2xl font-bold text-gray-800">Welcome</h1>
            <p class="text-gray-500 text-sm mt-2">Let's set up the tracker.</p>
        </div>

        <div class="space-y-4">
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Last Period Start Date</label>
                <input type="date" id="startDateInput" class="w-full p-3 rounded-xl border border-pink-200 focus:outline-none focus:ring-2 focus:ring-pink-400 bg-pink-50">
            </div>
            
            <div>
                <label class="block text-sm font-medium text-gray-700 mb-1">Average Cycle Length (Days)</label>
                <input type="number" id="cycleLengthInput" value="28" class="w-full p-3 rounded-xl border border-pink-200 focus:outline-none focus:ring-2 focus:ring-pink-400 bg-pink-50">
            </div>

            <button onclick="saveData()" class="w-full bg-pink-500 hover:bg-pink-600 text-white font-bold py-4 rounded-xl shadow-lg transition-all transform active:scale-95 mt-4">
                Start Tracking
            </button>
        </div>
    </div>

    <!-- MAIN DASHBOARD (Hidden if no data) -->
    <div id="dashboard" class="w-full max-w-md hidden flex-col h-full justify-between py-6">
        
        <!-- Header -->
        <div class="flex justify-between items-center px-2">
            <h2 class="text-xl font-bold text-gray-700">Cycle Tracker</h2>
            <button onclick="showSettings()" class="text-gray-400 hover:text-gray-600"><i class="fa-solid fa-gear text-xl"></i></button>
        </div>

        <!-- Main Status Card -->
        <div id="statusCard" class="phase-menstrual text-white rounded-[2rem] p-8 shadow-2xl relative overflow-hidden mt-6 mb-6">
            <!-- Background Decoration -->
            <div class="absolute top-0 right-0 -mt-4 -mr-4 w-32 h-32 bg-white opacity-10 rounded-full blur-2xl"></div>
            
            <div class="relative z-10 text-center">
                <p class="text-white/80 text-sm uppercase tracking-wider font-semibold mb-2">Current Phase</p>
                <h1 id="phaseName" class="text-4xl font-bold mb-1">Menstrual</h1>
                <p id="dayCount" class="text-xl opacity-90 font-medium">Day 1</p>
                
                <div class="mt-8 bg-black/20 rounded-full h-2 w-full overflow-hidden">
                    <div id="progressBar" class="bg-white h-full rounded-full transition-all" style="width: 10%;"></div>
                </div>
                <p id="daysLeft" class="text-xs mt-2 text-right opacity-75">27 days left in cycle</p>
            </div>
        </div>

        <!-- Info Grid -->
        <div class="grid grid-cols-2 gap-4 mb-auto">
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-pink-100">
                <i class="fa-solid fa-calendar-check text-pink-400 mb-2"></i>
                <p class="text-xs text-gray-400 font-bold uppercase">Next Period</p>
                <p id="nextPeriodDate" class="text-lg font-bold text-gray-700">--</p>
            </div>
            <div class="bg-white p-5 rounded-2xl shadow-sm border border-pink-100">
                <i class="fa-solid fa-venus text-purple-400 mb-2"></i>
                <p class="text-xs text-gray-400 font-bold uppercase">Fertility</p>
                <p id="fertilityStatus" class="text-lg font-bold text-gray-700">Low</p>
            </div>
        </div>

        <!-- Description Box -->
        <div class="bg-white p-6 rounded-2xl shadow-sm border border-pink-100 mt-4">
            <h3 class="font-bold text-gray-800 mb-2"><i class="fa-solid fa-circle-info text-blue-400 mr-2"></i>What to expect</h3>
            <p id="phaseDescription" class="text-sm text-gray-500 leading-relaxed">
                Energy levels might be lower. Good time for rest and self-care.
            </p>
        </div>

        <!-- Update Button -->
        <div class="mt-6 text-center">
             <button onclick="periodStartedToday()" class="text-sm text-pink-500 font-semibold underline decoration-pink-300 decoration-2 underline-offset-4">
                Period started today? Reset
            </button>
        </div>

    </div>

    <script>
        // --- LOGIC ---

        const STORAGE_KEY = 'gf_cycle_data';
        
        // Load data on startup
        window.onload = function() {
            const data = JSON.parse(localStorage.getItem(STORAGE_KEY));
            if (data) {
                updateDashboard(data);
                document.getElementById('setupScreen').classList.add('hidden');
                document.getElementById('dashboard').classList.remove('hidden');
                document.getElementById('dashboard').classList.add('flex');
            } else {
                document.getElementById('setupScreen').classList.remove('hidden');
            }
        };

        function saveData() {
            const startStr = document.getElementById('startDateInput').value;
            const lengthStr = document.getElementById('cycleLengthInput').value;

            if (!startStr || !lengthStr) {
                alert("Please fill in both fields.");
                return;
            }

            const data = {
                startDate: startStr,
                cycleLength: parseInt(lengthStr)
            };

            localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
            location.reload(); // Refresh to show dashboard
        }

        function showSettings() {
            if(confirm("Do you want to reset the settings?")) {
                localStorage.removeItem(STORAGE_KEY);
                location.reload();
            }
        }

        function periodStartedToday() {
            if(confirm("Confirm that a new period started today? This will reset the cycle to Day 1.")) {
                const data = JSON.parse(localStorage.getItem(STORAGE_KEY));
                const today = new Date().toISOString().split('T')[0];
                data.startDate = today;
                localStorage.setItem(STORAGE_KEY, JSON.stringify(data));
                location.reload();
            }
        }

        function updateDashboard(data) {
            const start = new Date(data.startDate);
            const today = new Date();
            
            // Calculate day of cycle (difference in time / milliseconds in a day)
            const diffTime = Math.abs(today - start);
            const diffDays = Math.ceil(diffTime / (1000 * 60 * 60 * 24)); 
            
            // Cycle math
            const dayOfCycle = diffDays;
            const length = data.cycleLength;
            
            // Determine Phase
            let phase = "";
            let phaseClass = "";
            let description = "";
            let fertility = "Low";

            // Standard approximations
            if (dayOfCycle >= 1 && dayOfCycle <= 5) {
                phase = "Menstrual";
                phaseClass = "phase-menstrual";
                description = "Energy is low. Cramps are possible. Great time for rest, comfort food, and low-stress activities.";
                fertility = "Low";
            } else if (dayOfCycle > 5 && dayOfCycle <= 13) {
                phase = "Follicular";
                phaseClass = "phase-follicular";
                description = "Energy is rising! She might feel more social, creative, and energetic as estrogen increases.";
                fertility = "Medium";
            } else if (dayOfCycle === 14 || dayOfCycle === 15) {
                phase = "Ovulation";
                phaseClass = "phase-ovulation";
                description = "Peak energy and mood. This is the most fertile window.";
                fertility = "High";
            } else if (dayOfCycle > 15 && dayOfCycle <= length) {
                phase = "Luteal";
                phaseClass = "phase-luteal";
                description = "PMS phase. Energy drops, cravings start. Be patient, offer chocolate, and give extra support.";
                fertility = "Low";
            } else {
                phase = "Late / Irregular";
                phaseClass = "bg-gray-500";
                description = "Cycle is longer than average setting. Period should arrive soon.";
            }

            // Update UI Elements
            document.getElementById('phaseName').innerText = phase;
            document.getElementById('dayCount').innerText = `Day ${dayOfCycle}`;
            document.getElementById('phaseDescription').innerText = description;
            document.getElementById('fertilityStatus').innerText = fertility;
            
            // Card Styling
            const card = document.getElementById('statusCard');
            card.className = `${phaseClass} text-white rounded-[2rem] p-8 shadow-2xl relative overflow-hidden mt-6 mb-6`;

            // Progress Bar
            const percentage = Math.min((dayOfCycle / length) * 100, 100);
            document.getElementById('progressBar').style.width = `${percentage}%`;
            
            const daysLeft = length - dayOfCycle;
            if (daysLeft > 0) {
                document.getElementById('daysLeft').innerText = `${daysLeft} days until next period`;
            } else {
                document.getElementById('daysLeft').innerText = `Period expected`;
            }

            // Next Period Date
            const nextDate = new Date(start);
            nextDate.setDate(start.getDate() + length);
            document.getElementById('nextPeriodDate').innerText = nextDate.toLocaleDateString('en-US', { month: 'short', day: 'numeric' });
        }
    </script>
</body>
</html>
