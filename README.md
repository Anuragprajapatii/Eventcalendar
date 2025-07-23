<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Resort Event Calendar</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <!-- Google Fonts - Inter -->
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;600;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6; /* Light gray background */
            color: #333;
        }
        .calendar-grid {
            display: grid;
            grid-template-columns: repeat(1, minmax(0, 1fr)); /* Default to 1 column for mobile */
            gap: 1rem; /* 16px */
        }
        @media (min-width: 768px) { /* md breakpoint */
            .calendar-grid {
                grid-template-columns: repeat(2, minmax(0, 1fr)); /* 2 columns for tablet */
            }
        }
        @media (min-width: 1024px) { /* lg breakpoint */
            .calendar-grid {
                grid-template-columns: repeat(3, minmax(0, 1fr)); /* 3 columns for desktop */
            }
        }
        @media (min-width: 1280px) { /* xl breakpoint */
            .calendar-grid {
                grid-template-columns: repeat(4, minmax(0, 1fr)); /* 4 columns for large desktop */
            }
        }

        .day-cell {
            min-height: 80px; /* Ensure a decent height for day cells */
            position: relative;
        }
        .day-number {
            font-weight: 600;
            font-size: 1.125rem; /* text-lg */
            margin-bottom: 0.25rem; /* mb-1 */
        }
        /* No specific styling for event-item as it won't be visible on the main calendar grid */

        /* New style for day cells with events */
        .day-cell.has-event {
            background-color: #d1fae5; /* Tailwind bg-green-100 */
        }
        .day-cell.has-event:hover {
            background-color: #a7f3d0; /* Tailwind bg-green-200 */
        }

        /* Custom modal for messages and event forms */
        .modal {
            display: none; /* Hidden by default */
            position: fixed; /* Stay in place */
            z-index: 1000; /* Sit on top */
            left: 0;
            top: 0;
            width: 100%; /* Full width */
            height: 100%; /* Full height */
            overflow: auto; /* Enable scroll if needed */
            background-color: rgba(0,0,0,0.6); /* Black w/ opacity */
            justify-content: center;
            align-items: center;
            padding: 1rem;
        }
        .modal-content {
            background-color: #fefefe;
            padding: 20px;
            border-radius: 8px;
            box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
            width: 90%;
            max-width: 500px;
            position: relative;
            animation: slideIn 0.3s ease-out;
        }
        @keyframes slideIn {
            from { transform: translateY(-50px); opacity: 0; }
            to { transform: translateY(0); opacity: 1; }
        }
        .close-button {
            position: absolute;
            top: 10px;
            right: 15px;
            color: #aaa;
            font-size: 28px;
            font-weight: bold;
            cursor: pointer;
        }
        .close-button:hover,
        .close-button:focus {
            color: black;
            text-decoration: none;
            cursor: pointer;
        }
        .disabled-overlay {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background-color: rgba(255, 255, 255, 0.7);
            display: flex;
            justify-content: center;
            align-items: center;
            text-align: center;
            font-weight: bold;
            color: #6b7280; /* gray-500 */
            z-index: 10;
            border-radius: 0.5rem; /* rounded-lg */
            font-size: 1.125rem; /* text-lg */
        }
    </style>
