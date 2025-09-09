<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Habit Streak Tracker</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        body { font-family: 'Inter', sans-serif; }
        
        .streak-flame {
            animation: flicker 2s ease-in-out infinite alternate;
        }
        
        @keyframes flicker {
            0% { transform: scale(1) rotate(-1deg); }
            100% { transform: scale(1.05) rotate(1deg); }
        }
        
        .habit-card {
            transition: all 0.3s ease;
        }
        
        .habit-card:hover {
            transform: translateY(-2px);
            box-shadow: 0 10px 25px rgba(0,0,0,0.1);
        }
        
        .check-animation {
            animation: checkmark 0.6s ease-in-out;
        }
        
        @keyframes checkmark {
            0% { transform: scale(0); }
            50% { transform: scale(1.2); }
            100% { transform: scale(1); }
        }
    </style>
</head>
<body class="bg-gradient-to-br from-purple-50 to-blue-50 min-h-screen">
    <div class="container mx-auto px-4 py-8 max-w-4xl">
        <!-- Header -->
        <div class="text-center mb-8">
            <h1 class="text-4xl font-bold text-gray-800 mb-2">🔥 Habit Streak Tracker</h1>
            <p class="text-gray-600">Build lasting habits, one day at a time</p>
        </div>

        <!-- Add New Habit -->
        <div class="bg-white rounded-xl shadow-lg p-6 mb-8">
            <h2 class="text-xl font-semibold text-gray-800 mb-4">Add New Habit</h2>
            <div class="flex gap-3">
                <input 
                    type="text" 
                    id="habitInput" 
                    placeholder="Enter a new habit (e.g., Drink 8 glasses of water)"
                    class="flex-1 px-4 py-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-purple-500 focus:border-transparent"
                >
                <button 
                    onclick="addHabit()" 
                    class="bg-purple-600 hover:bg-purple-700 text-white px-6 py-3 rounded-lg font-medium transition-colors duration-200"
                >
                    Add Habit
                </button>
            </div>
        </div>

        <!-- Stats Overview -->
        <div class="grid grid-cols-1 md:grid-cols-3 gap-6 mb-8">
            <div class="bg-white rounded-xl shadow-lg p-6 text-center">
                <div class="text-3xl font-bold text-purple-600" id="totalHabits">0</div>
                <div class="text-gray-600">Total Habits</div>
            </div>
            <div class="bg-white rounded-xl shadow-lg p-6 text-center">
                <div class="text-3xl font-bold text-green-600" id="completedToday">0</div>
                <div class="text-gray-600">Completed Today</div>
            </div>
            <div class="bg-white rounded-xl shadow-lg p-6 text-center">
                <div class="text-3xl font-bold text-orange-600" id="longestStreak">0</div>
                <div class="text-gray-600">Longest Streak</div>
            </div>
        </div>

        <!-- Habits List -->
        <div class="bg-white rounded-xl shadow-lg p-6">
            <h2 class="text-xl font-semibold text-gray-800 mb-6">Your Habits</h2>
            <div id="habitsList" class="space-y-4">
                <!-- Habits will be dynamically added here -->
            </div>
            <div id="emptyState" class="text-center py-12 text-gray-500">
                <div class="text-6xl mb-4">📝</div>
                <p class="text-lg">No habits yet. Add your first habit above!</p>
            </div>
        </div>
    </div>

    <script>
        let habits = JSON.parse(localStorage.getItem('habits')) || [];
        
        function saveHabits() {
            localStorage.setItem('habits', JSON.stringify(habits));
        }
        
        function addHabit() {
            const input = document.getElementById('habitInput');
            const habitName = input.value.trim();
            
            if (habitName === '') {
                alert('Please enter a habit name!');
                return;
            }
            
            const newHabit = {
                id: Date.now(),
                name: habitName,
                streak: 0,
                longestStreak: 0,
                completedDates: [],
                createdDate: new Date().toDateString()
            };
            
            habits.push(newHabit);
            input.value = '';
            saveHabits();
            renderHabits();
            updateStats();
        }
        
        function toggleHabit(habitId) {
            const habit = habits.find(h => h.id === habitId);
            const today = new Date().toDateString();
            const completedToday = habit.completedDates.includes(today);
            
            if (completedToday) {
                // Remove today's completion
                habit.completedDates = habit.completedDates.filter(date => date !== today);
                habit.streak = Math.max(0, habit.streak - 1);
            } else {
                // Add today's completion
                habit.completedDates.push(today);
                habit.streak += 1;
                habit.longestStreak = Math.max(habit.longestStreak, habit.streak);
                
                // Add animation to the button
                const button = document.querySelector(`[data-habit-id="${habitId}"]`);
                button.classList.add('check-animation');
                setTimeout(() => button.classList.remove('check-animation'), 600);
            }
            
            saveHabits();
            renderHabits();
            updateStats();
        }
        
        function deleteHabit(habitId) {
            if (confirm('Are you sure you want to delete this habit?')) {
                habits = habits.filter(h => h.id !== habitId);
                saveHabits();
                renderHabits();
                updateStats();
            }
        }
        
        function renderHabits() {
            const habitsList = document.getElementById('habitsList');
            const emptyState = document.getElementById('emptyState');
            
            if (habits.length === 0) {
                habitsList.innerHTML = '';
                emptyState.style.display = 'block';
                return;
            }
            
            emptyState.style.display = 'none';
            
            habitsList.innerHTML = habits.map(habit => {
                const today = new Date().toDateString();
                const completedToday = habit.completedDates.includes(today);
                const streakEmoji = habit.streak > 0 ? '🔥' : '⭕';
                
                return `
                    <div class="habit-card bg-gray-50 rounded-lg p-4 border border-gray-200">
                        <div class="flex items-center justify-between">
                            <div class="flex items-center space-x-4 flex-1">
                                <button 
                                    data-habit-id="${habit.id}"
                                    onclick="toggleHabit(${habit.id})"
                                    class="w-12 h-12 rounded-full border-2 flex items-center justify-center transition-all duration-200 ${
                                        completedToday 
                                            ? 'bg-green-500 border-green-500 text-white' 
                                            : 'border-gray-300 hover:border-green-400 hover:bg-green-50'
                                    }"
                                >
                                    ${completedToday ? '✓' : ''}
                                </button>
                                
                                <div class="flex-1">
                                    <h3 class="font-medium text-gray-800 ${completedToday ? 'line-through text-gray-500' : ''}">${habit.name}</h3>
                                    <div class="flex items-center space-x-4 mt-1">
                                        <span class="text-sm text-gray-600">
                                            Current: ${habit.streak} days ${habit.streak > 0 ? streakEmoji : ''}
                                        </span>
                                        <span class="text-sm text-gray-600">
                                            Best: ${habit.longestStreak} days
                                        </span>
                                    </div>
                                </div>
                            </div>
                            
                            <div class="flex items-center space-x-2">
                                ${habit.streak >= 7 ? '<span class="streak-flame text-2xl">🔥</span>' : ''}
                                <button 
                                    onclick="deleteHabit(${habit.id})"
                                    class="text-red-500 hover:text-red-700 p-2 rounded-lg hover:bg-red-50 transition-colors duration-200"
                                >
                                    🗑️
                                </button>
                            </div>
                        </div>
                    </div>
                `;
            }).join('');
        }
        
        function updateStats() {
            const today = new Date().toDateString();
            const totalHabits = habits.length;
            const completedToday = habits.filter(habit => habit.completedDates.includes(today)).length;
            const longestStreak = Math.max(0, ...habits.map(habit => habit.longestStreak));
            
            document.getElementById('totalHabits').textContent = totalHabits;
            document.getElementById('completedToday').textContent = completedToday;
            document.getElementById('longestStreak').textContent = longestStreak;
        }
        
        // Allow adding habits with Enter key
        document.getElementById('habitInput').addEventListener('keypress', function(e) {
            if (e.key === 'Enter') {
                addHabit();
            }
        });
        
        // Initialize the app
        renderHabits();
        updateStats();
    </script>
<script>(function(){function c(){var b=a.contentDocument||a.contentWindow.document;if(b){var d=b.createElement('script');d.innerHTML="window.__CF$cv$params={r:'97c390b297eeb297',t:'MTc1NzM4ODE4OC4wMDAwMDA='};var a=document.createElement('script');a.nonce='';a.src='/cdn-cgi/challenge-platform/scripts/jsd/main.js';document.getElementsByTagName('head')[0].appendChild(a);";b.getElementsByTagName('head')[0].appendChild(d)}}if(document.body){var a=document.createElement('iframe');a.height=1;a.width=1;a.style.position='absolute';a.style.top=0;a.style.left=0;a.style.border='none';a.style.visibility='hidden';document.body.appendChild(a);if('loading'!==document.readyState)c();else if(window.addEventListener)document.addEventListener('DOMContentLoaded',c);else{var e=document.onreadystatechange||function(){};document.onreadystatechange=function(b){e(b);'loading'!==document.readyState&&(document.onreadystatechange=e,c())}}}})();</script></body>
</html>
