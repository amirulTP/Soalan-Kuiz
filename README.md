// Komponen utama aplikasi kuiz
function App() {
  // Set soalan kuiz awal berkaitan Faraid
  const initialQuizQuestions = [
    {
      question: 'Apakah faktor utama yang menentukan jumlah bahagian waris dalam faraid?',
      options: ['Usia waris', 'Hubungan waris dengan si mati', 'Status kewangan waris', ' Jenis pekerjaan waris'],
      correctAnswer: 'Hubungan waris dengan si mati',
    },
    {
      question: 'Seorang lelaki meninggal dunia dan meninggalkan seorang anak lelaki serta seorang anak perempuan. Bagaimanakah harta mereka akan dibahagikan?',
      options: ['Sama rata', 'Anak lelaki mendapat dua kali ganda daripada anak perempuan', 'Hanya anak lelaki menerima harta', 'Anak perempuan menerima lebih daripada anak lelaki'],
      correctAnswer: 'Anak lelaki mendapat dua kali ganda daripada anak perempuan',
    },
    {
      question: 'Jika si mati hanya meninggalkan isteri dan ibu, berapakah bahagian faraid yang diterima isteri?',
      options: ['1/2', '1/3', '1/4', '1/8'],
      correctAnswer: '1/4',
    },
    {
      question: 'Apakah maksud Asobah dalam hukum faraid?',
      options: ['Waris yang menerima harta secara mutlak', 'Waris yang menerima baki harta selepas pembahagian', 'Waris yang menerima harta secara tetap', 'Waris yang tidak layak menerima harta'],
      correctAnswer: ' Waris yang menerima baki harta selepas pembahagian',
    },
  ];

  // Pemboleh ubah keadaan untuk menguruskan aliran kuiz
  const [quizQuestions, setQuizQuestions] = useState(initialQuizQuestions); // Menyimpan semua soalan kuiz
  const [currentQuestionIndex, setCurrentQuestionIndex] = useState(0); // Mengesan soalan semasa
  const [score, setScore] = useState(0); // Mengesan skor pengguna
  const [showResult, setShowResult] = useState(false); // Mengawal paparan skrin keputusan
  const [selectedOption, setSelectedOption] = useState(null); // Menyimpan pilihan yang dipilih
  const [isAnswered, setIsAnswered] = useState(false); // Menyemak jika soalan semasa telah dijawab
  const [isLoadingQuestion, setIsLoadingQuestion] = useState(false); // Mengesan keadaan memuatkan untuk menjana soalan
  const [isLoadingExplanation, setIsLoadingExplanation] = useState(false); // Mengesan keadaan memuatkan untuk penjelasan
  const [isLoadingFaraidHelper, setIsLoadingFaraidHelper] = useState(false); // Mengesan keadaan memuatkan untuk pembantu Faraid
  const [error, setError] = useState(null); // Menyimpan sebarang mesej ralat
  const [explanation, setExplanation] = useState(''); // Menyimpan penjelasan jawapan
  const [userInputQuery, setUserInputQuery] = useState(''); // Menyimpan input pengguna untuk soalan Faraid
  const [faraidResponse, setFaraidResponse] = useState(''); // Menyimpan respons daripada pembantu Faraid

  // Mengendalikan pilihan jawapan
  const handleOptionSelect = (option) => {
    // Hanya benarkan pemilihan jika soalan belum dijawab
    if (!isAnswered) {
      setSelectedOption(option);
    }
  };

  // Mengendalikan semakan jawapan dan mengemas kini skor
  const handleCheckAnswer = () => {
    if (selectedOption === null) {
      // Halang semakan jika tiada pilihan dipilih
      return;
    }

    setIsAnswered(true); // Tandakan soalan sebagai dijawab

    // Semak jika pilihan yang dipilih betul dan kemas kini skor
    if (selectedOption === quizQuestions[currentQuestionIndex].correctAnswer) {
      setScore(score + 1);
    }
  };

  // Mengendalikan pergerakan ke soalan seterusnya
  const handleNextQuestion = () => {
    // Hanya teruskan jika soalan telah dijawab
    if (!isAnswered) {
      return;
    }

    // Jika ada lebih banyak soalan, bergerak ke soalan seterusnya
    if (currentQuestionIndex < quizQuestions.length - 1) {
      setCurrentQuestionIndex(currentQuestionIndex + 1);
      setSelectedOption(null); // Tetapkan semula pilihan yang dipilih untuk soalan seterusnya
      setIsAnswered(false); // Tetapkan semula keadaan dijawab untuk soalan seterusnya
      setExplanation(''); // Kosongkan penjelasan
    } else {
      // Jika semua soalan telah dijawab, paparkan skrin keputusan
      setShowResult(true);
    }
  };

  // Mengendalikan memulakan semula kuiz
  const handleRestartQuiz = () => {
    setCurrentQuestionIndex(0);
    setScore(0);
    setShowResult(false);
    setSelectedOption(null);
    setIsAnswered(false);
    setError(null); // Kosongkan sebarang ralat sebelumnya
    setExplanation(''); // Kosongkan penjelasan
    setUserInputQuery(''); // Kosongkan input carian
    setFaraidResponse(''); // Kosongkan respons pembantu Faraid
  };

  /**
   * Menjana soalan kuiz baharu menggunakan Gemini API.
   * Mengarahkan LLM untuk menyediakan soalan, empat pilihan, dan jawapan yang betul dalam format JSON.
   */
  const generateQuizQuestion = async () => {
    setIsLoadingQuestion(true); // Tetapkan keadaan memuatkan kepada benar
    setError(null); // Kosongkan sebarang ralat sebelumnya
    setExplanation(''); // Kosongkan penjelasan apabila menjana soalan baharu

    try {
      // Prompt untuk LLM menjana soalan kuiz dalam format JSON tertentu
      const prompt = `Generate a single multiple-choice quiz question about Faraid (Islamic inheritance law) in Malay.
                      The response should be in JSON format with the following structure:
                      {
                        "question": "The quiz question text",
                        "options": ["Option A", "Option B", "Option C", "Option D"],
                        "correctAnswer": "The correct option text"
                      }`;

      // Mulakan sejarah sembang dengan prompt pengguna
      let chatHistory = [];
      chatHistory.push({ role: "user", parts: [{ text: prompt }] });

      // Tentukan payload untuk permintaan Gemini API
      const payload = {
        contents: chatHistory,
        generationConfig: {
          responseMimeType: "application/json", // Minta output JSON
          responseSchema: { // Tentukan skema JSON yang dijangka
            type: "OBJECT",
            properties: {
              "question": { "type": "STRING" },
              "options": {
                "type": "ARRAY",
                "items": { "type": "STRING" }
              },
              "correctAnswer": { "type": "STRING" }
            },
            "propertyOrdering": ["question", "options", "correctAnswer"]
          }
        }
      };

      // Kunci API (string kosong untuk persekitaran Canvas untuk menyediakannya)
      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      // Buat panggilan API
      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      // Huraikan respons JSON
      const result = await response.json();

      // Semak jika respons mengandungi kandungan yang sah
      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        const jsonText = result.candidates[0].content.parts[0].text;
        const newQuestion = JSON.parse(jsonText);

        // Sahkan struktur soalan yang dijana
        if (newQuestion.question && Array.isArray(newQuestion.options) &&
            newQuestion.options.length === 4 && newQuestion.correctAnswer) {
          // Tambah soalan baharu ke senarai soalan kuiz
          setQuizQuestions((prevQuestions) => [...prevQuestions, newQuestion]);
          // Tetapkan semula kuiz untuk bermula dengan soalan yang baru ditambah, atau tunjukkan soalan baharu jika ia satu-satunya
          setCurrentQuestionIndex(quizQuestions.length); // Pergi ke soalan baharu (terakhir dalam tatasusunan)
          setScore(0);
          setShowResult(false);
          setSelectedOption(null);
          setIsAnswered(false);
        } else {
          setError("Gagal menjana soalan kuiz yang sah. Sila cuba lagi.");
          console.error("Struktur soalan tidak sah dari LLM:", newQuestion);
        }
      } else {
        setError("Gagal mendapatkan respons daripada AI. Sila cuba lagi.");
        console.error("Struktur respons LLM tidak dijangka:", result);
      }
    } catch (err) {
      setError(`Ralat menjana soalan: ${err.message || 'Ralat yang tidak dijangka berlaku.'}`);
      console.error("Ralat memanggil Gemini API untuk soalan:", err);
    } finally {
      setIsLoadingQuestion(false); // Tamatkan keadaan memuatkan
    }
  };

  /**
   * Menjana penjelasan untuk jawapan yang betul menggunakan Gemini API.
   */
  const handleExplainAnswer = async () => {
    setIsLoadingExplanation(true);
    setError(null);
    setExplanation(''); // Kosongkan penjelasan sebelum menjana yang baharu

    const currentQuestion = quizQuestions[currentQuestionIndex];
    if (!currentQuestion) {
      setError("Tiada soalan untuk dijelaskan.");
      setIsLoadingExplanation(false);
      return;
    }

    try {
      const prompt = `Terangkan secara ringkas mengapa jawapan "${currentQuestion.correctAnswer}" adalah betul untuk soalan "${currentQuestion.question}" dalam konteks Faraid (undang-undang pewarisan Islam) dalam Bahasa Melayu.`;

      let chatHistory = [];
      chatHistory.push({ role: "user", parts: [{ text: prompt }] });

      const payload = {
        contents: chatHistory,
        generationConfig: {
          responseMimeType: "text/plain", // Minta output teks biasa
        }
      };

      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();

      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        setExplanation(result.candidates[0].content.parts[0].text);
      } else {
        setError("Gagal mendapatkan penjelasan daripada AI. Sila cuba lagi.");
        console.error("Unexpected LLM response structure for explanation:", result);
      }
    } catch (err) {
      setError(`Ralat menjana penjelasan: ${err.message || 'Ralat yang tidak dijangka berlaku.'}`);
      console.error("Ralat memanggil Gemini API untuk penjelasan:", err);
    } finally {
      setIsLoadingExplanation(false);
    }
  };

  /**
   * Menjana respons kepada pertanyaan umum tentang Faraid menggunakan Gemini API.
   */
  const handleGetFaraidAnswer = async () => {
    setIsLoadingFaraidHelper(true);
    setError(null);
    setFaraidResponse(''); // Kosongkan respons sebelum menjana yang baharu

    if (!userInputQuery.trim()) {
      setError("Sila masukkan soalan anda tentang Faraid.");
      setIsLoadingFaraidHelper(false);
      return;
    }

    try {
      // Prompt LLM untuk menjawab soalan berkaitan Faraid
      const prompt = `Jawab soalan ini dalam Bahasa Melayu, dalam konteks Faraid (undang-undang pewarisan Islam) secara ringkas dan padat: "${userInputQuery}"`;

      let chatHistory = [];
      chatHistory.push({ role: "user", parts: [{ text: prompt }] });

      const payload = {
        contents: chatHistory,
        generationConfig: {
          responseMimeType: "text/plain", // Minta output teks biasa
        }
      };

      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();

      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        setFaraidResponse(result.candidates[0].content.parts[0].text);
      } else {
        setError("Gagal mendapatkan jawapan daripada AI. Sila cuba lagi.");
        console.error("Unexpected LLM response structure for general Faraid question:", result);
      }
    } catch (err) {
      setError(`Ralat menjana jawapan: ${err.message || 'Ralat yang tidak dijangka berlaku.'}`);
      console.error("Ralat memanggil Gemini API untuk soalan umum Faraid:", err);
    } finally {
      setIsLoadingFaraidHelper(false);
    }
  };

  /**
   * Menjana senario latihan Faraid berdasarkan input pengguna.
   */
  const handleGenerateFaraidScenario = async () => {
    setIsLoadingFaraidHelper(true);
    setError(null);
    setFaraidResponse(''); // Kosongkan respons sebelum menjana yang baharu

    if (!userInputQuery.trim()) {
      setError("Sila masukkan topik untuk senario latihan Faraid.");
      setIsLoadingFaraidHelper(false);
      return;
    }

    try {
      const prompt = `Jana satu senario latihan Faraid yang ringkas berdasarkan topik ini: "${userInputQuery}". Senario ini haruslah mencabar pemahaman tentang pembahagian harta dalam konteks Islam. Berikan senario dan bukan jawapan. Tulis dalam Bahasa Melayu.`;

      let chatHistory = [];
      chatHistory.push({ role: "user", parts: [{ text: prompt }] });

      const payload = {
        contents: chatHistory,
        generationConfig: {
          responseMimeType: "text/plain",
        }
      };

      const apiKey = "";
      const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/gemini-2.0-flash:generateContent?key=${apiKey}`;

      const response = await fetch(apiUrl, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify(payload)
      });

      const result = await response.json();

      if (result.candidates && result.candidates.length > 0 &&
          result.candidates[0].content && result.candidates[0].content.parts &&
          result.candidates[0].content.parts.length > 0) {
        setFaraidResponse(result.candidates[0].content.parts[0].text);
      } else {
        setError("Gagal menjana senario latihan. Sila cuba lagi.");
        console.error("Unexpected LLM response structure for scenario generation:", result);
      }
    } catch (err) {
      setError(`Ralat menjana senario: ${err.message || 'Ralat yang tidak dijangka berlaku.'}`);
      console.error("Ralat memanggil Gemini API untuk senario:", err);
    } finally {
      setIsLoadingFaraidHelper(false);
    }
  };

  // Paparkan aplikasi kuiz
  return (
    <div className="min-h-screen flex items-center justify-center bg-gray-100 p-4 font-sans">
      <div className="bg-white rounded-lg shadow-xl p-6 w-full max-w-md">
        <h1 className="text-3xl font-bold text-center text-indigo-700 mb-6">Kuiz Pemahaman Faraid</h1>

        {error && (
          <div className="bg-red-100 border border-red-400 text-red-700 px-4 py-3 rounded relative mb-4" role="alert">
            <span className="block sm:inline">{error}</span>
          </div>
        )}

        {showResult ? (
          // Paparkan skrin keputusan jika kuiz tamat
          <ResultScreen score={score} totalQuestions={quizQuestions.length} onRestart={handleRestartQuiz} />
        ) : (
          // Paparkan kad soalan jika kuiz sedang berjalan
          <QuestionCard
            question={quizQuestions[currentQuestionIndex]}
            currentQuestionNumber={currentQuestionIndex + 1}
            totalQuestions={quizQuestions.length}
            selectedOption={selectedOption}
            onOptionSelect={handleOptionSelect}
            onCheckAnswer={handleCheckAnswer}
            onNextQuestion={handleNextQuestion}
            isAnswered={isAnswered}
            onExplainAnswer={handleExplainAnswer} // Hantar fungsi penjelasan
            explanation={explanation} // Hantar penjelasan
            isLoadingExplanation={isLoadingExplanation} // Hantar status memuatkan penjelasan
          />
        )}

        {/* Bahagian Pembantu Faraid */}
        <div className="mt-8 pt-6 border-t-2 border-gray-200">
          <h2 className="text-2xl font-bold text-indigo-700 mb-4 text-center">Tanya je apa apa tentang Faraid</h2>
          <div className="flex flex-col space-y-4">
            <input
              type="text"
              value={userInputQuery}
              onChange={(e) => setUserInputQuery(e.target.value)}
              placeholder="Tulis soalan atau topik senario anda di sini..."
              className="p-3 border border-gray-300 rounded-lg focus:outline-none focus:ring-2 focus:ring-indigo-500"
            />
            <div className="flex justify-between space-x-2">
              <button
                onClick={handleGetFaraidAnswer}
                disabled={isLoadingFaraidHelper || !userInputQuery.trim()}
                className={`
                  flex-1 px-4 py-3 rounded-lg font-bold text-white transition duration-300 ease-in-out
                  ${isLoadingFaraidHelper || !userInputQuery.trim()
                    ? 'bg-gray-400 cursor-not-allowed'
                    : 'bg-green-600 hover:bg-green-700 focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50'
                  }
                `}
              >
                {isLoadingFaraidHelper ? 'Mencari... ' : 'Dapatkan Jawapan ✨'}
              </button>
              <button
                onClick={handleGenerateFaraidScenario}
                disabled={isLoadingFaraidHelper || !userInputQuery.trim()}
                className={`
                  flex-1 px-4 py-3 rounded-lg font-bold text-white transition duration-300 ease-in-out
                  ${isLoadingFaraidHelper || !userInputQuery.trim()
                    ? 'bg-yellow-400 cursor-not-allowed'
                    : 'bg-yellow-600 hover:bg-yellow-700 focus:outline-none focus:ring-2 focus:ring-yellow-500 focus:ring-opacity-50'
                  }
                `}
              >
                {isLoadingFaraidHelper ? 'Menjana... ' : 'Jana Senario Latihan ✨'}
              </button>
            </div>
          </div>
          {faraidResponse && (
            <div className="mt-4 p-4 bg-lime-50 border border-lime-200 rounded-lg text-lime-800">
              <h3 className="font-semibold mb-2">Respons:</h3>
              <p>{faraidResponse}</p>
            </div>
          )}
        </div>
      </div>
    </div>
  );
}

// Komponen QuestionCard untuk memaparkan satu soalan
function QuestionCard({
  question,
  currentQuestionNumber,
  totalQuestions,
  selectedOption,
  onOptionSelect,
  onCheckAnswer,
  onNextQuestion,
  isAnswered,
  onExplainAnswer, // Terima fungsi penjelasan
  explanation, // Terima penjelasan
  isLoadingExplanation, // Terima status memuatkan penjelasan
}) {
  return (
    <div>
      <div className="text-sm text-gray-600 mb-4 text-center">
        Soalan {currentQuestionNumber} daripada {totalQuestions}
      </div>
      <h2 className="text-xl font-semibold mb-6 text-gray-800 text-center">
        {question.question}
      </h2>
      <div className="space-y-3 mb-6">
        {question.options.map((option) => (
          <button
            key={option}
            onClick={() => onOptionSelect(option)}
            disabled={isAnswered} // Lumpuhkan butang setelah dijawab
            className={`
              w-full p-3 rounded-lg border-2 text-left transition duration-300 ease-in-out
              ${selectedOption === option
                ? (isAnswered && option === question.correctAnswer
                  ? 'bg-green-100 border-green-500 text-green-800' // Dipilih dengan betul
                  : isAnswered && option !== question.correctAnswer
                    ? 'bg-red-100 border-red-500 text-red-800' // Dipilih dengan salah
                    : 'bg-indigo-50 border-indigo-500 text-indigo-700') // Dipilih tetapi belum disemak
                : (isAnswered && option === question.correctAnswer
                  ? 'bg-green-100 border-green-500 text-green-800' // Jawapan betul dipaparkan selepas semakan
                  : 'bg-white border-gray-300 text-gray-700 hover:bg-gray-50') // Lalai atau tidak disemak
              }
              ${isAnswered ? 'cursor-not-allowed' : 'cursor-pointer'}
            `}
          >
            {option}
          </button>
        ))}
      </div>
      <div className="flex justify-center space-x-4">
        {!isAnswered ? (
          <button
            onClick={onCheckAnswer}
            disabled={selectedOption === null} // Lumpuhkan butang semak jika tiada pilihan dipilih
            className={`
              px-6 py-3 rounded-lg font-bold text-white transition duration-300 ease-in-out
              ${selectedOption === null
                ? 'bg-gray-400 cursor-not-allowed'
                : 'bg-indigo-600 hover:bg-indigo-700 focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50'
              }
            `}
          >
            Semak Jawapan
          </button>
        ) : (
          <>
            <button
              onClick={onNextQuestion}
              className="px-6 py-3 rounded-lg font-bold text-white bg-green-600 hover:bg-green-700 transition duration-300 ease-in-out focus:outline-none focus:ring-2 focus:ring-green-500 focus:ring-opacity-50"
            >
              {currentQuestionNumber === totalQuestions ? 'Lihat Keputusan' : 'Soalan Seterusnya'}
            </button>
            <button
              onClick={onExplainAnswer}
              disabled={isLoadingExplanation}
              className={`
                px-6 py-3 rounded-lg font-bold text-white transition duration-300 ease-in-out
                ${isLoadingExplanation
                  ? 'bg-blue-400 cursor-not-allowed'
                  : 'bg-blue-600 hover:bg-blue-700 focus:outline-none focus:ring-2 focus:ring-blue-500 focus:ring-opacity-50'
                }
              `}
            >
              {isLoadingExplanation ? 'Menjana Penjelasan... ' : 'Terangkan Jawapan ✨'}
            </button>
          </>
        )}
      </div>

      {explanation && (
        <div className="mt-6 p-4 bg-blue-50 border border-blue-200 rounded-lg text-blue-800">
          <h3 className="font-semibold mb-2">Penjelasan:</h3>
          <p>{explanation}</p>
        </div>
      )}
    </div>
  );
}

// Komponen ResultScreen untuk memaparkan skor akhir
function ResultScreen({ score, totalQuestions, onRestart }) {
  const percentageCorrect = (score / totalQuestions) * 100;
  const incorrectAnswers = totalQuestions - score;
  const percentageIncorrect = (incorrectAnswers / totalQuestions) * 100;

  return (
    <div className="text-center">
      <h2 className="text-3xl font-bold text-indigo-700 mb-4">Kuiz Selesai!</h2>
      <p className="text-xl text-gray-800 mb-2">
        Anda menjawab {score} daripada {totalQuestions} soalan dengan betul.
      </p>
      <p className="text-2xl font-semibold text-green-600 mb-2">
        Peratusan Betul: {percentageCorrect.toFixed(0)}%
      </p>
      <p className="text-2xl font-semibold text-red-600 mb-6">
        Peratusan Salah: {percentageIncorrect.toFixed(0)}%
      </p>
      <button
        onClick={onRestart}
        className="px-8 py-4 rounded-lg font-bold text-white bg-indigo-600 hover:bg-indigo-700 transition duration-300 ease-in-out focus:outline-none focus:ring-2 focus:ring-indigo-500 focus:ring-opacity-50 shadow-lg"
      >
        Mula Semula Kuiz
      </button>
    </div>
  );
}
