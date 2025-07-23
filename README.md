<!DOCTYPE html>
<html lang="fr">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Connexion simple avec rôle admin</title>
<style>
  body, html {
    margin: 0; padding: 0; font-family: Arial, sans-serif; background: #f5f5f5;
  }
  #login-page {
    display: flex; flex-direction: column; align-items: center; justify-content: center;
    height: 100vh; background: #222; color: white;
  }
  #login-page input, #login-page button {
    width: 300px; padding: 12px; margin: 8px 0; border-radius: 6px; border: none; font-size: 16px;
  }
  #login-page button {
    background: #007bff; color: white; font-weight: bold; cursor: pointer;
  }
  #main-site {
    display: none; max-width: 900px; margin: 20px auto; padding: 20px; background: white; border-radius: 10px;
  }
  #admin-controls {
    background: #ffeb3b; padding: 15px; border-radius: 8px; margin-bottom: 20px;
    display: flex; align-items: center; justify-content: space-between;
  }
  #upload-btn {
    font-size: 24px; cursor: pointer; padding: 5px 12px; background: green; color: white; border-radius: 50%;
    user-select: none;
  }
  #gallery {
    display: flex; flex-wrap: wrap; gap: 15px;
  }
  .photo-card {
    position: relative;
    width: 200px;
    border-radius: 8px;
    box-shadow: 0 4px 8px rgba(0,0,0,0.2);
    background: #fff;
    padding: 10px;
    box-sizing: border-box;
    text-align: center;
  }
  .photo-card img {
    max-width: 100%;
    border-radius: 6px;
    cursor: pointer;
    transition: transform 0.2s ease;
  }
  .photo-card img:hover {
    transform: scale(1.05);
  }
  .comment {
    margin-top: 8px;
    font-size: 14px;
    color: #333;
    min-height: 20px;
  }
  .add-comment {
    margin-top: 8px;
  }
  .add-comment input {
    width: 100%;
    padding: 5px 6px;
    font-size: 14px;
    border-radius: 4px;
    border: 1px solid #ccc;
    box-sizing: border-box;
  }
  .add-comment button {
    margin-top: 4px;
    width: 100%;
    padding: 6px;
    background: #007bff;
    color: white;
    border: none;
    border-radius: 4px;
    cursor: pointer;
    font-weight: bold;
  }
  .like-section {
    margin-top: 8px;
    user-select: none;
    display: flex;
    justify-content: center;
    align-items: center;
    gap: 8px;
    font-size: 18px;
    cursor: pointer;
  }
  .like-section span {
    font-size: 16px;
  }
  button#logout-btn {
    margin-top: 20px; padding: 10px 20px; background: #d32f2f; border: none; color: white; border-radius: 6px; cursor: pointer;
  }
</style>
</head>
<body>

<div id="login-page">
  <h2>Connexion</h2>
  <input type="text" id="name" placeholder="Votre nom" required />
  <input type="email" id="email" placeholder="Votre email" required />
  <input type="password" id="password" placeholder="Mot de passe (optionnel)" />
  <button onclick="login()">Entrer</button>
  <div id="login-message"></div>
</div>

<div id="main-site">
  <div id="admin-controls" style="display:none;">
    <span>Bienvenue Admin!</span>
    <span id="upload-btn" title="Ajouter une photo">+</span>
    <input type="file" id="file-input" accept="image/*" multiple />
  </div>

  <h1>Galerie de Photos</h1>
  <div id="gallery"></div>

  <p>Connecté en tant que: <strong><span id="user-name"></span></strong> | Email: <strong><span id="user-email"></span></strong> | Rôle: <strong><span id="user-role"></span></strong></p>

  <button id="logout-btn" onclick="logout()">Se déconnecter</button>
</div>

<script>
const ADMIN_EMAIL = 'jdirfiras122@gmail.com';
const ADMIN_PASSWORD = 'admin123';

window.onload = function() {
  const loggedIn = localStorage.getItem('loggedIn');
  if(loggedIn === 'true') {
    showMainSite();
  }
  loadGallery();
};