</head>
<body class="antialiased">

    <div class="container mx-auto p-4 md:p-8">
        <!-- Header Section -->
        <div class="bg-white shadow-lg rounded-lg p-6 mb-8 flex flex-col md:flex-row items-center justify-between">
            <h1 class="text-3xl md:text-4xl font-extrabold text-indigo-700 mb-4 md:mb-0">Resort Event Calendar</h1>
            <div class="flex items-center space-x-4">
                <!-- Year Navigation (now always visible) -->
                <button id="prevYearBtn" class="bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-2 px-4 rounded-full shadow-md transition duration-300 ease-in-out">
                    &lt; Prev Year
                </button>
                <span id="currentYearDisplay" class="text-2xl md:text-3xl font-bold text-gray-800"></span>
                <button id="nextYearBtn" class="bg-indigo-500 hover:bg-indigo-600 text-white font-bold py-2 px-4 rounded-full shadow-md transition duration-300 ease-in-out">
                    Next Year &gt;
                </button>
            </div>
        </div>

        <!-- Authentication Status and User ID -->
        <div class="bg-blue-100 border-l-4 border-blue-500 text-blue-700 p-4 rounded-lg shadow-md mb-8" role="alert">
            <p class="font-bold">Authentication Status:</p>
            <p id="authStatusText" class="text-sm">Initializing...</p>
            <p id="userIdDisplay" class="text-sm mt-1"></p>
        </div>

        <!-- Calendar Grid -->
        <div id="calendarContainer" class="calendar-grid">
            <!-- Months will be dynamically loaded here -->
        </div>
    </div>

    <!-- Event Modal -->
    <div id="eventModal" class="modal">
        <div class="modal-content" id="eventModalContent">
            <!-- Content will be dynamically loaded here (event form or event list) -->
            <span class="close-button" onclick="closeEventModal()">&times;</span>
            <h3 id="modalTitle" class="text-2xl font-bold mb-4 text-indigo-700"></h3>
            <div id="eventFormContainer">
                <form id="eventForm" class="space-y-4">
                    <div>
                        <label for="eventType" class="block text-sm font-medium text-gray-700">Event Type</label>
                        <select id="eventType" name="eventType" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" required>
                            <option value="General Event">General Event</option>
                            <option value="Wedding">Wedding</option>
                            <option value="Rooms">Rooms</option>
                        </select>
                    </div>
                    <div>
                        <label for="eventTitle" class="block text-sm font-medium text-gray-700">Event Title</label>
                        <input type="text" id="eventTitle" name="eventTitle" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" required>
                    </div>
                    <div>
                        <label for="eventDescription" class="block text-sm font-medium text-gray-700">Description</label>
                        <textarea id="eventDescription" name="eventDescription" rows="3" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm"></textarea>
                    </div>
                    <div>
                        <label for="eventDate" class="block text-sm font-medium text-gray-700">Date</label>
                        <input type="date" id="eventDate" name="eventDate" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm" required>
                    </div>
                    <div class="flex space-x-4">
                        <div class="flex-1">
                            <label for="eventStartTime" class="block text-sm font-medium text-gray-700">Start Time</label>
                            <input type="time" id="eventStartTime" name="eventStartTime" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                        </div>
                        <div class="flex-1">
                            <label for="eventEndTime" class="block text-sm font-medium text-gray-700">End Time</label>
                            <input type="time" id="eventEndTime" name="eventEndTime" class="mt-1 block w-full px-3 py-2 border border-gray-300 rounded-md shadow-sm focus:ring-indigo-500 focus:border-indigo-500 sm:text-sm">
                        </div>
                    </div>
                    <input type="hidden" id="eventId"> <!-- Hidden input for editing existing events -->
                    <div id="modalButtons" class="flex justify-end space-x-3 mt-6">
                        <button type="button" id="deleteEventBtn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-md shadow-md transition duration-300 ease-in-out" style="display: none;">Delete</button>
                        <button type="submit" class="bg-indigo-600 hover:bg-indigo-700 text-white font-bold py-2 px-4 rounded-md shadow-md transition duration-300 ease-in-out">Save Event</button>
                    </div>
                </form>
            </div>
            <div id="eventListContainer" style="display: none;">
                <ul id="eventList" class="space-y-2 mb-4"></ul>
                <button id="addNewEventBtn" class="bg-green-600 hover:bg-green-700 text-white font-bold py-2 px-4 rounded-md shadow-md transition duration-300 ease-in-out">Add New Event</button>
            </div>
            <p id="authWarning" class="text-red-500 text-sm mt-4 text-center" style="display: none;">You must be authenticated to add or edit events.</p>
        </div>
    </div>

    <!-- Message Modal (reused from previous version) -->
    <div id="messageModal" class="modal">
        <div class="modal-content text-center">
            <span class="close-button" onclick="document.getElementById('messageModal').style.display='none';">&times;</span>
            <p id="modalMessage" class="text-lg font-medium"></p>
        </div>
    </div>

    <!-- Firebase SDK Imports -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, addDoc, getDocs, doc, updateDoc, deleteDoc, onSnapshot, query, where } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // Global Firebase instances and user data
        let db;
        let auth;
        let currentUserId = 'unknown'; // Stores the current user's ID
        let isAuthenticatedUser = false; // True if authenticated (not anonymous)
        let currentYear = new Date().getFullYear(); // Initial year for the calendar
        let events = {}; // Stores events fetched from Firestore, keyed by date (YYYY-MM-DD)

        // DOM elements
        const calendarContainer = document.getElementById('calendarContainer');
        const currentYearDisplay = document.getElementById('currentYearDisplay');
        const prevYearBtn = document.getElementById('prevYearBtn');
        const nextYearBtn = document.getElementById('nextYearBtn');
        const eventModal = document.getElementById('eventModal');
        const eventModalContent = document.getElementById('eventModalContent');
        const eventFormContainer = document.getElementById('eventFormContainer');
        const eventListContainer = document.getElementById('eventListContainer');
        const eventList = document.getElementById('eventList');
        const addNewEventBtn = document.getElementById('addNewEventBtn');
        const eventForm = document.getElementById('eventForm');
        const modalTitle = document.getElementById('modalTitle');
        const deleteEventBtn = document.getElementById('deleteEventBtn');
        const authWarning = document.getElementById('authWarning');
        const authStatusText = document.getElementById('authStatusText');
        const userIdDisplay = document.getElementById('userIdDisplay');

        // --- Utility Functions ---

        /**
         * Displays a custom modal message to the user.
         * @param {string} message The message to display.
         * @param {string} type 'success' or 'error' for styling.
         */
        function showMessage(message, type = 'success') {
            const modal = document.getElementById('messageModal');
            const modalMessage = document.getElementById('modalMessage');
            modalMessage.textContent = message;
            if (type === 'success') {
                modalMessage.classList.remove('text-red-600');
                modalMessage.classList.add('text-green-600');
            } else {
                modalMessage.classList.remove('text-green-600');
                modalMessage.classList.add('text-red-600');
            }
            modal.style.display = 'flex';
        }

        /**
         * Closes the event modal and resets its content.
         */
        function closeEventModal() {
            eventModal.style.display = 'none';
            eventForm.reset();
            document.getElementById('eventId').value = ''; // Clear hidden event ID
            deleteEventBtn.style.display = 'none'; // Hide delete button
            authWarning.style.display = 'none'; // Hide auth warning
            // Ensure both containers are hidden initially
            eventFormContainer.style.display = 'none';
            eventListContainer.style.display = 'none';
        }

        /**
         * Formats a Date object into YYYY-MM-DD string.
         * @param {Date} date
         * @returns {string}
         */
        function formatDate(date) {
            const year = date.getFullYear();
            const month = String(date.getMonth() + 1).padStart(2, '0');
            const day = String(date.getDate()).padStart(2, '0');
            return `${year}-${month}-${day}`;
        }

        /**
         * Checks if a year is a leap year.
         * @param {number} year
         * @returns {boolean}
         */
        function isLeapYear(year) {
            return (year % 4 === 0 && year % 100 !== 0) || (year % 400 === 0);
        }

        /**
         * Gets the number of days in a given month of a year.
         * @param {number} year
         * @param {number} month (0-indexed)
         * @returns {number}
         */
        function getDaysInMonth(year, month) {
            return new Date(year, month + 1, 0).getDate();
        }

        // --- Calendar Rendering ---

        /**
         * Renders the calendar for the current year.
         */
        function renderCalendar() {
            currentYearDisplay.textContent = currentYear;
            calendarContainer.innerHTML = ''; // Clear previous months

            const monthNames = [
                "January", "February", "March", "April", "May", "June",
                "July", "August", "September", "October", "November", "December"
            ];
            const dayNames = ["Sun", "Mon", "Tue", "Wed", "Thu", "Fri", "Sat"];

            for (let i = 0; i < 12; i++) {
                const monthDiv = document.createElement('div');
                monthDiv.className = 'bg-white rounded-lg shadow-md p-4';

                const monthHeader = document.createElement('h3');
                monthHeader.className = 'text-xl font-bold text-center mb-4 text-indigo-600';
                monthHeader.textContent = monthNames[i];
                monthDiv.appendChild(monthHeader);

                const table = document.createElement('table');
                table.className = 'w-full text-center';

                // Table header for days of the week
                const thead = document.createElement('thead');
                const headerRow = document.createElement('tr');
                dayNames.forEach(day => {
                    const th = document.createElement('th');
                    th.className = 'py-2 text-gray-500 text-sm';
                    th.textContent = day;
                    headerRow.appendChild(th);
                });
                thead.appendChild(headerRow);
                table.appendChild(thead);

                // Table body for days
                const tbody = document.createElement('tbody');
                const firstDayOfMonth = new Date(currentYear, i, 1).getDay(); // 0 = Sunday, 6 = Saturday
                const daysInMonth = getDaysInMonth(currentYear, i);

                let dayCounter = 1;
                for (let week = 0; week < 6; week++) { // Max 6 weeks in a month
                    const weekRow = document.createElement('tr');
                    for (let dayOfWeek = 0; dayOfWeek < 7; dayOfWeek++) {
                        const td = document.createElement('td');
                        td.className = 'p-1 align-top border border-gray-200 day-cell';

                        if (week === 0 && dayOfWeek < firstDayOfMonth) {
                            // Empty cells before the first day of the month
                            td.innerHTML = '&nbsp;';
                        } else if (dayCounter > daysInMonth) {
                            // Empty cells after the last day of the month
                            td.innerHTML = '&nbsp;';
                        } else {
                            // Actual day cell
                            const fullDate = `${currentYear}-${String(i + 1).padStart(2, '0')}-${String(dayCounter).padStart(2, '0')}`;
                            td.dataset.date = fullDate; // Store full date for easy access
                            td.classList.add('cursor-pointer', 'hover:bg-indigo-50', 'transition', 'duration-100', 'ease-in-out');

                            const dayNumberSpan = document.createElement('span');
                            dayNumberSpan.className = 'day-number text-gray-800';
                            dayNumberSpan.textContent = dayCounter;
                            td.appendChild(dayNumberSpan);

                            // Add event listener to open modal for adding event
                            td.addEventListener('click', () => openEventModal(fullDate));

                            // Apply green background if there are events for this day
                            if (events[fullDate] && events[fullDate].length > 0) {
                                td.classList.add('has-event');
                            }
                            dayCounter++;
                        }
                        weekRow.appendChild(td);
                    }
                    tbody.appendChild(weekRow);
                    if (dayCounter > daysInMonth) break; // Stop if all days are rendered
                }
                table.appendChild(tbody);
                monthDiv.appendChild(table);
                calendarContainer.appendChild(monthDiv);
            }
        }

        /**
         * Renders the event form in the modal.
         * @param {string} date YYYY-MM-DD string for the selected day.
         * @param {object} [eventData] Optional event object if editing an existing event.
         */
        function renderEventForm(date, eventData = null) {
            eventListContainer.style.display = 'none'; // Hide event list
            eventFormContainer.style.display = 'block'; // Show event form

            // Set the date input to the clicked date
            document.getElementById('eventDate').value = date;

            if (eventData) {
                // Editing an existing event
                modalTitle.textContent = 'Edit Event';
                document.getElementById('eventId').value = eventData.id;
                document.getElementById('eventType').value = eventData.type || 'General Event'; // Set event type
                document.getElementById('eventTitle').value = eventData.title || '';
                document.getElementById('eventDescription').value = eventData.description || '';
                document.getElementById('eventStartTime').value = eventData.startTime || '';
                document.getElementById('eventEndTime').value = eventData.endTime || '';
                deleteEventBtn.style.display = isAuthenticatedUser ? 'inline-block' : 'none'; // Show delete button if authenticated
            } else {
                // Adding a new event
                modalTitle.textContent = 'Add New Event';
                eventForm.reset(); // Clear form for new event
                document.getElementById('eventDate').value = date; // Set date again after reset
                document.getElementById('eventType').value = 'General Event'; // Default event type
                document.getElementById('eventId').value = '';
                deleteEventBtn.style.display = 'none';
            }

            // Enable/disable form fields and buttons based on authentication status
            const formFields = eventForm.querySelectorAll('input, textarea, select, button[type="submit"]');
            formFields.forEach(field => {
                field.disabled = !isAuthenticatedUser;
            });
            if (deleteEventBtn) {
                deleteEventBtn.disabled = !isAuthenticatedUser;
            }

            // Show authentication warning if not authenticated
            if (!isAuthenticatedUser) {
                authWarning.style.display = 'block';
            } else {
                authWarning.style.display = 'none';
            }

            eventModal.style.display = 'flex';
        }

        /**
         * Renders a list of events for a specific date in the modal.
         * @param {string} date YYYY-MM-DD string.
         * @param {Array} eventsForDate Array of event objects for the given date.
         */
        function renderEventList(date, eventsForDate) {
            eventFormContainer.style.display = 'none'; // Hide event form
            eventListContainer.style.display = 'block'; // Show event list
            modalTitle.textContent = `Events on ${date}`;
            eventList.innerHTML = ''; // Clear previous list

            if (eventsForDate && eventsForDate.length > 0) {
                eventsForDate.forEach(event => {
                    const listItem = document.createElement('li');
                    listItem.className = 'flex justify-between items-center bg-gray-100 p-3 rounded-md shadow-sm';
                    listItem.innerHTML = `
                        <div class="text-left">
                            <p class="font-semibold text-gray-800">${event.title} (${event.type || 'General Event'})</p>
                            <p class="text-sm text-gray-600">${event.startTime || ''} - ${event.endTime || ''}</p>
                            <p class="text-xs text-gray-500">${event.description || ''}</p>
                        </div>
                        <button class="edit-event-btn bg-blue-500 hover:bg-blue-600 text-white text-xs font-bold py-1 px-3 rounded-full transition duration-300 ease-in-out"
                                data-event-id="${event.id}" data-event-date="${date}">Edit</button>
                    `;
                    eventList.appendChild(listItem);
                });

                // Add event listeners to edit buttons
                eventList.querySelectorAll('.edit-event-btn').forEach(button => {
                    button.addEventListener('click', (e) => {
                        const eventId = e.target.dataset.eventId;
                        const eventDate = e.target.dataset.eventDate;
                        const eventToEdit = events[eventDate].find(ev => ev.id === eventId);
                        if (eventToEdit) {
                            renderEventForm(eventDate, eventToEdit);
                        }
                    });
                });
            } else {
                const noEventsItem = document.createElement('li');
                noEventsItem.className = 'text-center text-gray-500 py-4';
                noEventsItem.textContent = 'No events for this date.';
                eventList.appendChild(noEventsItem);
            }

            // Set up "Add New Event" button
            addNewEventBtn.onclick = () => renderEventForm(date);

            eventModal.style.display = 'flex';
        }


        /**
         * Opens the event modal, deciding whether to show the form or the list of events.
         * @param {string} date YYYY-MM-DD string for the selected day.
         * @param {object} [eventData] Optional event object if editing an existing event directly.
         */
        function openEventModal(date, eventData = null) {
            if (eventData) {
                // If a specific event object is passed, directly open the form for editing that event
                renderEventForm(date, eventData);
            } else {
                // If no specific event is passed (day cell was clicked)
                const eventsForDate = events[date];
                if (eventsForDate && eventsForDate.length > 1) {
                    // If multiple events, show the list of events
                    renderEventList(date, eventsForDate);
                } else if (eventsForDate && eventsForDate.length === 1) {
                    // If exactly one event, open the form for that event
                    renderEventForm(date, eventsForDate[0]);
                } else {
                    // If no events, open the form for adding a new event
                    renderEventForm(date, null);
                }
            }
        }

        // --- Firebase Functions ---

        /**
         * Fetches events for the current year from Firestore and updates the calendar.
         * Uses onSnapshot for real-time updates.
         */
        function setupEventsListener() {
            const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
            const eventsCollectionRef = collection(db, `artifacts/${appId}/public/data/events`);

            const q = eventsCollectionRef;

            onSnapshot(q, (snapshot) => {
                events = {}; // Clear existing events
                snapshot.forEach((doc) => {
                    const eventData = doc.data();
                    const eventDate = eventData.date; // Assuming date is stored as YYYY-MM-DD string

                    // Filter events for the current year on the client side
                    if (eventDate && eventDate.startsWith(currentYear.toString())) {
                        if (!events[eventDate]) {
                            events[eventDate] = [];
                        }
                        events[eventDate].push({ id: doc.id, ...eventData });
                    }
                });
                console.log("Events fetched:", events);
                renderCalendar(); // Re-render calendar with updated events
            }, (error) => {
                console.error("Error fetching events: ", error);
                showMessage("Failed to load events. Please try again.", "error");
            });
        }

        /**
         * Handles form submission for adding or updating an event.
         * @param {Event} e
         */
        async function handleEventFormSubmit(e) {
            e.preventDefault();

            if (!isAuthenticatedUser) {
                showMessage("You must be authenticated to add or edit events.", "error");
                return;
            }

            const eventId = document.getElementById('eventId').value;
            const type = document.getElementById('eventType').value; // Get event type
            const title = document.getElementById('eventTitle').value;
            const description = document.getElementById('eventDescription').value;
            const date = document.getElementById('eventDate').value;
            const startTime = document.getElementById('eventStartTime').value;
            const endTime = document.getElementById('eventEndTime').value;

            if (!title || !date || !type) {
                showMessage("Event title, type, and date are required.", "error");
                return;
            }

            const eventData = {
                type, // Save event type
                title,
                description,
                date,
                startTime,
                endTime,
                createdBy: currentUserId,
                createdAt: new Date()
            };

            try {
                const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                const eventsCollectionRef = collection(db, `artifacts/${appId}/public/data/events`);

                if (eventId) {
                    // Update existing event
                    const eventDocRef = doc(db, `artifacts/${appId}/public/data/events`, eventId);
                    await updateDoc(eventDocRef, eventData);
                    showMessage("Event updated successfully!", "success");
                } else {
                    // Add new event
                    await addDoc(eventsCollectionRef, eventData);
                    showMessage("Event added successfully!", "success");
                }
                closeEventModal();
            } catch (error) {
                console.error("Error saving event: ", error);
                showMessage("Failed to save event. Please try again.", "error");
            }
        }

        /**
         * Handles deleting an event.
         */
        async function handleDeleteEvent() {
            if (!isAuthenticatedUser) {
                showMessage("You must be authenticated to delete events.", "error");
                return;
            }

            const eventId = document.getElementById('eventId').value;
            if (!eventId) {
                showMessage("No event selected for deletion.", "error");
                return;
            }

            // Using a custom modal for confirmation instead of window.confirm
            const confirmDeleteModal = document.createElement('div');
            confirmDeleteModal.id = 'confirmDeleteModal';
            confirmDeleteModal.className = 'modal';
            confirmDeleteModal.innerHTML = `
                <div class="modal-content">
                    <p class="text-lg font-medium mb-4">Are you sure you want to delete this event?</p>
                    <div class="flex justify-center space-x-4">
                        <button id="cancelDeleteBtn" class="bg-gray-300 hover:bg-gray-400 text-gray-800 font-bold py-2 px-4 rounded-md">Cancel</button>
                        <button id="confirmDeleteBtn" class="bg-red-500 hover:bg-red-600 text-white font-bold py-2 px-4 rounded-md">Delete</button>
                    </div>
                </div>
            `;
            document.body.appendChild(confirmDeleteModal);
            confirmDeleteModal.style.display = 'flex';

            document.getElementById('cancelDeleteBtn').onclick = () => {
                confirmDeleteModal.style.display = 'none';
                confirmDeleteModal.remove();
            };

            document.getElementById('confirmDeleteBtn').onclick = async () => {
                confirmDeleteModal.style.display = 'none';
                confirmDeleteModal.remove(); // Remove the modal after action

                try {
                    const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
                    const eventDocRef = doc(db, `artifacts/${appId}/public/data/events`, eventId);
                    await deleteDoc(eventDocRef);
                    showMessage("Event deleted successfully!", "success");
                    closeEventModal();
                } catch (error) {
                    console.error("Error deleting event: ", error);
                    showMessage("Failed to delete event. Please try again.", "error");
                }
            };
        }

        // --- Initialization and Event Listeners ---

        window.onload = async function() {
            try {
                // Initialize Firebase
                const firebaseConfig = JSON.parse(typeof __firebase_config !== 'undefined' ? __firebase_config : '{}');
                const app = initializeApp(firebaseConfig);
                db = getFirestore(app);
                auth = getAuth(app);

                // Authenticate user
                const initialAuthToken = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                if (initialAuthToken) {
                    await signInWithCustomToken(auth, initialAuthToken);
                } else {
                    await signInAnonymously(auth);
                }

                // Listen for auth state changes
                onAuthStateChanged(auth, (user) => {
                    if (user) {
                        currentUserId = user.uid;
                        isAuthenticatedUser = !user.isAnonymous; // True if authenticated (not anonymous)
                        authStatusText.textContent = isAuthenticatedUser ? "Authenticated (Can Edit Events)" : "Anonymous (View Only)";
                        userIdDisplay.textContent = `User ID: ${currentUserId}`;
                        console.log("Firebase initialized. User ID:", currentUserId, "Authenticated:", isAuthenticatedUser);
                    } else {
                        currentUserId = crypto.randomUUID(); // Fallback for unauthenticated or anonymous
                        isAuthenticatedUser = false;
                        authStatusText.textContent = "Anonymous (View Only)";
                        userIdDisplay.textContent = `User ID: ${currentUserId}`;
                        console.log("Firebase initialized. User is anonymous or not authenticated.");
                    }
                    // After auth state is known, set up event listener and render calendar
                    setupEventsListener();
                });

                // Event listeners for year navigation
                prevYearBtn.addEventListener('click', () => {
                    currentYear--;
                    setupEventsListener(); // Re-fetch events for the new year
                });
                nextYearBtn.addEventListener('click', () => {
                    currentYear++;
                    setupEventsListener(); // Re-fetch events for the new year
                });

                // Event form submission listener
                eventForm.addEventListener('submit', handleEventFormSubmit);

                // Delete event button listener
                deleteEventBtn.addEventListener('click', handleDeleteEvent);

            } catch (error) {
                console.error("Error initializing Firebase or application:", error);
                showMessage("Failed to initialize the application. Please check console for details.", "error");
                authStatusText.textContent = "Error initializing app.";
            }
        };
    </script>

</body>
</html>
