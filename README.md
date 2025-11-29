# Kitchen-Helper
Hi,this was my first GitHub project.AND I am writing code to make food with help of AI by your kichen item.


<!DOCTYPE html>
<html lang="hi">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>KYA BANAU</title>
    <!-- Tailwind CSS CDN -->
    <script src="https://cdn.tailwindcss.com"></script>
    <style>

        
            font-family: 'Poppins', sans-serif;
            background-color: #f7f3f0; /* हल्का बेज पृष्ठभूमि */
        }
        .card {
            box-shadow: 0 10px 25px -3px rgba(0, 0, 0, 0.1), 0 4px 6px -2px rgba(0, 0, 0, 0.05);
            border-radius: 1.5rem;
            background-color: white;
        }
        .btn-primary {
            background-color: #ef4444; /* लाल रंग */
            transition: transform 0.1s ease-in-out, background-color 0.1s;
        }
        .btn-primary:hover {
            background-color: #dc2626;
            transform: translateY(-1px);
        }
        .loader {
            border-top-color: #dc2626;
        }
    </style>
</head>
<body class="p-4 sm:p-8 min-h-screen flex items-start justify-center">

    <div class="w-full max-w-2xl mt-4 card p-6 sm:p-8">
        <!-- हेडर (Header) -->
        <h1 class="text-3xl sm:text-4xl font-bold text-gray-800 text-center mb-2">
           KYA KHAYEGA
        </h1>
        <p class="text-center text-gray-500 mb-8">
           CHAL BATA KYA HAI KITCHEN MAI,MAI JAADU DIKHATA HU
        </p>

        <!-- सामग्री इनपुट (Ingredients Input) -->
        <div class="mb-6">
            <label for="ingredients" class="block text-lg font-medium text-gray-700 mb-2">
               TERAI PASS JO HAI LIKH DAI CHUP-CHAP{ex. ATTA,DAAL,CHINI,NAKAM,OR SINGLE HU BHI CHAL SAKTA HAI😂}
            </label>
            <textarea id="ingredients" rows="4" class="w-full p-4 border border-gray-300 rounded-xl focus:ring-red-500 focus:border-red-500 transition duration-150" placeholder="चावल, दाल, आलू, हरी मिर्च, दही, नमक..."></textarea>
        </div>

        <!-- बटन और लोडिंग इंडिकेटर (Button and Loading Indicator) -->
        <div class="flex flex-col sm:flex-row items-center space-y-4 sm:space-y-0 sm:space-x-4">
            <button id="generateBtn" class="btn-primary text-white font-semibold py-3 px-6 rounded-xl w-full sm:w-auto shadow-lg hover:shadow-xl flex items-center justify-center">
                रेसिपी तैयार करें
            </button>
            <div id="loadingIndicator" class="hidden text-red-600 font-medium flex items-center space-x-2">
                <div class="loader ease-linear rounded-full border-4 border-t-4 border-gray-200 h-6 w-6"></div>
                <span>AJ KHANA TERA BHAI SET KAREGA..</span>
            </div>
            <p id="errorBox" class="text-red-500 hidden font-semibold mt-4"></p>
        </div>

        <!-- रेसिपी आउटपुट (Recipe Output) -->
        <div id="recipeOutput" class="mt-10 pt-6 border-t border-gray-200">
            <h2 class="text-2xl font-semibold text-gray-800 mb-4 hidden" id="outputHeader">
                💡 TERAI LIYA SUGGEST KIYA HAI PYAR SAI🤤
            </h2>
            <div id="recipeContent" class="prose max-w-none">
                <!-- रेसिपी यहाँ प्रदर्शित होगी -->
            </div>
        </div>
    </div>

    <script>
        // *******************************************************************
        // ******* आपकी API Key यहाँ डाल दी गई है (FINAL VERSION) *******
        // *******************************************************************
        const apiKey = "AIzaSyAEtLV55mMB3_PX5Ln3BFkdv8Dg3sApPyU"; 
        
        // बाकी कॉन्फ़िगरेशन
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'default-app-id';
        const modelName = 'gemini-2.5-flash-preview-09-2025';
        
        // API URL में Key को पैरामीटर के रूप में जोड़ा जा रहा है (सबसे विश्वसनीय तरीका)
        const apiUrl = `https://generativelanguage.googleapis.com/v1beta/models/${modelName}:generateContent?key=${apiKey}`; 

        const ingredientsInput = document.getElementById('ingredients');
        const generateBtn = document.getElementById('generateBtn');
        const loadingIndicator = document.getElementById('loadingIndicator');
        const recipeOutput = document.getElementById('recipeOutput');
        const recipeContent = document.getElementById('recipeContent');
        const outputHeader = document.getElementById('outputHeader');
        const errorBox = document.getElementById('errorBox');

        // लोडर CSS
        const style = document.createElement('style');
        style.textContent = `
            .loader {
                border-top-color: #ef4444;
                animation: spin 1s linear infinite;
            }
            @keyframes spin {
                0% { transform: rotate(0deg); }
                100% { transform: rotate(360deg); }
            }
        `;
        document.head.appendChild(style);


        // API कॉल के लिए एक्सपोनेंशियल बैकऑफ
        async function fetchWithExponentialBackoff(url, options, maxRetries = 5) {
            for (let i = 0; i < maxRetries; i++) {
                try {
                    const response = await fetch(url, options);

                    if (response.status === 429 && i < maxRetries - 1) {
                        const delay = Math.pow(2, i) * 1000 + Math.random() * 500;
                        await new Promise(resolve => setTimeout(resolve, delay));
                        continue; 
                    }

                    if (!response.ok) {
                        const errorText = await response.text();
                        const errorMessage = `API अनुरोध विफल: ${response.status} ${response.statusText}. विवरण: ${errorText}`;
                        console.error("API त्रुटि:", errorMessage);
                        throw new Error(errorMessage);
                    }
                    
                    return response; // सफल प्रतिक्रिया
                } catch (error) {
                    console.error('fetch के दौरान एक त्रुटि हुई:', error.message);
                    
                    if (i === maxRetries - 1) {
                        throw error; 
                    }
                    
                    const delay = Math.pow(2, i) * 1000 + Math.random() * 500;
                    await new Promise(resolve => setTimeout(resolve, delay));
                }
            }
        }

        async function generateRecipe() {
            // यदि Key गलती से हट गई हो तो चेक करें
            if (apiKey === "" || !apiKey.startsWith("AIzaSy")) {
                errorBox.textContent = "त्रुटि: API Key सही से सेट नहीं है।";
                errorBox.classList.remove('hidden');
                return;
            }

            const ingredients = ingredientsInput.value.trim();

            if (!ingredients) {
                errorBox.textContent = "कृपया अपनी सामग्री की सूची दर्ज करें।";
                errorBox.classList.remove('hidden');
                return;
            }

            // UI स्टेट अपडेट करें
            errorBox.classList.add('hidden');
            generateBtn.disabled = true;
            generateBtn.classList.add('opacity-50', 'cursor-not-allowed');
            loadingIndicator.classList.remove('hidden');
            recipeContent.innerHTML = '';
            outputHeader.classList.add('hidden');

            const systemPrompt = "आप एक रचनात्मक, सहायक शेफ (रसोई विशेषज्ञ) हैं। आपका काम उपयोगकर्ता द्वारा प्रदान की गई सामग्री के आधार पर एक स्वादिष्ट और बनाने में आसान व्यंजन (रेसिपी) का सुझाव देना है। प्रतिक्रिया हिंदी में होनी चाहिए। उत्तर को एक आकर्षक शीर्षक, 'आवश्यक सामग्री' (केवल वे जो उपयोगकर्ता ने प्रदान की हैं) की सूची, 'तैयारी का समय', और 'बनाने की विधि' के साथ विस्तृत निर्देशों के रूप में स्पष्ट रूप से संरचित करें। केवल उन्हीं व्यंजनों का सुझाव दें जो उपलब्ध सामग्री से बन सकें, या कम से कम उनमें 90% सामग्री उपलब्ध हो।";
            
            const userQuery = `मेरे पास ये सामग्री हैं: ${ingredients}। मुझे क्या बनाना चाहिए? मुझे एक विस्तृत रेसिपी चाहिए।`;

            const payload = {
                contents: [{ parts: [{ text: userQuery }] }],
                systemInstruction: {
                    parts: [{ text: systemPrompt }]
                },
            };

            const options = {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(payload)
            };

            try {
                const response = await fetchWithExponentialBackoff(apiUrl, options);
                const result = await response.json();
                
                const candidate = result.candidates?.[0];

                if (candidate && candidate.content?.parts?.[0]?.text) {
                    const text = candidate.content.parts[0].text;
                    recipeContent.innerHTML = formatMarkdownToHtml(text);
                    outputHeader.classList.remove('hidden');
                } else {
                    console.error("अपेक्षित API परिणाम संरचना नहीं मिली। पूर्ण प्रतिक्रिया:", result);
                    errorBox.textContent = "रेसिपी उत्पन्न करने में कोई समस्या आई। कृपया सामग्री की जाँच करें और पुनः प्रयास करें।";
                    errorBox.classList.remove('hidden');
                }

            } catch (error) {
                console.error('Gemini API कॉल त्रुटि (मुख्य कैच):', error);
                errorBox.textContent = `रेसिपी उत्पन्न करने में समस्या: नेटवर्क त्रुटि या सर्वर से अवैध प्रतिक्रिया। कृपया कंसोल (F12) में विवरण देखें।`;
                errorBox.classList.remove('hidden');
            } finally {
                generateBtn.disabled = false;
                generateBtn.classList.remove('opacity-50', 'cursor-not-allowed');
                loadingIndicator.classList.add('hidden');
            }
        }

        // मार्कडाउन को HTML में बदलने के लिए सरल फ़ंक्शन
        function formatMarkdownToHtml(markdown) {
            // बोल्ड, हेडिंग्स, और सूचियों को संभालने के लिए सरल परिवर्तन
            let html = markdown
                .replace(/^## (.*$)/gim, '<h3 class="text-xl font-semibold mt-4 mb-2 text-gray-700">$1</h3>') // H2 को H3 में
                .replace(/^# (.*$)/gim, '<h2 class="text-2xl font-bold mt-6 mb-3 text-red-600">$1</h2>') // H1 को H2 में
                .replace(/\*\*(.*?)\*\*/gim, '<strong>$1</strong>') // बोल्ड
                .replace(/^- (.*$)/gim, '<li>$1</li>') // बुलेटेड सूची आइटम
                .replace(/\* (.*$)/gim, '<li>$1</li>'); // बुलेटेड सूची आइटम (वैकल्पिक)

            // यदि सूची आइटम हैं, तो उन्हें ul टैग में लपेटें
            if (html.includes('<li>')) {
                html = html.replace(/(<li>.*<\/li>)/gs, '<ul class="list-disc pl-5 space-y-2 text-gray-600 mb-4">$1</ul>');
            }
            
            // सुनिश्चित करें कि लगातार बुलेट पॉइंट्स एक ही ul में रहें
            html = html.replace(/<\/ul>\s*<ul class="list-disc pl-5 space-y-2 text-gray-600 mb-4">/g, '');

            // पैराग्राफ को <p> टैग में लपेटें
            // यह थोड़ा जटिल है, लेकिन सरल टेक्स्ट लाइनों को पैराग्राफ में बदलता है
            html = html.split('\n').map(line => {
                line = line.trim();
                // उन लाइनों को अनदेखा करें जो पहले से ही किसी टैग का हिस्सा हैं
                if (line === '' || line.startsWith('<h') || line.startsWith('<ul') || line.startsWith('<li>') || line.startsWith('<strong>')) {
                    return line;
                }
                // अन्यथा, इसे एक पैराग्राफ के रूप में मानें
                return `<p class="mb-4 text-gray-600 leading-relaxed">${line}</p>`;
            }).join('\n');


            return html;
        }

        generateBtn.addEventListener('click', generateRecipe);
        ingredientsInput.addEventListener('keypress', function(e) {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                generateRecipe();
            }
        });
    </script>
</body>
</html>
