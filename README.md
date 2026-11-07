# storm-watch-
for watching movies 
<!DOCTYPE html>
<html>
<head>
<title>Storm Watch - TikTok Style</title>
<meta name="viewport" content="width=device-width, initial-scale=1, maximum-scale=1">
<style>
* { margin:0; padding:0; box-sizing:border-box; font-family: Arial; }
body { background:#000; color:#fff; overflow:hidden; }
#app { height:100vh; display:flex; flex-direction:column; }
.header { display:flex; justify-content:space-between; padding:12px 15px; font-weight:bold; font-size:18px; background:#000; z-index:10; }
.header span.active { border-bottom:2px solid #fe2c55; }
.feed { flex:1; overflow-y:scroll; scroll-snap-type:y mandatory; }
.video-card { height:calc(100vh - 110px); scroll-snap-align:start; position:relative; background:#111; display:flex; align-items:center; justify-content:center; border-bottom:1px solid #222; }
.video-card video, .video-card img { width:100%; height:100%; object-fit:cover; }
.overlay { position:absolute; bottom:20px; left:12px; right:70px; }
.overlay h3 { font-size:16px; }
.overlay p { font-size:13px; opacity:0.9; margin:5px 0; }
.actions { position:absolute; right:10px; bottom:80px; display:flex; flex-direction:column; gap:18px; text-align:center; }
.actions button { background:#222; color:#fff; border:none; width:50px; height:50px; border-radius:50%; font-size:22px; }
.actions span { font-size:12px; display:block; margin-top:4px; }
.bottom-nav { display:flex; justify-content:space-around; background:#000; padding:10px 0; border-top:1px solid #222; }
.bottom-nav button { background:none; border:none; color:#fff; font-size:13px; }
.bottom-nav .plus { background:#fe2c55; width:45px; border-radius:8px; font-size:22px; font-weight:bold; }
.modal { display:none; position:fixed; inset:0; background:rgba(0,0,0,0.9); padding:20px; z-index:99; overflow-y:auto; }
.modal input, .modal textarea { width:100%; padding:12px; margin:8px 0; border-radius:10px; border:none; }
.modal button { width:100%; padding:12px; background:#fe2c55; color:#fff; border:none; border-radius:10px; margin-top:8px; }
.close { text-align:right; font-size:22px; cursor:pointer; }
.comment-item { background:#1a1a1a; padding:10px; border-radius:10px; margin:8px 0; font-size:14px; }
.profile-pic { width:45px; height:45px; border-radius:50%; border:2px solid #fe2c55; }
</style>
</head>
<body>
<div id="app">
  <div class="header">
    <span onclick="showTab('following')">Following</span>
    <span class="active">⛈️ Storm Watch - For You</span>
    <span onclick="openModal('searchModal')">🔍</span>
  </div>

  <div class="feed" id="feed"></div>

  <div class="bottom-nav">
    <button onclick="goHome()">🏠 Home</button>
    <button onclick="openModal('searchModal')">🔍 Discover</button>
    <button class="plus" onclick="openModal('uploadModal')">+</button>
    <button onclick="openModal('inboxModal')">💬 Inbox <span id="inboxCount">3</span></button>
    <button onclick="openModal('profileModal')">👤 Me</button>
  </div>
</div>

<!-- UPLOAD -->
<div class="modal" id="uploadModal">
  <div class="close" onclick="closeModal('uploadModal')">✕</div>
  <h2>Upload Storm Video</h2>
  <input type="file" id="videoFile" accept="video/*,image/*">
  <input id="caption" placeholder="Add caption... #storm #warri">
  <input id="hashtags" placeholder="#hashtags">
  <button onclick="uploadVideo()">Post Video 🚀</button>
</div>

<!-- COMMENTS -->
<div class="modal" id="commentModal">
  <div class="close" onclick="closeModal('commentModal')">✕</div>
  <h2>Comments</h2>
  <div id="commentList"></div>
  <input id="commentInput" placeholder="Add comment...">
  <button onclick="addComment()">Send Comment</button>
</div>

<!-- SEARCH -->
<div class="modal" id="searchModal">
  <div class="close" onclick="closeModal('searchModal')">✕</div>
  <h2>Discover</h2>
  <input id="searchInput" placeholder="Search storm, user, hashtag..." oninput="searchVideos()">
  <div id="searchResult"></div>
  <p style="margin-top:15px">#stormwatch 🔥 2.1M views</p>
  <p>#raininwarri 🔥 850K views</p>
  <p>#vietnamstorm 🔥 1.5M views</p>
  <p>#tornado 🔥 3M views</p>
</div>

<!-- INBOX -->
<div class="modal" id="inboxModal">
  <div class="close" onclick="closeModal('inboxModal')">✕</div>
  <h2>Inbox</h2>
  <div class="comment-item"><b>@sister_vibes:</b> See this storm for Warri 😱</div>
  <div class="comment-item"><b>@grab_rider:</b> I dey road, rain dey fall o</div>
  <div class="comment-item"><b>@stormwatch:</b> New alert for your area ⚠️</div>
</div>

<!-- PROFILE -->
<div class="modal" id="profileModal">
  <div class="close" onclick="closeModal('profileModal')">✕</div>
  <center>
    <img class="profile-pic" src="https://i.pravatar.cc/100">
    <h2>@you_storm</h2>
    <p><b id="myFollowers">1.2K</b> Followers • <b>340</b> Following • <b id="myLikes">5.6K</b> Likes</p>
    <button onclick="followMe()">Follow / Edit Profile</button>
  </center>
  <h3 style="margin-top:15px">Your Videos</h3>
  <div id="myVideos"></div>
</div>

<script>
let videos = [
  { user:"@stormwatch", caption:"Heavy storm for Warri today ⛈️ #stormwatch #warri", likes:12500, comments:["Omo rain too much!","Stay safe o","Warri no dey smile"], img:"https://images.unsplash.com/photo-1428592953211-077101b2021b?w=600", followed:false },
  { user:"@sister_vibes", caption:"Night lounge vibe before rain 🌧️ #vietnam #night", likes:8200, comments:["Fine place","Where be this?"], img:"https://images.unsplash.com/photo-1534088568595-a066f410bcda?w=600", followed:false },
  { user:"@grab_rider", caption:"Grab rider for rain still dey deliver 🛵 #grab #vietnam", likes:15200, comments:["Respect!","Rain no stop am"], img:"https://images.unsplash.com/photo-1501691223384-9f9e2c9b3a0e?w=600", followed:false },
];
let currentVideo = 0;

function renderFeed(){
  let feed = document.getElementById('feed');
  feed.innerHTML = "";
  videos.forEach((v,i)=>{
    feed.innerHTML += `
    <div class="video-card">
      <img src="${v.img}">
      <div class="overlay">
        <h3>${v.user} ${v.followed ? '✓' : ''}</h3>
        <p>${v.caption}</p>
        <p>🎵 Original Sound - Storm Watch</p>
      </div>
      <div class="actions">
        <div><button onclick="likeVideo(${i})">❤️</button><span id="like-${i}">${v.likes}</span></div>
        <div><button onclick="openComments(${i})">💬</button><span>${v.comments.length}</span></div>
        <div><button onclick="shareVideo(${i})">↗️</button><span>Share</span></div>
        <div><button onclick="followUser(${i})">${v.followed ? '✓' : '+'}</button><span>${v.followed ? 'Following' : 'Follow'}</span></div>
        <div><button onclick="saveVideo(${i})">🔖</button><span>Save</span></div>
      </div>
    </div>`;
  });
}
function likeVideo(i){ videos[i].likes++; document.getElementById('like-'+i).innerText = videos[i].likes; }
function followUser(i){ videos[i].followed = !videos[i].followed; renderFeed(); }
function openComments(i){ currentVideo=i; document.getElementById('commentList').innerHTML = videos[i].comments.map(c=>`<div class="comment-item">${c}</div>`).join(''); openModal('commentModal'); }
function addComment(){ let t=document.getElementById('commentInput').value; if(!t) return; videos[currentVideo].comments.push("You: "+t); openComments(currentVideo); document.getElementById('commentInput').value=""; }
function shareVideo(i){ alert("Link copied! Share to WhatsApp, TikTok, Instagram: https://stormwatch.app/video/"+i); }
function saveVideo(i){ alert("Saved to Favorites 🔖"); }
function uploadVideo(){ let cap=document.getElementById('caption').value || "New storm video ⛈️"; videos.unshift({user:"@you_storm", caption:cap, likes:0, comments:[], img:"https://images.unsplash.com/photo-1428592953211-077101b2021b?w=600", followed:false}); closeModal('uploadModal'); renderFeed(); alert("Video Posted! 🚀"); }
function searchVideos(){ let q=document.getElementById('searchInput').value.toLowerCase(); let res=videos.filter(v=>v.caption.toLowerCase().includes(q)||v.user.toLowerCase().includes(q)); document.getElementById('searchResult').innerHTML = res.map(v=>`<div class="comment-item"><b>${v.user}</b><br>${v.caption}</div>`).join('') || "No result"; }
function followMe(){ alert("Profile Updated!"); }
function goHome(){ renderFeed(); }
function openModal(id){ document.getElementById(id).style.display="block"; }
function closeModal(id){ document.getElementById(id).style.display="none"; }
function showTab(t){ alert("Showing "+t+" feed"); }

renderFeed();
</script>
</body>
</html>