function login() {
  const name = document.getElementById('name').value.trim();
  const email = document.getElementById('email').value.trim();
  const password = document.getElementById('password').value.trim();
  const msg = document.getElementById('login-message');

  if(name === '' || email === '') {
    msg.style.color = 'red';
    msg.textContent = 'Veuillez remplir le nom et l\'email.';
    return;
  }

  if(email === ADMIN_EMAIL && password === ADMIN_PASSWORD) {
    // أدمن
    localStorage.setItem('userRole', 'admin');
  } else {
    // مستخدم عادي
    localStorage.setItem('userRole', 'user');
  }

  localStorage.setItem('loggedIn', 'true');
  localStorage.setItem('userName', name);
  localStorage.setItem('userEmail', email);

  msg.style.color = 'lightgreen';
  msg.textContent = 'Connexion réussie! Chargement du site...';

  setTimeout(showMainSite, 1000);
}

function showMainSite() {
  document.getElementById('login-page').style.display = 'none';
  document.getElementById('main-site').style.display = 'block';

  const role = localStorage.getItem('userRole');
  const name = localStorage.getItem('userName');
  const email = localStorage.getItem('userEmail');

  document.getElementById('user-name').textContent = name;
  document.getElementById('user-email').textContent = email;
  document.getElementById('user-role').textContent = role === 'admin' ? 'Admin' : 'Utilisateur';

  if(role === 'admin') {
    document.getElementById('admin-controls').style.display = 'flex';
  } else {
    document.getElementById('admin-controls').style.display = 'none';
  }
}

function logout() {
  localStorage.clear();
  document.getElementById('main-site').style.display = 'none';
  document.getElementById('login-page').style.display = 'flex';
  document.getElementById('login-message').textContent = '';
}

// تحميل الصور مع تعليقات وعدد الإعجابات
function loadGallery() {
  const gallery = document.getElementById('gallery');
  gallery.innerHTML = '';

  let images = JSON.parse(localStorage.getItem('galleryImages') || '[]');

  images.forEach((imgData, index) => {
    const photoCard = document.createElement('div');
    photoCard.className = 'photo-card';

    const img = document.createElement('img');
    img.src = imgData.data;
    img.alt = 'Photo ' + (index + 1);
    img.title = 'Cliquez pour télécharger';
    img.onclick = () => {
      const a = document.createElement('a');
      a.href = imgData.data;
      a.download = 'photo' + (index + 1) + '.jpg';
      a.click();
    };

    photoCard.appendChild(img);

    // تعليق الصورة
    const comment = document.createElement('div');
    comment.className = 'comment';
    comment.textContent = imgData.comment || '';
    photoCard.appendChild(comment);

    // منطقة الإعجاب مع ❤️ وعدد اللايكات
    const likeSection = document.createElement('div');
    likeSection.className = 'like-section';

    const heart = document.createElement('span');
    heart.textContent = '❤️';
    heart.style.userSelect = 'none';
    heart.style.cursor = 'pointer';

    const likeCount = document.createElement('span');
    likeCount.textContent = imgData.likes || 0;

    likeSection.appendChild(heart);
    likeSection.appendChild(likeCount);

    heart.onclick = () => {
      imgData.likes = (imgData.likes || 0) + 1;
      likeCount.textContent = imgData.likes;
      saveGallery(images);
    };

    photoCard.appendChild(likeSection);

    if(localStorage.getItem('userRole') === 'admin') {
      const addCommentDiv = document.createElement('div');
      addCommentDiv.className = 'add-comment';

      const commentInput = document.createElement('input');
      commentInput.type = 'text';
      commentInput.placeholder = 'Ajouter un commentaire...';
      commentInput.value = imgData.comment || '';

      const saveBtn = document.createElement('button');
      saveBtn.textContent = 'Enregistrer';

      saveBtn.onclick = () => {
        imgData.comment = commentInput.value;
        comment.textContent = imgData.comment;
        saveGallery(images);
      };

      addCommentDiv.appendChild(commentInput);
      addCommentDiv.appendChild(saveBtn);
      photoCard.appendChild(addCommentDiv);
    }

    gallery.appendChild(photoCard);
  });
}

function saveGallery(images) {
  localStorage.setItem('galleryImages', JSON.stringify(images));
}

document.getElementById('upload-btn').addEventListener('click', () => {
  document.getElementById('file-input').click();
});

document.getElementById('file-input').addEventListener('change', function(event) {
  const files = event.target.files;
  if(files.length === 0) return;

  let images = JSON.parse(localStorage.getItem('galleryImages') || '[]');

  Array.from(files).forEach(file => {
    const reader = new FileReader();
    reader.onload = function(e) {
      images.push({data: e.target.result, comment: '', likes: 0});
      saveGallery(images);
      loadGallery();
    };
    reader.readAsDataURL(file);
  });

  event.target.value = '';
});
</script>

</body>
</html>
