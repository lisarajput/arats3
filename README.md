import React, { useState, useEffect, useRef } from 'react';
import {
  Phone,
  Video,
  MessageCircle,
  User,
  Send,
  Paperclip,
  Check,
  CheckCheck,
  Pin,
  X,
  Image as ImageIcon,
  FileText,
  Clock,
  LogOut,
  ChevronLeft
} from 'lucide-react';

// --- FIREBASE SETUP ---
import { initializeApp } from 'firebase/app';
import { getAuth, signInAnonymously, signInWithCustomToken, onAuthStateChanged } from 'firebase/auth';
import { getFirestore, collection, onSnapshot, addDoc, doc, setDoc, deleteDoc, serverTimestamp } from 'firebase/firestore';

// Fallback to user's config if environment config is not provided
const userFirebaseConfig = {
  apiKey: "AIzaSyCK7hyqt97IjyvafKqGHNSYAKlbpsNv_IQc",
  authDomain: "arats-3.firebaseapp.com",
  databaseURL: "https://arats-3-default-rtdb.firebaseio.com",
  projectId: "arats-3",
  storageBucket: "arats-3.firebasestorage.app",
  messagingSenderId: "821801850941",
  appId: "1:821801850941:web:280ac07911e64814f8b254"
};

const firebaseConfig = typeof __firebase_config !== 'undefined' ? JSON.parse(__firebase_config) : userFirebaseConfig;
const app = initializeApp(firebaseConfig);
const auth = getAuth(app);
const db = getFirestore(app);
const globalAppId = typeof __app_id !== 'undefined' ? __app_id : 'default-arats-app';

export default function App() {
  const [user, setUser] = useState(null);
  const [userName, setUserName] = useState(localStorage.getItem('userName') || '');
  const [currentView, setCurrentView] = useState('home'); // home, namePrompt, serverSelect, calling, chat
  const [pendingAction, setPendingAction] = useState(null); // 'audio', 'video', 'chat'
  const [selectedServer, setSelectedServer] = useState(null);
  const [loadingText, setLoadingText] = useState('Firebase connect ho raha hai...');

  // Initialize Firebase Auth
  useEffect(() => {
    const initAuth = async () => {
      try {
        if (typeof __initial_auth_token !== 'undefined' && __initial_auth_token) {
          await signInWithCustomToken(auth, __initial_auth_token);
        } else {
          await signInAnonymously(auth);
        }
      } catch (error) {
        console.error("Auth Error:", error);
        setLoadingText("Connection error. Kripya page refresh karein.");
      }
    };
    initAuth();

    const unsubscribe = onAuthStateChanged(auth, (user) => {
      setUser(user);
      if (user) setLoadingText('');
    });
    return () => unsubscribe();
  }, []);

  const handleFeatureClick = (feature) => {
    if (!userName) {
      setPendingAction(feature);
      setCurrentView('namePrompt');
    } else {
      proceedToFeature(feature);
    }
  };

  const proceedToFeature = (feature) => {
    if (feature === 'chat') {
      setCurrentView('chat');
    } else {
      setPendingAction(feature);
      setCurrentView('serverSelect');
    }
  };

  const saveNameAndProceed = (name) => {
    if (name.trim().length < 2) return;
    setUserName(name.trim());
    localStorage.setItem('userName', name.trim());
    proceedToFeature(pendingAction);
  };

  const joinServer = (serverId) => {
    setSelectedServer(serverId);
    setCurrentView('calling');
  };

  const goBack = () => {
    if (currentView === 'calling' || currentView === 'chat' || currentView === 'serverSelect') {
      setCurrentView('home');
      setSelectedServer(null);
      setPendingAction(null);
    } else if (currentView === 'namePrompt') {
      setCurrentView('home');
      setPendingAction(null);
    }
  };

  if (!user && loadingText) {
    return (
      <div className="min-h-screen flex items-center justify-center bg-gray-50 text-gray-800">
        <div className="text-center">
          <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-green-600 mx-auto mb-4"></div>
          <p className="text-lg font-medium">{loadingText}</p>
        </div>
      </div>
    );
  }

  return (
    <div className="min-h-screen bg-gray-100 font-sans text-gray-900 flex flex-col">
      {/* App Header */}
      <header className="bg-emerald-600 text-white p-4 shadow-md flex items-center justify-between z-10">
        <div className="flex items-center gap-3">
          {(currentView !== 'home' && currentView !== 'namePrompt') && (
            <button onClick={goBack} className="p-1 hover:bg-emerald-700 rounded-full transition">
              <ChevronLeft size={24} />
            </button>
          )}
          <h1 className="text-xl font-bold tracking-wide">ConnectPro</h1>
        </div>
        {userName && currentView !== 'namePrompt' && (
          <div className="flex items-center gap-2 bg-emerald-700 px-3 py-1 rounded-full text-sm">
            <User size={16} />
            <span className="truncate max-w-[100px]">{userName}</span>
            <button 
              onClick={() => { setUserName(''); localStorage.removeItem('userName'); setCurrentView('home'); }}
              className="ml-2 hover:text-red-300 transition"
              title="Naam change karein"
            >
              <LogOut size={14} />
            </button>
          </div>
        )}
      </header>

      {/* Main Content Area */}
      <main className="flex-1 overflow-hidden relative flex flex-col">
        {currentView === 'home' && <HomeView onFeatureClick={handleFeatureClick} />}
        {currentView === 'namePrompt' && <NamePromptView onSave={saveNameAndProceed} onCancel={goBack} />}
        {currentView === 'serverSelect' && <ServerSelectionView type={pendingAction} onJoin={joinServer} />}
        {currentView === 'calling' && <CallingView type={pendingAction} server={selectedServer} userName={userName} onLeave={goBack} />}
        {currentView === 'chat' && <ChatView userName={userName} userId={user.uid} />}
      </main>
    </div>
  );
}

