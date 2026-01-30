<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KYA KHAYEGA | AI Recipe Assistant</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Poppins:wght@300;400;600&display=swap');
        body { font-family: 'Poppins', sans-serif; background-color: #f7f3f0; }
        .card { box-shadow: 0 10px 25px rgba(0,0,0,0.1); border-radius: 1.5rem; background-color: white; }
        .btn-red { background-color: #ef4444; transition: 0.2s; }
        .btn-red:hover { background-color: #dc2626; transform: translateY(-2px); }
        .loader { border-top-color: #ef4444; animation: spin 1s linear infinite; }
        @keyframes spin { 0% { transform: rotate(0deg); } 100% { transform: rotate(360deg); } }
    </style>
</head>
<body class="flex items-center justify-center min-h-screen p-4">

    <div class="w-full max-w-2xl card p-8 mt-4">
        <h1 class="text-4xl font-bold text-gray-800 text-center mb-2">KYA KHAYEGA</h1>
        <p class="text-center text-gray-500 mb-8 uppercase text-sm tracking-widest">CHAL BATA KYA HAI KITCHEN MAI, MAI JAADU DIKHATA HU</p>

        <div class="mb-6">
            <label class="block text-lg font-medium text-gray-700 mb-2">
                TERAI PASS JO HAI LIKH DAI CHUP-CHAP {ex. ATTA, DAAL, CHINI...}
            </label>
            <textarea id="ingredients" rows="4" class="w-full p-4 border border-gray-300 rounded-xl focus:ring-2 focus:ring-red-400 outline-none transition" placeholder="rice, milk, sugar..."></textarea>
        </div>

        <div class="flex flex-col items-center">
            <button id="generateBtn" class="btn-red text-white font-semibold py-3 px-10 rounded-full shadow-lg">
                रेसिपी तैयार करें
            </button>
            
            <div id="loadingIndicator" class="hidden mt-6 flex items-center space-x-3">
                <div class="loader rounded-full border-4 border-gray-200 h-8 w-8"></div>
                <span class="text-red-600 font-medium">AJ KHANA TERA BHAI SET KAREGA...</span>
            </div>
            <p id="errorBox" class="text-red-500 hidden mt-4 font-semibold text-center"></p>
        </div>

        <div id="recipeOutput" class="mt-10 pt-6 border-t border-gray-200 hidden">
            <h2 class="text-2xl font-semibold text-gray-800 mb-4 text-center">💡 TERAI LIYA SUGGEST KIYA HAI PYAR SAI 😍</h2>
            <div id="recipeContent" class="prose max-w-none text-gray-700 leading-relaxed bg-gray-50 p-6 rounded-xl italic">
                </div>
        </div>
    </div>

    <script>
        // TECHNICAL CONFIGURATION
        const apiKey = "AIzaSyCnizsVWuX_jVxTuevKfVxXUTAuKmSta1k"; // Aapki nayi key
        const modelName = "gemini-1.5-flash"; // Correct stable model
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${modelName}:generateContent?key=${apiKey}`; // Fixed URL

        const ingredientsInput = document.getElementById('ingredients');
        const generateBtn = document.getElementById('generateBtn');
        const loadingIndicator = document.getElementById('loadingIndicator');
        const recipeOutput = document.getElementById('recipeOutput');
        const recipeContent = document.getElementById('recipeContent');
        const errorBox = document.getElementById('errorBox');

        generateBtn.addEventListener('click', async () => {
            const items = ingredientsInput.value.trim();
            if (!items) { alert("Pehle kuch items toh likho!"); return; }

            errorBox.classList.add('hidden');
            recipeOutput.classList.add('hidden');
            loadingIndicator.classList.remove('hidden');
            generateBtn.disabled = true;

            const promptText = `I have these ingredients: ${items}. Suggest a simple and tasty Indian recipe. Give the recipe name and short cooking steps in Hinglish language. Make it funny and cool.`;

            try {
                const response = await fetch(apiUrl, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{ parts: [{ text: promptText }] }]
                    })
                });

                const data = await response.json();
                
                if (data.candidates && data.candidates[0].content.parts[0].text) {
                    const finalRecipe = data.candidates[0].content.parts[0].text;
                    recipeContent.innerHTML = finalRecipe.replace(/\n/g, '<br>');
                    recipeOutput.classList.remove('hidden');
                } else {
                    throw new Error(data.error ? data.error.message : "Kuch gadbad ho gayi!");
                }
            } catch (err) {
                console.error("DEBUG INFO:", err);
                errorBox.innerText = "Error: " + err.message;
                errorBox.classList.remove('hidden');
            } finally {
                loadingIndicator.classList.add('hidden');
                generateBtn.disabled = false;
            }
        });
    </script>
</body>
</html>
