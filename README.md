<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>MCQ Quiz</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            display: flex;
            justify-content: center;
            align-items: center;
            min-height: 100vh;
            margin: 0;
            background-color: #f0f0f0;
        }
        .quiz-container {
            background: white;
            padding: 20px;
            border-radius: 10px;
            box-shadow: 0 0 10px rgba(0,0,0,0.1);
            width: 90%;
            max-width: 600px;
        }
        h1, h2, h3 {
            text-align: center;
            color: #333;
        }
        .question, .add-question {
            margin-bottom: 20px;
        }
        .options label {
            display: block;
            margin: 10px 0;
            padding: 10px;
            background: #f9f9f9;
            border-radius: 5px;
            cursor: pointer;
        }
        .options input {
            margin-right: 10px;
        }
        button {
            display: block;
            width: 100%;
            padding: 10px;
            background: #007BFF;
            color: white;
            border: none;
            border-radius: 5px;
            cursor: pointer;
            font-size: 16px;
            margin-top: 10px;
        }
        button:hover {
            background: #0056b3;
        }
        #result, #timer {
            margin-top: 20px;
            text-align: center;
            font-weight: bold;
        }
        #language-toggle {
            margin-bottom: 20px;
        }
        .add-question input, .add-question select {
            width: 100%;
            padding: 8px;
            margin: 5px 0;
            border: 1px solid #ccc;
            border-radius: 5px;
        }
    </style>