// ==========================================
// VIEWS
// ==========================================

function HomeView({ onFeatureClick }) {
  return (
    <div className="flex-1 flex flex-col items-center justify-center p-6 bg-gradient-to-br from-green-50 to-emerald-100">
      <div className="max-w-md w-full text-center mb-10">
        <h2 className="text-3xl font-extrabold text-gray-800 mb-2">Aapka Swagat Hai!</h2>
        <p className="text-gray-600">Kisi bhi feature par click karke doston ke sath connect karein.</p>
      </div>

      <div className="grid grid-cols-1 gap-6 w-full max-w-sm">
        <button 
          onClick={() => onFeatureClick('audio')}
          className="flex items-center gap-4 bg-white p-5 rounded-2xl shadow-lg hover:shadow-xl hover:scale-105 transition transform duration-200 border-l-4 border-blue-500"
        >
          <div className="bg-blue-100 p-4 rounded-full text-blue-600">
            <Phone size={32} />
          </div>
          <div className="text-left">
            <h3 className="text-xl font-bold text-gray-800">Audio Calling</h3>
            <p className="text-sm text-gray-500">HD voice ke sath 5 servers</p>
          </div>
        </button>

        <button 
          onClick={() => onFeatureClick('video')}
          className="flex items-center gap-4 bg-white p-5 rounded-2xl shadow-lg hover:shadow-xl hover:scale-105 transition transform duration-200 border-l-4 border-purple-500"
        >
          <div className="bg-purple-100 p-4 rounded-full text-purple-600">
            <Video size={32} />
          </div>
          <div className="text-left">
            <h3 className="text-xl font-bold text-gray-800">Video Calling</h3>
            <p className="text-sm text-gray-500">Zoom jaise features ke sath</p>
          </div>
        </button>

        <button 
          onClick={() => onFeatureClick('chat')}
          className="flex items-center gap-4 bg-white p-5 rounded-2xl shadow-lg hover:shadow-xl hover:scale-105 transition transform duration-200 border-l-4 border-green-500"
        >
          <div className="bg-green-100 p-4 rounded-full text-green-600">
            <MessageCircle size={32} />
          </div>
          <div className="text-left">
            <h3 className="text-xl font-bold text-gray-800">Chatting</h3>
            <p className="text-sm text-gray-500">WhatsApp style messaging</p>
          </div>
        </button>
      </div>
    </div>
  );
}

