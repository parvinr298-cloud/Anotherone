<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>SideQuest | Bridge Education & Employability</title>
  
  <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/10.12.0/firebase-firestore-compat.js"></script>

  <style>
    :root {
      --primary: #4f46e5;
      --primary-hover: #4338ca;
      --bg-main: #f8fafc;
      --bg-card: #ffffff;
      --text-main: #0f172a;
      --text-muted: #64748b;
      --border: #e2e8f0;
      --accent-spark: #ffd700;
    }

    * { box-sizing: border-box; margin: 0; padding: 0; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif; }
    body { background-color: var(--bg-main); color: var(--text-main); display: flex; flex-direction: column; min-height: 100vh; overflow-x: hidden; }
    
    /* Layout & Navigation */
    header { background: var(--bg-card); border-bottom: 1px solid var(--border); padding: 15px 30px; display: flex; justify-content: space-between; align-items: center; }
    
    /* FUN ANIMATION: Rocket tilt on logo hover */
    .logo { font-size: 24px; font-weight: bold; color: var(--primary); display: flex; align-items: center; gap: 8px; cursor: pointer; }
    .logo span { transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); display: inline-block; }
    .logo:hover span { transform: translate(4px, -4px) rotate(15deg) scale(1.2); }
    
    .user-badge { font-size: 14px; color: var(--text-muted); display: flex; align-items: center; gap: 10px; }
    
    /* FUN ANIMATION: Micro-pops on buttons */
    .btn { padding: 8px 16px; border-radius: 6px; border: none; cursor: pointer; font-weight: 500; font-size: 14px; transition: all 0.2s cubic-bezier(0.175, 0.885, 0.32, 1.275); }
    .btn-primary { background: var(--primary); color: white; box-shadow: 0 4px 6px -1px rgba(79, 70, 229, 0.2); }
    .btn-primary:hover { background: var(--primary-hover); transform: translateY(-2px); box-shadow: 0 6px 12px -1px rgba(79, 70, 229, 0.4); }
    .btn-primary:active { transform: translateY(1px); }
    
    .btn-secondary { background: transparent; border: 1px solid var(--border); color: var(--text-muted); }
    .btn-secondary:hover { background: #f1f5f9; transform: translateY(-1px); }
    .btn-danger { background: #ef4444; color: white; }
    .btn-danger:hover { background: #dc2626; transform: scale(1.03); }

    .main-container { display: flex; flex: 1; max-width: 1400px; width: 100%; margin: 0 auto; padding: 20px; gap: 20px; }
    
    /* Sidebar Filters */
    aside { width: 260px; background: var(--bg-card); border: 1px solid var(--border); padding: 20px; border-radius: 12px; height: fit-content; }
    .filter-group { margin-bottom: 20px; }
    .filter-group h4 { margin-bottom: 10px; color: var(--text-muted); font-size: 12px; text-transform: uppercase; letter-spacing: 0.05em; }
    
    /* FUN ANIMATION: Smooth sidebar sliding tags */
    .category-btn { display: block; width: 100%; text-align: left; padding: 10px; margin: 4px 0; border: none; background: transparent; border-radius: 6px; cursor: pointer; color: var(--text-main); font-size: 14px; transition: all 0.2s ease; position: relative; padding-left: 12px; }
    .category-btn::before { content: '•'; position: absolute; left: -10px; opacity: 0; transition: all 0.2s ease; color: var(--primary); }
    .category-btn:hover { background: #f1f5f9; padding-left: 16px; }
    .category-btn:hover::before { opacity: 1; left: 6px; }
    .category-btn.active { background: #e0e7ff; color: var(--primary); font-weight: 600; padding-left: 18px; }
    .category-btn.active::before { opacity: 1; left: 6px; content: '⚡'; }

    /* Main Content Area */
    main { flex: 1; }
    .action-bar { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
    
    /* FUN ANIMATION: Smooth fade and scale overlay entry */
    .auth-overlay { position: fixed; inset: 0; background: rgba(15, 23, 42, 0.6); backdrop-filter: blur(4px); display: none; justify-content: center; align-items: center; z-index: 1000; animation: fadeIn 0.2s ease forwards; }
    .auth-modal { background: var(--bg-card); padding: 30px; border-radius: 16px; width: 100%; max-width: 400px; box-shadow: 0 20px 25px -5px rgba(0,0,0,0.1), 0 10px 10px -5px rgba(0,0,0,0.04); animation: modalPop 0.3s cubic-bezier(0.34, 1.56, 0.64, 1) forwards; }
    .auth-modal h3 { margin-bottom: 15px; font-size: 22px; display: flex; gap: 8px; align-items: center; }
    .form-group { margin-bottom: 15px; }
    .form-group label { display: block; margin-bottom: 5px; font-size: 14px; color: var(--text-muted); }
    
    input, select, textarea { width: 100%; padding: 10px 14px; border: 2px solid var(--border); border-radius: 8px; font-size: 14px; transition: all 0.2s; outline: none; }
    input:focus, select:focus, textarea:focus { border-color: var(--primary); box-shadow: 0 0 0 4px rgba(79, 70, 229, 0.15); transform: translateY(-1px); }

    /* Quest Cards */
    .quest-grid { display: grid; grid-template-columns: 1fr; gap: 15px; }
    
    /* FUN ANIMATION: Wiggle and lift on cards */
    .quest-card { background: var(--bg-card); border: 1px solid var(--border); border-radius: 12px; padding: 20px; display: flex; flex-direction: column; gap: 12px; position: relative; transition: all 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.1); animation: cardReveal 0.4s ease forwards; }
    .quest-card:hover { transform: translateY(-4px) scale(1.01); box-shadow: 0 12px 20px -5px rgba(0,0,0,0.08); border-color: #cbd5e1; }
    .quest-header { display: flex; justify-content: space-between; align-items: flex-start; }
    .quest-tags { display: flex; gap: 8px; flex-wrap: wrap; }
    .tag { font-size: 11px; padding: 4px 10px; border-radius: 20px; font-weight: 700; text-transform: uppercase; letter-spacing: 0.02em; }
    .tag-type { background: #e0e7ff; color: var(--primary); }
    .tag-stipend { background: #dcfce7; color: #15803d; animation: pulseGlow 2s infinite; }
    
    /* Animation Keyframes */
    @keyframes fadeIn { from { opacity: 0; } to { opacity: 1; } }
    @keyframes modalPop { from { transform: scale(0.85); opacity: 0; } to { transform: scale(1); opacity: 1; } }
    @keyframes cardReveal { from { transform: translateY(15px); opacity: 0; } to { transform: translateY(0); opacity: 1; } }
    @keyframes pulseGlow { 0% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0.2); } 70% { box-shadow: 0 0 0 6px rgba(34, 197, 94, 0); } 100% { box-shadow: 0 0 0 0 rgba(34, 197, 94, 0); } }

    /* Confetti/Success Notification style */
    .toast-success { position: fixed; bottom: 20px; right: 20px; background: #10b981; color: white; padding: 12px 24px; border-radius: 8px; box-shadow: 0 10px 15px -3px rgba(0,0,0,0.1); display: none; z-index: 2000; font-weight: bold; animation: modalPop 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275); }

    /* Responsive design */
    @media (max-width: 768px) {
      .main-container { flex-direction: column; }
      aside { width: 100%; }
    }
  </style>
</head>
<body>

  <header>
    <div class="logo" onclick="location.reload()"><span>🚀</span> SideQuest</div>
    <div id="user-section" class="user-badge">
      <!-- State handled by JS below -->
      <button class="btn btn-secondary" onclick="openAuthModal('login')">Log In</button>
      <button class="btn btn-primary" onclick="openAuthModal('signup')">Sign Up</button>
    </div>
  </header>

  <div class="main-container">
    
    <aside>
      <div class="filter-group">
        <h4>Explore Quests</h4>
        <button class="category-btn active" onclick="filterCategory('all', this)">🌐 All Opportunities</button>
        <button class="category-btn" onclick="filterCategory('job', this)">💼 Part-Time Work</button>
        <button class="category-btn" onclick="filterCategory('project', this)">🛠️ Micro-Projects</button>
      </div>
    </aside>

    <main>
      <div class="action-bar">
        <h2 id="view-title">Available Quests</h2>
        <button class="btn btn-primary" onclick="openCreateQuestModal()">+ Post a Quest</button>
      </div>

      <!-- Live-Filtering Quest Grid -->
      <div class="quest-grid" id="quest-container">
        <div class="quest-card" data-category="job">
          <div class="quest-header">
            <h3>Social Media Assistant</h3>
            <span class="tag tag-stipend">$20/hr</span>
          </div>
          <p>Help local business grow their digital ecosystem via creative visual design and schedules.</p>
          <div class="quest-tags">
            <span class="tag tag-type">Part-Time Work</span>
            <span class="tag" style="background:#f1f5f9; color:#475569;">Marketing</span>
          </div>
          <button class="btn btn-primary" style="margin-top:10px; width:100%;" onclick="applyToQuest('Social Media Assistant')">Accept Quest ⚔️</button>
        </div>

        <div class="quest-card" data-category="project">
          <div class="quest-header">
            <h3>Landing Page Development</h3>
            <span class="tag tag-stipend">$350 Fixed</span>
          </div>
          <p>Build a clean, highly converting single-page website using semantic HTML and CSS grids.</p>
          <div class="quest-tags">
            <span class="tag tag-type">Micro-Project</span>
            <span class="tag" style="background:#f1f5f9; color:#475569;">Web Dev</span>
          </div>
          <button class="btn btn-primary" style="margin-top:10px; width:100%;" onclick="applyToQuest('Landing Page Development')">Accept Quest ⚔️</button>
        </div>
      </div>
    </main>

  </div>

  <!-- AUTH INTERACTIVE MODAL -->
  <div class="auth-overlay" id="authOverlay" onclick="closeAuthModal(event)">
    <div class="auth-modal" onclick="event.stopPropagation()">
      <h3 id="modalTitle">🌟 Join SideQuest</h3>
      <form id="authForm" onsubmit="handleAuthSubmit(event)">
        <div class="form-group" id="nameGroup">
          <label>Full Name</label>
          <input type="text" id="authName" placeholder="Alex Mercer">
        </div>
        <div class="form-group">
          <label>Email Address</label>
          <input type="email" id="authEmail" required placeholder="you@example.com">
        </div>
        <div class="form-group">
          <label>Password</label>
          <input type="password" id="authPassword" required placeholder="••••••••">
        </div>
        <button type="submit" class="btn btn-primary" style="width:100%; padding:12px; font-size:16px;" id="authSubmitBtn">Create Account 🎉</button>
      </form>
      <p style="text-align:center; margin-top:15px; font-size:13px; color:var(--text-muted); cursor:pointer;" id="switchAuthLink" onclick="toggleAuthMode()">Already have an account? Log In</p>
    </div>
  </div>

  <!-- SUCCESS POPUP TOAST -->
  <div class="toast-success" id="successToast">Success Alert! 🚀</div>

  <!-- APP FUNCTIONALITY JAVASCRIPT -->
  <script>
    let currentAuthMode = 'signup';
    let loggedInUser = null;

    function openAuthModal(mode) {
      currentAuthMode = mode;
      const overlay = document.getElementById('authOverlay');
      const title = document.getElementById('modalTitle');
      const btn = document.getElementById('authSubmitBtn');
      const nameGroup = document.getElementById('nameGroup');
      const switchLink = document.getElementById('switchAuthLink');
      
      overlay.style.display = 'flex';
      
      if(mode === 'login') {
        title.innerHTML = '🔑 Welcome Back';
        btn.innerText = 'Log In ⚡';
        nameGroup.style.display = 'none';
        switchLink.innerText = "Don't have an account? Sign Up";
      } else {
        title.innerHTML = '🌟 Join SideQuest';
        btn.innerText = 'Create Account 🎉';
        nameGroup.style.display = 'block';
        switchLink.innerText = 'Already have an account? Log In';
      }
    }

    function toggleAuthMode() {
      openAuthModal(currentAuthMode === 'signup' ? 'login' : 'signup');
    }

    function closeAuthModal(e) {
      document.getElementById('authOverlay').style.display = 'none';
    }

    function handleAuthSubmit(event) {
      event.preventDefault();
      const email = document.getElementById('authEmail').value;
      const name = document.getElementById('authName').value || email.split('@')[0];
      
      loggedInUser = { name: name, email: email };
      closeAuthModal();
      updateUserHeader();
      showToast(`Welcome aboard, ${name}! 🚀`);
    }

    function updateUserHeader() {
      const section = document.getElementById('user-section');
      if(loggedInUser) {
        section.innerHTML = `
          <span style="font-weight:600; color:var(--primary)">👤 ${loggedInUser.name}</span>
          <button class="btn btn-secondary btn-danger" style="color:white" onclick="logoutUser()">Log Out</button>
        `;
      } else {
        section.innerHTML = `
          <button class="btn btn-secondary" onclick="openAuthModal('login')">Log In</button>
          <button class="btn btn-primary" onclick="openAuthModal('signup')">Sign Up</button>
        `;
      }
    }

    function logoutUser() {
      loggedInUser = null;
      updateUserHeader();
      showToast('Logged out cleanly. Come back soon! 👋');
    }

    function filterCategory(category, element) {
      // Manage active classes
      document.querySelectorAll('.category-btn').forEach(btn => btn.classList.remove('active'));
      element.classList.add('active');
      
      // Filter the items
      const cards = document.querySelectorAll('.quest-card');
      cards.forEach(card => {
        if(category === 'all' || card.getAttribute('data-category') === category) {
          card.style.display = 'flex';
        } else {
          card.style.display = 'none';
        }
      });
    }

    function applyToQuest(questName) {
      if(!loggedInUser) {
        openAuthModal('signup');
        showToast('Please sign up or sign in to accept this quest! ⚔️');
        return;
      }
      showToast(`Quest Accepted: "${questName}"! Check your email dashboard. 📬`);
    }

    function openCreateQuestModal() {
      if(!loggedInUser) {
        openAuthModal('signup');
        return;
      }
      showToast('Quest Submission module unlocked! (Mock Database active) 🚀');
    }

    function showToast(message) {
      const toast = document.getElementById('successToast');
      toast.innerText = message;
      toast.style.display = 'block';
      setTimeout(() => {
        toast.style.display = 'none';
      }, 3500);
    }
  </script>
</body>
</html>

