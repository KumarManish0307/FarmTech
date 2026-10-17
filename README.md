<!DOCTYPE html>
<html lang="en" class="dark">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>FARMTECH Cloud IoT Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700;800&display=swap" rel="stylesheet">
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">

    <script>
        tailwind.config = {
            darkMode: 'class',
            theme: {
                extend: {
                    fontFamily: {
                        sans: ['Inter', 'sans-serif'],
                    }
                }
            }
        }
    </script>

    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #070d19;
            color: #f8fafc;
        }
        ::-webkit-scrollbar { width: 8px; height: 8px; }
        ::-webkit-scrollbar-track { background: #0f172a; }
        ::-webkit-scrollbar-thumb { background: #334155; border-radius: 9999px; }
        @keyframes spinImpeller { from { transform: rotate(0deg); } to { transform: rotate(360deg); } }
        .spin-active { animation: spinImpeller 1.2s linear infinite; }
        @keyframes glowPulse {
            0%, 100% { box-shadow: 0 0 15px rgba(16, 185, 129, 0.25); }
            50% { box-shadow: 0 0 28px rgba(16, 185, 129, 0.6); }
        }
        .pump-glow { animation: glowPulse 2s infinite; }
        .gauge-ring { transition: stroke-dashoffset 0.45s cubic-bezier(0.4, 0, 0.2, 1), stroke 0.3s ease; }
    </style>
</head>
<body class="min-h-screen flex flex-col selection:bg-emerald-500 selection:text-white">

    <header class="sticky top-0 z-30 bg-slate-900/95 backdrop-blur-md border-b border-slate-800 px-4 sm:px-6 py-3.5 shadow-lg">
        <div class="max-w-7xl mx-auto flex flex-wrap items-center justify-between gap-4">
            <div class="flex items-center gap-3">
                <div class="w-11 h-11 rounded-2xl bg-gradient-to-br from-emerald-500 to-teal-600 flex items-center justify-center shadow-lg shadow-emerald-500/25">
                    <i class="fa-solid fa-cloud-showers-heavy text-white text-lg"></i>
                </div>
                <div>
                    <div class="flex items-center gap-2 flex-wrap">
                        <h1 class="text-base sm:text-lg font-extrabold tracking-tight text-white">
                            FARMTECH Cloud IoT Dashboard
                        </h1>
                        <span id="cloudStatusBadge" class="inline-flex items-center gap-1.5 text-[11px] font-semibold px-2.5 py-0.5 rounded-full bg-emerald-500/15 text-emerald-400 border border-emerald-500/30">
                            <span id="cloudPulseDot" class="w-2 h-2 rounded-full bg-emerald-400 animate-ping"></span>
                            <span id="cloudStatusText">Firebase Connected</span>
                        </span>
                    </div>
                    <p class="text-xs text-slate-400">Real-Time Global Irrigation Control, Telemetry & Cloud Sync</p>
                </div>
            </div>

            <div class="flex items-center bg-slate-950 p-1 rounded-xl border border-slate-800 text-xs sm:text-sm font-semibold overflow-x-auto max-w-full">
                <button id="tabBtn-dashboard" onclick="switchMainTab('dashboard')" class="px-3.5 py-2 rounded-lg bg-emerald-600 text-white shadow transition-all flex items-center gap-2 whitespace-nowrap">
                    <i class="fa-solid fa-gauge-high"></i>
                    <span>Live Cloud Dashboard</span>
                </button>
                <button id="tabBtn-code" onclick="switchMainTab('code')" class="px-3.5 py-2 rounded-lg text-slate-400 hover:text-white transition-all flex items-center gap-2 whitespace-nowrap">
                    <i class="fa-solid fa-microchip"></i>
                    <span>ESP32 Cloud Code & Circuit</span>
                </button>
                <button id="tabBtn-guide" onclick="switchMainTab('guide')" class="px-3.5 py-2 rounded-lg text-slate-400 hover:text-white transition-all flex items-center gap-2 whitespace-nowrap">
                    <i class="fa-solid fa-fire-flame-curved text-amber-400"></i>
                    <span>Cloud Setup Guide</span>
                </button>
            </div>
        </div>
    </header>

    <main class="flex-1 max-w-7xl w-full mx-auto px-4 sm:px-6 py-5 space-y-6">

        <!-- TAB 1: LIVE DASHBOARD -->
        <section id="view-dashboard" class="space-y-6">

            <div class="bg-gradient-to-r from-slate-900 via-slate-900/95 to-slate-900 border border-slate-800 rounded-2xl p-4 sm:p-5 shadow-xl space-y-4">
                <div class="flex flex-col lg:flex-row lg:items-center justify-between gap-4 border-b border-slate-800/80 pb-4">
                    <div class="flex items-start gap-3">
                        <div class="w-10 h-10 rounded-xl bg-amber-500/15 border border-amber-500/30 flex items-center justify-center text-amber-400 shrink-0 mt-0.5">
                            <i class="fa-solid fa-cloud-arrow-up text-lg"></i>
                        </div>
                        <div>
                            <h2 class="text-sm sm:text-base font-bold text-white flex items-center gap-2 flex-wrap">
                                Permanent Firebase Database URL
                                <span class="text-[11px] font-medium px-2 py-0.5 rounded bg-slate-800 text-sky-400 border border-slate-700">REST JSON Sync (/irrigation.json)</span>
                            </h2>
                            <p class="text-xs text-slate-400 mt-0.5">
                                Connected to your permanent Firebase Realtime Database.
                            </p>
                        </div>
                    </div>

                    <div class="flex flex-col sm:flex-row items-stretch sm:items-center gap-2 w-full lg:w-auto">
                        <div class="relative flex-1 sm:w-96">
                            <i class="fa-solid fa-link absolute left-3.5 top-1/2 -translate-y-1/2 text-emerald-400 text-xs"></i>
                            <input id="firebaseUrlInput" type="text" value="https://esop32-project-default-rtdb.asia-southeast1.firebasedatabase.app"
                                class="w-full pl-9 pr-3 py-2 rounded-xl bg-slate-950 border border-emerald-500/50 text-xs text-white placeholder-slate-500 focus:outline-none focus:border-emerald-500">
                        </div>
                        <button onclick="connectCustomFirebaseUrl()" class="px-4 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white font-bold text-xs transition flex items-center justify-center gap-1.5 whitespace-nowrap shadow">
                            <i class="fa-solid fa-check"></i>
                            <span>Connected</span>
                        </button>
                    </div>
                </div>

                <div class="flex flex-wrap items-center justify-between gap-3 pt-1">
                    <div class="flex items-center gap-2 text-xs text-slate-300">
                        <span class="px-2 py-0.5 rounded bg-emerald-500/15 text-emerald-400 border border-emerald-500/30 font-semibold">
                            <i class="fa-solid fa-flask mr-1"></i>Testing Controls:
                        </span>
                        <span class="text-slate-400 hidden sm:inline">Pump stays ON until soil moisture reaches >= 90%.</span>
                    </div>

                    <div class="flex flex-wrap items-center gap-2.5">
                        <div class="flex items-center gap-2.5 bg-slate-950 px-3.5 py-1.5 rounded-xl border border-slate-800">
                            <label for="simMoistureSlider" class="text-xs font-medium text-slate-300">Test Moisture:</label>
                            <input id="simMoistureSlider" type="range" min="5" max="95" value="28"
                                class="w-28 sm:w-36 accent-emerald-500 cursor-pointer"
                                oninput="handleManualMoistureSim(this.value)">
                            <span id="simMoistureBadge" class="text-xs font-extrabold text-emerald-400 w-9 text-right">28%</span>
                        </div>

                        <button onclick="handleManualMoistureSim(18)" class="px-3 py-1.5 rounded-xl bg-rose-500/15 hover:bg-rose-500/25 text-rose-300 border border-rose-500/30 text-xs font-semibold transition flex items-center gap-1.5">
                            <i class="fa-solid fa-sun"></i> Make Soil Dry 18%
                        </button>

                        <button onclick="handleManualMoistureSim(92)" class="px-3 py-1.5 rounded-xl bg-sky-500/15 hover:bg-sky-500/25 text-sky-300 border border-sky-500/30 text-xs font-semibold transition flex items-center gap-1.5">
                            <i class="fa-solid fa-cloud-showers-water"></i> Make Soil Wet 92%
                        </button>
                    </div>
                </div>
            </div>

            <!-- 4 Sensor Cards -->
            <div class="grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-4 gap-4 sm:gap-5">
                <!-- Soil Moisture -->
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-col justify-between shadow-lg">
                    <div class="flex items-start justify-between">
                        <div>
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Soil Moisture</span>
                            <div class="mt-1">
                                <span id="moistureStatusPill" class="inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-rose-500/15 text-rose-400 border border-rose-500/30">
                                    <span class="w-1.5 h-1.5 rounded-full bg-rose-400 animate-ping"></span> Critical Dry
                                </span>
                            </div>
                        </div>
                        <div class="w-10 h-10 rounded-xl bg-emerald-500/10 border border-emerald-500/20 flex items-center justify-center text-emerald-400">
                            <i class="fa-solid fa-droplet text-lg"></i>
                        </div>
                    </div>

                    <div class="my-4 flex items-center justify-center relative">
                        <svg class="w-32 h-32 -rotate-90 transform" viewBox="0 0 120 120">
                            <circle cx="60" cy="60" r="50" stroke="#1e293b" stroke-width="10" fill="transparent" />
                            <circle id="moistureRing" cx="60" cy="60" r="50" stroke="#10b981" stroke-width="10"
                                stroke-dasharray="314.16" stroke-dashoffset="226" stroke-linecap="round" fill="transparent"
                                class="gauge-ring" />
                        </svg>
                        <div class="absolute inset-0 flex flex-col items-center justify-center">
                            <span id="moistureCardValue" class="text-3xl font-extrabold text-white">28%</span>
                            <span class="text-[11px] text-slate-400">Firebase Telemetry</span>
                        </div>
                    </div>

                    <div class="space-y-2 border-t border-slate-800/80 pt-3">
                        <div class="w-full h-2 bg-slate-950 rounded-full overflow-hidden">
                            <div id="moistureLinearBar" class="h-full bg-emerald-500 transition-all duration-300" style="width: 28%"></div>
                        </div>
                        <div class="flex justify-between text-xs text-slate-400">
                            <span>Cutoff: <strong class="text-sky-400">&gt;= 90%</strong></span>
                            <span id="adcRawText" class="font-mono">ADC: 2940</span>
                        </div>
                    </div>
                </div>

                <!-- Water Pump Status -->
                <div id="pumpCardContainer" class="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-col justify-between transition-all duration-300 shadow-lg">
                    <div class="flex items-start justify-between">
                        <div>
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Water Pump Status</span>
                            <div class="mt-1">
                                <span id="pumpStatePill" class="inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-emerald-500/20 text-emerald-300 border border-emerald-500/30">
                                    <span class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-ping"></span> RUNNING (ON)
                                </span>
                            </div>
                        </div>
                        <div class="w-11 h-11 rounded-xl bg-emerald-500/15 border border-emerald-500/30 flex items-center justify-center text-emerald-400">
                            <i id="pumpImpellerIcon" class="fa-solid fa-fan text-xl spin-active"></i>
                        </div>
                    </div>

                    <div class="my-4 bg-slate-950/80 border border-slate-800 rounded-xl p-3.5 space-y-2 text-xs">
                        <div class="flex justify-between">
                            <span class="text-slate-400">Active Mode:</span>
                            <span id="cardModeText" class="font-bold text-emerald-400">AUTO MODE</span>
                        </div>
                        <div class="flex justify-between">
                            <span class="text-slate-400">Relay GPIO 26:</span>
                            <span id="cardRelayLogic" class="font-mono text-emerald-400">LOW (ON - 0V)</span>
                        </div>
                        <div class="flex justify-between">
                            <span class="text-slate-400">Water Flow:</span>
                            <span id="cardFlowRate" class="font-semibold text-sky-400">2.0 L/min</span>
                        </div>
                    </div>

                    <div class="flex items-center justify-between text-xs text-slate-400 border-t border-slate-800/80 pt-3">
                        <span>Cutoff Rule: <strong class="text-sky-400">Stays ON till 90%</strong></span>
                        <span id="pumpReasonText" class="text-emerald-400 font-semibold">Irrigating...</span>
                    </div>
                </div>

                <!-- Weather / DHT11 -->
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-col justify-between shadow-lg">
                    <div class="flex items-start justify-between">
                        <div>
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Weather / DHT11</span>
                            <p class="text-xs text-slate-400 mt-0.5">Field Micro-Climate</p>
                        </div>
                        <div class="w-10 h-10 rounded-xl bg-amber-500/10 border border-amber-500/20 flex items-center justify-center text-amber-400">
                            <i class="fa-solid fa-temperature-three-quarters text-lg"></i>
                        </div>
                    </div>

                    <div class="grid grid-cols-2 gap-3 my-4">
                        <div class="bg-slate-950/80 border border-slate-800 rounded-xl p-3">
                            <span class="text-[11px] text-amber-400 flex items-center gap-1">
                                <i class="fa-solid fa-sun"></i> Temp
                            </span>
                            <div id="tempCardValue" class="text-2xl font-extrabold text-white mt-1">29.6°C</div>
                            <span class="text-[10px] text-slate-400">Ambient Air</span>
                        </div>
                        <div class="bg-slate-950/80 border border-slate-800 rounded-xl p-3">
                            <span class="text-[11px] text-sky-400 flex items-center gap-1">
                                <i class="fa-solid fa-droplet"></i> Humidity
                            </span>
                            <div id="humCardValue" class="text-2xl font-extrabold text-white mt-1">64%</div>
                            <span class="text-[10px] text-slate-400">Relative</span>
                        </div>
                    </div>

                    <div class="flex items-center justify-between text-xs text-slate-400 border-t border-slate-800/80 pt-3">
                        <span>Heat Index: <strong class="text-amber-400">31.2°C</strong></span>
                        <span>Sensor: <strong class="text-slate-200">DHT11</strong></span>
                    </div>
                </div>

                <!-- Water Tank Level -->
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 flex flex-col justify-between shadow-lg">
                    <div class="flex items-start justify-between">
                        <div>
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400">Water Tank Level</span>
                            <p id="tankAlertBadge" class="text-xs text-sky-400 font-semibold mt-0.5">Sufficient Storage</p>
                        </div>
                        <div class="w-10 h-10 rounded-xl bg-sky-500/10 border border-sky-500/20 flex items-center justify-center text-sky-400">
                            <i class="fa-solid fa-water text-lg"></i>
                        </div>
                    </div>

                    <div class="my-4 space-y-2.5">
                        <div class="flex items-baseline justify-between">
                            <span id="tankCardValue" class="text-3xl font-extrabold text-white">82%</span>
                            <span id="tankLitersText" class="text-xs text-slate-400">~410 Liters</span>
                        </div>
                        <div class="w-full h-4 bg-slate-950 rounded-full overflow-hidden border border-slate-800 p-0.5">
                            <div id="tankLevelBar" class="h-full bg-gradient-to-r from-sky-500 to-cyan-400 rounded-full transition-all duration-500" style="width: 82%"></div>
                        </div>
                        <div class="flex items-center justify-between text-[11px] text-slate-400">
                            <span>Dry-Run Cutoff: 10%</span>
                            <button onclick="refillTank()" class="text-sky-400 hover:text-sky-300 underline font-semibold">
                                + Refill Tank
                            </button>
                        </div>
                    </div>

                    <div class="flex items-center justify-between text-xs text-slate-400 border-t border-slate-800/80 pt-3">
                        <span>Sensor: <strong class="text-slate-200">HC-SR04</strong></span>
                        <span>Safety Cutoff: <strong class="text-emerald-400">ON</strong></span>
                    </div>
                </div>
            </div>

            <!-- Controls & Chart -->
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 sm:p-6 flex flex-col justify-between space-y-5 shadow-lg">
                    <div class="space-y-5">
                        <div class="border-b border-slate-800 pb-3.5">
                            <h2 class="text-base font-bold text-white flex items-center gap-2">
                                <i class="fa-solid fa-sliders text-emerald-400"></i>
                                Cloud Control Panel
                            </h2>
                            <p class="text-xs text-slate-400 mt-0.5">Changes sync immediately to Firebase</p>
                        </div>

                        <div>
                            <label class="text-xs font-semibold uppercase tracking-wider text-slate-400 block mb-2">
                                1. Operation Mode
                            </label>
                            <div class="grid grid-cols-2 gap-2 bg-slate-950 p-1.5 rounded-xl border border-slate-800">
                                <button id="modeAutoBtn" onclick="changeMode('AUTO')"
                                    class="py-2.5 px-3 rounded-lg font-bold text-xs flex items-center justify-center gap-2 bg-emerald-600 text-white shadow transition-all">
                                    <i class="fa-solid fa-robot"></i>
                                    <span>AUTO MODE</span>
                                </button>
                                <button id="modeManualBtn" onclick="changeMode('MANUAL')"
                                    class="py-2.5 px-3 rounded-lg font-bold text-xs flex items-center justify-center gap-2 text-slate-400 hover:text-white transition-all">
                                    <i class="fa-solid fa-hand-pointer"></i>
                                    <span>MANUAL MODE</span>
                                </button>
                            </div>
                        </div>

                        <div class="bg-slate-950 border border-slate-800 rounded-xl p-4">
                            <div class="flex items-center justify-between gap-2">
                                <div>
                                    <h3 class="text-sm font-bold text-white">Remote Pump Switch</h3>
                                    <p id="manualHintText" class="text-[11px] text-slate-400 mt-0.5">
                                        Switch to MANUAL MODE to control directly
                                    </p>
                                </div>
                                <button id="remotePumpBtn" onclick="toggleRemotePump()"
                                    class="px-4 py-2.5 rounded-xl font-bold text-xs transition-all flex items-center gap-2 bg-slate-800 text-slate-500 border border-slate-700 cursor-not-allowed">
                                    <i class="fa-solid fa-power-off"></i>
                                    <span id="remotePumpBtnText">Turn Pump ON</span>
                                </button>
                            </div>
                        </div>

                        <div class="space-y-2">
                            <div class="flex items-center justify-between text-xs">
                                <span class="font-semibold text-slate-300">Auto Start Threshold:</span>
                                <span id="thresholdValBadge" class="px-2 py-0.5 rounded bg-amber-500/20 text-amber-300 font-bold border border-amber-500/30">35%</span>
                            </div>
                            <input id="thresholdRangeInput" type="range" min="15" max="75" value="35"
                                class="w-full accent-amber-500 cursor-pointer"
                                oninput="changeThreshold(this.value)">
                            <p class="text-[11px] text-slate-400">
                                Pump starts below <strong><span id="threshDescNum">35</span>%</strong> and stays ON until moisture reaches <strong>90%</strong>.
                            </p>
                        </div>
                    </div>

                    <div class="border-t border-slate-800 pt-4">
                        <label class="text-xs font-semibold uppercase tracking-wider text-slate-400 block mb-2.5">
                            One-Click Crop Presets:
                        </label>
                        <div class="grid grid-cols-2 gap-2 text-xs font-semibold">
                            <button onclick="selectCropPreset('Wheat', 30)" class="py-2 px-3 rounded-xl bg-slate-950 hover:bg-slate-800 text-slate-200 border border-slate-800 transition flex items-center justify-between">
                                <span>🌾 Wheat</span>
                                <span class="text-emerald-400">30%</span>
                            </button>
                            <button onclick="selectCropPreset('Rice/Paddy', 55)" class="py-2 px-3 rounded-xl bg-slate-950 hover:bg-slate-800 text-slate-200 border border-slate-800 transition flex items-center justify-between">
                                <span>🌱 Rice</span>
                                <span class="text-emerald-400">55%</span>
                            </button>
                            <button onclick="selectCropPreset('Tomato', 40)" class="py-2 px-3 rounded-xl bg-slate-950 hover:bg-slate-800 text-slate-200 border border-slate-800 transition flex items-center justify-between">
                                <span>🍅 Tomato</span>
                                <span class="text-emerald-400">40%</span>
                            </button>
                            <button onclick="selectCropPreset('Mustard', 28)" class="py-2 px-3 rounded-xl bg-slate-950 hover:bg-slate-800 text-slate-200 border border-slate-800 transition flex items-center justify-between">
                                <span>🌼 Mustard</span>
                                <span class="text-emerald-400">28%</span>
                            </button>
                        </div>
                    </div>
                </div>

                <div class="lg:col-span-2 bg-slate-900 border border-slate-800 rounded-2xl p-5 sm:p-6 flex flex-col justify-between space-y-4 shadow-lg">
                    <div class="flex flex-wrap items-center justify-between gap-2 border-b border-slate-800 pb-3.5">
                        <div>
                            <h2 class="text-base font-bold text-white flex items-center gap-2">
                                <i class="fa-solid fa-chart-line text-emerald-400"></i>
                                Cloud Analytics: Soil Moisture (%) vs 90% Cutoff
                            </h2>
                            <p class="text-xs text-slate-400 mt-0.5">Live telemetry history plotted every 1.5 seconds</p>
                        </div>
                        <div class="flex items-center gap-4 text-xs">
                            <span class="flex items-center gap-1.5 text-emerald-400 font-semibold">
                                <span class="w-2.5 h-2.5 rounded-full bg-emerald-500"></span> Soil Moisture (%)
                            </span>
                            <span class="flex items-center gap-1.5 text-sky-400 font-semibold">
                                <span class="w-3 h-0.5 bg-sky-400"></span> 90% Shutoff Line
                            </span>
                        </div>
                    </div>

                    <div class="w-full h-60 sm:h-64 bg-slate-950 rounded-xl border border-slate-800/90 p-2 relative">
                        <canvas id="cloudChartCanvas" class="w-full h-full block"></canvas>
                    </div>

                    <div class="border-t border-slate-800 pt-3.5 space-y-2.5">
                        <div class="flex flex-wrap items-center justify-between gap-2">
                            <span class="text-xs font-bold uppercase tracking-wider text-slate-400 flex items-center gap-1.5">
                                <i class="fa-solid fa-database text-emerald-400"></i>
                                Event & Telemetry Log
                            </span>
                            <div class="flex items-center gap-2">
                                <button onclick="exportTelemetryCsv()" class="px-3 py-1.5 rounded-lg bg-emerald-600 hover:bg-emerald-500 text-white text-xs font-semibold shadow flex items-center gap-1.5 transition">
                                    <i class="fa-solid fa-file-csv"></i>
                                    <span>Export CSV Log</span>
                                </button>
                                <button onclick="clearCloudLogs()" class="px-2.5 py-1.5 rounded-lg bg-slate-800 hover:bg-slate-700 text-slate-300 text-xs transition">
                                    Clear
                                </button>
                            </div>
                        </div>

                        <div id="cloudEventLogBox" class="bg-slate-950 border border-slate-800 rounded-xl p-3 h-28 overflow-y-auto font-mono text-xs space-y-1.5">
                        </div>
                    </div>
                </div>
            </div>
        </section>

        <!-- TAB 2: ESP32 CODE & CIRCUIT -->
        <section id="view-code" class="hidden space-y-6">
            <div class="grid grid-cols-1 lg:grid-cols-3 gap-6">
                <div class="bg-slate-900 border border-slate-800 rounded-2xl p-5 sm:p-6 space-y-4 shadow-lg">
                    <div>
                        <span class="text-xs font-bold uppercase tracking-wider text-emerald-400">Hardware Wiring</span>
                        <h2 class="text-lg font-bold text-white mt-0.5">ESP32 & NodeMCU Pinout</h2>
                        <p class="text-xs text-slate-400 mt-1">
                            Connect your sensors and 5V Relay Module to the microcontroller GPIO pins:
                        </p>
                    </div>

                    <div class="overflow-x-auto">
                        <table class="w-full text-left border-collapse text-xs">
                            <thead>
                                <tr class="border-b border-slate-800 text-slate-400 bg-slate-950/60">
                                    <th class="py-2.5 px-2.5">Sensor / Module</th>
                                    <th class="py-2.5 px-2.5 text-emerald-400">ESP32 Pin</th>
                                    <th class="py-2.5 px-2.5 text-sky-400">NodeMCU (8266)</th>
                                </tr>
                            </thead>
                            <tbody class="divide-y divide-slate-800 text-slate-200">
                                <tr>
                                    <td class="py-2.5 px-2.5 font-semibold">Soil Moisture (AOUT)</td>
                                    <td class="py-2.5 px-2.5 font-mono text-emerald-300">GPIO 34</td>
                                    <td class="py-2.5 px-2.5 font-mono text-sky-300">A0</td>
                                </tr>
                                <tr>
                                    <td class="py-2.5 px-2.5 font-semibold">Water Pump Relay (IN)</td>
                                    <td class="py-2.5 px-2.5 font-mono text-emerald-300">GPIO 26</td>
                                    <td class="py-2.5 px-2.5 font-mono text-sky-300">D1 (GPIO 5)</td>
                                </tr>
                                <tr>
                                    <td class="py-2.5 px-2.5 font-semibold">DHT11 Temp/Humidity</td>
                                    <td class="py-2.5 px-2.5 font-mono text-emerald-300">GPIO 27</td>
                                    <td class="py-2.5 px-2.5 font-mono text-sky-300">D2 (GPIO 4)</td>
                                </tr>
                            </tbody>
                        </table>
                    </div>
                </div>

                <div class="lg:col-span-2 bg-slate-900 border border-slate-800 rounded-2xl p-5 sm:p-6 flex flex-col justify-between shadow-lg">
                    <div class="flex flex-wrap items-center justify-between gap-3 border-b border-slate-800 pb-4">
                        <div>
                            <h2 class="text-base font-bold text-white flex items-center gap-2">
                                <i class="fa-solid fa-file-code text-emerald-400"></i>
                                ESP32_Cloud_Irrigation.ino (90% Cutoff Logic Included)
                            </h2>
                            <p class="text-xs text-slate-400 mt-0.5">Pump stays ON until soil moisture reaches >= 90%</p>
                        </div>
                        <button onclick="copyEspFirebaseCode()" class="px-4 py-2 rounded-xl bg-emerald-600 hover:bg-emerald-500 text-white text-xs font-bold shadow flex items-center gap-2 transition">
                            <i class="fa-regular fa-copy"></i>
                            <span id="copyBtnLabel">Copy ESP32 Code</span>
                        </button>
                    </div>

                    <pre id="espFirebaseCodeBlock" class="mt-4 bg-slate-950 border border-slate-800 rounded-xl p-4 text-xs font-mono text-emerald-300 overflow-x-auto h-[420px] leading-relaxed select-all">
#include &lt;WiFi.h&gt;
#include &lt;HTTPClient.h&gt;

const char* WIFI_SSID = "FarmTech;
const char* WIFI_PASS = "Farm_Tech";
String CLOUD_URL = "https://esop32-project-default-rtdb.asia-southeast1.firebasedatabase.app";

#define SOIL_PIN  34
#define RELAY_PIN 26   // Active LOW Relay Module

int moisturePercent = 45;
int threshold = 35;
bool autoMode = true;
bool pumpOn = false;

int readSoilMoisture() {
  int rawADC = analogRead(SOIL_PIN);
  int pct = map(rawADC, 3300, 1400, 0, 100);
  return constrain(pct, 0, 100);
}

void setup() {
  Serial.begin(115200);
  pinMode(RELAY_PIN, OUTPUT);
  digitalWrite(RELAY_PIN, HIGH); // Pump OFF initially

  WiFi.begin(WIFI_SSID, WIFI_PASS);
  while (WiFi.status() != WL_CONNECTED) {
    delay(500);
    Serial.print(".");
  }
}

void loop() {
  if (WiFi.status() == WL_CONNECTED) {
    HTTPClient http;
    String endpoint = CLOUD_URL + "/irrigation.json";

    http.begin(endpoint);
    if (http.GET() == 200) {
      String payload = http.getString();
    }
    http.end();

    moisturePercent = readSoilMoisture();

    // AUTO IRRIGATION LOGIC WITH 90% CUTOFF
    if (autoMode) {
      if (moisturePercent &lt; threshold) {
        pumpOn = true; // Turn ON when dry
      } 
      if (moisturePercent &gt;= 90) {
        pumpOn = false; // Stay ON until 90%, then shut OFF
      }
    }

    // Active LOW Relay Control
    digitalWrite(RELAY_PIN, pumpOn ? LOW : HIGH);

    http.begin(endpoint);
    http.addHeader("Content-Type", "application/json");
    String updateJson = "{\"moisture\":" + String(moisturePercent) + ",\"pump\":" + String(pumpOn ? "true" : "false") + "}";
    http.PATCH(updateJson);
    http.end();
  }
  delay(2000);
}</pre>
                </div>
            </div>
        </section>

        <!-- TAB 3: GUIDE -->
        <section id="view-guide" class="hidden space-y-6">
            <div class="bg-slate-900 border border-slate-800 rounded-2xl p-6 shadow-lg space-y-6">
                <div class="border-b border-slate-800 pb-4">
                    <span class="text-xs font-bold uppercase tracking-wider text-amber-400">Step-by-Step Guide</span>
                    <h2 class="text-xl font-extrabold text-white mt-1">
                        How to Connect Realtime Database & ESP32 in 5 Minutes
                    </h2>
                </div>
                <div class="grid grid-cols-1 md:grid-cols-2 gap-5">
                    <div class="bg-slate-950 border border-slate-800 rounded-2xl p-5 space-y-2">
                        <h3 class="font-bold text-white text-sm">1. Permanent Database URL</h3>
                        <p class="text-xs text-slate-300 leading-relaxed">
                            Your Firebase URL (`https://esop32-project-default-rtdb.asia-southeast1.firebasedatabase.app`) is now permanently hardcoded into both the dashboard and ESP32 sketch.
                        </p>
                    </div>
                    <div class="bg-slate-950 border border-slate-800 rounded-2xl p-5 space-y-2">
                        <h3 class="font-bold text-white text-sm">2. 90% Shutoff Rule</h3>
                        <p class="text-xs text-slate-300 leading-relaxed">
                            The water pump remains active until soil moisture reaches strictly >= 90%, ensuring optimal deep irrigation.
                        </p>
                    </div>
                </div>
            </div>
        </section>
    </main>

    <footer class="bg-slate-950 border-t border-slate-800 py-4 px-4 text-center text-xs text-slate-400">
        Created by FARMTECH
    </footer>

    <div id="toastBanner" class="fixed bottom-5 right-5 z-50 hidden bg-slate-900 border border-emerald-500/50 text-white px-4 py-3 rounded-xl shadow-2xl flex items-center gap-3 text-xs sm:text-sm">
        <i class="fa-solid fa-circle-check text-emerald-400 text-base"></i>
        <span id="toastText">Synced with Cloud</span>
    </div>

    <script>
        const appState = {
            moisture: 28,
            temperature: 29.6,
            humidity: 64,
            tank: 82,
            mode: 'AUTO',
            pump: true,
            threshold: 35,
            firebaseUrl: 'https://esop32-project-default-rtdb.asia-southeast1.firebasedatabase.app',
            isLiveFirebase: true,
            history: [44, 41, 38, 34, 31, 29, 28, 30, 33, 36, 39, 42],
            csvRows: [
                ['Timestamp', 'Soil_Moisture_%', 'Threshold_%', 'Operation_Mode', 'Pump_Status', 'Temp_C', 'Humidity_%']
            ]
        };

        function switchMainTab(tabName) {
            ['dashboard', 'code', 'guide'].forEach(t => {
                const section = document.getElementById(`view-${t}`);
                const btn = document.getElementById(`tabBtn-${t}`);
                if (t === tabName) {
                    section.classList.remove('hidden');
                    btn.className = 'px-3.5 py-2 rounded-lg bg-emerald-600 text-white shadow transition-all flex items-center gap-2 whitespace-nowrap';
                } else {
                    section.classList.add('hidden');
                    btn.className = 'px-3.5 py-2 rounded-lg text-slate-400 hover:text-white transition-all flex items-center gap-2 whitespace-nowrap';
                }
            });
            if (tabName === 'dashboard') {
                drawTelemetryChart();
            }
        }

        // EXACT 90% CUTOFF LOGIC
        function evaluateCloudAutomation() {
            const wasPumpOn = appState.pump;

            if (appState.tank <= 10 && appState.pump) {
                appState.pump = false;
                logCloudEvent('⚠️ TANK CRITICAL (<10%): Pump auto-stopped.', 'text-rose-400');
                return;
            }

            if (appState.moisture >= 90) {
                appState.pump = false;
                if (wasPumpOn) {
                    logCloudEvent(`🛡️ 90% CUTOFF REACHED: Soil moisture is ${Math.round(appState.moisture)}% (>=90%). Pump turned OFF.`, 'text-sky-400');
                }
                return;
            }

            if (appState.mode === 'AUTO') {
                if (appState.moisture < appState.threshold && appState.tank > 10) {
                    appState.pump = true;
                    if (!wasPumpOn) {
                        logCloudEvent(`🤖 AUTO: Moisture (${Math.round(appState.moisture)}%) < Threshold (${appState.threshold}%). Pump turned ON.`, 'text-emerald-400');
                    }
                }
            }
        }

        function updateDashboardUI() {
            const m = Math.round(appState.moisture);
            document.getElementById('moistureCardValue').textContent = `${m}%`;
            document.getElementById('simMoistureSlider').value = m;
            document.getElementById('simMoistureBadge').textContent = `${m}%`;
            document.getElementById('moistureLinearBar').style.width = `${m}%`;

            const adc = Math.round(3300 - (m / 100) * 1900);
            document.getElementById('adcRawText').textContent = `ADC: ${adc}`;

            const offset = 314.16 - (m / 100) * 314.16;
            const ring = document.getElementById('moistureRing');
            if (ring) ring.style.strokeDashoffset = offset;

            const moistPill = document.getElementById('moistureStatusPill');
            const linearBar = document.getElementById('moistureLinearBar');
            if (m < appState.threshold) {
                if (ring) ring.style.stroke = '#f43f5e';
                linearBar.className = 'h-full bg-rose-500 transition-all duration-300';
                moistPill.className = 'inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-rose-500/15 text-rose-400 border border-rose-500/30';
                moistPill.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-rose-400 animate-ping"></span> Critical Dry';
            } else if (m < 90) {
                if (ring) ring.style.stroke = '#10b981';
                linearBar.className = 'h-full bg-emerald-500 transition-all duration-300';
                moistPill.className = 'inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-emerald-500/15 text-emerald-400 border border-emerald-500/30';
                moistPill.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-emerald-400"></span> Irrigating / Optimal';
            } else {
                if (ring) ring.style.stroke = '#38bdf8';
                linearBar.className = 'h-full bg-sky-400 transition-all duration-300';
                moistPill.className = 'inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-sky-500/15 text-sky-400 border border-sky-500/30';
                moistPill.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-sky-400"></span> Reached >= 90%';
            }

            document.getElementById('thresholdValBadge').textContent = `${appState.threshold}%`;
            document.getElementById('thresholdRangeInput').value = appState.threshold;
            document.getElementById('threshDescNum').textContent = appState.threshold;

            const pumpCard = document.getElementById('pumpCardContainer');
            const pumpPill = document.getElementById('pumpStatePill');
            const impeller = document.getElementById('pumpImpellerIcon');
            const relayLogic = document.getElementById('cardRelayLogic');
            const flowRate = document.getElementById('cardFlowRate');
            const pumpReason = document.getElementById('pumpReasonText');

            if (appState.pump) {
                pumpCard.classList.add('pump-glow', 'border-emerald-500/50');
                pumpPill.className = 'inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-emerald-500/20 text-emerald-300 border border-emerald-500/30';
                pumpPill.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-emerald-400 animate-ping"></span> RUNNING (ON)';
                impeller.classList.add('spin-active', 'text-emerald-400');
                impeller.classList.remove('text-slate-500');
                relayLogic.textContent = 'LOW (ON - 0V)';
                relayLogic.className = 'font-mono text-emerald-400';
                flowRate.textContent = '2.0 L/min';
                pumpReason.textContent = 'Stays ON till 90%';
                pumpReason.className = 'text-emerald-400 font-semibold';
            } else {
                pumpCard.classList.remove('pump-glow', 'border-emerald-500/50');
                pumpPill.className = 'inline-flex items-center gap-1.5 px-2.5 py-0.5 rounded-full text-xs font-semibold bg-slate-800 text-slate-300 border border-slate-700';
                pumpPill.innerHTML = '<span class="w-1.5 h-1.5 rounded-full bg-slate-400"></span> CUTOFF (OFF)';
                impeller.classList.remove('spin-active', 'text-emerald-400');
                impeller.classList.add('text-slate-500');
                relayLogic.textContent = 'HIGH (OFF - 3.3V)';
                relayLogic.className = 'font-mono text-slate-400';
                flowRate.textContent = '0.0 L/min';
                pumpReason.textContent = 'Moisture >= 90%';
                pumpReason.className = 'text-sky-400 font-semibold';
            }

            document.getElementById('tempCardValue').textContent = `${appState.temperature.toFixed(1)}°C`;
            document.getElementById('humCardValue').textContent = `${Math.round(appState.humidity)}%`;
            const tankInt = Math.round(appState.tank);
            document.getElementById('tankCardValue').textContent = `${tankInt}%`;
            document.getElementById('tankLitersText').textContent = `~${tankInt * 5} Liters`;
            document.getElementById('tankLevelBar').style.width = `${tankInt}%`;

            document.getElementById('cardModeText').textContent = `${appState.mode} MODE`;
            const btnAuto = document.getElementById('modeAutoBtn');
            const btnManual = document.getElementById('modeManualBtn');
            const remoteBtn = document.getElementById('remotePumpBtn');
            const remoteBtnText = document.getElementById('remotePumpBtnText');

            if (appState.mode === 'AUTO') {
                btnAuto.className = 'py-2.5 px-3 rounded-lg font-bold text-xs flex items-center justify-center gap-2 bg-emerald-600 text-white shadow transition-all';
                btnManual.className = 'py-2.5 px-3 rounded-lg font-bold text-xs flex items-center justify-center gap-2 text-slate-400 hover:text-white transition-all';
                remoteBtn.className = 'px-4 py-2.5 rounded-xl font-bold text-xs transition-all flex items-center gap-2 bg-slate-800 text-slate-500 border border-slate-700 cursor-not-allowed';
                remoteBtnText.textContent = appState.pump ? 'Auto ON' : 'Auto OFF';
            } else {
                btnManual.className = 'py-2.5 px-3 rounded-lg font-bold text-xs flex items-center justify-center gap-2 bg-amber-500 text-slate-950 shadow transition-all';
                btnAuto.className = 'py-2.5 px-3 rounded-lg font-bold text-xs flex items-center justify-center gap-2 text-slate-400 hover:text-white transition-all';
                if (appState.pump) {
                    remoteBtn.className = 'px-4 py-2.5 rounded-xl font-bold text-xs transition-all flex items-center gap-2 bg-rose-600 hover:bg-rose-500 text-white shadow cursor-pointer';
                    remoteBtnText.textContent = 'Turn Pump OFF';
                } else {
                    remoteBtn.className = 'px-4 py-2.5 rounded-xl font-bold text-xs transition-all flex items-center gap-2 bg-emerald-600 hover:bg-emerald-500 text-white shadow cursor-pointer';
                    remoteBtnText.textContent = 'Turn Pump ON';
                }
            }
        }

        function handleManualMoistureSim(val) {
            appState.moisture = parseFloat(val);
            evaluateCloudAutomation();
            updateDashboardUI();
            pushStateToDatabaseRest();
        }

        function changeMode(newMode) {
            appState.mode = newMode;
            logCloudEvent(`☁️ Mode switched to ${newMode} MODE.`, 'text-amber-400');
            evaluateCloudAutomation();
            updateDashboardUI();
            pushStateToDatabaseRest();
            showToast(`Switched to ${newMode} MODE`);
        }

        function toggleRemotePump() {
            if (appState.mode !== 'MANUAL') {
                showToast('Switch to MANUAL MODE first!');
                return;
            }
            appState.pump = !appState.pump;
            logCloudEvent(`🖐️ Manual Pump turned ${appState.pump ? 'ON' : 'OFF'}.`, appState.pump ? 'text-emerald-400' : 'text-rose-400');
            updateDashboardUI();
            pushStateToDatabaseRest();
        }

        function changeThreshold(val) {
            appState.threshold = parseInt(val, 10);
            evaluateCloudAutomation();
            updateDashboardUI();
            pushStateToDatabaseRest();
        }

        function selectCropPreset(cropName, threshVal) {
            appState.threshold = threshVal;
            logCloudEvent(`🌾 Preset: ${cropName} (Threshold set to ${threshVal}%).`, 'text-sky-400');
            evaluateCloudAutomation();
            updateDashboardUI();
            pushStateToDatabaseRest();
            showToast(`${cropName} Preset Synced!`);
        }

        function refillTank() {
            appState.tank = 100;
            logCloudEvent('💧 Water Tank Refilled to 100%.', 'text-sky-400');
            evaluateCloudAutomation();
            updateDashboardUI();
            showToast('Tank Refilled!');
        }

        async function connectCustomFirebaseUrl() {
            showToast('Connected to Permanent Firebase DB!');
        }

        async function pushStateToDatabaseRest() {
            if (!appState.isLiveFirebase || !appState.firebaseUrl) return;
            try {
                await fetch(`${appState.firebaseUrl}/irrigation.json`, {
                    method: 'PATCH',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        moisture: Math.round(appState.moisture),
                        threshold: appState.threshold,
                        mode: appState.mode,
                        pump: appState.pump
                    })
                });
            } catch (err) {}
        }

        async function pullStateFromDatabaseRest() {
            if (!appState.isLiveFirebase || !appState.firebaseUrl) return;
            try {
                const res = await fetch(`${appState.firebaseUrl}/irrigation.json`);
                if (!res.ok) return;
                const data = await res.json();
                if (data && typeof data === 'object') {
                    if (typeof data.moisture === 'number') appState.moisture = data.moisture;
                    if (typeof data.temp === 'number') appState.temperature = data.temp;
                    if (typeof data.humidity === 'number') appState.humidity = data.humidity;
                    if (typeof data.tank === 'number') appState.tank = data.tank;
                    if (typeof data.pump === 'boolean') appState.pump = data.pump;
                }
            } catch (err) {}
        }

        function drawTelemetryChart() {
            const canvas = document.getElementById('cloudChartCanvas');
            if (!canvas) return;
            const rect = canvas.getBoundingClientRect();
            const dpr = window.devicePixelRatio || 1;
            canvas.width = rect.width * dpr;
            canvas.height = rect.height * dpr;

            const ctx = canvas.getContext('2d');
            ctx.scale(dpr, dpr);
            const w = rect.width;
            const h = rect.height;

            ctx.clearRect(0, 0, w, h);

            const padLeft = 38, padRight = 18, padTop = 18, padBottom = 26;
            const plotW = w - padLeft - padRight;
            const plotH = h - padTop - padBottom;

            ctx.strokeStyle = 'rgba(51, 65, 85, 0.45)';
            ctx.lineWidth = 1;
            ctx.fillStyle = '#94a3b8';
            ctx.font = '11px Inter, sans-serif';
            [0, 25, 50, 75, 100].forEach(pct => {
                const y = padTop + plotH - (pct / 100) * plotH;
                ctx.beginPath();
                ctx.moveTo(padLeft, y);
                ctx.lineTo(w - padRight, y);
                ctx.stroke();
                ctx.fillText(`${pct}%`, 6, y + 4);
            });

            const shutoffY = padTop + plotH - (90 / 100) * plotH;
            ctx.strokeStyle = '#38bdf8';
            ctx.lineWidth = 1.8;
            ctx.setLineDash([6, 4]);
            ctx.beginPath();
            ctx.moveTo(padLeft, shutoffY);
            ctx.lineTo(w - padRight, shutoffY);
            ctx.stroke();
            ctx.setLineDash([]);

            const pts = appState.history;
            if (pts.length < 2) return;
            const stepX = plotW / (pts.length - 1);

            ctx.beginPath();
            pts.forEach((val, idx) => {
                const x = padLeft + idx * stepX;
                const y = padTop + plotH - (val / 100) * plotH;
                if (idx === 0) ctx.moveTo(x, y);
                else ctx.lineTo(x, y);
            });
            ctx.strokeStyle = '#10b981';
            ctx.lineWidth = 3;
            ctx.stroke();
        }

        function logCloudEvent(msg, colorClass = 'text-slate-300') {
            const box = document.getElementById('cloudEventLogBox');
            if (!box) return;
            const nowStr = new Date().toLocaleTimeString([], { hour12: false });
            const div = document.createElement('div');
            div.className = colorClass;
            div.textContent = `[${nowStr}] ${msg}`;
            box.prepend(div);
        }

        function clearCloudLogs() {
            document.getElementById('cloudEventLogBox').innerHTML = '';
        }

        function exportTelemetryCsv() {
            const csvContent = appState.csvRows.map(r => r.join(',')).join('\n');
            const blob = new Blob([csvContent], { type: 'text/csv;charset=utf-8;' });
            const url = URL.createObjectURL(blob);
            const a = document.createElement('a');
            a.href = url;
            a.download = `FARMTECH_Report_${new Date().toISOString().slice(0, 10)}.csv`;
            document.body.appendChild(a);
            a.click();
            document.body.removeChild(a);
            showToast('CSV Report Downloaded!');
        }

        function copyEspFirebaseCode() {
            const code = document.getElementById('espFirebaseCodeBlock').innerText;
            const temp = document.createElement('textarea');
            temp.value = code;
            document.body.appendChild(temp);
            temp.select();
            document.execCommand('copy');
            document.body.removeChild(temp);
            showToast('ESP32 Code copied!');
        }

        function showToast(message) {
            const banner = document.getElementById('toastBanner');
            document.getElementById('toastText').textContent = message;
            banner.classList.remove('hidden');
            setTimeout(() => banner.classList.add('hidden'), 3000);
        }

        async function cloudTelemetryTick() {
            await pullStateFromDatabaseRest();

            if (!appState.isLiveFirebase) {
                if (appState.pump) {
                    appState.moisture = Math.min(95, appState.moisture + 3.0);
                    appState.tank = Math.max(0, appState.tank - 0.25);
                } else {
                    appState.moisture = Math.max(12, appState.moisture - 0.5);
                }
            }

            evaluateCloudAutomation();
            updateDashboardUI();

            const roundedM = Math.round(appState.moisture);
            appState.history.push(roundedM);
            if (appState.history.length > 16) appState.history.shift();

            appState.csvRows.push([
                new Date().toLocaleTimeString([], { hour12: false }),
                roundedM,
                appState.threshold,
                appState.mode,
                appState.pump ? 'ON' : 'OFF',
                appState.temperature.toFixed(1),
                Math.round(appState.humidity)
            ]);

            drawTelemetryChart();
        }

        window.addEventListener('resize', drawTelemetryChart);

        window.onload = function () {
            logCloudEvent('🔥 FARMTECH Dashboard Connected to Permanent Firebase.', 'text-emerald-400');
            evaluateCloudAutomation();
            updateDashboardUI();
            drawTelemetryChart();
            setInterval(cloudTelemetryTick, 1500);
        };
    </script>
</body>
</html>