function NamePromptView({ onSave, onCancel }) {
  const [name, setName] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    onSave(name);
  };

  return (
    <div className="absolute inset-0 flex items-center justify-center bg-black/60 p-4 z-50 backdrop-blur-sm">
      <div className="bg-white rounded-2xl shadow-2xl p-8 max-w-sm w-full transform transition-all">
        <div className="flex justify-center mb-6">
          <div className="bg-emerald-100 text-emerald-600 p-4 rounded-full">
            <User size={40} />
          </div>
        </div>
        <h2 className="text-2xl font-bold text-center text-gray-800 mb-2">Apna Naam Darj Karein</h2>
        <p className="text-center text-gray-500 mb-6 text-sm">Aage badhne ke liye naam likhna jaruri hai.</p>
        
        <form onSubmit={handleSubmit}>
          <input
            type="text"
            value={name}
            onChange={(e) => setName(e.target.value)}
            placeholder="Aapka Shubh Naam..."
            className="w-full px-4 py-3 rounded-lg border border-gray-300 focus:ring-2 focus:ring-emerald-500 focus:border-emerald-500 outline-none transition mb-6 bg-gray-50"
            autoFocus
            required
            minLength={2}
          />
          <div className="flex gap-3">
            <button 
              type="button" 
              onClick={onCancel}
              className="flex-1 py-3 px-4 bg-gray-100 text-gray-700 font-medium rounded-lg hover:bg-gray-200 transition"
            >
              Cancel
            </button>
            <button 
              type="submit" 
              className="flex-1 py-3 px-4 bg-emerald-600 text-white font-medium rounded-lg hover:bg-emerald-700 transition"
            >
              Start
            </button>
          </div>
        </form>
      </div>
    </div>
  );
}

function ServerSelectionView({ type, onJoin }) {
  const servers = [1, 2, 3, 4, 5];
  const isVideo = type === 'video';
  const colorClass = isVideo ? 'text-purple-600 bg-purple-100 border-purple-500' : 'text-blue-600 bg-blue-100 border-blue-500';
  const btnClass = isVideo ? 'bg-purple-600 hover:bg-purple-700' : 'bg-blue-600 hover:bg-blue-700';

  return (
    <div className="flex-1 overflow-y-auto p-6 bg-gray-50">
      <div className="max-w-2xl mx-auto">
        <div className="text-center mb-10">
          <h2 className="text-3xl font-bold text-gray-800">
            {isVideo ? 'Video Calling Servers' : 'Audio Calling Servers'}
          </h2>
          <p className="text-gray-600 mt-2">Kripya kisi ek server ko chunein jisme aap join hona chahte hain.</p>
        </div>

        <div className="grid grid-cols-1 md:grid-cols-2 gap-4">
          {servers.map(num => (
            <div key={num} className="bg-white rounded-xl shadow-md p-6 flex items-center justify-between border-l-4 hover:shadow-lg transition cursor-pointer" style={{borderLeftColor: isVideo ? '#8b5cf6' : '#3b82f6'}}>
              <div className="flex items-center gap-4">
                <div className={`p-3 rounded-full ${colorClass.split(' ').slice(0,2).join(' ')}`}>
                  {isVideo ? <Video size={24} /> : <Phone size={24} />}
                </div>
                <div>
                  <h3 className="text-lg font-bold text-gray-800">Server {num}</h3>
                  <p className="text-sm text-green-600 font-medium flex items-center gap-1">
                    <span className="w-2 h-2 rounded-full bg-green-500 inline-block"></span> Active & HD
                  </p>
                </div>
              </div>
              <button 
                onClick={() => onJoin(num)}
                className={`text-white px-5 py-2 rounded-lg font-medium transition ${btnClass}`}
              >
                Join
              </button>
            </div>
          ))}
        </div>
      </div>
    </div>
  );
}

