<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Colorful Alarm Clock</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/tone/14.8.49/Tone.js"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            overscroll-behavior-y: contain;
        }
        ::-webkit-scrollbar { width: 8px; }
        ::-webkit-scrollbar-track { background: #99f6e4; border-radius: 10px; } /* Teal-100 */
        ::-webkit-scrollbar-thumb { background: #14b8a6; border-radius: 10px; } /* Teal-500 */
        ::-webkit-scrollbar-thumb:hover { background: #0d9488; } /* Teal-600 */

        .alarm-ringing-overlay {
            position: fixed; top: 0; left: 0; right: 0; bottom: 0;
            background-color: rgba(0, 0, 0, 0.7);
            display: flex; align-items: center; justify-content: center;
            z-index: 1000; opacity: 0; visibility: hidden;
            transition: opacity 0.3s ease, visibility 0.3s ease;
        }
        .alarm-ringing-overlay.show { opacity: 1; visibility: visible; }
        .alarm-ringing-content {
            background-color: white; color: #1f2937; /* Gray-800 */
            padding: 30px 40px; border-radius: 16px; text-align: center;
            box-shadow: 0 10px 30px rgba(0,0,0,0.25);
            animation: pulseRinging 1.2s infinite ease-in-out;
        }
        @keyframes pulseRinging {
            0%, 100% { transform: scale(1); box-shadow: 0 10px 30px rgba(0,0,0,0.25); }
            50% { transform: scale(1.03); box-shadow: 0 15px 35px rgba(20, 184, 166, 0.4); } /* Teal-500 shadow */
        }
        .toast {
            position: fixed; bottom: 20px; left: 50%; transform: translateX(-50%);
            padding: 12px 20px; border-radius: 8px; font-size: 0.9rem;
            z-index: 1100; opacity: 0;
            transition: opacity 0.3s ease-in-out, transform 0.3s ease-in-out;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.15);
        }
        .toast.show { opacity: 1; transform: translateX(-50%) translateY(0); }
        .toast.success { background-color: #10b981; color: white; } /* Emerald-500 */
        .toast.error { background-color: #ef4444; color: white; } /* Red-500 */
        .toast.info { background-color: #3b82f6; color: white; } /* Blue-500 */
        
        .spinner {
            display: inline-block; width: 1em; height: 1em;
            border: 2px solid currentColor; border-right-color: transparent;
            border-radius: 50%; animation: spinner-border .75s linear infinite;
            margin-left: 0.5em;
        }
        @keyframes spinner-border { to { transform: rotate(360deg); } }
        button:disabled .spinner-text { opacity: 0; }
        button:disabled .spinner { display: inline-block !important; }
    </style>
</head>
<body class="bg-gradient-to-br from-cyan-600 to-teal-700 text-white min-h-screen flex flex-col items-center justify-center p-4 selection:bg-pink-500 selection:text-white">

    <div class="bg-white/10 backdrop-blur-md p-6 sm:p-8 rounded-2xl shadow-2xl w-full max-w-lg">
        <div class="text-center mb-6">
            <h1 class="text-5xl sm:text-6xl font-bold text-cyan-300" id="currentTime">00:00:00 AM</h1>
            <p class="text-teal-100 text-lg" id="currentDate">January 1, 2024</p>
        </div>

        <form id="alarmForm" class="space-y-4 mb-8">
            <div>
                <label for="alarmLabel" class="block text-sm font-medium text-teal-200 mb-1">Alarm Label (Optional)</label>
                <input type="text" id="alarmLabel" placeholder="e.g., Morning Meeting" class="mt-1 block w-full py-2.5 px-3 border border-teal-500 bg-white/20 text-white placeholder-teal-300 rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-pink-400 focus:border-pink-400 sm:text-sm">
            </div>
            <div class="grid grid-cols-3 gap-3">
                <div>
                    <label for="alarmHour" class="block text-sm font-medium text-teal-200 mb-1">Hour</label>
                    <select id="alarmHour" required class="mt-1 block w-full py-2.5 px-3 border border-teal-500 bg-white/20 text-white rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-pink-400 focus:border-pink-400 sm:text-sm appearance-none text-center">
                    </select>
                </div>
                <div>
                    <label for="alarmMinute" class="block text-sm font-medium text-teal-200 mb-1">Minute</label>
                    <select id="alarmMinute" required class="mt-1 block w-full py-2.5 px-3 border border-teal-500 bg-white/20 text-white rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-pink-400 focus:border-pink-400 sm:text-sm appearance-none text-center">
                    </select>
                </div>
                <div>
                    <label for="alarmAmPm" class="block text-sm font-medium text-teal-200 mb-1">AM/PM</label>
                    <select id="alarmAmPm" required class="mt-1 block w-full py-2.5 px-3 border border-teal-500 bg-white/20 text-white rounded-lg shadow-sm focus:outline-none focus:ring-2 focus:ring-pink-400 focus:border-pink-400 sm:text-sm appearance-none text-center">
                        <option value="AM">AM</option>
                        <option value="PM">PM</option>
                    </select>
                </div>
            </div>
            <button type="submit" class="w-full bg-pink-500 hover:bg-pink-600 text-white font-semibold py-2.5 px-4 rounded-lg shadow-md focus:outline-none focus:ring-2 focus:ring-pink-400 focus:ring-offset-2 focus:ring-offset-teal-700 transition duration-150 ease-in-out">
                Set Alarm
            </button>
        </form>

        <div>
            <h2 class="text-xl font-semibold text-teal-100 mb-3">Active Alarms</h2>
            <div id="alarmsList" class="space-y-3 max-h-60 overflow-y-auto p-2 rounded-lg bg-black/20">
                <p id="noAlarmsMessage" class="text-teal-200 text-center py-4">No alarms set yet. Sweet dreams!</p>
            </div>
        </div>
    </div>

    <div id="alarmRingingOverlay" class="alarm-ringing-overlay">
        <div class="alarm-ringing-content">
            <div class="text-5xl mb-3 animate-bounce">⏰</div>
            <h2 class="text-3xl font-bold text-pink-600 mb-1" id="ringingAlarmLabel">ALARM!</h2>
            <p class="text-xl text-gray-700 mb-6" id="ringingAlarmTime">It's time!</p>
            <div class="flex gap-4 justify-center">
                <button id="snoozeAlarmButton" class="bg-yellow-500 hover:bg-yellow-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg transition duration-150 ease-in-out">
                    Snooze (5m)
                </button>
                <button id="dismissAlarmButton" class="bg-red-500 hover:bg-red-600 text-white font-bold py-3 px-6 rounded-lg shadow-lg transition duration-150 ease-in-out flex items-center justify-center">
                    <span class="spinner-text">Dismiss</span>
                    <span class="spinner hidden"></span>
                </button>
            </div>
        </div>
    </div>

    <div id="toastNotification" class="toast"></div>

    <script>
        const currentTimeDisplay = document.getElementById('currentTime');
        const currentDateDisplay = document.getElementById('currentDate');
        const alarmForm = document.getElementById('alarmForm');
        const alarmLabelInput = document.getElementById('alarmLabel');
        const alarmHourSelect = document.getElementById('alarmHour');
        const alarmMinuteSelect = document.getElementById('alarmMinute');
        const alarmAmPmSelect = document.getElementById('alarmAmPm');
        const alarmsList = document.getElementById('alarmsList');
        const noAlarmsMessage = document.getElementById('noAlarmsMessage');
        const alarmRingingOverlay = document.getElementById('alarmRingingOverlay');
        const ringingAlarmLabelDisplay = document.getElementById('ringingAlarmLabel');
        const ringingAlarmTimeDisplay = document.getElementById('ringingAlarmTime');
        const snoozeAlarmButton = document.getElementById('snoozeAlarmButton');
        const dismissAlarmButton = document.getElementById('dismissAlarmButton');
        const dismissSpinner = dismissAlarmButton.querySelector('.spinner');
        const dismissSpinnerText = dismissAlarmButton.querySelector('.spinner-text');
        const toastNotification = document.getElementById('toastNotification');

        let alarms = [];
        let alarmInterval;
        let currentRingingAlarmId = null;
        let synth, alarmLoop;
        const SNOOZE_DURATION_MS = 5 * 60 * 1000; // 5 minutes

        function showToast(message, type = 'success') {
            toastNotification.textContent = message;
            toastNotification.className = `toast ${type} show`;
            setTimeout(() => { toastNotification.className = `toast ${type}`; }, type === 'info' ? 5000 : 3000);
        }

        async function initializeAudio() {
            if (Tone.context.state !== 'running') {
                await Tone.start();
                console.log("AudioContext started");
            }
            if (!synth) {
                synth = new Tone.MembraneSynth({
                    pitchDecay: 0.01,
                    octaves: 6,
                    oscillator: { type: "sine" },
                    envelope: { attack: 0.001, decay: 0.2, sustain: 0.01, release: 0.8, attackCurve: "exponential"}
                }).toDestination();
            }
            if(!alarmLoop){
                alarmLoop = new Tone.Loop(time => {
                    if (synth) { // Check if synth is initialized
                        synth.triggerAttackRelease("C3", "8n", time);
                        synth.triggerAttackRelease("G3", "8n", time + Tone.Time("8n").toSeconds());
                    }
                }, "0.7s"); // Loop every 0.7 seconds
            }
        }
        document.body.addEventListener('click', initializeAudio, { once: true });

        function populateSelectOptions() {
            for (let i = 1; i <= 12; i++) {
                const option = document.createElement('option');
                option.value = i.toString().padStart(2, '0');
                option.textContent = i.toString().padStart(2, '0');
                alarmHourSelect.appendChild(option);
            }
            for (let i = 0; i <= 59; i++) {
                const option = document.createElement('option');
                option.value = i.toString().padStart(2, '0');
                option.textContent = i.toString().padStart(2, '0');
                alarmMinuteSelect.appendChild(option);
            }
        }

        function updateTime() {
            const now = new Date();
            const hours = now.getHours();
            const minutes = now.getMinutes();
            const seconds = now.getSeconds();
            const amPm = hours >= 12 ? 'PM' : 'AM';
            const displayHours = (hours % 12) || 12;
            currentTimeDisplay.textContent = `${displayHours.toString().padStart(2, '0')}:${minutes.toString().padStart(2, '0')}:${seconds.toString().padStart(2, '0')} ${amPm}`;
            currentDateDisplay.textContent = now.toLocaleDateString(undefined, { weekday: 'long', year: 'numeric', month: 'long', day: 'numeric' });
        }

        function renderAlarms() {
            alarmsList.innerHTML = '';
            if (alarms.length === 0) {
                alarmsList.appendChild(noAlarmsMessage);
                noAlarmsMessage.style.display = 'block';
            } else {
                noAlarmsMessage.style.display = 'none';
                alarms.sort((a, b) => { /* More robust sorting might be needed */
                    const getSortableTime = (alarm) => {
                        let hour = parseInt(alarm.hour);
                        if (alarm.amPm === 'PM' && hour !== 12) hour += 12;
                        if (alarm.amPm === 'AM' && hour === 12) hour = 0; // Midnight case
                        return hour * 60 + parseInt(alarm.minute);
                    };
                    return getSortableTime(a) - getSortableTime(b);
                });

                alarms.forEach(alarm => {
                    const alarmElement = document.createElement('div');
                    alarmElement.className = 'flex justify-between items-center bg-white/10 p-3.5 rounded-lg shadow-md hover:bg-white/20 transition-colors';
                    alarmElement.innerHTML = `
                        <div>
                            <span class="text-lg font-semibold text-cyan-200">${alarm.hour}:${alarm.minute} ${alarm.amPm}</span>
                            ${alarm.label ? `<p class="text-sm text-teal-200">${alarm.label}</p>` : ''}
                        </div>
                        <button data-id="${alarm.id}" class="delete-alarm-btn text-pink-400 hover:text-pink-300 transition-colors p-1.5 rounded-md focus:outline-none focus:ring-2 focus:ring-pink-500 focus:ring-offset-2 focus:ring-offset-current">
                            <svg xmlns="http://www.w3.org/2000/svg" class="h-5 w-5 pointer-events-none" viewBox="0 0 20 20" fill="currentColor"><path fill-rule="evenodd" d="M9 2a1 1 0 00-.894.553L7.382 4H4a1 1 0 000 2v10a2 2 0 002 2h8a2 2 0 002-2V6a1 1 0 100-2h-3.382l-.724-1.447A1 1 0 0011 2H9zM7 8a1 1 0 012 0v6a1 1 0 11-2 0V8zm5-1a1 1 0 00-1 1v6a1 1 0 102 0V8a1 1 0 00-1-1z" clip-rule="evenodd" /></svg>
                        </button>
                    `;
                    alarmsList.appendChild(alarmElement);
                });
            }
        }

        function saveAlarms() {
            localStorage.setItem('colorfulAlarmClockAlarms', JSON.stringify(alarms));
        }

        function loadAlarms() {
            const storedAlarms = localStorage.getItem('colorfulAlarmClockAlarms');
            if (storedAlarms) alarms = JSON.parse(storedAlarms);
            renderAlarms();
        }

        alarmForm.addEventListener('submit', (e) => {
            e.preventDefault();
            const label = alarmLabelInput.value.trim();
            const hour = alarmHourSelect.value;
            const minute = alarmMinuteSelect.value;
            const amPm = alarmAmPmSelect.value;

            if (alarms.find(a => a.hour === hour && a.minute === minute && a.amPm === amPm && a.label === label)) {
                showToast("This exact alarm (time and label) is already set!", "error"); return;
            }

            alarms.push({ id: Date.now().toString(), label, hour, minute, amPm, active: true, snoozeUntil: null });
            saveAlarms(); renderAlarms(); alarmForm.reset();
            showToast("Alarm set: " + (label || `${hour}:${minute} ${amPm}`), "success");
        });

        alarmsList.addEventListener('click', (e) => {
            const deleteButton = e.target.closest('.delete-alarm-btn');
            if (deleteButton) {
                const alarmId = deleteButton.dataset.id;
                alarms = alarms.filter(alarm => alarm.id !== alarmId);
                saveAlarms(); renderAlarms();
                showToast("Alarm deleted.", "success");
            }
        });
        
        async function callGeminiAPI(prompt) {
            const apiKey = ""; // Provided by Canvas environment
            const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;
            const payload = { contents: [{ role: "user", parts: [{ text: prompt }] }] };
            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify(payload)
                });
                if (!response.ok) throw new Error(`API request failed: ${response.status}`);
                const result = await response.json();
                if (result.candidates?.[0]?.content?.parts?.[0]?.text) {
                    return result.candidates[0].content.parts[0].text;
                }
                throw new Error("Invalid API response structure.");
            } catch (error) {
                console.error("Error calling Gemini API:", error);
                return "Have a great day!"; // Fallback quote
            }
        }


        function checkAlarms() {
            if (currentRingingAlarmId && alarmRingingOverlay.classList.contains('show')) return;

            const now = new Date();
            const currentHour = now.getHours();
            const currentMinute = now.getMinutes().toString().padStart(2, '0');
            const currentAmPm = currentHour >= 12 ? 'PM' : 'AM';
            const displayHour = ((currentHour % 12) || 12).toString().padStart(2, '0');

            for (const alarm of alarms) {
                if (alarm.active && alarm.hour === displayHour && alarm.minute === currentMinute && alarm.amPm === currentAmPm) {
                    if (alarm.snoozeUntil && now.getTime() < alarm.snoozeUntil) {
                        console.log(`Alarm ${alarm.label || alarm.id} is snoozed. Skipping.`);
                        continue; // Snoozed, skip
                    }
                    // Clear snooze if it's time to ring again
                    alarm.snoozeUntil = null; 
                    saveAlarms(); // Save change to snoozeUntil
                    triggerAlarm(alarm);
                    break; // Only trigger one alarm at a time
                }
            }
        }

        function triggerAlarm(alarm) {
            if (currentRingingAlarmId) return; // Already an alarm trying to ring

            currentRingingAlarmId = alarm.id;
            console.log(`Alarm ringing: ${alarm.label || alarm.id}`);
            ringingAlarmLabelDisplay.textContent = alarm.label || "Time's Up!";
            ringingAlarmTimeDisplay.textContent = `It's ${alarm.hour}:${alarm.minute} ${alarm.amPm}!`;
            alarmRingingOverlay.classList.add('show');

            if (Tone.context.state === 'running' && alarmLoop && synth) {
                Tone.Transport.start();
                alarmLoop.start(0);
            } else {
                console.warn("Tone.js or synth/loop not ready. Visual alarm only.");
                initializeAudio(); // Try to init audio again
            }
        }
        
        function stopAlarmSound() {
            if (Tone.context.state === 'running' && alarmLoop) {
                alarmLoop.stop();
                Tone.Transport.stop(); // Also stop transport if it was started for the loop
                // synth.triggerRelease(); // For MembraneSynth, loop stop is enough.
            }
        }

        snoozeAlarmButton.addEventListener('click', () => {
            if (!currentRingingAlarmId) return;
            
            stopAlarmSound();
            alarmRingingOverlay.classList.remove('show');

            const alarm = alarms.find(a => a.id === currentRingingAlarmId);
            if (alarm) {
                alarm.snoozeUntil = Date.now() + SNOOZE_DURATION_MS;
                saveAlarms();
                showToast(`Snoozed: ${alarm.label || alarm.id} for 5 minutes.`, "info");
            }
            currentRingingAlarmId = null; // Allow checkAlarms to run again
        });

        dismissAlarmButton.addEventListener('click', async () => {
            if (!currentRingingAlarmId) return;

            dismissAlarmButton.disabled = true;
            dismissSpinnerText.classList.add('hidden');
            dismissSpinner.classList.remove('hidden');

            stopAlarmSound();
            alarmRingingOverlay.classList.remove('show');
            
            const alarm = alarms.find(a => a.id === currentRingingAlarmId);
            if (alarm) {
                alarm.snoozeUntil = null; // Clear any snooze state
                // Optional: Mark alarm as inactive for the day, or remove if one-time
                // For simplicity, we just let it be re-triggerable next day.
                saveAlarms();
            }
            
            const quotePrompt = "Give me a very short, positive, and uplifting motivational quote for starting the day (max 10-12 words).";
            const motivationalQuote = await callGeminiAPI(quotePrompt);
            showToast(motivationalQuote, "info");
            
            currentRingingAlarmId = null;
            dismissAlarmButton.disabled = false;
            dismissSpinnerText.classList.remove('hidden');
            dismissSpinner.classList.add('hidden');
        });

        document.addEventListener('DOMContentLoaded', () => {
            populateSelectOptions();
            updateTime();
            setInterval(updateTime, 1000);
            loadAlarms();
            alarmInterval = setInterval(checkAlarms, 1000); // Check alarms more frequently
        });
    </script>
</body>
</html>
