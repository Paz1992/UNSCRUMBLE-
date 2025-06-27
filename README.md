<html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Unscramble the Verb</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;700&display=swap" rel="stylesheet">
    <style>
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f0f0f0;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            padding: 20px;
            box-sizing: border-box;
        }
        .game-container {
            background-color: #ffffff;
            padding: 30px;
            border-radius: 20px;
            box-shadow: 0 10px 25px rgba(0, 0, 0, 0.1);
            text-align: center;
            max-width: 600px;
            width: 100%;
        }
        .scrambled-letters-container {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-top: 30px;
            margin-bottom: 30px;
        }
        .scrambled-letter-box, .empty-letter-box {
            width: 60px;
            height: 60px;
            background-color: #4a5568; /* Darker gray for letters */
            color: #ffffff;
            font-size: 2.5rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            border-radius: 12px;
            cursor: grab;
            transition: transform 0.2s, background-color 0.2s;
            box-shadow: 0 4px 8px rgba(0, 0, 0, 0.2);
        }
        .scrambled-letter-box:active {
            cursor: grabbing;
            transform: scale(1.05);
        }
        .empty-boxes-container {
            display: flex;
            justify-content: center;
            gap: 15px;
            margin-bottom: 40px;
        }
        .empty-letter-box {
            background-color: #e2e8f0; /* Light gray for empty boxes */
            border: 2px dashed #a0aec0;
            color: #4a5568;
            font-size: 2.5rem;
            font-weight: bold;
            display: flex;
            justify-content: center;
            align-items: center;
            cursor: default;
            box-shadow: none;
        }
        .feedback-message {
            margin-top: 20px;
            font-size: 1.5rem;
            font-weight: bold;
            min-height: 40px; /* To prevent layout shift */
        }
        .correct {
            color: #48bb78; /* Green for success */
        }
        .incorrect {
            color: #f56565; /* Red for error */
        }
        .button-group {
            display: flex;
            justify-content: center;
            gap: 20px;
            margin-top: 20px;
        }
        .button-next, .button-retry {
            background-color: #4299e1; /* Blue */
            color: #ffffff;
            padding: 15px 30px;
            border-radius: 12px;
            font-size: 1.25rem;
            font-weight: bold;
            cursor: pointer;
            border: none;
            transition: background-color 0.3s, transform 0.2s;
            box-shadow: 0 6px 12px rgba(0, 0, 0, 0.2);
        }
        .button-next:hover, .button-retry:hover {
            background-color: #3182ce;
            transform: translateY(-2px);
        }
        .button-next:disabled, .button-retry:disabled {
            background-color: #a0aec0;
            cursor: not-allowed;
            box-shadow: none;
            transform: none;
        }
        .draggable {
            position: absolute;
            z-index: 1000;
        }

        /* Responsive adjustments */
        @media (max-width: 640px) {
            .scrambled-letter-box, .empty-letter-box {
                width: 50px;
                height: 50px;
                font-size: 2rem;
            }
            .scrambled-letters-container, .empty-boxes-container {
                gap: 10px;
            }
            .game-container {
                padding: 20px;
            }
            .button-next, .button-retry {
                padding: 12px 24px;
                font-size: 1rem;
            }
            .button-group {
                flex-direction: column;
                gap: 10px;
            }
        }
    </style>