</head>
<body>
    <div class="quiz-container">
        <h1>MCQ Quiz / एमसीक्यू क्विज</h1>
        <div id="language-toggle">
            <label>Language / भाषा: </label>
            <select id="language" onchange="changeLanguage()">
                <option value="en">English</option>
                <option value="hi">हिंदी</option>
            </select>
        </div>
        <div class="add-question">
            <h2>Add Question / प्रश्न जोड़ें</h2>
            <input type="text" id="question-en" placeholder="Enter question in English">
            <input type="text" id="question-hi" placeholder="प्रश्न हिंदी में दर्ज करें">
            <input type="text" id="option1-en" placeholder="Option 1 (English)">
            <input type="text" id="option1-hi" placeholder="विकल्प 1 (हिंदी)">
            <input type="text" id="option2-en" placeholder="Option 2 (English)">
            <input type="text" id="option2-hi" placeholder="विकल्प 2 (हिंदी)">
            <input type="text" id="option3-en" placeholder="Option 3 (English)">
            <input type="text" id="option3-hi" placeholder="विकल्प 3 (हिंदी)">
            <input type="text" id="option4-en" placeholder="Option 4 (English)">
            <input type="text" id="option4-hi" placeholder="विकल्प 4 (हिंदी)">
            <select id="correct-answer">
                <option value="">Select correct answer / सही उत्तर चुनें</option>
                <option value="0">Option 1 / विकल्प 1</option>
                <option value="1">Option 2 / विकल्प 2</option>
                <option value="2">Option 3 / विकल्प 3</option>
                <option value="3">Option 4 / विकल्प 4</option>
            </select>
            <button onclick="addQuestion()">Add Question / प्रश्न जोड़ें</button>
        </div>
        <div>
            <label>Upload PDF / पीडीएफ अपलोड करें:</label>
            <input type="file" id="pdf-upload" accept="application/pdf">
        </div>
        <div id="timer">Time Remaining: 0s</div>
        <div id="quiz">
            <!-- Questions will be added dynamically -->
        </div>
        <button onclick="submitQuiz()">Submit / जमा करें</button>
        <div id="result"></div>
    </div>

    <script src="https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.10.377/pdf.min.js"></script>
    <script>
        pdfjsLib.GlobalWorkerOptions.workerSrc = 'https://cdnjs.cloudflare.com/ajax/libs/pdf.js/2.10.377/pdf.worker.min.js';

        let questions = [];
        let currentQuestion = 0;
        let score = 0;
        let timer;
        let timeLeft = 60;
        let language = 'en';
        let userAnswers = [];

        // Sample questions (for testing)
        const sampleQuestions = [
            {
                en: { question: "What is the capital of India?", options: ["Mumbai", "Delhi", "Kolkata", "Chennai"], correct: "Delhi" },
                hi: { question: "भारत की राजधानी क्या है?", options: ["मुंबई", "दिल्ली", "कोलकाता", "चेन्नई"], correct: "दिल्ली" }
            },
            {
                en: { question: "What is the color of the sun?", options: ["Red", "Blue", "Yellow", "Green"], correct: "Yellow" },
                hi: { question: "सूर्य का रंग क्या है?", options: ["लाल", "नीला", "पीला", "हरा"], correct: "पीला" }
            }
        ];

        // Add question manually
        function addQuestion() {
            const questionEn = document.getElementById('question-en').value;
            const questionHi = document.getElementById('question-hi').value;
            const optionsEn = [
                document.getElementById('option1-en').value,
                document.getElementById('option2-en').value,
                document.getElementById('option3-en').value,
                document.getElementById('option4-en').value
            ];
            const optionsHi = [
                document.getElementById('option1-hi').value,
                document.getElementById('option2-hi').value,
                document.getElementById('option3-hi').value,
                document.getElementById('option4-hi').value
            ];
            const correctIndex = document.getElementById('correct-answer').value;

            if (questionEn && questionHi && optionsEn.every(opt => opt) && optionsHi.every(opt => opt) && correctIndex !== "") {
                questions.push({
                    en: { question: questionEn, options: optionsEn, correct: optionsEn[correctIndex] },
                    hi: { question: questionHi, options: optionsHi, correct: optionsHi[correctIndex] }
                });
                alert(language === 'en' ? 'Question added!' : 'प्रश्न जोड़ा गया!');
                loadQuiz();
                // Clear form
                document.getElementById('question-en').value = '';
                document.getElementById('question-hi').value = '';
                for (let i = 1; i <= 4; i++) {
                    document.getElementById(`option${i}-en`).value = '';
                    document.getElementById(`option${i}-hi`).value = '';
                }
                document.getElementById('correct-answer').value = '';
            } else {
                alert(language === 'en' ? 'Please fill all fields!' : 'कृपया सभी क्षेत्र भरें!');
            }
        }

        // Load PDF and extract questions
        document.getElementById('pdf-upload').addEventListener('change', async function(e) {
            const file = e.target.files[0];
            if (file) {
                const arrayBuffer = await file.arrayBuffer();
                const pdf = await pdfjsLib.getDocument(arrayBuffer).promise;
                let text = '';

                for (let i = 1; i <= pdf.numPages; i++) {
                    const page = await pdf.getPage(i);
                    const content = await page.getTextContent();
                    text += content.items.map(item => item.str).join(' ') + '\n';
                }

                questions = parsePDFText(text);
                loadQuiz();
                startTimer();
            }
        });

        // Parse PDF text to questions
        function parsePDFText(text) {
            const lines = text.split('\n');
            const parsedQuestions = [];
            let currentQuestion = null;

            lines.forEach(line => {
                if (line.startsWith('Q')) {
                    currentQuestion = {
                        en: { question: '', options: [], correct: '' },
                        hi: { question: '', options: [], correct: '' }
                    };
                    const [enQ, hiQ] = line.split('(');
                    currentQuestion.en.question = enQ.replace('Q', '').trim();
                    currentQuestion.hi.question = hiQ ? hiQ.replace(')', '').trim() : enQ.replace('Q', '').trim();
                } else if (line.match(/^[A-D]\)/)) {
                    const [enOpt, hiOpt] = line.split('(');
                    currentQuestion.en.options.push(enOpt.replace(/[A-D]\)/, '').trim());
                    currentQuestion.hi.options.push(hiOpt ? hiOpt.replace(')', '').trim() : enOpt.replace(/[A-D]\)/, '').trim());
                } else if (line.startsWith('Correct:')) {
                    const [enCorr, hiCorr] = line.replace('Correct:', '').split('(');
                    currentQuestion.en.correct = enCorr.trim();
                    currentQuestion.hi.correct = hiCorr ? hiCorr.replace(')', '').trim() : enCorr.trim();
                    parsedQuestions.push(currentQuestion);
                }
            });

            return parsedQuestions.length ? parsedQuestions : sampleQuestions;
        }

        // Load quiz
        function loadQuiz() {
            const quizContainer = document.getElementById('quiz');
            quizContainer.innerHTML = '';

            questions.forEach((q, index) => {
                const questionDiv = document.createElement('div');
                questionDiv.classList.add('question');
                questionDiv.innerHTML = `<h3>${index + 1}. ${q[language].question}</h3>`;
                
                const optionsDiv = document.createElement('div');
                optionsDiv.classList.add('options');
                
                q[language].options.forEach(option => {
                    optionsDiv.innerHTML += `
                        <label>
                            <input type="radio" name="question${index}" value="${option}">
                            ${option}
                        </label>`;
                });
                
                questionDiv.appendChild(optionsDiv);
                quizContainer.appendChild(questionDiv);
            });
        }

        // Timer
        function startTimer() {
            clearInterval(timer);
            timeLeft = 60 * questions.length;
            document.getElementById('timer').innerText = `Time Remaining: ${timeLeft}s`;
            timer = setInterval(() => {
                timeLeft--;
                document.getElementById('timer').innerText = `Time Remaining: ${timeLeft}s`;
                if (timeLeft <= 0) {
                    clearInterval(timer);
                    submitQuiz();
                }
            }, 1000);
        }

        // Submit quiz and show progress report
        function submitQuiz() {
            clearInterval(timer);
            score = 0;
            userAnswers = [];

            questions.forEach((q, index) => {
                const selectedOption = document.querySelector(`input[name="question${index}"]:checked`);
                const userAnswer = selectedOption ? selectedOption.value : null;
                userAnswers.push({ question: q[language].question, userAnswer, correct: q[language].correct });
                if (userAnswer === q[language].correct) {
                    score++;
                }
            });

            let report = language === 'en' ? `<h3>Quiz Results</h3>` : `<h3>क्विज़ परिणाम</h3>`;
            report += `<p>${language === 'en' ? 'Score' : 'स्कोर'}: ${score}/${questions.length}</p>`;
            report += `<h4>${language === 'en' ? 'Detailed Analysis' : 'विस्तृत विश्लेषण'}</h4>`;
            userAnswers.forEach((ans, i) => {
                report += `<p><b>${i + 1}. ${ans.question}</b><br>`;
                report += `${language === 'en' ? 'Your Answer' : 'आपका उत्तर'}: ${ans.userAnswer || 'None'}<br>`;
                report += `${language === 'en' ? 'Correct Answer' : 'सही उत्तर'}: ${ans.correct}<br>`;
                report += `${ans.userAnswer === ans.correct ? '✅ Correct' : '❌ Incorrect'}</p>`;
            });

            document.getElementById('result').innerHTML = report;
        }

        // Change language
        function changeLanguage() {
            language = document.getElementById('language').value;
            loadQuiz();
        }

        // Load sample quiz initially
        questions = sampleQuestions;
        loadQuiz();
        startTimer();
    </script>
</body>
</html># Quiz-type
