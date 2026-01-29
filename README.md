# abdohk-site.
Public
<!DOCTYPE html>
<html lang="ar">
<head>
<meta charset="UTF-8">
<title>ABDOHK | الحساب الشخصي</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
body{
    margin:0;
    font-family: monospace;
    background:#000;
    color:#00b7ff;
    overflow:hidden;
}
canvas{
    position:fixed;
    inset:0;
    z-index:-1;
}
.box, .profile, #fbProfile{
    width:360px;
    padding:30px;
    background:rgba(0,0,0,.85);
    box-shadow:0 0 25px #00b7ff;
    border-radius:15px;
    text-align:center;
    position:absolute;
    top:50%; left:50%;
    transform:translate(-50%,-50%);
}
input{
    width:100%;
    padding:12px;
    margin:10px 0;
    background:transparent;
    border:none;
    border-bottom:1px solid #00b7ff;
    color:#00b7ff;
}
input[type="file"]{border:none}
button{
    width:100%;
    padding:12px;
    margin-top:15px;
    border:none;
    border-radius:8px;
    background:#00b7ff;
    color:#000;
    font-weight:bold;
    cursor:pointer;
}
button:hover{background:#0ff}
.logout{background:#ff004c;color:#fff}
.avatar{
    width:120px;
    height:120px;
    border-radius:50%;
    border:3px solid #00b7ff;
    margin:0 auto 15px;
    background:#111;
    display:flex;
    align-items:center;
    justify-content:center;
    overflow:hidden;
}
.avatar img{
    width:100%;
    height:100%;
    object-fit:cover;
}
.name{font-size:20px;font-weight:bold}
.email{color:#0ff;font-size:14px}

/* ===== ملف شخصي شبيه فيسبوك ===== */
#fbProfile{display:none; max-width:400px; margin:20px auto; background:#111; border-radius:15px; padding:10px; color:#00b7ff;}
#fbProfile .cover{position:relative; height:150px; background:#222; border-radius:10px; overflow:hidden;}
#fbProfile .cover img{width:100%; height:100%; object-fit:cover;}
#fbProfile .cover input{position:absolute; bottom:5px; right:5px;}
#fbProfile .avatar-big{width:100px; height:100px; margin:auto; border-radius:50%; overflow:hidden; border:3px solid #00b7ff;}
#fbProfile .avatar-big img{width:100%; height:100%; object-fit:cover;}
#fbProfile .avatar-big input{margin-top:5px;}
#fbProfile .post-box{background:#000; margin:10px; padding:10px; border-radius:10px;}
#fbProfile .post-box textarea{width:100%; height:60px; background:#111; color:#0ff; border:none; padding:5px;}
#fbProfile .post-box button{margin-top:5px; width:100%;}
#fbProfile .post{background:#222; margin:5px; padding:5px; border-radius:8px;}
#fbProfile .post small{color:#888;}
</style>
</head>
<body>

<canvas id="matrix"></canvas>

<!-- تسجيل دخول -->
<div id="loginBox" class="box">
    <h2>تسجيل الدخول</h2>
    <input id="email" placeholder="البريد">
    <input id="pass" type="password" placeholder="كلمة المرور">
    <button onclick="login()">دخول</button>
    <p id="msg"></p>
    <a href="#" onclick="showSignup()">إنشاء حساب</a>
</div>

<!-- إنشاء حساب -->
<div id="signupBox" class="box" style="display:none">
    <h2>إنشاء حساب</h2>
    <input id="name" placeholder="الاسم">
    <input id="emailS" placeholder="البريد">
    <input id="passS" type="password" placeholder="كلمة المرور">
    <button onclick="signup()">تسجيل</button>
    <p id="msgS"></p>
    <a href="#" onclick="showLogin()">تسجيل دخول</a>
</div>

<!-- الملف الشخصي شبيه فيسبوك -->
<div id="fbProfile">
    <div class="cover">
        <img id="coverImg" src="">
        <input type="file" onchange="uploadCover(this)">
    </div>
    <div style="text-align:center; margin-top:-60px;">
        <div class="avatar-big">
            <img id="avatarImg" src="">
            <input type="file" onchange="uploadAvatar(this)">
        </div>
        <h2 id="profileName"></h2>
        <p id="profileEmail"></p>
    </div>
    <div class="post-box">
        <textarea id="postText" placeholder="بم تفكر؟"></textarea>
        <button onclick="addPost()">نشر</button>
    </div>
    <div id="posts"></div>
    <button class="logout" onclick="logout()">تسجيل خروج</button>
</div>

<script>
// ===== عناصر الصفحة =====
const fbProfile = document.getElementById("fbProfile");
const avatarImg = document.getElementById("avatarImg");
const coverImg = document.getElementById("coverImg");
const profileName = document.getElementById("profileName");
const profileEmail = document.getElementById("profileEmail");
const postsDiv = document.getElementById("posts");
const postText = document.getElementById("postText");

// ===== بيانات المستخدمين =====
const users = JSON.parse(localStorage.getItem("users")) || [];
const saveUsers = () => localStorage.setItem("users", JSON.stringify(users));
const saveSession = u => localStorage.setItem("session", JSON.stringify(u));
const getSession = () => JSON.parse(localStorage.getItem("session"));

// ===== ماتريكس =====
const c=document.getElementById("matrix"),x=c.getContext("2d");
c.width=innerWidth;c.height=innerHeight;
const chars="01",fs=18,cols=c.width/fs,d=[];
for(let i=0;i<cols;i++)d[i]=Math.random()*c.height;
setInterval(()=>{
    x.fillStyle="rgba(0,0,0,.1)";
    x.fillRect(0,0,c.width,c.height);
    x.fillStyle="#00b7ff";
    x.font=fs+"px monospace";
    d.forEach((y,i)=>{
        x.fillText(chars[Math.random()*2|0],i*fs,y);
        d[i]=y>c.height?0:y+fs;
    });
},33);

// ===== تنقل بين الصفحات =====
function showSignup(){loginBox.style.display="none";signupBox.style.display="block";}
function showLogin(){signupBox.style.display="none";loginBox.style.display="block";}

// ===== إنشاء حساب =====
function signup(){
    const u = {
        id: Date.now(),
        name: name.value,
        email: emailS.value,
        password: passS.value,
        avatar: "",
        cover: "",
        posts: [],
        createdAt: new Date().toLocaleString(),
        lastLogin: new Date().toLocaleString(),
        loginCount: 1
    };
    if(!u.name || !u.email || !u.password){ msgS.innerText="أكمل البيانات"; return; }
    if(users.find(x=>x.email===u.email)){ msgS.innerText="هذا البريد موجود"; return; }

    users.push(u);
    saveUsers();
    saveSession(u);
    showFBProfile();
}

// ===== تسجيل دخول =====
function login(){
    const u = users.find(x => x.email===email.value && x.password===pass.value);
    if(!u){ msg.innerText="خطأ في البريد أو كلمة السر"; return; }

    u.lastLogin = new Date().toLocaleString();
    u.loginCount += 1;

    saveUsers();
    saveSession(u);
    showFBProfile();
}

// ===== عرض الملف الشخصي فيسبوك =====
function showFBProfile(){
    const u = getSession();
    if(!u) return;

    fbProfile.style.display = "block";
    loginBox.style.display = signupBox.style.display = "none";

    profileName.innerText = u.name;
    profileEmail.innerText = u.email;
    avatarImg.src = u.avatar || "";
    coverImg.src = u.cover || "";

    loadPosts();
}

// ===== رفع صورة شخصية =====
function uploadAvatar(input){
    const r = new FileReader();
    r.onload = () => {
        const u = getSession();
        u.avatar = r.result;
        saveUserData(u);
        avatarImg.src = r.result;
    };
    r.readAsDataURL(input.files[0]);
}

// ===== رفع غلاف =====
function uploadCover(input){
    const r = new FileReader();
    r.onload = () => {
        const u = getSession();
        u.cover = r.result;
        saveUserData(u);
        coverImg.src = r.result;
    };
    r.readAsDataURL(input.files[0]);
}

// ===== إضافة منشور =====
function addPost(){
    const u = getSession();
    if(!u.posts) u.posts=[];
    const text = postText.value.trim();
    if(!text) return;
    u.posts.unshift({ text, date: new Date().toLocaleString() });
    postText.value="";
    saveUserData(u);
    loadPosts();
}

// ===== عرض المنشورات =====
function loadPosts(){
    const u = getSession();
    postsDiv.innerHTML = "";
    (u.posts||[]).forEach(p=>{
        postsDiv.innerHTML += `<div class="post"><b>${u.name}</b><p>${p.text}</p><small>${p.date}</small></div>`;
    });
}

// ===== حفظ بيانات المستخدم =====
function saveUserData(u){
    saveSession(u);
    let i = users.findIndex(x=>x.email===u.email);
    if(i !== -1) users[i] = u;
    saveUsers();
}

// ===== تسجيل خروج =====
function logout(){
    localStorage.removeItem("session");
    fbProfile.style.display = "none";
    showLogin();
}

// ===== عند فتح الصفحة =====
window.onload = ()=>{
    if(getSession()) showFBProfile();
};
</script>
</body>
</html>
