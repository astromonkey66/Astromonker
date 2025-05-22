# Astromonker
<!DOCTYPE html>
<html lang="de">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>AstroMonker</title>
  <style>
    body {
      font-family: sans-serif;
      background-color: #0b0c10;
      color: #ffffff;
      margin: 0;
      padding: 1rem;
    }
    header {
      background-color: #1f2833;
      padding: 1rem;
      text-align: center;
    }
    h1 { color: #66fcf1; }
    button {
      background-color: #45a29e;
      color: white;
      border: none;
      padding: 0.5rem 1rem;
      margin: 0.5rem;
      cursor: pointer;
    }
    textarea {
      width: 100%;
      height: 100px;
      margin-bottom: 0.5rem;
    }
    #post-list { margin-top: 1rem; }
    .post {
      background: #1f2833;
      padding: 1rem;
      margin-bottom: 1rem;
      border-left: 3px solid #66fcf1;
    }
  </style>
  <!-- Firebase SDKs -->
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-app-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-auth-compat.js"></script>
  <script src="https://www.gstatic.com/firebasejs/9.23.0/firebase-firestore-compat.js"></script>
</head>
<body>
  <header>
    <h1>AstroMonker</h1>
    <button id="login-btn">Mit Google anmelden</button>
    <button id="logout-btn" style="display:none;">Abmelden</button>
  </header>

  <main>
    <section id="new-post" style="display:none;">
      <h2>Neue Idee</h2>
      <textarea id="post-text" placeholder="Deine astronomische Idee..."></textarea>
      <button id="submit-post">Veröffentlichen</button>
    </section>
    <section id="posts">
      <h2>Ideen aus dem Universum</h2>
      <div id="post-list"></div>
    </section>
  </main>

  <script>
    // DEINE FIREBASE-KONFIGURATION HIER EINTRAGEN
    const firebaseConfig = {
      apiKey: "DEIN_API_KEY",
      authDomain: "DEIN_AUTH_DOMAIN",
      projectId: "DEIN_PROJECT_ID",
      storageBucket: "DEIN_BUCKET",
      messagingSenderId: "DEIN_SENDER_ID",
      appId: "DEINE_APP_ID"
    };

    firebase.initializeApp(firebaseConfig);
    const auth = firebase.auth();
    const db = firebase.firestore();

    const loginBtn = document.getElementById("login-btn");
    const logoutBtn = document.getElementById("logout-btn");
    const newPostSection = document.getElementById("new-post");
    const postText = document.getElementById("post-text");
    const submitPost = document.getElementById("submit-post");
    const postList = document.getElementById("post-list");

    loginBtn.onclick = () => {
      const provider = new firebase.auth.GoogleAuthProvider();
      auth.signInWithPopup(provider);
    };

    logoutBtn.onclick = () => auth.signOut();

    submitPost.onclick = async () => {
      const text = postText.value.trim();
      if (text) {
        await db.collection("posts").add({
          text,
          author: auth.currentUser.displayName,
          timestamp: firebase.firestore.FieldValue.serverTimestamp()
        });
        postText.value = "";
      }
    };

    auth.onAuthStateChanged(user => {
      if (user) {
        loginBtn.style.display = "none";
        logoutBtn.style.display = "inline-block";
        newPostSection.style.display = "block";
      } else {
        loginBtn.style.display = "inline-block";
        logoutBtn.style.display = "none";
        newPostSection.style.display = "none";
      }
    });

    function renderPosts() {
      db.collection("posts").orderBy("timestamp", "desc").onSnapshot(snapshot => {
        postList.innerHTML = "";
        snapshot.forEach(doc => {
          const post = doc.data();
          const div = document.createElement("div");
          div.className = "post";
          div.innerHTML = `<strong>${post.author}:</strong><p>${post.text}</p>`;
          postList.appendChild(div);
        });
      });
    }

    renderPosts();
  </script>
</body>
</html>