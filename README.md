<!DOCTYPE html>
<html lang="he" dir="rtl">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>נחום Translate - StackBlitz Edition</title>
    <script src="https://cdn.jsdelivr.net/npm/@tailwindcss/browser@4"></script>
    <style>
        canvas {
            position: fixed;
            top: 0;
            left: 0;
            width: 100vw;
            height: 100vh;
            z-index: -1;
            pointer-events: none;
        }
    </style>
</head>
<body class="bg-slate-950 text-slate-100 min-h-screen flex items-center justify-center p-4">

    <canvas id="bgCanvas"></canvas>

    <div class="w-full max-w-xl bg-slate-900/90 backdrop-blur-md border border-slate-800 p-6 md:p-8 rounded-2xl shadow-2xl relative z-10">
        <div class="flex justify-between items-center mb-6">
            <div>
                <h1 class="text-3xl font-black text-sky-400 mb-1 flex items-center gap-2">
                    <span>🤖</span> נחום Translate
                </h1>
                <p class="text-slate-400 text-sm">הדבק כאן הודעת נחום, ותקבל סיכום קצר, מדויק ותכל'סי</p>
            </div>
        </div>

        <div class="space-y-4">
            <div>
                <textarea 
                    id="userInput" 
                    placeholder="הדבק לכאן את הודעת נחום הארוכה..."
                    class="w-full h-36 bg-slate-950/60 border border-slate-700 rounded-xl p-4 text-slate-100 placeholder-slate-500 focus:outline-none focus:border-sky-400 focus:ring-1 focus:ring-sky-400 transition resize-none text-base"
                ></textarea>
            </div>

            <button 
                onclick="translateText()" 
                id="translateBtn"
                class="w-full bg-sky-500 hover:bg-sky-400 text-slate-950 font-bold py-3.5 px-6 rounded-xl transition duration-200 shadow-lg shadow-sky-500/20 active:scale-[0.99] cursor-pointer text-lg"
            >
                תביא לי את התכל'ס
            </button>

            <div class="mt-6 pt-4 border-t border-slate-800">
                <div class="text-xs font-semibold text-slate-400 mb-2 uppercase tracking-wider">השורה התחתונה (תכל'ס):</div>
                <div 
                    id="resultArea" 
                    class="w-full min-h-[80px] bg-slate-950/80 border border-slate-800 rounded-xl p-4 text-slate-400 whitespace-pre-wrap leading-relaxed text-base flex items-center justify-center text-center"
                >
                    התשובה תופיע כאן...
                </div>
            </div>
        </div>
    </div>

    <script>
        const canvas = document.getElementById('bgCanvas');
        const ctx = canvas.getContext('2d');

        function resizeCanvas() {
            canvas.width = window.innerWidth;
            canvas.height = window.innerHeight;
        }
        window.addEventListener('resize', resizeCanvas);
        resizeCanvas();

        const particles = Array.from({ length: 40 }, () => ({
            x: Math.random() * canvas.width,
            y: Math.random() * canvas.height,
            size: Math.random() * 2 + 1,
            speedX: (Math.random() - 0.5) * 0.5,
            speedY: (Math.random() - 0.5) * 0.5,
            opacity: Math.random() * 0.5 + 0.2
        }));

        function animate() {
            ctx.clearRect(0, 0, canvas.width, canvas.height);
            ctx.fillStyle = '#0f172a';
            ctx.fillRect(0, 0, canvas.width, canvas.height);

            particles.forEach(p => {
                p.x += p.speedX;
                p.y += p.speedY;

                if (p.x < 0) p.x = canvas.width;
                if (p.x > canvas.width) p.x = 0;
                if (p.y < 0) p.y = canvas.height;
                if (p.y > canvas.height) p.y = 0;

                ctx.beginPath();
                ctx.arc(p.x, p.y, p.size, 0, Math.PI * 2);
                ctx.fillStyle = `rgba(56, 189, 248, ${p.opacity})`;
                ctx.fill();
            });

            requestAnimationFrame(animate);
        }
        animate();

        async function translateText() {
            const text = document.getElementById('userInput').value.trim();
            const resultArea = document.getElementById('resultArea');
            const btn = document.getElementById('translateBtn');

            if (!text) {
                resultArea.innerText = 'נא להדביק הודעה תחילה.';
                resultArea.className = 'w-full min-h-[80px] bg-slate-950/80 border border-slate-800 rounded-xl p-4 text-amber-400 whitespace-pre-wrap leading-relaxed text-base flex items-center justify-center text-center';
                return;
            }

            btn.disabled = true;
            btn.innerText = 'מנקה חפירות...';
            resultArea.innerText = 'חושב...';
            resultArea.className = 'w-full min-h-[80px] bg-slate-950/80 border border-slate-800 rounded-xl p-4 text-amber-400 whitespace-pre-wrap leading-relaxed text-base flex items-center justify-center text-center';

            const apiKey = "AQ.Ab8RN6Kt9LzSL27CfvajSBHgdSMXSQBUU7yfOXgoT0aanJRaTg";
            const url = `https://generativelanguage.googleapis.com/v1beta/models/gemini-3-flash-preview:generateContent`;

            const systemPrompt = `אתה "נחום Translate" - כלי קצה אגרסיבי ומהיר שממיר הודעות ארוכות ומלאות בחפירות להודעה קצרה, ישירה וברורה בפורמט "תכל'ס" נטו.

חוקי ברזל חובה:
1. אסור לחלוטין לכתוב מבואות, משפטי פתיחה או סיכומים כמו "הכוונה היא", "לסיכום", מרכאות או כותרות. פלט את התשובה הישירה בלבד.
2. תמצת כל הודעה למשפטים קצרים וקולעים בשורות נפרדות אם יש מספר נתונים.
3. שמור בדיוק על הסגנון והשפה של כותב ההודעה המקורית אך ללא שום מילים מיותרות.
4. הוצא אך ורק את השורה התחתונה המעשית המדויקת.`;

            try {
                const response = await fetch(url, {
                    method: 'POST',
                    headers: { 
                        'Content-Type': 'application/json',
                        'x-goog-api-key': apiKey
                    },
                    body: JSON.stringify({
                        contents: [{
                            role: 'user',
                            parts: [{
                                text: systemPrompt + "\n\nהודעה לתרגום:\n" + text
                            }]
                        }],
                        generationConfig: { 
                            temperature: 0.1, 
                            maxOutputTokens: 2048 
                        }
                    })
                });

                const data = await response.json();
                if (!response.ok) {
                    throw new Error(data.error?.message || 'שגיאת תקשורת עם המודל.');
                }

                const result = data.candidates?.[0]?.content?.parts?.[0]?.text;
                if (result) {
                    resultArea.innerText = result.trim();
                    resultArea.className = 'w-full min-h-[80px] bg-slate-950/80 border border-slate-800 rounded-xl p-4 text-emerald-400 whitespace-pre-wrap leading-relaxed text-base text-right';
                } else {
                    throw new Error('לא התקבלה תשובה תקינה');
                }
            } catch (err) {
                resultArea.innerText = 'שגיאה: ' + err.message;
                resultArea.className = 'w-full min-h-[80px] bg-slate-950/80 border border-slate-800 rounded-xl p-4 text-rose-400 whitespace-pre-wrap leading-relaxed text-base';
            } finally {
                btn.disabled = false;
                btn.innerText = "תביא לי את התכל'ס";
            }
        }
    </script>
</body>
</html>
