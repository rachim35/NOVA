<!DOCTYPE html>
<html lang="fr">
<head>
  <meta charset="UTF-8" />
  <title>Site de Jeu + Chat</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      background: #121212;
      color: #eee;
      margin: 0; padding: 0;
      display: flex; flex-direction: column; align-items: center;
      min-height: 100vh;
    }
    #login, #main {
      margin-top: 50px;
      width: 90%;
      max-width: 600px;
      background: #222;
      padding: 20px;
      border-radius: 10px;
      box-shadow: 0 0 10px #00ff99;
    }
    input[type=text] {
      padding: 10px;
      font-size: 1.1rem;
      border-radius: 5px;
      border: none;
      width: 70%;
      margin-right: 10px;
    }
    button {
      background: #00ff99;
      border: none;
      padding: 10px 15px;
      font-size: 1.1rem;
      border-radius: 5px;
      cursor: pointer;
      color: #121212;
      font-weight: bold;
      transition: background 0.3s ease;
    }
    button:hover {
      background: #00cc77;
    }
    #chat {
      height: 300px;
      overflow-y: auto;
      background: #111;
      padding: 10px;
      border-radius: 8px;
      margin-bottom: 10px;
    }
    .message {
      margin-bottom: 8px;
      padding: 5px 10px;
      border-radius: 6px;
      background: #00ff99;
      color: #121212;
      max-width: 80%;
      word-wrap: break-word;
    }
    .message.self {
      background: #007733;
      color: #ddd;
      margin-left: auto;
    }
    #game {
      margin-top: 20px;
      background: #222;
      padding: 15px;
      border-radius: 8px;
      text-align: center;
    }
  </style>
</head>
<body>

<div id="login">
  <h2>Connexion</h2>
  <input type="text" id="usernameInput" placeholder="Ton pseudo" maxlength="20" />
  <button onclick="login()">Entrer</button>
</div>

<div id="main" style="display:none;">
  <h2>Chat en direct</h2>
  <div id="chat"></div>
  <input type="text" id="messageInput" placeholder="Écris un message..." maxlength="200" />
  <button onclick="sendMessage()">Envoyer</button>

  <div id="game">
    <h3>Mini-jeu : Devine le Nombre</h3>
    <p>Je pense à un nombre entre 1 et 100. Essaie de deviner !</p>
    <input type="number" id="guessInput" min="1" max="100" placeholder="1-100" />
    <button onclick="guessNumber()">Deviner</button>
    <div id="messageGame"></div>
    <button onclick="resetGame()" style="margin-top:10px; background:#555; color:#eee;">Recommencer</button>
  </div>
</div>

<!-- Firebase -->
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-app-compat.js"></script>
<script src="https://www.gstatic.com/firebasejs/9.22.2/firebase-database-compat.js"></script>

<script>
  // TODO : Remplace avec ta config Firebase ici (obtenue à l'étape 1)
  const firebaseConfig = {
    apiKey: "TON_API_KEY",
    authDomain: "TON_PROJECT.firebaseapp.com",
    databaseURL: "https://TON_PROJECT.firebaseio.com",
    projectId: "TON_PROJECT",
    storageBucket: "TON_PROJECT.appspot.com",
    messagingSenderId: "TON_SENDER_ID",
    appId: "TON_APP_ID"
  };

  firebase.initializeApp(firebaseConfig);
  const db = firebase.database();

  let username = '';
  const chatRef = db.ref('chatMessages');

  function login() {
    const input = document.getElementById('usernameInput');
    if (!input.value.trim()) {
      alert('Merci de saisir un pseudo');
      return;
    }
    username = input.value.trim();
    document.getElementById('login').style.display = 'none';
    document.getElementById('main').style.display = 'block';
    loadMessages();
  }

  function loadMessages() {
    chatRef.off(); // Déconnecte les anciens listeners

    chatRef.limitToLast(50).on('child_added', snapshot => {
      const msg = snapshot.val();
      displayMessage(msg.user, msg.text);
    });
  }

  function displayMessage(user, text) {
    const chat = document.getElementById('chat');
    const div = document.createElement('div');
    div.classList.add('message');
    if(user === username) div.classList.add('self');
    div.textContent = `${user}: ${text}`;
    chat.appendChild(div);
    chat.scrollTop = chat.scrollHeight;
  }

  function sendMessage() {
    const input = document.getElementById('messageInput');
    const text = input.value.trim();
    if (!text) return;
    chatRef.push({ user: username, text: text });
    input.value = '';
  }

  // Jeu "Devine le Nombre"
  let secretNumber = Math.floor(Math.random() * 100) + 1;
  let attempts = 0;

  function guessNumber() {
    const input = document.getElementById('guessInput');
    const msg = document.getElementById('messageGame');
    const guess = Number(input.value);

    if (!guess || guess < 1 || guess > 100) {
      msg.textContent = 'Merci de saisir un nombre entre 1 et 100.';
      msg.style.color = '#ff4444';
      return;
    }

    attempts++;
    if (guess === secretNumber) {
      msg.textContent = `Bravo ${username} ! Tu as deviné en ${attempts} essai${attempts > 1 ? 's' : ''} 🎉`;
      msg.style.color = '#00ff99';
    } else if (guess < secretNumber) {
      msg.textContent = 'Trop petit, essaie encore.';
      msg.style.color = '#ff4444';
    } else {
      msg.textContent = 'Trop grand, essaie encore.';
      msg.style.color = '#ff4444';
    }
    input.value = '';
    input.focus();
  }

  function resetGame() {
    secretNumber = Math.floor(Math.random() * 100) + 1;
    attempts = 0;
    document.getElementById('messageGame').textContent = '';
    document.getElementById('guessInput').value = '';
    document.getElementById('guessInput').focus();
  }
</script>

</body>
</html>
