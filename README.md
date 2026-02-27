<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0, maximum-scale=1.0, user-scalable=no">
    <title>ConnectPro - Realtime Calls & Chat</title>
    
    <!-- Tailwind CSS for Styling -->
    <script src="https://cdn.tailwindcss.com"></script>
    
    <!-- FontAwesome for Icons -->
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/6.4.0/css/all.min.css">
    
    <!-- Jitsi Meet API for robust WebRTC Audio/Video Calling -->
    <script src="https://8x8.vc/external_api.js"></script>

    <style>
        @import url('https://fonts.googleapis.com/css2?family=Inter:wght@300;400;500;600;700&display=swap');
        
        body {
            font-family: 'Inter', sans-serif;
            background-color: #f3f4f6;
            margin: 0;
            overflow: hidden; /* Prevent body scroll, manage in app container */
        }

        /* Custom Scrollbar */
        ::-webkit-scrollbar { width: 6px; }
        ::-webkit-scrollbar-track { background: transparent; }
        ::-webkit-scrollbar-thumb { background: #cbd5e1; border-radius: 10px; }
        ::-webkit-scrollbar-thumb:hover { background: #94a3b8; }

        /* Animations */
        .fade-in { animation: fadeIn 0.3s ease-in-out; }
        @keyframes fadeIn { from { opacity: 0; transform: translateY(10px); } to { opacity: 1; transform: translateY(0); } }
        
        .pulse-mic { animation: pulse 1.5s infinite; }
        @keyframes pulse {
            0% { box-shadow: 0 0 0 0 rgba(239, 68, 68, 0.7); }
            70% { box-shadow: 0 0 0 15px rgba(239, 68, 68, 0); }
            100% { box-shadow: 0 0 0 0 rgba(239, 68, 68, 0); }
        }

        /* Chat Background Pattern */
        .chat-bg {
            background-color: #efeae2;
            background-image: url('https://w0.peakpx.com/wallpaper/818/148/HD-wallpaper-whatsapp-background-cool-dark-green-new-theme-whatsapp.jpg');
            background-blend-mode: overlay;
            background-size: cover;
            background-position: center;
        }

        /* App Container for Mobile View */
        #app-container {
            height: 100vh;
            height: 100dvh;
            display: flex;
            flex-direction: column;
            background: white;
            box-shadow: 0 0 20px rgba(0,0,0,0.1);
        }

        /* Utility classes for hiding/showing screens */
        .screen { display: none; height: 100%; flex-direction: column; }
        .screen.active { display: flex; }
    </style>
</head>
<body>

    <div id="app-container" class="max-w-md mx-auto relative w-full">
        
        <!-- Header -->
        <header class="bg-emerald-600 text-white p-4 shadow-md flex items-center justify-between z-20 shrink-0">
            <div class="flex items-center gap-3">
                <button id="back-btn" class="hidden p-2 hover:bg-emerald-700 rounded-full transition" onclick="goBack()">
                    <i class="fa-solid fa-arrow-left"></i>
                </button>
                <h1 class="text-xl font-bold tracking-wide">ConnectPro</h1>
            </div>
            <div id="user-profile-badge" class="hidden items-center gap-2 bg-emerald-700 px-3 py-1.5 rounded-full text-sm font-medium shadow-inner">
                <i id="header-badge-icon" class="fa-solid fa-user text-xs"></i>
                <span id="header-username" class="truncate max-w-[100px]">User</span>
                <button onclick="logout()" class="ml-2 hover:text-red-300 transition" title="Change Name">
                    <i class="fa-solid fa-right-from-bracket text-xs"></i>
                </button>
            </div>
        </header>

        <!-- Screen 1: Loading -->
        <div id="screen-loading" class="screen active bg-gray-50 items-center justify-center">
            <div class="text-center">
                <div class="animate-spin rounded-full h-12 w-12 border-b-2 border-emerald-600 mx-auto mb-4"></div>
                <p id="loading-text" class="text-lg font-medium text-gray-700">Connecting to secure servers...</p>
            </div>
        </div>

        <!-- Screen 2: Name Prompt -->
        <div id="screen-name" class="screen bg-gray-900 bg-opacity-60 items-center justify-center p-4 absolute inset-0 z-50 backdrop-blur-sm">
            <div class="bg-white rounded-2xl shadow-2xl p-8 w-full max-w-sm fade-in">
                <div class="flex justify-center mb-6">
                    <div class="bg-emerald-100 text-emerald-600 w-20 h-20 rounded-full flex items-center justify-center shadow-inner">
                        <i class="fa-solid fa-user-astronaut text-4xl"></i>
                    </div>
                </div>
                <h2 class="text-2xl font-bold text-center text-gray-800 mb-2">Create Identity</h2>
                <p class="text-center text-gray-500 mb-6 text-sm">Enter your name to join calls and chats.</p>
                
                <input type="text" id="name-input" placeholder="Your Full Name..." class="w-full px-4 py-3 rounded-xl border border-gray-300 focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition mb-6 bg-gray-50 font-medium text-gray-800" onkeypress="if(event.key === 'Enter') saveName()">
                
                <button onclick="saveName()" class="w-full py-3 px-4 bg-emerald-600 text-white font-bold rounded-xl hover:bg-emerald-700 transition shadow-lg hover:shadow-xl transform hover:-translate-y-0.5">
                    Start Connecting <i class="fa-solid fa-arrow-right ml-2"></i>
                </button>
            </div>
        </div>

        <!-- Screen 3: Home Dashboard -->
        <div id="screen-home" class="screen bg-gradient-to-br from-emerald-50 to-teal-100 p-6 overflow-y-auto">
            <div class="text-center mb-8 mt-4 fade-in">
                <h2 class="text-3xl font-extrabold text-gray-800 mb-2">Welcome!</h2>
                <p class="text-gray-600 text-sm">Select a feature to connect with others globally.</p>
            </div>

            <div class="grid grid-cols-1 gap-5 w-full fade-in" style="animation-delay: 0.1s;">
                <button onclick="openServerSelect('video')" class="flex items-center gap-4 bg-white p-5 rounded-2xl shadow-md hover:shadow-xl transition transform hover:scale-[1.02] border-l-4 border-purple-500 group">
                    <div class="bg-purple-100 w-14 h-14 rounded-full flex items-center justify-center text-purple-600 group-hover:bg-purple-600 group-hover:text-white transition-colors">
                        <i class="fa-solid fa-video text-xl"></i>
                    </div>
                    <div class="text-left flex-1">
                        <h3 class="text-lg font-bold text-gray-800">Video Calling</h3>
                        <p class="text-xs text-gray-500">HD Multi-user video rooms</p>
                    </div>
                    <i class="fa-solid fa-chevron-right text-gray-300"></i>
                </button>

                <button onclick="openServerSelect('audio')" class="flex items-center gap-4 bg-white p-5 rounded-2xl shadow-md hover:shadow-xl transition transform hover:scale-[1.02] border-l-4 border-blue-500 group">
                    <div class="bg-blue-100 w-14 h-14 rounded-full flex items-center justify-center text-blue-600 group-hover:bg-blue-600 group-hover:text-white transition-colors">
                        <i class="fa-solid fa-phone text-xl"></i>
                    </div>
                    <div class="text-left flex-1">
                        <h3 class="text-lg font-bold text-gray-800">Audio Calling</h3>
                        <p class="text-xs text-gray-500">Crystal clear voice servers</p>
                    </div>
                    <i class="fa-solid fa-chevron-right text-gray-300"></i>
                </button>

                <button onclick="openChat()" class="flex items-center gap-4 bg-white p-5 rounded-2xl shadow-md hover:shadow-xl transition transform hover:scale-[1.02] border-l-4 border-green-500 group relative overflow-hidden">
                    <div class="bg-green-100 w-14 h-14 rounded-full flex items-center justify-center text-green-600 group-hover:bg-green-600 group-hover:text-white transition-colors">
                        <i class="fa-solid fa-message text-xl"></i>
                    </div>
                    <div class="text-left flex-1">
                        <h3 class="text-lg font-bold text-gray-800">Global Chat</h3>
                        <p class="text-xs text-gray-500">Text & Voice notes</p>
                    </div>
                    <span class="absolute top-4 right-4 flex h-3 w-3">
                        <span class="animate-ping absolute inline-flex h-full w-full rounded-full bg-green-400 opacity-75"></span>
                        <span class="relative inline-flex rounded-full h-3 w-3 bg-green-500"></span>
                    </span>
                </button>
            </div>
        </div>

        <!-- Screen 4: Server Selection -->
        <div id="screen-servers" class="screen bg-gray-50 p-6 overflow-y-auto">
            <div class="text-center mb-8 fade-in">
                <h2 id="server-title" class="text-2xl font-bold text-gray-800">Select Server</h2>
                <p class="text-gray-500 text-sm mt-1">Join an active room to connect.</p>
            </div>
            <div id="server-list" class="grid grid-cols-1 gap-4 pb-10 fade-in">
                <!-- Servers populated by JS -->
            </div>
        </div>

        <!-- Screen 5: A/V Calling Interface (Jitsi Wrapper) -->
        <div id="screen-call" class="screen bg-gray-900 relative">
            <div class="absolute inset-0 z-0" id="jitsi-container"></div>
            <!-- Loading overlay for Jitsi -->
            <div id="jitsi-loading" class="absolute inset-0 z-10 bg-gray-900 flex flex-col items-center justify-center text-white">
                <i class="fa-solid fa-satellite-dish animate-pulse text-4xl text-emerald-500 mb-4"></i>
                <p class="text-lg font-medium">Connecting to Server...</p>
                <p class="text-sm text-gray-400 mt-2">Please allow Camera & Mic permissions.</p>
            </div>
        </div>

        <!-- Screen 6: Chat Interface -->
        <div id="screen-chat" class="screen chat-bg relative flex-col h-full">
            <!-- Messages Area -->
            <div id="chat-messages" class="flex-1 overflow-y-auto p-4 space-y-3 pb-20 z-0 flex flex-col">
                <div class="text-center mb-4">
                    <span class="bg-yellow-100/90 text-yellow-800 text-xs font-medium px-3 py-1 rounded-md shadow-sm border border-yellow-200 backdrop-blur-sm">
                        <i class="fa-solid fa-lock mr-1"></i> Messages are end-to-end functional. Delete available after 60s.
                    </span>
                </div>
                <!-- Messages injected here -->
            </div>

            <!-- Input Area -->
            <div class="bg-[#f0f2f5] p-2 px-3 flex items-end gap-2 z-10 shrink-0 border-t border-gray-300 w-full relative">
                <!-- Recording Indicator overlay -->
                <div id="recording-indicator" class="hidden absolute inset-0 bg-white/95 z-20 flex items-center justify-between px-6 rounded-t-xl">
                    <div class="flex items-center gap-3 text-red-500 font-bold">
                        <div class="w-3 h-3 rounded-full bg-red-500 pulse-mic"></div>
                        <span id="recording-time">Recording...</span>
                    </div>
                    <button onclick="cancelRecording()" class="text-gray-500 hover:text-red-500 font-medium">Cancel</button>
                </div>

                <div class="flex-1 bg-white rounded-2xl flex items-end shadow-sm border border-gray-300 overflow-hidden focus-within:ring-1 focus-within:ring-emerald-500 transition-all">
                    <textarea id="chat-input" rows="1" placeholder="Type a message..." class="w-full max-h-24 min-h-[44px] py-3 px-4 bg-transparent outline-none resize-none text-gray-800 text-sm overflow-y-auto"></textarea>
                </div>
                
                <button id="send-btn" class="hidden w-11 h-11 bg-emerald-600 text-white rounded-full flex items-center justify-center hover:bg-emerald-700 transition shadow-md shrink-0 mb-0.5" onclick="sendTextMessage()">
                    <i class="fa-solid fa-paper-plane text-sm ml-[-2px]"></i>
                </button>

                <button id="mic-btn" class="w-11 h-11 bg-emerald-600 text-white rounded-full flex items-center justify-center hover:bg-emerald-700 transition shadow-md shrink-0 mb-0.5" onmousedown="startRecording()" onmouseup="stopRecording()" onmouseleave="stopRecording()" ontouchstart="startRecording()" ontouchend="stopRecording()">
                    <i class="fa-solid fa-microphone text-lg"></i>
                </button>
            </div>
        </div>

    </div>

    <!-- Firebase SDKs -->
    <script type="module">
        import { initializeApp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-app.js";
        import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-auth.js";
        import { getFirestore, collection, onSnapshot, addDoc, doc, deleteDoc, serverTimestamp } from "https://www.gstatic.com/firebasejs/11.6.1/firebase-firestore.js";

        // --- Configuration & Initialization ---
        const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : {
            apiKey: "AIzaSyCK7hyqt97IjyvafKqGHNSYAKlbpsNv_IQc", // Fallback for local testing if env is missing
            authDomain: "arats-3.firebaseapp.com",
            projectId: "arats-3",
        };
        const appId = typeof __app_id !== 'undefined' ? __app_id : 'connectpro-app-v1';
        
        const app = initializeApp(firebaseConfig);
        const auth = getAuth(app);
        const db = getFirestore(app);

        // --- Global State ---
        window.appState = {
            user: null,
            profile: null,
            currentView: 'loading',
            callType: null, // 'audio' or 'video'
            jitsiApi: null,
            mediaRecorder: null,
            audioChunks: [],
            isRecording: false,
            recordingInterval: null,
            recStartTime: null,
            chatUnsubscribe: null,
            deleteIntervals: []
        };

        // --- Badges & Colors Generator ---
        const badgeIcons = ['fa-ghost', 'fa-robot', 'fa-dragon', 'fa-cat', 'fa-dog', 'fa-frog', 'fa-user-ninja', 'fa-user-astronaut', 'fa-crown', 'fa-meteor'];
        const userColors = ['text-red-600', 'text-blue-600', 'text-green-600', 'text-purple-600', 'text-pink-600', 'text-orange-600', 'text-teal-600', 'text-indigo-600'];

        // --- Auth Setup ---
        const initAuth = async () => {
            try {
                if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
                    await signInWithCustomToken(auth, __initial_auth_token);
                } else {
                    await signInAnonymously(auth);
                }
            } catch (error) {
                console.error("Auth Error:", error);
                document.getElementById('loading-text').innerText = "Connection failed. Please refresh.";
                document.getElementById('loading-text').classList.add('text-red-500');
            }
        };

        onAuthStateChanged(auth, (user) => {
            if (user) {
                window.appState.user = user;
                checkProfileAndRoute();
            }
        });

        // --- Navigation & Routing ---
        window.showScreen = (screenId, showBackBtn = true) => {
            document.querySelectorAll('.screen').forEach(el => el.classList.remove('active'));
            document.getElementById(`screen-${screenId}`).classList.add('active');
            window.appState.currentView = screenId;
            
            const backBtn = document.getElementById('back-btn');
            if (showBackBtn && !['home', 'loading', 'name'].includes(screenId)) {
                backBtn.classList.remove('hidden');
            } else {
                backBtn.classList.add('hidden');
            }
        };

        window.goBack = () => {
            // Cleanups before leaving screens
            if (window.appState.jitsiApi) {
                window.appState.jitsiApi.dispose();
                window.appState.jitsiApi = null;
            }
            if (window.appState.chatUnsubscribe) {
                window.appState.chatUnsubscribe();
                window.appState.chatUnsubscribe = null;
                clearDeleteIntervals();
            }
            showScreen('home', false);
        };

        const checkProfileAndRoute = () => {
            const savedProfile = localStorage.getItem('connectProProfile');
            if (savedProfile) {
                window.appState.profile = JSON.parse(savedProfile);
                updateHeaderProfile();
                showScreen('home', false);
            } else {
                showScreen('name', false);
            }
        };

        // --- Profile Management ---
        window.saveName = () => {
            const name = document.getElementById('name-input').value.trim();
            if (name.length < 2) return alert("Please enter a valid name (min 2 chars).");
            
            const profile = {
                name: name,
                color: userColors[Math.floor(Math.random() * userColors.length)],
                badge: badgeIcons[Math.floor(Math.random() * badgeIcons.length)]
            };
            
            localStorage.setItem('connectProProfile', JSON.stringify(profile));
            window.appState.profile = profile;
            updateHeaderProfile();
            showScreen('home', false);
        };

        window.logout = () => {
            if(confirm("Change your name and avatar?")) {
                localStorage.removeItem('connectProProfile');
                window.appState.profile = null;
                document.getElementById('user-profile-badge').classList.add('hidden');
                document.getElementById('name-input').value = '';
                goBack();
                showScreen('name', false);
            }
        };

        const updateHeaderProfile = () => {
            if (!window.appState.profile) return;
            document.getElementById('header-username').innerText = window.appState.profile.name;
            const icon = document.getElementById('header-badge-icon');
            icon.className = `fa-solid ${window.appState.profile.badge}`;
            document.getElementById('user-profile-badge').classList.remove('hidden');
        };

        // --- Server Selection ---
        window.openServerSelect = (type) => {
            window.appState.callType = type;
            document.getElementById('server-title').innerText = type === 'video' ? 'Video Servers' : 'Audio Servers';
            
            const list = document.getElementById('server-list');
            list.innerHTML = '';
            
            const colorClass = type === 'video' ? 'text-purple-600 bg-purple-100 border-purple-500' : 'text-blue-600 bg-blue-100 border-blue-500';
            const btnClass = type === 'video' ? 'bg-purple-600 hover:bg-purple-700' : 'bg-blue-600 hover:bg-blue-700';
            const icon = type === 'video' ? 'fa-video' : 'fa-phone';

            for(let i=1; i<=5; i++) {
                list.innerHTML += `
                    <div class="bg-white rounded-xl shadow-sm p-4 flex items-center justify-between border-l-4 ${colorClass.split(' ').pop()} hover:shadow-md transition">
                        <div class="flex items-center gap-4">
                            <div class="w-12 h-12 rounded-full ${colorClass.split(' ').slice(0,2).join(' ')} flex items-center justify-center">
                                <i class="fa-solid ${icon} text-xl"></i>
                            </div>
                            <div>
                                <h3 class="font-bold text-gray-800">Server ${i}</h3>
                                <p class="text-xs text-green-600 font-medium"><i class="fa-solid fa-circle text-[8px] mr-1"></i>Active & Global</p>
                            </div>
                        </div>
                        <button onclick="joinCall(${i})" class="text-white px-5 py-2 rounded-lg font-medium text-sm transition shadow-sm ${btnClass}">
                            Join
                        </button>
                    </div>
                `;
            }
            showScreen('servers');
        };

        // --- Jitsi Call Implementation ---
        window.joinCall = (serverNum) => {
            showScreen('call');
            const container = document.getElementById('jitsi-container');
            container.innerHTML = ''; // Clear previous
            document.getElementById('jitsi-loading').style.display = 'flex';

            const roomName = `VpaasMagicCookie820846061386455bb3a8-${appId}-${window.appState.callType}-Room-${serverNum}`.replace(/[^a-zA-Z0-9-]/g, '');

            const domain = '8x8.vc';
            const options = {
                roomName: roomName,
                parentNode: container,
                userInfo: {
                    displayName: window.appState.profile.name
                },
                configOverwrite: {
                    startWithAudioMuted: false,
                    startWithVideoMuted: window.appState.callType === 'audio',
                    prejoinPageEnabled: false, // Instant join as requested
                    disableDeepLinking: true,
                },
                interfaceConfigOverwrite: {
                    TOOLBAR_BUTTONS: [
                        'microphone', 'camera', 'desktop', 'fullscreen',
                        'hangup', 'chat', 'settings', 'raisehand',
                        'videoquality', 'filmstrip', 'tileview'
                    ],
                    SHOW_JITSI_WATERMARK: false,
                    DEFAULT_BACKGROUND: '#111827', // Gray-900
                }
            };

            const api = new window.JitsiMeetExternalAPI(domain, options);
            window.appState.jitsiApi = api;

            api.addEventListener('videoConferenceJoined', () => {
                document.getElementById('jitsi-loading').style.display = 'none';
            });

            api.addEventListener('readyToClose', () => {
                goBack();
            });

            if (window.appState.callType === 'audio') {
                api.executeCommand('toggleVideo'); // Force video off
            }
        };

        // --- Chat Implementation ---
        
        // Input toggle (Text vs Mic button)
        const chatInput = document.getElementById('chat-input');
        const sendBtn = document.getElementById('send-btn');
        const micBtn = document.getElementById('mic-btn');

        chatInput.addEventListener('input', function() {
            this.style.height = 'auto';
            this.style.height = (this.scrollHeight) + 'px';
            if (this.value.trim().length > 0) {
                sendBtn.classList.remove('hidden');
                micBtn.classList.add('hidden');
            } else {
                sendBtn.classList.add('hidden');
                micBtn.classList.remove('hidden');
            }
        });

        chatInput.addEventListener('keydown', (e) => {
            if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                sendTextMessage();
            }
        });

        window.openChat = () => {
            showScreen('chat');
            loadChat();
        };

        const loadChat = () => {
            const chatCol = collection(db, 'artifacts', appId, 'public', 'data', 'global_chat');
            
            // Unsubscribe existing listener if any
            if(window.appState.chatUnsubscribe) window.appState.chatUnsubscribe();

            window.appState.chatUnsubscribe = onSnapshot(chatCol, (snapshot) => {
                const msgs = [];
                snapshot.forEach(doc => msgs.push({ id: doc.id, ...doc.data() }));
                // Sort by timestamp memory logic (Rule 2)
                msgs.sort((a, b) => (a.timestamp || 0) - (b.timestamp || 0));
                renderMessages(msgs);
            }, (error) => {
                console.error("Chat Error:", error);
            });
        };

        const renderMessages = (msgs) => {
            const container = document.getElementById('chat-messages');
            container.innerHTML = `
                <div class="text-center mb-4">
                    <span class="bg-yellow-100/90 text-yellow-800 text-[11px] font-medium px-3 py-1.5 rounded-md shadow-sm border border-yellow-200 backdrop-blur-sm uppercase tracking-wider">
                        <i class="fa-solid fa-lock mr-1"></i> End-to-end connected
                    </span>
                </div>
            `;
            
            clearDeleteIntervals();

            msgs.forEach(msg => {
                const isMe = msg.senderId === window.appState.user.uid;
                
                // Calculate if deletable (only me, and > 60 seconds old)
                const ageSecs = (Date.now() - (msg.timestamp || Date.now())) / 1000;
                const isDeletable = isMe && ageSecs > 60;
                
                const timeStr = msg.timestamp ? new Date(msg.timestamp).toLocaleTimeString([], {hour: '2-digit', minute:'2-digit'}) : '...';

                const msgDiv = document.createElement('div');
                msgDiv.className = `flex ${isMe ? 'justify-end' : 'justify-start'} group animate-fade-in mb-3`;
                msgDiv.id = `msg-container-${msg.id}`;
                
                let contentHtml = '';
                if (msg.type === 'text') {
                    contentHtml = `<p class="text-sm text-gray-800 whitespace-pre-wrap leading-relaxed">${escapeHtml(msg.text)}</p>`;
                } else if (msg.type === 'audio') {
                    contentHtml = `
                        <div class="flex items-center gap-2 min-w-[150px]">
                            <button onclick="playAudio('${msg.id}')" id="play-btn-${msg.id}" class="w-8 h-8 bg-emerald-500 rounded-full text-white flex items-center justify-center hover:bg-emerald-600">
                                <i class="fa-solid fa-play ml-0.5"></i>
                            </button>
                            <div class="flex-1 h-1 bg-gray-300 rounded-full overflow-hidden">
                                <div class="h-full bg-emerald-500 w-0" id="progress-${msg.id}"></div>
                            </div>
                            <audio id="audio-${msg.id}" src="${msg.audioData}" ontimeupdate="updateAudioProgress('${msg.id}')" onended="resetAudio('${msg.id}')"></audio>
                        </div>
                    `;
                }

                // Delete button logic
                const deleteBtnHtml = `<button onclick="deleteMessage('${msg.id}')" id="del-btn-${msg.id}" class="${isDeletable ? '' : 'hidden'} absolute -top-2 ${isMe ? '-left-2' : '-right-2'} bg-red-100 text-red-600 w-6 h-6 rounded-full shadow-md text-[10px] hover:bg-red-200 transition-colors z-10"><i class="fa-solid fa-trash"></i></button>`;

                msgDiv.innerHTML = `
                    <div class="relative max-w-[85%] md:max-w-[70%] rounded-xl px-3 py-2 shadow-sm ${isMe ? 'bg-[#d9fdd3] rounded-tr-none' : 'bg-white rounded-tl-none'}">
                        ${deleteBtnHtml}
                        ${!isMe ? `
                            <div class="text-[11px] font-bold ${msg.senderColor || 'text-gray-600'} mb-1 flex items-center gap-1">
                                <i class="fa-solid ${msg.senderBadge || 'fa-user'}"></i> ${escapeHtml(msg.senderName)}
                            </div>
                        ` : ''}
                        
                        ${contentHtml}
                        
                        <div class="flex items-center justify-end gap-1 mt-1 opacity-70">
                            <span class="text-[10px] text-gray-500">${timeStr}</span>
                            ${isMe ? `<i class="fa-solid fa-check-double text-[10px] text-blue-500"></i>` : ''}
                        </div>
                    </div>
                `;
                
                container.appendChild(msgDiv);

                // Setup interval to show delete button if it's mine and not yet deletable
                if (isMe && !isDeletable && msg.timestamp) {
                    const timeRemaining = 60000 - (Date.now() - msg.timestamp);
                    if (timeRemaining > 0) {
                        const t = setTimeout(() => {
                            const btn = document.getElementById(`del-btn-${msg.id}`);
                            if(btn) btn.classList.remove('hidden');
                        }, timeRemaining);
                        window.appState.deleteIntervals.push(t);
                    }
                }
            });

            // Scroll to bottom
            setTimeout(() => {
                container.scrollTop = container.scrollHeight;
            }, 100);
        };

        const clearDeleteIntervals = () => {
            window.appState.deleteIntervals.forEach(clearTimeout);
            window.appState.deleteIntervals = [];
        };

        window.sendTextMessage = async () => {
            const text = chatInput.value.trim();
            if (!text) return;
            
            chatInput.value = '';
            chatInput.style.height = 'auto'; // reset height
            sendBtn.classList.add('hidden');
            micBtn.classList.remove('hidden');

            const msgData = {
                type: 'text',
                text: text,
                senderId: window.appState.user.uid,
                senderName: window.appState.profile.name,
                senderColor: window.appState.profile.color,
                senderBadge: window.appState.profile.badge,
                timestamp: Date.now() // Client time for sorting
            };

            try {
                await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'global_chat'), msgData);
            } catch (error) {
                console.error("Send error:", error);
                alert("Failed to send message.");
            }
        };

        window.deleteMessage = async (msgId) => {
            if(confirm("Delete this message for everyone?")) {
                try {
                    await deleteDoc(doc(db, 'artifacts', appId, 'public', 'data', 'global_chat', msgId));
                } catch (e) {
                    console.error(e);
                    alert("Cannot delete.");
                }
            }
        };

        // --- Audio Recording Features ---
        window.startRecording = async (e) => {
            if(e) e.preventDefault(); // prevent touch events from firing mouse events
            if(window.appState.isRecording) return;
            
            try {
                const stream = await navigator.mediaDevices.getUserMedia({ audio: true });
                window.appState.mediaRecorder = new MediaRecorder(stream);
                window.appState.audioChunks = [];
                
                window.appState.mediaRecorder.ondataavailable = event => {
                    if (event.data.size > 0) window.appState.audioChunks.push(event.data);
                };

                window.appState.mediaRecorder.onstop = () => processAudioAndSend();
                
                window.appState.mediaRecorder.start();
                window.appState.isRecording = true;
                window.appState.recStartTime = Date.now();
                
                // UI Updates
                document.getElementById('recording-indicator').classList.remove('hidden');
                document.getElementById('recording-time').innerText = "0:00";
                
                window.appState.recordingInterval = setInterval(() => {
                    const elapsed = Math.floor((Date.now() - window.appState.recStartTime) / 1000);
                    const mins = Math.floor(elapsed / 60);
                    const secs = (elapsed % 60).toString().padStart(2, '0');
                    document.getElementById('recording-time').innerText = `${mins}:${secs}`;
                    
                    // Force stop after 30 seconds to prevent massive base64 strings
                    if(elapsed >= 30) {
                        stopRecording();
                        document.getElementById('recording-time').innerText = "Max limit reached.";
                    }
                }, 1000);

            } catch (err) {
                console.error("Mic access denied", err);
                alert("Please allow microphone access to record voice notes.");
            }
        };

        window.stopRecording = () => {
            if (!window.appState.isRecording || !window.appState.mediaRecorder) return;
            
            clearInterval(window.appState.recordingInterval);
            document.getElementById('recording-indicator').classList.add('hidden');
            
            if (window.appState.mediaRecorder.state !== 'inactive') {
                window.appState.mediaRecorder.stop();
            }
            window.appState.isRecording = false;
            
            // Stop all mic tracks
            window.appState.mediaRecorder.stream.getTracks().forEach(t => t.stop());
        };

        window.cancelRecording = () => {
            if (!window.appState.isRecording || !window.appState.mediaRecorder) return;
            clearInterval(window.appState.recordingInterval);
            document.getElementById('recording-indicator').classList.add('hidden');
            
            // Nullify chunks so processAudioAndSend ignores it
            window.appState.audioChunks = [];
            window.appState.mediaRecorder.stop();
            window.appState.isRecording = false;
            window.appState.mediaRecorder.stream.getTracks().forEach(t => t.stop());
        };

        const processAudioAndSend = () => {
            if (window.appState.audioChunks.length === 0) return; // cancelled
            
            const audioBlob = new Blob(window.appState.audioChunks, { type: 'audio/webm' });
            
            // Prevent empty clicks
            if(audioBlob.size < 1000) return; 

            const reader = new FileReader();
            reader.readAsDataURL(audioBlob);
            reader.onloadend = async () => {
                const base64Audio = reader.result;
                
                const msgData = {
                    type: 'audio',
                    audioData: base64Audio,
                    senderId: window.appState.user.uid,
                    senderName: window.appState.profile.name,
                    senderColor: window.appState.profile.color,
                    senderBadge: window.appState.profile.badge,
                    timestamp: Date.now()
                };

                try {
                    await addDoc(collection(db, 'artifacts', appId, 'public', 'data', 'global_chat'), msgData);
                } catch (error) {
                    console.error("Send audio error:", error);
                    alert("Failed to send voice note. (Size limit exceeded?)");
                }
            };
        };

        // Custom Audio Player logic for UI
        window.playAudio = (id) => {
            const audioEl = document.getElementById(`audio-${id}`);
            const btnIcon = document.querySelector(`#play-btn-${id} i`);
            
            if (audioEl.paused) {
                // Pause all other audios
                document.querySelectorAll('audio').forEach(a => {
                    if(a.id !== `audio-${id}`) {
                        a.pause();
                        document.querySelector(`#play-btn-${a.id.split('-')[1]} i`).className = 'fa-solid fa-play ml-0.5';
                    }
                });
                audioEl.play();
                btnIcon.className = 'fa-solid fa-pause';
            } else {
                audioEl.pause();
                btnIcon.className = 'fa-solid fa-play ml-0.5';
            }
        };

        window.updateAudioProgress = (id) => {
            const audioEl = document.getElementById(`audio-${id}`);
            const progress = document.getElementById(`progress-${id}`);
            if(audioEl.duration) {
                const percent = (audioEl.currentTime / audioEl.duration) * 100;
                progress.style.width = `${percent}%`;
            }
        };

        window.resetAudio = (id) => {
            const btnIcon = document.querySelector(`#play-btn-${id} i`);
            btnIcon.className = 'fa-solid fa-play ml-0.5';
            document.getElementById(`progress-${id}`).style.width = '0%';
        };

        const escapeHtml = (unsafe) => {
            return (unsafe || '').replace(/&/g, "&amp;").replace(/</g, "&lt;").replace(/>/g, "&gt;").replace(/"/g, "&quot;").replace(/'/g, "&#039;");
        };

        // Init process
        initAuth();

    </script>
</body>
</html>