</head>
<body>
    <div class="game-container">
        <h1 class="text-4xl font-bold text-gray-800 mb-6">Unscramble the Verb</h1>

        <!-- Visual clue image -->
        <div class="mb-8">
            <img id="verb-image" src="https://placehold.co/300x200/cccccc/333333?text=Image+of+Action" alt="Visual clue for the verb" class="mx-auto rounded-lg shadow-lg">
        </div>

        <!-- Container for draggable scrambled letters -->
        <div id="scrambled-letters-container" class="scrambled-letters-container">
            <!-- Scrambled letters will be inserted here by JavaScript -->
        </div>

        <!-- Container for empty boxes where letters are dropped -->
        <div id="empty-boxes-container" class="empty-boxes-container">
            <!-- Empty boxes will be inserted here by JavaScript -->
        </div>

        <!-- Feedback message area -->
        <div id="feedback-message" class="feedback-message text-gray-700"></div>

        <!-- Button Group for Next and Retry -->
        <div class="button-group">
            <button id="retry-button" class="button-retry" style="display:none;">Try Again</button>
            <button id="next-button" class="button-next" disabled>Next</button>
        </div>
    </div>

    <script>
        // Array of verb puzzles. Each puzzle includes:
        // - word: The correct verb
        // - image: URL for the visual clue
        // - hint: (Optional) A hint for the word (not used in this version but good for future expansion)
        const verbPuzzles = [
            { word: "run", image: "https://placehold.co/300x200/ffcc00/000000?text=Running+Kids" }, // Placeholder for the provided image
            { word: "jump", image: "https://placehold.co/300x200/a0e0a0/000000?text=Jumping+Person" },
            { word: "read", image: "https://placehold.co/300x200/c0c0ff/000000?text=Reading+Book" },
            { word: "eat", image: "https://placehold.co/300x200/ff99aa/000000?text=Eating+Food" },
            { word: "play", image: "https://placehold.co/300x200/99ff99/000000?text=Playing+Ball" }
        ];

        let currentPuzzleIndex = 0; // Tracks the current puzzle
        let currentWord = ""; // The correct word for the current puzzle
        let scrambledLetters = []; // Array to hold the scrambled letters
        let placedLetters = []; // Array to hold letters placed in empty boxes
        let dragSrcEl = null; // Element being dragged
        let dragData = {}; // Stores data about the dragged element (its original position, etc.)

        const scrambledLettersContainer = document.getElementById('scrambled-letters-container');
        const emptyBoxesContainer = document.getElementById('empty-boxes-container');
        const feedbackMessage = document.getElementById('feedback-message');
        const nextButton = document.getElementById('next-button');
        const retryButton = document.getElementById('retry-button'); // Get the new retry button
        const verbImage = document.getElementById('verb-image');

        // Initialize AudioContext
        const audioContext = new (window.AudioContext || window.webkitAudioContext)();

        /**
         * Plays a success sound (short, high-pitched tone).
         */
        function playSuccessSound() {
            if (!audioContext) return;
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();

            oscillator.type = 'sine';
            oscillator.frequency.setValueAtTime(880, audioContext.currentTime); // A5
            gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);

            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);

            oscillator.start();
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.2); // Fade out
            oscillator.stop(audioContext.currentTime + 0.2); // Stop after 0.2 seconds
        }

        /**
         * Plays an error sound (short, low-pitched tone).
         */
        function playErrorSound() {
            if (!audioContext) return;
            const oscillator = audioContext.createOscillator();
            const gainNode = audioContext.createGain();

            oscillator.type = 'sawtooth';
            oscillator.frequency.setValueAtTime(220, audioContext.currentTime); // A3
            gainNode.gain.setValueAtTime(0.3, audioContext.currentTime);

            oscillator.connect(gainNode);
            gainNode.connect(audioContext.destination);

            oscillator.start();
            gainNode.gain.exponentialRampToValueAtTime(0.001, audioContext.currentTime + 0.3); // Fade out
            oscillator.stop(audioContext.currentTime + 0.3); // Stop after 0.3 seconds
        }

        /**
         * Shuffles an array randomly.
         * @param {Array} array - The array to shuffle.
         * @returns {Array} The shuffled array.
         */
        function shuffleArray(array) {
            for (let i = array.length - 1; i > 0; i--) {
                const j = Math.floor(Math.random() * (i + 1));
                [array[i], array[j]] = [array[j], array[i]]; // ES6 swap
            }
            return array;
        }

        /**
         * Initializes a new puzzle.
         */
        function initializePuzzle() {
            // Reset feedback and button state
            feedbackMessage.textContent = "";
            nextButton.disabled = true;
            retryButton.style.display = 'none'; // Hide retry button at the start of a new puzzle

            // Clear containers
            scrambledLettersContainer.innerHTML = '';
            emptyBoxesContainer.innerHTML = '';

            // Get the current puzzle
            const puzzle = verbPuzzles[currentPuzzleIndex];
            currentWord = puzzle.word;

            // Update the image clue
            verbImage.src = puzzle.image;

            // Create scrambled letters
            scrambledLetters = shuffleArray(currentWord.split(''));
            placedLetters = new Array(currentWord.length).fill(null); // Initialize with nulls

            scrambledLetters.forEach((letter, index) => {
                const letterBox = document.createElement('div');
                letterBox.classList.add('scrambled-letter-box', 'rounded-xl', 'draggable-item', 'cursor-grab');
                letterBox.textContent = letter.toUpperCase();
                letterBox.setAttribute('draggable', 'true');
                letterBox.setAttribute('data-original-index', index); // Store original index in scrambledLetters array
                letterBox.setAttribute('data-letter', letter.toUpperCase());

                // Add drag event listeners
                letterBox.addEventListener('dragstart', handleDragStart);
                letterBox.addEventListener('touchstart', handleTouchStart, { passive: false }); // For touch devices
                scrambledLettersContainer.appendChild(letterBox);
            });

            // Create empty boxes
            for (let i = 0; i < currentWord.length; i++) {
                const emptyBox = document.createElement('div');
                emptyBox.classList.add('empty-letter-box', 'rounded-xl', 'droppable-target');
                emptyBox.setAttribute('data-index', i); // Store the index of this empty box

                // Add drop event listeners
                emptyBox.addEventListener('dragover', handleDragOver);
                emptyBox.addEventListener('drop', handleDrop);
                emptyBox.addEventListener('dragleave', handleDragLeave); // For visual feedback
                emptyBox.addEventListener('touchmove', handleTouchMove, { passive: false });
                emptyBox.addEventListener('touchend', handleTouchEnd);
                emptyBoxesContainer.appendChild(emptyBox);
            }
        }

        /**
         * Handles the start of a drag operation.
         * @param {Event} e - The drag event.
         */
        function handleDragStart(e) {
            dragSrcEl = this; // 'this' refers to the element being dragged
            e.dataTransfer.effectAllowed = 'move';
            e.dataTransfer.setData('text/plain', this.getAttribute('data-letter')); // Set data for transfer
            this.classList.add('opacity-50'); // Add a visual cue for dragging
        }

        /**
         * Handles an element being dragged over a drop target.
         * @param {Event} e - The drag event.
         */
        function handleDragOver(e) {
            e.preventDefault(); // Necessary to allow dropping
            e.dataTransfer.dropEffect = 'move';
            this.classList.add('border-blue-500', 'bg-blue-100'); // Visual feedback
        }

        /**
         * Handles an element leaving a drop target.
         * @param {Event} e - The drag event.
         */
        function handleDragLeave(e) {
            this.classList.remove('border-blue-500', 'bg-blue-100'); // Remove visual feedback
        }

        /**
         * Handles an element being dropped onto a target.
         * @param {Event} e - The drop event.
         */
        function handleDrop(e) {
            e.preventDefault();
            this.classList.remove('border-blue-500', 'bg-blue-100');

            if (this.textContent === '') { // Only allow drop if box is empty
                const letter = dragSrcEl.getAttribute('data-letter');
                const originalScrambledIndex = parseInt(dragSrcEl.getAttribute('data-original-index'));
                const emptyBoxIndex = parseInt(this.getAttribute('data-index'));

                // Update the visual of the dropped letter
                this.textContent = letter;
                this.style.backgroundColor = '#4a5568'; // Match scrambled letter box color
                this.style.color = '#ffffff';
                this.style.border = 'none'; // Remove dashed border

                // Hide the original draggable letter
                dragSrcEl.style.display = 'none';

                // Record the placement
                placedLetters[emptyBoxIndex] = {
                    letter: letter,
                    originalScrambledIndex: originalScrambledIndex,
                    dragSrcEl: dragSrcEl // Keep reference to the original element for 'undo' or moving back
                };

                // Clear the dragged element reference
                dragSrcEl = null;

                // Check for completion
                checkWord();
            } else {
                // If trying to drop on an occupied box, revert styles
                dragSrcEl.classList.remove('opacity-50');
            }
        }

        /**
         * Handles the start of a touch drag operation.
         * @param {Event} e - The touch event.
         */
        function handleTouchStart(e) {
            e.preventDefault(); // Prevent scrolling and other default touch behaviors
            dragSrcEl = this;
            dragSrcEl.classList.add('opacity-50');
            dragSrcEl.style.position = 'absolute';
            dragSrcEl.style.zIndex = '1000';

            const touch = e.touches[0];
            dragData = {
                startX: touch.clientX,
                startY: touch.clientY,
                initialLeft: dragSrcEl.offsetLeft,
                initialTop: dragSrcEl.offsetTop,
                originalParent: dragSrcEl.parentNode,
                originalIndexInParent: Array.from(dragSrcEl.parentNode.children).indexOf(dragSrcEl)
            };

            // Append to body to allow free movement
            document.body.appendChild(dragSrcEl);
            // Position the element at the touch point
            dragSrcEl.style.left = (touch.clientX - dragSrcEl.offsetWidth / 2) + 'px';
            dragSrcEl.style.top = (touch.clientY - dragSrcEl.offsetHeight / 2) + 'px';
        }

        /**
         * Handles an element being dragged via touch.
         * @param {Event} e - The touch event.
         */
        function handleTouchMove(e) {
            e.preventDefault(); // Prevent scrolling
            if (!dragSrcEl) return;

            const touch = e.touches[0];
            dragSrcEl.style.left = (touch.clientX - dragSrcEl.offsetWidth / 2) + 'px';
            dragSrcEl.style.top = (touch.clientY - dragSrcEl.offsetHeight / 2) + 'px';

            // Highlight potential drop targets
            const targetElement = document.elementFromPoint(touch.clientX, touch.clientY);
            document.querySelectorAll('.empty-letter-box').forEach(box => {
                box.classList.remove('border-blue-500', 'bg-blue-100');
            });
            if (targetElement && targetElement.classList.contains('empty-letter-box') && targetElement.textContent === '') {
                targetElement.classList.add('border-blue-500', 'bg-blue-100');
            }
        }

        /**
         * Handles an element being dropped via touch.
         * @param {Event} e - The touch event.
         */
        function handleTouchEnd(e) {
            if (!dragSrcEl) return;

            dragSrcEl.classList.remove('opacity-50');
            dragSrcEl.style.position = ''; // Remove absolute positioning
            dragSrcEl.style.zIndex = '';

            const touch = e.changedTouches[0];
            const targetElement = document.elementFromPoint(touch.clientX, touch.clientY);

            // Remove all highlightings
            document.querySelectorAll('.empty-letter-box').forEach(box => {
                box.classList.remove('border-blue-500', 'bg-blue-100');
            });

            if (targetElement && targetElement.classList.contains('empty-letter-box') && targetElement.textContent === '') {
                // Perform the drop logic
                const letter = dragSrcEl.getAttribute('data-letter');
                const originalScrambledIndex = parseInt(dragSrcEl.getAttribute('data-original-index'));
                const emptyBoxIndex = parseInt(targetElement.getAttribute('data-index'));

                targetElement.textContent = letter;
                targetElement.style.backgroundColor = '#4a5568';
                targetElement.style.color = '#ffffff';
                targetElement.style.border = 'none';

                dragSrcEl.style.display = 'none'; // Hide the original draggable letter

                placedLetters[emptyBoxIndex] = {
                    letter: letter,
                    originalScrambledIndex: originalScrambledIndex,
                    dragSrcEl: dragSrcEl // Keep reference
                };

                // Check for completion
                checkWord();
            } else {
                // If not dropped on a valid target, revert to original position
                if (dragData.originalParent) {
                    dragData.originalParent.insertBefore(dragSrcEl, dragData.originalParent.children[dragData.originalIndexInParent]);
                    dragSrcEl.style.left = '';
                    dragSrcEl.style.top = '';
                }
            }
            dragSrcEl = null; // Clear the dragged element reference
            dragData = {};
        }

        /**
         * Checks if the word has been correctly unscrambled.
         */
        function checkWord() {
            // Check if all empty boxes are filled
            if (placedLetters.includes(null)) {
                feedbackMessage.textContent = ""; // Clear message if not all boxes filled
                nextButton.disabled = true;
                retryButton.style.display = 'none'; // Hide retry button if not all boxes filled
                return;
            }

            // Construct the guessed word from placed letters
            const guessedWord = placedLetters.map(item => item.letter).join('').toLowerCase();

            if (guessedWord === currentWord.toLowerCase()) {
                feedbackMessage.textContent = "Correct! 🎉";
                feedbackMessage.classList.remove('incorrect');
                feedbackMessage.classList.add('correct');
                playSuccessSound(); // Play success sound
                nextButton.disabled = false; // Enable "Next" button
                retryButton.style.display = 'none'; // Hide retry button on success
            } else {
                feedbackMessage.textContent = "Incorrect! Try again. 🤔";
                feedbackMessage.classList.remove('correct');
                feedbackMessage.classList.add('incorrect');
                playErrorSound(); // Play error sound
                nextButton.disabled = true; // Keep "Next" button disabled
                retryButton.style.display = 'block'; // Show retry button on error
            }
        }

        /**
         * Resets the current puzzle, clearing placed letters and showing scrambled letters again.
         */
        function resetCurrentPuzzle() {
            // Clear feedback and hide retry button
            feedbackMessage.textContent = "";
            retryButton.style.display = 'none';

            // Clear empty boxes and reset their styles
            const emptyBoxes = emptyBoxesContainer.querySelectorAll('.empty-letter-box');
            emptyBoxes.forEach((box, index) => {
                box.textContent = '';
                box.style.backgroundColor = '#e2e8f0';
                box.style.color = '#4a5568';
                box.style.border = '2px dashed #a0aec0';
            });

            // Make all original scrambled letter boxes visible again
            const originalScrambledLetterBoxes = scrambledLettersContainer.querySelectorAll('.scrambled-letter-box');
            originalScrambledLetterBoxes.forEach(box => {
                box.style.display = 'flex'; // Restore visibility
            });

            // Reset placedLetters array
            placedLetters = new Array(currentWord.length).fill(null);

            // Ensure Next button is disabled until correctly solved
            nextButton.disabled = true;
        }


        /**
         * Moves to the next puzzle or restarts the game.
         */
        function goToNextPuzzle() {
            currentPuzzleIndex++;
            if (currentPuzzleIndex < verbPuzzles.length) {
                initializePuzzle();
            } else {
                // All puzzles completed, restart the game
                feedbackMessage.textContent = "Congratulations! You've completed all verbs. Play again! 🥳";
                currentPuzzleIndex = 0; // Reset for next round
                nextButton.textContent = "Restart Game";
                // Hide the next button temporarily and then re-initialize
                nextButton.disabled = true;
                setTimeout(() => {
                    nextButton.textContent = "Next"; // Reset text
                    initializePuzzle();
                }, 3000); // Give a moment to read the message
            }
        }

        // Event listener for the "Next" button
        nextButton.addEventListener('click', goToNextPuzzle);

        // Event listener for the new "Retry" button
        retryButton.addEventListener('click', resetCurrentPuzzle);

        // Initial setup when the page loads
        document.addEventListener('DOMContentLoaded', initializePuzzle);
    </script>
</body>
</html>
ELABORADO POR: MARIA PAZ PERALTA