// Jitsi Meet Wrapper for robust Audio/Video with Zoom features
function CallingView({ type, server, userName, onLeave }) {
  const jitsiContainerRef = useRef(null);
  const [isLoading, setIsLoading] = useState(true);

  useEffect(() => {
    // Load Jitsi API dynamically
    const script = document.createElement('script');
    script.src = 'https://8x8.vc/external_api.js';
    script.async = true;
    script.onload = initJitsi;
    document.body.appendChild(script);

    let api = null;

    function initJitsi() {
      if (!window.JitsiMeetExternalAPI) return;
      
      const domain = '8x8.vc';
      // Unique room name combining app id, type and server number to avoid overlap
      const roomName = `vpaas-magic-cookie-820846061386455bb3a8/${globalAppId}-ConnectPro-${type}-Server-${server}`;
      
      const options = {
        roomName: roomName,
        parentNode: jitsiContainerRef.current,
        userInfo: {
          displayName: userName
        },
        configOverwrite: {
          startVideoMuted: type === 'audio' ? 0 : 0, // In modern Jitsi, to force audio only we use interface config
          startWithAudioMuted: false,
          startWithVideoMuted: type === 'audio', 
          prejoinPageEnabled: false,
          disableDeepLinking: true,
        },
        interfaceConfigOverwrite: {
          TOOLBAR_BUTTONS: [
            'microphone', 'camera', 'closedcaptions', 'desktop', 'fullscreen',
            'fodeviceselection', 'hangup', 'profile', 'chat', 'recording',
            'livestreaming', 'etherpad', 'sharedvideo', 'settings', 'raisehand',
            'videoquality', 'filmstrip', 'invite', 'feedback', 'stats', 'shortcuts',
            'tileview', 'videobackgroundblur', 'download', 'help', 'mute-everyone', 'security'
          ],
          SHOW_JITSI_WATERMARK: false,
          SHOW_WATERMARK_FOR_GUESTS: false,
          DEFAULT_BACKGROUND: '#111827', // dark bg
        }
      };

      api = new window.JitsiMeetExternalAPI(domain, options);
      
      api.addEventListener('videoConferenceJoined', () => {
        setIsLoading(false);
      });

      api.addEventListener('readyToClose', () => {
        onLeave();
      });

      // Force video off for audio calls
      if (type === 'audio') {
        api.executeCommand('toggleVideo');
      }
    }

    return () => {
      if (api) api.dispose();
      if (script.parentNode) script.parentNode.removeChild(script);
    };
  }, [type, server, userName]);

  return (
    <div className="flex-1 bg-gray-900 relative flex flex-col">
      <div className="bg-gray-800 text-white p-3 flex justify-between items-center shadow-md">
        <div className="flex items-center gap-2">
          {type === 'video' ? <Video size={20} className="text-purple-400" /> : <Phone size={20} className="text-blue-400" />}
          <span className="font-semibold">{type === 'video' ? 'Video' : 'Audio'} Server {server}</span>
          <span className="text-xs bg-green-600/20 text-green-400 px-2 py-0.5 rounded ml-2 border border-green-600/30">HD Quality</span>
        </div>
        <button onClick={onLeave} className="bg-red-600 hover:bg-red-700 text-white px-4 py-1.5 rounded-md text-sm font-medium transition">
          Leave
        </button>
      </div>
      
      <div className="flex-1 relative">
        {isLoading && (
          <div className="absolute inset-0 flex flex-col items-center justify-center text-white bg-gray-900 z-10">
            <div className="animate-spin rounded-full h-12 w-12 border-b-2 border-white mb-4"></div>
            <p>Server se connect ho raha hai...</p>
            <p className="text-gray-400 text-sm mt-2">Kripya camera aur mic permissions allow karein.</p>
          </div>
        )}
        <div ref={jitsiContainerRef} className="w-full h-full" />
      </div>
    </div>
  );
}

