import React, { useState, useEffect, useRef } from 'react';
import { 
  BookOpen, 
  LayoutDashboard, 
  Bot, 
  CheckCircle2, 
  Lock, 
  User, 
  Mail, 
  LogOut, 
  ChevronDown, 
  ChevronUp, 
  Send,
  Atom,
  FlaskConical,
  Dna,
  Calculator,
  Sigma
} from 'lucide-react';

// --- Configuration & Data ---

const TARGET_DATE = new Date('2026-04-21T10:00:00');

// Helper to calculate progress
const calculateSubjectProgress = (subjectId, progressData, totalItems) => {
  if (!progressData[subjectId]) return 0;
  const completed = Object.keys(progressData[subjectId]).filter(k => progressData[subjectId][k] && progressData[subjectId][k].trim() !== '').length;
  // Each chapter has 2 inputs (Question + Task), so totalItems is chapters * 2
  return Math.min(100, Math.round((completed / totalItems) * 100));
};

const SYLLABUS_DATA = [
  {
    id: 'physics',
    name: 'Physics',
    nameBn: 'পদার্থবিজ্ঞান',
    color: 'text-cyan-400',
    borderColor: 'border-cyan-500/50',
    bgGradient: 'from-cyan-900/20 to-black',
    icon: <Atom className="w-6 h-6" />,
    chapters: [
      { id: 'p1', title: 'অধ্যায় ১: ভৌত রাশি এবং তাদের পরিমান', question: 'প্রথম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'p2', title: 'অধ্যায় ২: গতি', question: 'দ্বিতীয় অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'p3', title: 'অধ্যায় ৩: বল', question: 'তৃতীয় অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'p4', title: 'অধ্যায় ৪: কাজ ক্ষমতা ও শক্তি', question: 'চতুর্থ অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'p7', title: 'অধ্যায় ৭: তরঙ্গ ও শব্দ', question: 'সপ্তম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'p8', title: 'অধ্যায় ৮: আলোর প্রতিফলন', question: 'অষ্টম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'p10', title: 'অধ্যায় ১০: স্থির বিদ্যুৎ', question: 'দশম অধ্যায় এর সকল সূত্র লিখ।' },
    ]
  },
  {
    id: 'chemistry',
    name: 'Chemistry',
    nameBn: 'রসায়ন',
    color: 'text-pink-400',
    borderColor: 'border-pink-500/50',
    bgGradient: 'from-pink-900/20 to-black',
    icon: <FlaskConical className="w-6 h-6" />,
    chapters: [
      { id: 'c3', title: 'অধ্যায় ৩: পদার্থের গঠন', question: null }, // Special case based on prompt (only task)
      { id: 'c4', title: 'অধ্যায় ৪: পর্যায় সারনি', question: 'চতুর্থ অধ্যায় এর ৩০টি মৌলের সংকেত লিখ।' },
      { id: 'c5', title: 'অধ্যায় ৫: রাসায়নিক বন্ধন', question: null },
      { id: 'c6', title: 'অধ্যায় ৬: মোলের ধারণা ও রাসায়নিক', question: null },
      { id: 'c7', title: 'অধ্যায় ৭: রাসায়নিক বিক্রিয়া', question: null },
      { id: 'c11', title: 'অধ্যায় ১১: খনিজ সম্পদ: জীবাশ্ম', question: null },
    ]
  },
  {
    id: 'biology',
    name: 'Biology',
    nameBn: 'জীববিজ্ঞান',
    color: 'text-lime-400',
    borderColor: 'border-lime-500/50',
    bgGradient: 'from-lime-900/20 to-black',
    icon: <Dna className="w-6 h-6" />,
    chapters: [
      { id: 'b1', title: 'অধ্যায় ১: জীবন পাঠ', question: null },
      { id: 'b2', title: 'অধ্যায় ২: জীবনকোষ ও টিস্যু', question: null },
      { id: 'b3', title: 'অধ্যায় ৩: কোষ বিভাজন', question: null },
      { id: 'b4', title: 'অধ্যায় ৪: জীবনীশক্তি', question: null },
      { id: 'b11', title: 'অধ্যায় ১১: জীবের প্রজনন', question: null },
      { id: 'b12', title: 'অধ্যায় ১২: জীবের বংশগতি ও জৈব অভিব্যক্তি', question: null },
      { id: 'b13', title: 'অধ্যায় ১৩: জীবের পরিবেশ', question: null },
    ]
  },
  {
    id: 'hmath',
    name: 'Higher Math',
    nameBn: 'উচ্চতর গণিত',
    color: 'text-purple-400',
    borderColor: 'border-purple-500/50',
    bgGradient: 'from-purple-900/20 to-black',
    icon: <Calculator className="w-6 h-6" />,
    chapters: [
      { id: 'hm2', title: 'অধ্যায় ২: বিজগাণিতিক ও রাশি', question: 'দ্বিতীয় অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm7', title: 'অধ্যায় ৭: অসীম ধারা', question: 'সপ্তম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm10', title: 'অধ্যায় ১০: দ্বিপদী বিস্তৃতি', question: 'দশম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm8', title: 'অধ্যায় ৮: ত্রিকোণমিতি', question: 'অষ্টম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm12', title: 'অধ্যায় ১২: সমতলীয় ভেক্টর', question: 'দ্বাদশ অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm9', title: 'অধ্যায় ৯: সূচক ও লগারিদমীয় ফাংশন', question: 'নবম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm11', title: 'অধ্যায় ১১: স্থানাঙ্ক জ্যামিতি', question: 'একাদশ অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'hm14', title: 'অধ্যায় ১৪: সম্ভাবনা', question: 'চতুর্দশ অধ্যায় এর সকল সূত্র লিখ।' },
    ]
  },
  {
    id: 'math',
    name: 'Math',
    nameBn: 'গণিত',
    color: 'text-yellow-400',
    borderColor: 'border-yellow-500/50',
    bgGradient: 'from-yellow-900/20 to-black',
    icon: <Sigma className="w-6 h-6" />,
    chapters: [
      { id: 'm2', title: 'অধ্যায় ২: সেট ও ফাংশন', question: 'দ্বিতীয় অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm3', title: 'অধ্যায় ৩: বিজগাণিতিক ও রাশি', question: 'তৃতীয় অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm7', title: 'অধ্যায় ৭: ব্যবহারিক জ্যামিতি', question: 'সপ্তম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm8', title: 'অধ্যায় ৮: বৃত্ত', question: 'অষ্টম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm9', title: 'অধ্যায় ৯: ত্রিকোণমিতি', question: 'নবম অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm11', title: 'অধ্যায় ১১: বিজগাণিতিক অনুপাত ও সমানুপাত', question: 'একাদশ অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm16', title: 'অধ্যায় ১৬: পরিমিতি', question: 'ষোড়শ অধ্যায় এর সকল সূত্র লিখ।' },
      { id: 'm17', title: 'অধ্যায় ১৭: পরিসংখ্যান', question: 'সপ্তদশ অধ্যায় এর সকল সূত্র লিখ।' },
    ]
  }
];

// --- Main Application Component ---

export default function App() {
  // State
  const [user, setUser] = useState(null); // { name, email, password }
  const [authMode, setAuthMode] = useState('login'); // 'login' or 'register'
  const [activeTab, setActiveTab] = useState('dashboard'); // 'dashboard', 'syllabus', 'ai'
  const [selectedSubject, setSelectedSubject] = useState(null);
  const [progressData, setProgressData] = useState({});
  const [timeLeft, setTimeLeft] = useState({ days: 0, hours: 0, minutes: 0, seconds: 0 });
  const [chatMessages, setChatMessages] = useState([
    { role: 'model', text: 'আমি কিরা... মানে লাইট ইয়াগামি। তোমার পড়াশোনা নিয়ে কোনো সমস্যা? নির্ভয়ে বলো।' }
  ]);
  const [isLoadingAi, setIsLoadingAi] = useState(false);
  
  // Inputs
  const [loginForm, setLoginForm] = useState({ identifier: '', password: '' });
  const [regForm, setRegForm] = useState({ name: '', email: '', password: '' });
  const [chatInput, setChatInput] = useState('');

  // --- Effects ---

  // Load from local storage on mount
  useEffect(() => {
    const storedUser = localStorage.getItem('ssc2026_user');
    if (storedUser) setUser(JSON.parse(storedUser));
    
    const storedProgress = localStorage.getItem('ssc2026_progress');
    if (storedProgress) setProgressData(JSON.parse(storedProgress));
  }, []);

  // Timer Logic
  useEffect(() => {
    const timer = setInterval(() => {
      const now = new Date();
      const difference = TARGET_DATE - now;

      if (difference > 0) {
        setTimeLeft({
          days: Math.floor(difference / (1000 * 60 * 60 * 24)),
          hours: Math.floor((difference / (1000 * 60 * 60)) % 24),
          minutes: Math.floor((difference / 1000 / 60) % 60),
          seconds: Math.floor((difference / 1000) % 60),
        });
      }
    }, 1000);
    return () => clearInterval(timer);
  }, []);

  // Save progress
  const handleProgressChange = (subjectId, chapterId, fieldType, value) => {
    const newProgress = {
      ...progressData,
      [subjectId]: {
        ...progressData[subjectId],
        [`${chapterId}_${fieldType}`]: value
      }
    };
    setProgressData(newProgress);
    localStorage.setItem('ssc2026_progress', JSON.stringify(newProgress));
  };

  // --- Authentication Handlers ---

  const handleRegister = (e) => {
    e.preventDefault();
    if (!regForm.name || !regForm.email || !regForm.password) return alert("All fields required");
    
    const userData = { ...regForm };
    localStorage.setItem('ssc2026_user', JSON.stringify(userData));
    setUser(userData);
    alert(`Welcome ${userData.name}!`);
  };

  const handleLogin = (e) => {
    e.preventDefault();
    const storedUser = JSON.parse(localStorage.getItem('ssc2026_user'));
    
    if (storedUser && 
        (storedUser.email === loginForm.identifier || storedUser.phone === loginForm.identifier) && 
        storedUser.password === loginForm.password) {
      setUser(storedUser);
    } else {
      alert("Invalid credentials. Please register first if you haven't.");
    }
  };

  const handleLogout = () => {
    setUser(null);
    setActiveTab('dashboard');
  };

  // --- AI Handler ---

  const handleAiChat = async () => {
    if (!chatInput.trim()) return;
    
    const userMsg = { role: 'user', text: chatInput };
    setChatMessages(prev => [...prev, userMsg]);
    setChatInput('');
    setIsLoadingAi(true);

    try {
      const apiKey = ""; // Runtime Environment Key
      const systemPrompt = "You are Light Yagami from the anime Death Note. You are helping a student prepare for their SSC 2026 exam. Your tone is intelligent, calm, slightly arrogant but helpful, and confident. You MUST speak in PURE BANGLA (Bengali). Do not use English script. Solve their doubts effectively but maintain the persona.";
      
      const response = await fetch(`https://generativelanguage.googleapis.com/v1beta/models/gemini-2.5-flash-preview-09-2025:generateContent?key=${apiKey}`, {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({
          contents: [{ parts: [{ text: chatInput }] }],
          systemInstruction: { parts: [{ text: systemPrompt }] }
        })
      });

      const data = await response.json();
      const aiText = data.candidates?.[0]?.content?.parts?.[0]?.text || "নেটওয়ার্ক সমস্যা। পরে আবার চেষ্টা করো।";
      
      setChatMessages(prev => [...prev, { role: 'model', text: aiText }]);
    } catch (error) {
      setChatMessages(prev => [...prev, { role: 'model', text: "আমি এখন ব্যস্ত। কিছুক্ষণ পর আবার এসো।" }]);
    } finally {
      setIsLoadingAi(false);
    }
  };

  // --- Renderers ---

  if (!user) {
    return (
      <div className="min-h-screen bg-black text-white flex flex-col items-center justify-center p-4 font-sans relative overflow-hidden">
        {/* Abstract Background */}
        <div className="absolute top-[-20%] left-[-20%] w-[500px] h-[500px] bg-blue-900/30 rounded-full blur-[100px]" />
        <div className="absolute bottom-[-20%] right-[-20%] w-[500px] h-[500px] bg-indigo-900/30 rounded-full blur-[100px]" />

        <div className="z-10 w-full max-w-md bg-white/5 backdrop-blur-xl border border-white/10 rounded-2xl p-8 shadow-2xl">
          <h1 className="text-3xl font-bold text-center mb-2">
            <span className="text-white">SSC BATCH</span> <span className="text-blue-500">2026</span>
          </h1>
          <p className="text-gray-400 text-center mb-8">Your ultimate study companion</p>

          <div className="flex bg-black/40 rounded-lg p-1 mb-6">
            <button 
              onClick={() => setAuthMode('login')}
              className={`flex-1 py-2 rounded-md transition-all ${authMode === 'login' ? 'bg-blue-600 text-white shadow-lg' : 'text-gray-400 hover:text-white'}`}
            >
              Login
            </button>
            <button 
              onClick={() => setAuthMode('register')}
              className={`flex-1 py-2 rounded-md transition-all ${authMode === 'register' ? 'bg-blue-600 text-white shadow-lg' : 'text-gray-400 hover:text-white'}`}
            >
              Register
            </button>
          </div>

          {authMode === 'login' ? (
            <form onSubmit={handleLogin} className="space-y-4">
              <div className="space-y-2">
                <label className="text-sm text-gray-400 ml-1">Gmail or Number</label>
                <div className="relative">
                  <User className="absolute left-3 top-3 w-5 h-5 text-gray-500" />
                  <input 
                    type="text" 
                    className="w-full bg-black/50 border border-white/10 rounded-lg py-2.5 pl-10 px-4 focus:outline-none focus:border-blue-500 transition-colors"
                    placeholder="Enter your ID"
                    value={loginForm.identifier}
                    onChange={(e) => setLoginForm({...loginForm, identifier: e.target.value})}
                  />
                </div>
              </div>
              <div className="space-y-2">
                <label className="text-sm text-gray-400 ml-1">Password</label>
                <div className="relative">
                  <Lock className="absolute left-3 top-3 w-5 h-5 text-gray-500" />
                  <input 
                    type="password" 
                    className="w-full bg-black/50 border border-white/10 rounded-lg py-2.5 pl-10 px-4 focus:outline-none focus:border-blue-500 transition-colors"
                    placeholder="••••••••"
                    value={loginForm.password}
                    onChange={(e) => setLoginForm({...loginForm, password: e.target.value})}
                  />
                </div>
              </div>
              <button className="w-full bg-gradient-to-r from-blue-600 to-blue-800 text-white font-semibold py-3 rounded-lg mt-4 hover:shadow-lg hover:shadow-blue-500/30 transition-all">
                Access Dashboard
              </button>
            </form>
          ) : (
            <form onSubmit={handleRegister} className="space-y-4">
              <div className="space-y-2">
                <label className="text-sm text-gray-400 ml-1">Full Name</label>
                <div className="relative">
                  <User className="absolute left-3 top-3 w-5 h-5 text-gray-500" />
                  <input 
                    type="text" 
                    className="w-full bg-black/50 border border-white/10 rounded-lg py-2.5 pl-10 px-4 focus:outline-none focus:border-blue-500 transition-colors"
                    placeholder="Your Name"
                    value={regForm.name}
                    onChange={(e) => setRegForm({...regForm, name: e.target.value})}
                  />
                </div>
              </div>
              <div className="space-y-2">
                <label className="text-sm text-gray-400 ml-1">Gmail or Number</label>
                <div className="relative">
                  <Mail className="absolute left-3 top-3 w-5 h-5 text-gray-500" />
                  <input 
                    type="text" 
                    className="w-full bg-black/50 border border-white/10 rounded-lg py-2.5 pl-10 px-4 focus:outline-none focus:border-blue-500 transition-colors"
                    placeholder="contact@example.com"
                    value={regForm.email}
                    onChange={(e) => setRegForm({...regForm, email: e.target.value})}
                  />
                </div>
              </div>
              <div className="space-y-2">
                <label className="text-sm text-gray-400 ml-1">Create Password</label>
                <div className="relative">
                  <Lock className="absolute left-3 top-3 w-5 h-5 text-gray-500" />
                  <input 
                    type="password" 
                    className="w-full bg-black/50 border border-white/10 rounded-lg py-2.5 pl-10 px-4 focus:outline-none focus:border-blue-500 transition-colors"
                    placeholder="Create a strong password"
                    value={regForm.password}
                    onChange={(e) => setRegForm({...regForm, password: e.target.value})}
                  />
                </div>
              </div>
              <button className="w-full bg-gradient-to-r from-blue-600 to-blue-800 text-white font-semibold py-3 rounded-lg mt-4 hover:shadow-lg hover:shadow-blue-500/30 transition-all">
                Create Account
              </button>
            </form>
          )}
        </div>
      </div>
    );
  }

  // --- Main App UI ---
  return (
    <div className="min-h-screen bg-black text-white font-sans flex flex-col relative pb-20">
      
      {/* Header */}
      <header className="sticky top-0 z-50 bg-black/80 backdrop-blur-md border-b border-white/10 p-4 shadow-lg">
        <div className="flex justify-between items-center max-w-5xl mx-auto">
          <h1 className="text-xl md:text-2xl font-bold tracking-tight">
            <span className="text-white">SSC BATCH</span> <span className="text-blue-500">2026</span>
          </h1>
          <div className="flex items-center gap-3">
            <div className="text-right hidden sm:block">
              <p className="text-xs text-gray-400">Welcome,</p>
              <p className="text-sm font-semibold text-blue-400">{user.name}</p>
            </div>
            <button onClick={handleLogout} className="p-2 bg-white/5 hover:bg-red-500/20 rounded-full transition-colors text-gray-300 hover:text-red-400">
              <LogOut size={18} />
            </button>
          </div>
        </div>
      </header>

      {/* Timer Section */}
      <div className="bg-gradient-to-r from-gray-900 to-blue-900/30 py-6 border-b border-white/5">
        <div className="max-w-5xl mx-auto px-4 text-center">
          <p className="text-blue-400 text-xs tracking-[0.2em] uppercase mb-2">Time Remaining Until Exam</p>
          <div className="flex justify-center gap-4 sm:gap-8">
            {['Days', 'Hours', 'Minutes', 'Seconds'].map((unit, i) => {
              const val = Object.values(timeLeft)[i];
              return (
                <div key={unit} className="flex flex-col items-center">
                  <div className="bg-black/50 border border-blue-500/30 w-16 h-16 sm:w-20 sm:h-20 rounded-xl flex items-center justify-center mb-1 shadow-[0_0_15px_rgba(59,130,246,0.2)]">
                    <span className="text-2xl sm:text-3xl font-mono font-bold text-white">{val}</span>
                  </div>
                  <span className="text-[10px] sm:text-xs text-gray-500 uppercase">{unit}</span>
                </div>
              );
            })}
          </div>
          <p className="text-gray-500 text-xs mt-3">Target: 21 April 2026, 10:00 AM</p>
        </div>
      </div>

      <main className="flex-1 max-w-5xl mx-auto w-full p-4 overflow-y-auto">
        
        {/* DASHBOARD VIEW */}
        {activeTab === 'dashboard' && (
          <div className="space-y-6 animate-in fade-in duration-500">
            <h2 className="text-2xl font-bold mb-4 flex items-center gap-2">
              <LayoutDashboard className="text-blue-500" /> Dashboard
            </h2>
            
            {/* Overall Progress Card */}
            <div className="bg-gradient-to-br from-gray-900 to-gray-800 rounded-2xl p-6 border border-white/10 shadow-xl relative overflow-hidden">
               <div className="absolute top-0 right-0 w-32 h-32 bg-blue-500/10 rounded-full blur-2xl pointer-events-none" />
               <h3 className="text-lg font-semibold text-gray-200 mb-6">Total Syllabus Completion</h3>
               
               <div className="grid grid-cols-1 md:grid-cols-2 gap-6 items-center">
                  <div className="space-y-4">
                    {SYLLABUS_DATA.map(sub => {
                      const prog = calculateSubjectProgress(sub.id, progressData, sub.chapters.length * (sub.id === 'biology' ? 1 : 2)); 
                      // Note: Biology only has Tasks (1 input/chapter), others have Question + Task (2 inputs/chapter) - logic adjusted dynamically or simplified here.
                      // Simplified logic correction: 
                      // Physics/Math/HMath/Chem = Question + Task = 2 points per chapter
                      // But wait, Chem ch 3,5,6,7,11 only have Tasks. Ch 4 has Question. 
                      // Biology only has Tasks.
                      // Let's refine calculation inside the component mapping for better accuracy or keep simple approximation.
                 
