<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>AI 스마트 물품 관리기 Pro</title>
    <!-- Tailwind CSS 및 Font Awesome 로드 -->
    <script src="https://cdn.tailwindcss.com"></script>
    <link href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.0.0/css/all.min.css" rel="stylesheet">
    <style>
        @import url('https://cdn.jsdelivr.net/gh/orioncactus/pretendard/dist/web/static/pretendard.css');
        
        body { 
            background-color: #f8fafc; 
            color: #1e293b; 
            font-family: 'Pretendard', sans-serif; 
            -webkit-font-smoothing: antialiased;
        }

        .glass { 
            background: rgba(255, 255, 255, 0.95); 
            backdrop-filter: blur(10px); 
        }

        .loader {
            border-top-color: #3b82f6;
            animation: spinner 1s linear infinite;
        }

        @keyframes spinner {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }

        .result-card {
            transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
        }

        .result-card:hover {
            transform: translateY(-4px);
            box-shadow: 0 10px 15px -3px rgba(0, 0, 0, 0.1);
        }
    </style>
</head>
<body class="min-h-screen p-4 md:p-8 flex flex-col items-center">

    <div class="max-w-2xl w-full space-y-6">
        <!-- 헤더 영역 -->
        <div class="text-center space-y-3 py-4">
            <div class="inline-flex items-center justify-center w-20 h-20 bg-blue-600 rounded-3xl shadow-xl mb-2 text-white text-4xl">
                <i class="fas fa-barcode"></i>
            </div>
            <h1 class="text-4xl font-black text-slate-900 tracking-tight">AI 스마트 <span class="text-blue-600">물품 대장</span></h1>
            <p class="text-slate-500 font-medium text-lg">사진 속 물품을 식별하여 시트에 기록합니다</p>
        </div>

        <!-- 설정 영역 (앱스 스크립트 URL) -->
        <div class="glass p-5 rounded-2xl shadow-sm border border-slate-200">
            <label class="block text-xs font-black text-slate-400 uppercase tracking-widest mb-2 text-center">연동된 구글 스프레드시트 시스템</label>
            <div class="flex items-center justify-center gap-2 text-blue-600 font-bold text-sm mb-3">
                <i class="fas fa-check-circle"></i>
                <span>기록 시스템 연결됨</span>
            </div>
            <input type="text" id="script-url" 
                   value="https://script.google.com/macros/s/AKfycbz7P7ORsIhO_bphJf3POExTIIqphCDIiAIZ4KmhBSeWUXPShI2afceST8ObsrGEGqsc/exec"
                   class="w-full bg-white border border-slate-200 rounded-xl px-4 py-3 text-sm focus:ring-2 focus:ring-blue-500 outline-none transition-all">
        </div>

        <!-- 메인 업로드 카드 -->
        <div class="bg-white rounded-[2.5rem] shadow-2xl shadow-slate-200/50 overflow-hidden border border-white">
            <div class="p-8 space-y-8">
                
                <!-- 이미지 업로드 영역 -->
                <div id="drop-zone" class="relative group cursor-pointer">
                    <div id="placeholder" class="w-full h-80 border-2 border-dashed border-slate-200 rounded-[2.5rem] flex flex-col items-center justify-center bg-slate-50 group-hover:bg-blue-50 transition-all duration-300">
                        <div class="bg-white p-5 rounded-full shadow-sm mb-4 group-hover:scale-110 transition-transform">
                            <i class="fas fa-camera text-4xl text-blue-500"></i>
                        </div>
                        <p class="font-bold text-slate-700 text-xl text-center px-4">물건 촬영 또는 업로드</p>
                        <p class="text-sm text-slate-400 mt-2 font-medium">클릭하여 사진을 선택하세요</p>
                    </div>
                    <img id="preview-img" class="hidden w-full h-80 object-contain rounded-[2rem] bg-slate-50 shadow-inner">
                    <button id="reset-btn" class="hidden absolute top-4 right-4 bg-white/90 text-slate-600 w-12 h-12 rounded-2xl shadow-xl hover:text-red-500 transition-all">
                        <i class="fas fa-trash"></i>
                    </button>
                </div>

                <input type="file" id="file-input" accept="image/*" class="hidden">

                <!-- 분석 버튼 -->
                <button id="analyze-btn" disabled class="w-full bg-slate-900 text-white font-bold py-5 rounded-[1.5rem] shadow-xl hover:bg-slate-800 disabled:bg-slate-200 transition-all flex items-center justify-center gap-3 text-lg">
                    <i class="fas fa-wand-magic-sparkles"></i> 분석 및 대장 기록
                </button>

                <!-- 상태 및 결과 표시 영역 -->
                <div id="status-area" class="hidden space-y-6 pt-6 border-t border-slate-100">
                    <div id="loading" class="flex flex-col items-center py-10">
                        <div class="loader rounded-full border-4 border-slate-100 h-16 w-16 mb-6"></div>
                        <p class="text-slate-600 font-black text-lg animate-pulse">AI가 물품을 분석하고 있습니다...</p>
                    </div>

                    <div id="result-box" class="hidden">
                        <div class="flex items-center justify-between mb-6 px-1">
                            <h3 class="font-black text-slate-800 text-2xl flex items-center gap-2">
                                <i class="fas fa-list-check text-blue-600"></i> 기록될 목록
                            </h3>
                            <span id="sync-badge" class="hidden px-4 py-2 bg-emerald-100 text-emerald-700 text-xs font-black rounded-xl">
                                <i class="fas fa-check-double mr-1"></i> 시트 전송 완료
                            </span>
                        </div>
                        <div id="items-list" class="grid grid-cols-1 gap-4">
                            <!-- 분석 결과 아이템이 동적으로 삽입됨 -->
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- 하단 푸터 -->
        <div class="text-center px-4">
            <p class="text-[10px] text-slate-400 font-bold uppercase tracking-[0.2em] leading-loose">
                Powered by Gemini 2.5 Flash API & Google Sheets Connector<br>
                Professional Inventory Management System
            </p>
        </div>
    </div>

    <script>
        // API 설정: GitHub 등 외부 호스팅 시에는 반드시 API 키가 포함되어야 합니다.
        const apiKey = "AIzaSyC61yO3g7qVSjpaEL_vrVVdziks_kANDRc"; 
        const model = "gemini-2.5-flash-preview-09-2025";

        const fileInput = document.getElementById('file-input');
        const dropZone = document.getElementById('drop-zone');
        const analyzeBtn = document.getElementById('analyze-btn');
        const previewImg = document.getElementById('preview-img');
        const placeholder = document.getElementById('placeholder');
        const resetBtn = document.getElementById('reset-btn');
        const statusArea = document.getElementById('status-area');
        const loading = document.getElementById('loading');
        const resultBox = document.getElementById('result-box');
        const itemsList = document.getElementById('items-list');
        const syncBadge = document.getElementById('sync-badge');
        const scriptUrlInput = document.getElementById('script-url');

        let base64Image = null;
        let mimeType = null;

        // 파일 핸들링
        dropZone.onclick = () => fileInput.click();
        fileInput.onchange = (e) => {
            const file = e.target.files[0];
            if (file) {
                mimeType = file.type;
                const reader = new FileReader();
                reader.onload = (ev) => {
                    base64Image = ev.target.result.split(',')[1];
                    previewImg.src = ev.target.result;
                    previewImg.classList.remove('hidden');
                    placeholder.classList.add('hidden');
                    resetBtn.classList.remove('hidden');
                    analyzeBtn.disabled = false;
                    resetResults();
                };
                reader.readAsDataURL(file);
            }
        };

        resetBtn.onclick = (e) => {
            e.stopPropagation();
            base64Image = null;
            fileInput.value = '';
            previewImg.classList.add('hidden');
            placeholder.classList.remove('hidden');
            resetBtn.classList.add('hidden');
            analyzeBtn.disabled = true;
            resetResults();
        };

        function resetResults() {
            statusArea.classList.add('hidden');
            resultBox.classList.add('hidden');
            itemsList.innerHTML = '';
            syncBadge.classList.add('hidden');
        }

        // AI 분석 실행
        analyzeBtn.onclick = async () => {
            if (!base64Image) return;

            statusArea.classList.remove('hidden');
            loading.classList.remove('hidden');
            resultBox.classList.add('hidden');
            analyzeBtn.disabled = true;

            const prompt = `사진 속 모든 물건들을 개별적으로 식별하여 분석해줘.
            JSON 형식으로만 응답하며, 비고(remark)는 반드시 빈 문자열("")로 채워줘.
            {
                "items": [
                    {"name": "물건 이름", "usage": "간략한 용도 설명", "emoji": "아이콘 이모지", "remark": ""}
                ]
            }`;

            try {
                const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/${model}:generateContent?key=${apiKey}`, {
                    method: 'POST',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({
                        contents: [{
                            parts: [
                                { text: prompt },
                                { inlineData: { mimeType, data: base64Image } }
                            ]
                        }]
                    })
                });

                const aiResult = await response.json();
                
                // 에러 처리 추가
                if (aiResult.error) {
                    throw new Error(aiResult.error.message);
                }

                const content = aiResult.candidates[0].content.parts[0].text.replace(/```json|```/g, '').trim();
                const data = JSON.parse(content);

                renderResults(data.items);
                await sendToSpreadsheet(data.items);

            } catch (err) {
                console.error(err);
                alert("분석 중 오류가 발생했습니다: " + err.message + "\n(API 키가 유효한지 확인해주세요)");
            } finally {
                loading.classList.add('hidden');
                analyzeBtn.disabled = false;
            }
        };

        function renderResults(items) {
            itemsList.innerHTML = '';
            items.forEach(item => {
                const div = document.createElement('div');
                div.className = "result-card p-5 bg-slate-50 border border-slate-100 rounded-[1.5rem] flex items-center gap-5 shadow-sm";
                div.innerHTML = `
                    <div class="w-16 h-16 bg-white rounded-2xl shadow-sm flex items-center justify-center text-3xl shrink-0">
                        ${item.emoji || '📦'}
                    </div>
                    <div class="flex-1 min-w-0">
                        <h4 class="font-black text-slate-800 text-lg truncate">${item.name}</h4>
                        <p class="text-xs text-slate-500 font-medium leading-relaxed">${item.usage}</p>
                    </div>
                `;
                itemsList.appendChild(div);
            });
            resultBox.classList.remove('hidden');
        }

        async function sendToSpreadsheet(items) {
            const url = scriptUrlInput.value.trim();
            if (!url) return;

            try {
                await fetch(url, {
                    method: 'POST',
                    mode: 'no-cors',
                    headers: { 'Content-Type': 'application/json' },
                    body: JSON.stringify({ items: items })
                });
                syncBadge.classList.remove('hidden');
            } catch (err) {
                console.warn("시트 전송 경고:", err);
            }
        }
    </script>
</body>
</html>
