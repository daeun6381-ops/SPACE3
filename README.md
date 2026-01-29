<!DOCTYPE html>
<html lang="ko">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Our Space | 우리만의 비밀 공간</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script src="https://unpkg.com/lucide@latest"></script>
    <style>
        @import url('https://fonts.googleapis.com/css2?family=Pretendard:wght@400;600;700&display=swap');
        @import url('https://fonts.googleapis.com/css2?family=Gaegu:wght@400;700&display=swap');
        
        body { 
            font-family: 'Pretendard', sans-serif; 
            transition: background-color 0.5s ease; 
        }

        .memo-font { 
            font-family: 'Gaegu', cursive; 
            font-weight: 700; 
        }

        .fade-in { animation: fadeIn 0.6s ease-out forwards; }
        @keyframes fadeIn { 
            from { opacity: 0; transform: translateY(10px); } 
            to { opacity: 1; transform: translateY(0); } 
        }

        .no-scrollbar::-webkit-scrollbar { display: none; }
        .no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

        .btn-active:active { transform: scale(0.95); }
        
        /* 채팅 말풍선 스타일 */
        .chat-memo {
            position: relative;
            max-width: 85%;
            padding: 14px 18px;
            border-radius: 22px;
            font-size: 1.25rem;
            line-height: 1.4;
            box-shadow: 2px 4px 10px rgba(0,0,0,0.08);
            margin-bottom: 10px;
            word-break: break-all;
        }
        
        .chat-memo.mine {
            background: #fff9c4;
            align-self: flex-end;
            border-bottom-right-radius: 4px;
            transform: rotate(1deg);
            color: #4a4200;
        }
        
        .chat-memo.others {
            background: #e3f2fd;
            align-self: flex-start;
            border-bottom-left-radius: 4px;
            transform: rotate(-1deg);
            color: #0d3b66;
        }

        .photo-badge-top {
            background: linear-gradient(135deg, #ff7eb3 0%, #ff758c 100%);
            box-shadow: 0 4px 12px rgba(255, 117, 140, 0.4);
            color: white;
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 13px;
            font-weight: 800;
        }
        .photo-badge-bottom {
            background: rgba(255, 255, 255, 0.9);
            backdrop-filter: blur(4px);
            color: #333;
            padding: 4px 12px;
            border-radius: 12px;
            font-size: 12px;
            font-weight: 700;
        }

        .delete-hint {
            position: absolute;
            top: 50%;
            left: 50%;
            transform: translate(-50%, -50%);
            background: rgba(0, 0, 0, 0.5);
            backdrop-filter: blur(2px);
            color: white;
            padding: 6px 14px;
            border-radius: 16px;
            font-size: 11px;
            font-weight: 600;
            opacity: 0;
            transition: opacity 0.3s ease;
            pointer-events: none;
            z-index: 10;
        }
        
        .photo-container.show-hint .delete-hint {
            opacity: 1;
        }

        .dark-mode { background-color: #1a1a1a; color: #f3f4f6; }
        .dark-mode .bg-white { background-color: #262626; border-color: #333; }
        .dark-mode .chat-memo.mine { background-color: #5c5623 !important; color: #fff; }
        .dark-mode .chat-memo.others { background-color: #233e5c !important; color: #fff; }

        /* 로딩 애니메이션 */
        .loading-spinner {
            border: 3px solid rgba(255, 255, 255, 0.3);
            border-radius: 50%;
            border-top: 3px solid #fff;
            width: 20px;
            height: 20px;
            animation: spin 1s linear infinite;
            display: inline-block;
        }
        @keyframes spin {
            0% { transform: rotate(0deg); }
            100% { transform: rotate(360deg); }
        }
    </style>
</head>
<body class="bg-gray-50 text-gray-800 overflow-x-hidden min-h-screen">

    <!-- [1] 접속 보안 화면 -->
    <div id="auth-screen" class="fixed inset-0 z-[100] flex flex-col items-center justify-center bg-white p-6 transition-opacity duration-500">
        <div class="w-full max-w-sm text-center space-y-10">
            <div class="flex justify-center">
                <div id="lock-icon-bg" class="p-6 bg-pink-50 rounded-full text-pink-500 transition-colors duration-500">
                    <i data-lucide="lock" class="w-14 h-14"></i>
                </div>
            </div>
            <div class="space-y-2">
                <h1 class="text-3xl font-bold tracking-tight">Our Secret Space</h1>
                <p class="text-gray-500 text-base">비밀 코드를 입력하세요</p>
            </div>
            <input type="password" id="secret-code" maxlength="4" placeholder="••••" 
                   class="w-full text-center text-5xl tracking-[1.5rem] p-4 border-b-2 border-gray-200 focus:outline-none focus:border-pink-400 transition-all uppercase placeholder:tracking-normal">
            <p id="auth-error" class="text-red-400 text-base opacity-0 transition-opacity">코드가 올바르지 않습니다.</p>
        </div>
    </div>

    <!-- [3] 커스텀 삭제 모달 -->
    <div id="delete-modal" class="fixed inset-0 z-[200] hidden flex items-center justify-center p-6 bg-black/60 backdrop-blur-sm">
        <div class="bg-white rounded-[2.5rem] w-full max-w-xs p-8 text-center space-y-6 fade-in shadow-2xl">
            <div class="flex justify-center">
                <div class="p-4 bg-red-50 text-red-500 rounded-full">
                    <i data-lucide="trash-2" class="w-10 h-10"></i>
                </div>
            </div>
            <div class="space-y-2">
                <h3 class="text-xl font-bold">추억을 삭제할까요?</h3>
                <p class="text-gray-500 text-sm">지워진 사진은 복구할 수 없어요.</p>
            </div>
            <div class="flex gap-3">
                <button onclick="closeDeleteModal()" class="flex-1 py-4 bg-gray-100 rounded-2xl font-bold text-gray-600 hover:bg-gray-200 transition-colors">취소</button>
                <button id="confirm-delete-btn" class="flex-1 py-4 bg-red-500 rounded-2xl font-bold text-white shadow-lg shadow-red-200 hover:bg-red-600 transition-colors">삭제</button>
            </div>
        </div>
    </div>

    <!-- [2] 메인 애플리케이션 -->
    <div id="main-app" class="hidden opacity-0 transition-opacity duration-700">
        
        <!-- 헤더 -->
        <header class="sticky top-0 z-40 bg-white/80 backdrop-blur-lg border-b border-gray-100 transition-colors duration-500">
            <div class="max-w-4xl mx-auto px-6 h-20 flex justify-between items-center">
                <div class="flex flex-col">
                    <div class="flex items-center gap-2">
                        <span id="header-dday" class="text-2xl font-black tracking-tighter text-pink-500 transition-colors duration-500">D+0</span>
                        <div id="online-indicator" class="w-2 h-2 rounded-full bg-green-400 animate-pulse hidden" title="실시간 동기화 중"></div>
                    </div>
                    <span class="text-[11px] text-gray-500 font-bold uppercase tracking-widest">Since 2025.05.28</span>
                </div>
                <div class="flex items-center gap-2">
                    <button onclick="toggleThemePanel()" class="p-2 rounded-full hover:bg-gray-100 btn-active">
                        <i data-lucide="palette" class="w-6 h-6 text-gray-500"></i>
                    </button>
                </div>
            </div>
            <div id="theme-panel" class="hidden max-w-4xl mx-auto px-6 pb-4 flex gap-3 overflow-x-auto no-scrollbar fade-in">
                <button onclick="changeTheme('pink')" class="flex-shrink-0 px-5 py-2 rounded-full bg-pink-100 text-pink-600 text-sm font-bold">Pink</button>
                <button onclick="changeTheme('blue')" class="flex-shrink-0 px-5 py-2 rounded-full bg-blue-100 text-blue-600 text-sm font-bold">Blue</button>
                <button onclick="changeTheme('green')" class="flex-shrink-0 px-5 py-2 rounded-full bg-emerald-100 text-emerald-600 text-sm font-bold">Green</button>
                <button onclick="changeTheme('dark')" class="flex-shrink-0 px-5 py-2 rounded-full bg-gray-800 text-white text-sm font-bold">Dark</button>
            </div>
        </header>

        <main class="max-w-5xl mx-auto p-6 pb-32 space-y-12">
            
            <div id="tab-home" class="tab-content space-y-12">
                <!-- 상단 슬라이더 -->
                <div class="relative">
                    <h4 class="font-bold text-xl mb-4 flex items-center justify-between">
                        <div class="flex items-center gap-2">
                            <i data-lucide="sparkles" class="w-6 h-6 text-yellow-400"></i>
                            STORY
                            <span class="text-[11px] font-normal text-gray-400 ml-1">옆으로 스크롤 해보세요</span>
                        </div>
                    </h4>
                    <div id="main-photo-slider" class="flex gap-4 overflow-x-auto no-scrollbar py-2 snap-x snap-mandatory min-h-[280px]">
                        <div class="min-w-full h-72 bg-gray-100 rounded-[2.5rem] flex items-center justify-center text-gray-500 text-base">
                            사진을 불러오는 중입니다...
                        </div>
                    </div>
                </div>

                <!-- 업로드와 메모 나란히 배치 -->
                <div class="grid grid-cols-1 lg:grid-cols-2 gap-8 items-start">
                    
                    <!-- 사진 업로드 카드 -->
                    <div class="bg-white rounded-[2.5rem] p-7 shadow-xl shadow-gray-200/50 border border-gray-50 space-y-5 h-full">
                        <div class="flex items-center gap-3">
                            <div id="upload-icon-box" class="p-2.5 bg-pink-100 rounded-xl text-pink-500">
                                <i data-lucide="camera" class="w-6 h-6"></i>
                            </div>
                            <h4 class="font-bold text-lg">사진 업로드</h4>
                        </div>
                        
                        <div id="image-preview-container" class="hidden w-full h-48 bg-gray-50 rounded-2xl overflow-hidden relative border-2 border-dashed border-gray-200">
                            <img id="image-preview" src="" class="w-full h-full object-cover">
                            <button onclick="clearImageSelection()" class="absolute top-3 right-3 bg-black/50 text-white p-1.5 rounded-full">
                                <i data-lucide="x" class="w-5 h-5"></i>
                            </button>
                        </div>

                        <textarea id="post-caption" placeholder="무슨 일이 있었나요?" 
                                  class="w-full p-5 bg-gray-50 rounded-2xl resize-none focus:outline-none focus:ring-2 ring-pink-100 transition-all h-24 text-base"></textarea>
                        
                        <div class="grid grid-cols-2 gap-3">
                            <div class="space-y-1.5">
                                <label class="text-[12px] text-gray-500 ml-2 font-bold uppercase">사진 날짜</label>
                                <input type="date" id="post-date" class="w-full p-3.5 bg-gray-50 rounded-xl text-sm focus:outline-none border-none">
                            </div>
                            <div class="space-y-1.5">
                                <label class="text-[12px] text-gray-500 ml-2 font-bold uppercase">사진 선택</label>
                                <label for="post-file" class="flex items-center justify-center w-full p-3.5 bg-gray-100 rounded-xl text-xs font-bold cursor-pointer hover:bg-gray-200 transition-colors overflow-hidden">
                                    <span id="file-name-label">파일 선택</span>
                                    <input type="file" id="post-file" accept="image/*" class="hidden" onchange="handleFileSelect(event)">
                                </label>
                            </div>
                        </div>
                        
                        <button onclick="uploadPost()" id="upload-btn" class="w-full bg-pink-500 text-white py-5 rounded-2xl font-bold text-lg btn-active transition-all flex items-center justify-center gap-2">
                            <span id="btn-text">기억 업로드</span>
                            <span id="btn-spinner" class="loading-spinner hidden"></span>
                        </button>
                    </div>

                    <!-- 채팅형 메모 섹션 -->
                    <div class="space-y-5 h-full flex flex-col">
                        <div class="flex items-center justify-between px-2">
                            <h4 class="font-bold text-xl flex items-center gap-2">
                                <i data-lucide="message-circle" class="w-6 h-6 text-pink-500"></i>
                                메모 남기기
                            </h4>
                        </div>
                        
                        <div id="memo-board" class="flex-1 flex flex-col gap-4 p-6 bg-orange-50/30 rounded-[2.5rem] min-h-[400px] max-h-[500px] overflow-y-auto no-scrollbar border-2 border-dashed border-orange-100">
                        </div>

                        <div class="flex gap-2.5 p-2">
                            <input type="text" id="memo-input" placeholder="메시지를 남겨주세요..." 
                                   onkeypress="if(event.keyCode==13) sendMemo()"
                                   class="flex-1 p-5 bg-white rounded-2xl shadow-sm border border-gray-100 text-base focus:outline-none">
                            <button onclick="sendMemo()" id="send-btn" class="p-5 bg-pink-500 text-white rounded-2xl shadow-md btn-active">
                                <i data-lucide="send" class="w-6 h-6"></i>
                            </button>
                        </div>
                    </div>
                </div>
            </div>

            <div id="tab-gallery" class="tab-content hidden space-y-8">
                <div class="px-2">
                    <h4 class="font-bold text-3xl tracking-tight">앨범</h4>
                    <p class="text-gray-500 text-base font-medium">추억 돌아보기</p>
                </div>
                <div id="gallery-grid" class="grid grid-cols-2 md:grid-cols-3 lg:grid-cols-4 gap-4"></div>
            </div>

        </main>

        <nav class="fixed bottom-8 left-1/2 -translate-x-1/2 w-[calc(100%-3rem)] max-w-md h-22 bg-white/90 backdrop-blur-xl border border-white/20 rounded-[2.5rem] shadow-2xl flex items-center justify-around px-4 z-50">
            <button onclick="switchTab('home')" id="nav-home" class="flex flex-col items-center gap-1.5 text-pink-500 transition-all duration-300 scale-110">
                <i data-lucide="heart" class="w-7 h-7"></i>
                <span class="text-[12px] font-bold">홈</span>
            </button>
            <button onclick="switchTab('gallery')" id="nav-gallery" class="flex flex-col items-center gap-1.5 text-gray-400 transition-all duration-300">
                <i data-lucide="layout-grid" class="w-7 h-7"></i>
                <span class="text-[12px] font-bold">앨범</span>
            </button>
        </nav>
    </div>

    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-app.js";
        import { getFirestore, collection, addDoc, onSnapshot, query, orderBy, doc, setDoc, deleteDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-firestore.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.0.2/firebase-auth.js";

        const firebaseConfig = {
            apiKey: "AIzaSyDDPLecqluEvnmfwUqtaUFHuUReqTkybkc",
            authDomain: "space1-b521a.firebaseapp.com",
            projectId: "space1-b521a",
            storageBucket: "space1-b521a.firebasestorage.app",
            messagingSenderId: "235223237323",
            appId: "1:235223237323:web:d5cbf1999654707e65514e",
            measurementId: "G-5K1ETJG4T6"
        };

        const app = initializeApp(firebaseConfig);
        const db = getFirestore(app);
        const auth = getAuth(app);
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'our-secret-space-v3';
        const START_DATE = new Date('2025-05-28');

        let isAuthInProgress = false;
        let isAppInitialized = false;
        let selectedImageData = null;
        let targetDeleteId = null;

        async function authenticate() {
            if (isAuthInProgress) return;
            isAuthInProgress = true;
            try {
                const token = typeof __initial_auth_token !== 'undefined' ? __initial_auth_token : null;
                if (token) {
                    try { await signInWithCustomToken(auth, token); } 
                    catch (tokenErr) { await signInAnonymously(auth); }
                } else { await signInAnonymously(auth); }
            } catch (err) {
                isAuthInProgress = false;
                setTimeout(authenticate, 2000);
            }
        }

        onAuthStateChanged(auth, (user) => {
            isAuthInProgress = false;
            if (user) {
                const indicator = document.getElementById('online-indicator');
                if (indicator) indicator.classList.remove('hidden');
                const appScreen = document.getElementById('main-app');
                if (appScreen && !appScreen.classList.contains('hidden') && !isAppInitialized) {
                    initApp();
                }
            }
        });

        document.getElementById('secret-code').addEventListener('input', async (e) => {
            const errorText = document.getElementById('auth-error');
            const val = e.target.value.toUpperCase();
            if (val === 'EVOL') {
                errorText.classList.add('opacity-0');
                if (!auth.currentUser) await authenticate();
                document.getElementById('auth-screen').classList.add('opacity-0');
                setTimeout(() => {
                    document.getElementById('auth-screen').style.display = 'none';
                    document.getElementById('main-app').classList.remove('hidden');
                    setTimeout(() => {
                        document.getElementById('main-app').classList.add('opacity-100');
                        if (auth.currentUser && !isAppInitialized) initApp();
                    }, 50);
                }, 500);
            } else if (e.target.value.length === 4) {
                errorText.classList.remove('opacity-0');
                setTimeout(() => { e.target.value = ''; errorText.classList.add('opacity-0'); }, 1500);
            }
        });

        // 사진 리사이징 및 압축 함수
        async function compressImage(file) {
            return new Promise((resolve) => {
                const reader = new FileReader();
                reader.readAsDataURL(file);
                reader.onload = (event) => {
                    const img = new Image();
                    img.src = event.target.result;
                    img.onload = () => {
                        const canvas = document.createElement('canvas');
                        let width = img.width;
                        let height = img.height;

                        // 최대 너비 설정 (1200px)
                        const MAX_WIDTH = 1200;
                        if (width > MAX_WIDTH) {
                            height = Math.round((height * MAX_WIDTH) / width);
                            width = MAX_WIDTH;
                        }

                        canvas.width = width;
                        canvas.height = height;
                        const ctx = canvas.getContext('2d');
                        ctx.drawImage(img, 0, 0, width, height);

                        // JPEG 품질 0.7 정도로 압축하여 Base64 반환
                        const dataUrl = canvas.toDataURL('image/jpeg', 0.7);
                        resolve(dataUrl);
                    };
                };
            });
        }

        window.handleFileSelect = async (event) => {
            const file = event.target.files[0];
            if (!file) return;

            // 로딩 표시
            const label = document.getElementById('file-name-label');
            const originalText = label.innerText;
            label.innerText = "처리 중...";

            try {
                // 용량이 크더라도 클라이언트 단에서 압축 진행
                selectedImageData = await compressImage(file);
                
                document.getElementById('image-preview').src = selectedImageData;
                document.getElementById('image-preview-container').classList.remove('hidden');
                document.getElementById('file-name-label').innerText = file.name;
                lucide.createIcons();
            } catch (err) {
                console.error("Image processing error:", err);
                alert("사진을 처리하는 중에 오류가 발생했어요.");
                label.innerText = originalText;
            }
        };

        window.clearImageSelection = () => {
            selectedImageData = null;
            document.getElementById('post-file').value = "";
            document.getElementById('image-preview-container').classList.add('hidden');
            document.getElementById('file-name-label').innerText = "파일 선택";
        };

        function initApp() {
            if (isAppInitialized) return;
            isAppInitialized = true;
            lucide.createIcons();
            updateDDay();
            listenToTheme();
            listenToPosts();
            listenToMemos();
            const dateInput = document.getElementById('post-date');
            if (dateInput) dateInput.valueAsDate = new Date();
            const deleteBtn = document.getElementById('confirm-delete-btn');
            if (deleteBtn) deleteBtn.onclick = deletePost;
        }

        function updateDDay() {
            const diff = Math.floor((new Date() - START_DATE) / (1000 * 60 * 60 * 24));
            const el = document.getElementById('header-dday');
            if (el) el.innerText = `D+${diff}`;
        }

        window.changeTheme = async (theme) => {
            if (!auth.currentUser) return;
            try { await setDoc(doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'currentTheme'), { name: theme }); } 
            catch (e) { console.error("Theme set error:", e); }
        };

        function applyTheme(theme) {
            const root = document.body;
            const logo = document.getElementById('header-dday');
            const uploadIcon = document.getElementById('upload-icon-box');
            const uploadBtn = document.getElementById('upload-btn');
            const sendBtn = document.getElementById('send-btn');
            root.classList.remove('dark-mode');
            const themes = {
                pink: { text: 'text-pink-500', btn: 'bg-pink-500', icon: 'bg-pink-100' },
                blue: { text: 'text-blue-500', btn: 'bg-blue-500', icon: 'bg-blue-100' },
                green: { text: 'text-emerald-500', btn: 'bg-emerald-500', icon: 'bg-emerald-100' },
                dark: { text: 'text-white', btn: 'bg-gray-700', icon: 'bg-gray-700' }
            };
            const t = themes[theme] || themes.pink;
            if (theme === 'dark') root.classList.add('dark-mode');
            if (logo) logo.className = `text-2xl font-black tracking-tighter ${t.text}`;
            if (uploadIcon) uploadIcon.className = `p-2.5 ${t.icon} rounded-xl ${t.text}`;
            if (uploadBtn) uploadBtn.className = `w-full ${t.btn} text-white py-5 rounded-2xl font-bold text-lg btn-active transition-all flex items-center justify-center gap-2`;
            if (sendBtn) sendBtn.className = `p-5 ${t.btn} text-white rounded-2xl shadow-md btn-active`;
        }

        function listenToTheme() {
            if (!auth.currentUser) return;
            onSnapshot(doc(db, 'artifacts', appId, 'public', 'data', 'settings', 'currentTheme'), (s) => {
                if (s.exists()) applyTheme(s.data().name);
            });
        }

        window.uploadPost = async () => {
            if (!auth.currentUser) return;
            const caption = document.getElementById('post-caption').value;
            const dateVal = document.getElementById('post-date').value;
            if (!caption.trim() && !selectedImageData) return;

            // 버튼 로딩 상태 시작
            const btnText = document.getElementById('btn-text');
            const btnSpinner = document.getElementById('btn-spinner');
            const uploadBtn = document.getElementById('upload-btn');
            
            btnText.innerText = "저장 중...";
            btnSpinner.classList.remove('hidden');
            uploadBtn.disabled = true;

            const imgUrl = selectedImageData || 'https://images.unsplash.com/photo-1518199266791-5375a83190b7?w=800';
            const postDate = new Date(dateVal);
            const daysDiff = Math.floor((postDate - START_DATE) / (1000 * 60 * 60 * 24)) + 1;
            
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'posts'), {
                    caption, imgUrl, dateVal, daysDiff,
                    createdAt: serverTimestamp(),
                    userId: auth.currentUser.uid
                });
                document.getElementById('post-caption').value = '';
                clearImageSelection();
            } catch (e) { 
                console.error("Upload error:", e); 
                alert("업로드에 실패했어요. 용량이 여전히 너무 큰지 확인해주세요.");
            } finally {
                // 버튼 상태 복구
                btnText.innerText = "기억 업로드";
                btnSpinner.classList.add('hidden');
                uploadBtn.disabled = false;
            }
        };

        window.openDeleteModal = (id) => {
            targetDeleteId = id;
            const modal = document.getElementById('delete-modal');
            modal.classList.remove('hidden');
            modal.classList.add('flex');
        };

        window.closeDeleteModal = () => {
            targetDeleteId = null;
            const modal = document.getElementById('delete-modal');
            modal.classList.add('hidden');
            modal.classList.remove('flex');
        };

        async function deletePost() {
            if (!targetDeleteId || !auth.currentUser) return;
            try {
                await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'posts', targetDeleteId));
                closeDeleteModal();
            } catch (e) { console.error("Delete error:", e); }
        }

        window.toggleDeleteHint = (element) => {
            const hasHint = element.classList.contains('show-hint');
            document.querySelectorAll('.photo-container').forEach(el => el.classList.remove('show-hint'));
            if (!hasHint) {
                element.classList.add('show-hint');
                setTimeout(() => element.classList.remove('show-hint'), 3000);
            }
        };

        function listenToPosts() {
            if (!auth.currentUser) return;
            const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'posts'), orderBy('createdAt', 'desc'));
            onSnapshot(q, (snapshot) => {
                const slider = document.getElementById('main-photo-slider');
                const grid = document.getElementById('gallery-grid');
                if (!slider || !grid) return;
                slider.innerHTML = ''; 
                grid.innerHTML = '';
                if (snapshot.empty) {
                    slider.innerHTML = `<div class="min-w-full h-72 bg-gray-100 rounded-[2.5rem] flex items-center justify-center text-gray-500 text-sm text-center px-10">첫 번째 추억을 기록해 보세요!</div>`;
                }
                snapshot.forEach(snap => {
                    const p = snap.data();
                    const id = snap.id;
                    const dStr = p.dateVal ? p.dateVal.replace(/-/g, '.') : '0000.00.00';
                    const slide = document.createElement('div');
                    slide.className = 'min-w-[85%] lg:min-w-[45%] h-72 relative rounded-[2.5rem] overflow-hidden shadow-xl snap-center flex-shrink-0 fade-in photo-container cursor-pointer select-none active:scale-[0.98] transition-all';
                    slide.addEventListener('click', () => toggleDeleteHint(slide));
                    slide.addEventListener('dblclick', () => openDeleteModal(id));
                    slide.innerHTML = `
                        <img src="${p.imgUrl}" class="w-full h-full object-cover pointer-events-none" onerror="this.src='https://images.unsplash.com/photo-1518199266791-5375a83190b7?w=800'">
                        <div class="delete-hint">더블 클릭 시 삭제</div>
                        <div class="absolute inset-0 bg-gradient-to-t from-black/70 via-transparent to-transparent pointer-events-none"></div>
                        <div class="absolute top-4 left-4 photo-badge-top">Day ${p.daysDiff}</div>
                        <div class="absolute bottom-4 right-4 photo-badge-bottom">${dStr}</div>
                        <div class="absolute bottom-4 left-4 text-white text-sm font-bold px-3 truncate w-[70%]">${p.caption || '기록된 순간'}</div>
                    `;
                    slider.appendChild(slide);
                    const card = document.createElement('div');
                    card.className = 'relative aspect-square rounded-2xl overflow-hidden shadow-sm fade-in photo-container cursor-pointer active:scale-95 transition-all';
                    card.addEventListener('click', () => toggleDeleteHint(card));
                    card.addEventListener('dblclick', () => openDeleteModal(id));
                    card.innerHTML = `
                        <img src="${p.imgUrl}" class="w-full h-full object-cover pointer-events-none" onerror="this.src='https://images.unsplash.com/photo-1518199266791-5375a83190b7?w=800'">
                        <div class="delete-hint">더블 클릭 시 삭제</div>
                        <div class="absolute top-2 left-2 photo-badge-top text-[10px]">Day ${p.daysDiff}</div>
                    `;
                    grid.appendChild(card);
                });
            });
        }

        window.sendMemo = async () => {
            if (!auth.currentUser) return;
            const input = document.getElementById('memo-input');
            const text = input.value;
            if (!text.trim()) return;
            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'memos'), {
                    text, createdAt: serverTimestamp(), userId: auth.currentUser.uid
                });
                input.value = '';
            } catch (e) { console.error("Memo error:", e); }
        };

        function listenToMemos() {
            if (!auth.currentUser) return;
            const q = query(collection(db, 'artifacts', appId, 'public', 'data', 'memos'), orderBy('createdAt', 'asc'));
            onSnapshot(q, (snapshot) => {
                const board = document.getElementById('memo-board');
                if (!board) return;
                board.innerHTML = '';
                snapshot.forEach(docSnap => {
                    const m = docSnap.data();
                    const isMine = m.userId === auth.currentUser.uid;
                    const note = document.createElement('div');
                    note.className = `chat-memo ${isMine ? 'mine' : 'others'} memo-font fade-in`;
                    note.innerText = m.text;
                    board.appendChild(note);
                });
                board.scrollTo({ top: board.scrollHeight, behavior: 'smooth' });
            });
        }

        window.switchTab = (tab) => {
            document.querySelectorAll('.tab-content').forEach(el => el.classList.add('hidden'));
            document.getElementById(`tab-${tab}`).classList.remove('hidden');
            ['home', 'gallery'].forEach(t => {
                const nav = document.getElementById(`nav-${t}`);
                if (t === tab) {
                    nav.classList.add('text-pink-500', 'scale-110');
                    nav.classList.remove('text-gray-400');
                } else {
                    nav.classList.remove('text-pink-500', 'scale-110');
                    nav.classList.add('text-gray-400');
                }
            });
            lucide.createIcons();
        };

        window.toggleThemePanel = () => document.getElementById('theme-panel').classList.toggle('hidden');
    </script>
</body>
</html>