// WhatsApp Clone Interface using Firestore
function ChatView({ userName, userId }) {
  const [messages, setMessages] = useState([]);
  const [inputText, setInputText] = useState('');
  const [pinnedMsg, setPinnedMsg] = useState(null);
  const [isUploading, setIsUploading] = useState(false);
  const [errorMsg, setErrorMsg] = useState('');
  const messagesEndRef = useRef(null);
  const fileInputRef = useRef(null);

  // Firestore paths
  const chatCollection = collection(db, 'artifacts', globalAppId, 'public', 'data', 'chat_messages');
  const metaDoc = doc(db, 'artifacts', globalAppId, 'public', 'data', 'chat_meta', 'info');

  useEffect(() => {
    // Listen to messages
    const unsubscribeMsgs = onSnapshot(chatCollection, (snapshot) => {
      const msgsData = [];
      snapshot.forEach(d => {
        const data = d.data();
        msgsData.push({ id: d.id, ...data });
      });
      // Sort in memory (Rule 2)
      msgsData.sort((a, b) => (a.timestamp || 0) - (b.timestamp || 0));
      setMessages(msgsData);
      scrollToBottom();
    }, (error) => {
      console.error("Chat fetch error:", error);
      setErrorMsg("Chat load nahi ho payi. Error.");
    });

    // Listen to pinned message info
    const unsubscribeMeta = onSnapshot(metaDoc, (docSnap) => {
      if (docSnap.exists()) {
        setPinnedMsg(docSnap.data().pinnedMessage);
      } else {
        setPinnedMsg(null);
      }
    });

    return () => {
      unsubscribeMsgs();
      unsubscribeMeta();
    };
  }, []);

  const scrollToBottom = () => {
    setTimeout(() => {
      messagesEndRef.current?.scrollIntoView({ behavior: "smooth" });
    }, 100);
  };

  const handleSendMessage = async (e) => {
    e?.preventDefault();
    if (!inputText.trim()) return;

    const msgData = {
      text: inputText.trim(),
      senderName: userName,
      senderId: userId,
      timestamp: Date.now(), // using client time for simple memory sorting
      type: 'text'
    };
    
    setInputText('');
    
    try {
      await addDoc(chatCollection, msgData);
    } catch (err) {
      console.error(err);
      setErrorMsg("Message send fail hua.");
    }
  };

  const handleFileUpload = (e) => {
    const file = e.target.files[0];
    if (!file) return;

    // Limit size to ~500KB to stay safe within Firestore limits and not freeze browser
    if (file.size > 500 * 1024) {
      alert("Kripya 500KB se choti file select karein. (Firestore Limits)");
      return;
    }

    setIsUploading(true);
    const reader = new FileReader();
    reader.onload = async (ev) => {
      const base64Data = ev.target.result;
      const isImage = file.type.startsWith('image/');

      const msgData = {
        text: file.name,
        senderName: userName,
        senderId: userId,
        timestamp: Date.now(),
        type: isImage ? 'image' : 'document',
        fileData: base64Data
      };

      try {
        await addDoc(chatCollection, msgData);
      } catch (err) {
        console.error(err);
        setErrorMsg("File upload fail hua.");
      } finally {
        setIsUploading(false);
      }
    };
    reader.readAsDataURL(file);
    e.target.value = null; // reset
  };

  const handlePinMessage = async (msg) => {
    try {
      await setDoc(metaDoc, {
        pinnedMessage: {
          id: msg.id,
          text: msg.type === 'text' ? msg.text : `[${msg.type === 'image' ? 'Image' : 'Document'}] ${msg.text}`,
          senderName: msg.senderName
        }
      }, { merge: true });
    } catch (err) {
      console.error(err);
    }
  };

  const handleUnpin = async () => {
    try {
      await setDoc(metaDoc, { pinnedMessage: null }, { merge: true });
    } catch (err) {
      console.error(err);
    }
  };

  const formatTime = (ts) => {
    if (!ts) return '';
    const date = new Date(ts);
    let h = date.getHours();
    let m = date.getMinutes();
    const ampm = h >= 12 ? 'PM' : 'AM';
    h = h % 12;
    h = h ? h : 12; 
    m = m < 10 ? '0' + m : m;
    return `${h}:${m} ${ampm}`;
  };

  return (
    <div className="flex-1 flex flex-col bg-[#efeae2] relative h-full">
      {/* WhatsApp Chat Background Pattern Simulation */}
      <div className="absolute inset-0 opacity-10 pointer-events-none" 
           style={{ backgroundImage: "url('https://camo.githubusercontent.com/92ebac89bc70b686e068fbaae226de3e98629550cb5287f3956e10eb8e622b7a/68747470733a2f2f7765622e77686174736170702e636f6d2f696d672f62672d636861742d74696c652d6461726b5f61346265353132653731393562366237333364393131306234303866303735642e706e67')", backgroundSize: "300px" }}>
      </div>

      {/* Pinned Message Banner */}
      {pinnedMsg && (
        <div className="bg-white/95 border-b border-gray-200 px-4 py-2 flex items-center justify-between shadow-sm z-10 sticky top-0 backdrop-blur-sm">
          <div className="flex items-center gap-2 overflow-hidden">
            <Pin size={16} className="text-gray-500 shrink-0" />
            <div className="text-sm truncate">
              <span className="font-semibold text-emerald-600">{pinnedMsg.senderName}: </span>
              <span className="text-gray-600">{pinnedMsg.text}</span>
            </div>
          </div>
          <button onClick={handleUnpin} className="text-gray-400 hover:text-gray-600 p-1">
            <X size={16} />
          </button>
        </div>
      )}

      {/* Messages Area */}
      <div className="flex-1 overflow-y-auto p-4 space-y-3 z-0">
        {errorMsg && <div className="text-center text-red-500 text-sm bg-red-50 p-2 rounded-lg">{errorMsg}</div>}
        
        {messages.length === 0 ? (
          <div className="text-center text-gray-500 mt-10 text-sm bg-white/50 w-fit mx-auto px-4 py-2 rounded-full">
            Yahan chat start karein. Sabhi messages end-to-end working hain.
          </div>
        ) : (
          messages.map((msg) => {
            const isMe = msg.senderId === userId;
            return (
              <div key={msg.id} className={`flex ${isMe ? 'justify-end' : 'justify-start'} group`}>
                <div className={`max-w-[80%] md:max-w-[60%] rounded-lg px-3 py-2 shadow-sm relative ${isMe ? 'bg-[#d9fdd3] rounded-tr-none' : 'bg-white rounded-tl-none'}`}>
                  
                  {/* Sender Name (Only for others) */}
                  {!isMe && (
                    <div className="text-xs font-bold text-emerald-600 mb-1">
                      {msg.senderName}
                    </div>
                  )}

                  {/* Message Content */}
                  {msg.type === 'text' && (
                    <p className="text-gray-800 text-sm md:text-base break-words whitespace-pre-wrap leading-relaxed">{msg.text}</p>
                  )}
                  
                  {msg.type === 'image' && (
                    <div className="mt-1 mb-1">
                      <img src={msg.fileData} alt="uploaded" className="rounded-md max-h-60 object-contain" />
                    </div>
                  )}

                  {msg.type === 'document' && (
                    <div className="flex items-center gap-2 bg-black/5 p-2 rounded mb-1 mt-1">
                      <FileText size={20} className="text-gray-600" />
                      <a href={msg.fileData} download={msg.text} className="text-blue-600 text-sm underline truncate">{msg.text}</a>
                    </div>
                  )}

                  {/* Metadata (Time & Ticks) */}
                  <div className={`flex items-center justify-end gap-1 mt-1 ${isMe ? 'text-gray-500' : 'text-gray-400'}`}>
                    <span className="text-[10px] whitespace-nowrap">{formatTime(msg.timestamp)}</span>
                    {isMe && (
                      <CheckCheck size={14} className="text-blue-500" /> // Simulating read tick
                    )}
                  </div>

                  {/* Pin button (Hover) */}
                  <button 
                    onClick={() => handlePinMessage(msg)}
                    className={`absolute top-1 ${isMe ? '-left-6' : '-right-6'} opacity-0 group-hover:opacity-100 text-gray-400 hover:text-gray-600 transition-opacity p-1 bg-white rounded-full shadow-sm border border-gray-100`}
                    title="Pin Message"
                  >
                    <Pin size={12} />
                  </button>
                </div>
              </div>
            );
          })
        )}
        <div ref={messagesEndRef} />
      </div>

      {/* Input Area */}
      <div className="bg-[#f0f2f5] p-3 flex items-end gap-2 z-10 sticky bottom-0">
        <button 
          onClick={() => fileInputRef.current?.click()}
          className="p-3 text-gray-500 hover:bg-gray-200 rounded-full transition shrink-0"
          disabled={isUploading}
        >
          {isUploading ? <div className="w-5 h-5 border-2 border-gray-400 border-t-transparent rounded-full animate-spin"></div> : <Paperclip size={22} />}
        </button>
        <input 
          type="file" 
          ref={fileInputRef} 
          onChange={handleFileUpload} 
          className="hidden" 
          accept="image/*,.pdf,.doc,.docx,.txt"
        />
        
        <form onSubmit={handleSendMessage} className="flex-1 flex items-end gap-2 bg-white rounded-2xl border border-gray-300 focus-within:ring-1 focus-within:ring-emerald-500 px-2 shadow-sm">
          <textarea
            value={inputText}
            onChange={(e) => setInputText(e.target.value)}
            onKeyDown={(e) => {
              if (e.key === 'Enter' && !e.shiftKey) {
                e.preventDefault();
                handleSendMessage();
              }
            }}
            placeholder="Message type karein..."
            className="flex-1 max-h-32 min-h-[44px] py-3 px-2 bg-transparent outline-none resize-none text-gray-800 text-sm"
            rows={1}
          />
        </form>

        <button 
          onClick={handleSendMessage}
          disabled={!inputText.trim() || isUploading}
          className="bg-emerald-600 text-white p-3 rounded-full hover:bg-emerald-700 transition shrink-0 disabled:opacity-50 disabled:cursor-not-allowed shadow-md"
        >
          <Send size={20} className="ml-1" />
        </button>
      </div>
    </div>
  );
}
