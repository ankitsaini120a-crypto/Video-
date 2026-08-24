# Video-
views growth
import os, zipfile, textwrap, json

root="/mnt/data/youtube-real-views-site"
os.makedirs(root, exist_ok=True)

files = {
"index.html": r'''<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>ViewTube — Real YouTube Watch Portal</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
<header class="topbar">
  <div class="brand">▶ ViewTube</div>
  <a class="admin-link" href="admin.html">Admin</a>
</header>

<main class="container">
  <section class="hero">
    <div>
      <span class="pill">REAL VIEWERS • YOUTUBE EMBED</span>
      <h1>Watch. Discover. Support creators.</h1>
      <p>Share your YouTube videos in a clean watch portal. Views are counted by YouTube, not by fake bots.</p>
    </div>
  </section>

  <section class="toolbar">
    <input id="search" type="search" placeholder="Search videos...">
    <select id="category">
      <option value="">All categories</option>
    </select>
  </section>

  <section id="videos" class="grid"></section>
  <p id="empty" class="empty" hidden>No videos found.</p>
</main>

<footer>ViewTube • A legitimate YouTube viewing portal</footer>
<script src="app.js"></script>
</body>
</html>''',

"admin.html": r'''<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>ViewTube Admin</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
<header class="topbar">
  <div class="brand">▶ ViewTube Admin</div>
  <a class="admin-link" href="index.html">View site</a>
</header>

<main class="container narrow">
  <section class="card">
    <h1>Video Manager</h1>
    <p class="muted">This demo stores videos in your browser using localStorage. For a multi-user production site, connect the included API to a database.</p>

    <form id="videoForm">
      <input type="hidden" id="editId">
      <label>Video title<input id="title" required maxlength="120"></label>
      <label>YouTube URL<input id="url" required placeholder="https://www.youtube.com/watch?v=..."></label>
      <label>Category<input id="cat" maxlength="40" placeholder="Gaming"></label>
      <label>Description<textarea id="desc" maxlength="500"></textarea></label>
      <div class="row">
        <button type="submit">Save video</button>
        <button type="button" class="secondary" id="cancel">Cancel</button>
      </div>
    </form>
    <p id="message" class="message"></p>
  </section>

  <section class="card">
    <h2>Your videos</h2>
    <div id="adminList"></div>
  </section>
</main>
<script src="admin.js"></script>
</body>
</html>''',

"watch.html": r'''<!doctype html>
<html lang="en">
<head>
  <meta charset="utf-8">
  <meta name="viewport" content="width=device-width,initial-scale=1">
  <title>Watch — ViewTube</title>
  <link rel="stylesheet" href="style.css">
</head>
<body>
<header class="topbar">
  <div class="brand">▶ ViewTube</div>
  <a class="admin-link" href="index.html">Home</a>
</header>
<main class="container">
  <div id="watch"></div>
</main>
<script src="watch.js"></script>
</body>
</html>''',

"style.css": r''':root{font-family:Inter,system-ui,-apple-system,Segoe UI,Roboto,sans-serif;color:#f5f7fb;background:#0b0d12}
*{box-sizing:border-box}body{margin:0;min-height:100vh}.topbar{height:64px;display:flex;align-items:center;justify-content:space-between;padding:0 5%;border-bottom:1px solid #202532;background:#10131a;position:sticky;top:0;z-index:5}.brand{font-size:21px;font-weight:800}.admin-link{color:#cbd3e1;text-decoration:none}.container{width:min(1150px,92%);margin:0 auto;padding:32px 0 60px}.narrow{max-width:820px}.hero{padding:45px 0 30px}.hero h1{font-size:clamp(34px,6vw,62px);line-height:1.02;margin:14px 0}.hero p{max-width:700px;color:#aab3c2;font-size:18px}.pill{font-size:12px;font-weight:800;letter-spacing:.12em;color:#8fb5ff}.toolbar{display:flex;gap:12px;margin:20px 0 26px}.toolbar input,.toolbar select,form input,form textarea{width:100%;background:#151a23;border:1px solid #2a3241;color:#fff;border-radius:12px;padding:13px 14px;outline:none}.toolbar select{max-width:190px}.grid{display:grid;grid-template-columns:repeat(auto-fill,minmax(260px,1fr));gap:20px}.video-card,.card{background:#11151d;border:1px solid #252c38;border-radius:18px;overflow:hidden}.thumb{aspect-ratio:16/9;background:#1b2230;position:relative}.thumb img{width:100%;height:100%;object-fit:cover}.play{position:absolute;left:50%;top:50%;transform:translate(-50%,-50%);background:#fff;color:#111;width:52px;height:52px;border-radius:50%;display:grid;place-items:center;font-size:20px}.video-info{padding:16px}.video-info h3{margin:0 0 8px;font-size:18px}.muted,.video-info p{color:#8993a5}.meta{font-size:13px;margin-bottom:12px}.btn,button{display:inline-block;border:0;background:#fff;color:#111;padding:11px 15px;border-radius:10px;font-weight:700;text-decoration:none;cursor:pointer}.secondary{background:#252c38;color:#fff}footer{text-align:center;color:#687386;border-top:1px solid #202532;padding:25px}.card{padding:24px;margin-bottom:22px}.card h1,.card h2{margin-top:0}label{display:block;font-weight:700;margin:14px 0}label input,label textarea{display:block;margin-top:7px;font-weight:400}textarea{min-height:110px;resize:vertical}.row{display:flex;gap:10px;margin-top:18px}.message{min-height:20px}.admin-item{display:flex;justify-content:space-between;gap:15px;padding:14px 0;border-bottom:1px solid #252c38}.admin-item:last-child{border-bottom:0}.empty{text-align:center;color:#8993a5;padding:50px}.player{aspect-ratio:16/9;background:#000;border-radius:16px;overflow:hidden}.player iframe{width:100%;height:100%;border:0}.watch-title{font-size:30px;margin:20px 0 8px}.notice{padding:12px 14px;background:#171d27;border:1px solid #2a3241;border-radius:10px;color:#aeb7c6;margin:18px 0}@media(max-width:650px){.toolbar{flex-direction:column}.toolbar select{max-width:none}.hero{padding-top:25px}.container{width:94%}}''',

"app.js": r'''const DEFAULT_VIDEOS=[
 {id:"demo1",title:"Add your first YouTube video",url:"https://www.youtube.com/watch?v=dQw4w9WgXcQ",category:"Demo",description:"Replace this demo video from the Admin page."}
];
const KEY="viewtube_videos";
function getVideos(){try{return JSON.parse(localStorage.getItem(KEY))||DEFAULT_VIDEOS}catch{return DEFAULT_VIDEOS}}
function saveVideos(v){localStorage.setItem(KEY,JSON.stringify(v))}
if(!localStorage.getItem(KEY))saveVideos(DEFAULT_VIDEOS);

const grid=document.querySelector("#videos"), search=document.querySelector("#search"), category=document.querySelector("#category"), empty=document.querySelector("#empty");
function idFromUrl(url){try{const u=new URL(url); if(u.hostname.includes("youtu.be"))return u.pathname.slice(1).split("/")[0]; if(u.searchParams.get("v"))return u.searchParams.get("v"); const m=u.pathname.match(/\/(?:shorts|embed)\/([^/?]+)/); return m?m[1]:null}catch{return null}}
function render(){
 const q=search.value.toLowerCase(), c=category.value;
 const vids=getVideos().filter(v=>(v.title+" "+v.description).toLowerCase().includes(q)&&(!c||v.category===c));
 grid.innerHTML=vids.map(v=>{const id=idFromUrl(v.url);return `<article class="video-card">
 <div class="thumb">${id?`<img loading="lazy" src="https://i.ytimg.com/vi/${encodeURIComponent(id)}/hqdefault.jpg" alt="">`:''}<div class="play">▶</div></div>
 <div class="video-info"><h3>${esc(v.title)}</h3><div class="meta">${esc(v.category||"General")}</div><p>${esc(v.description||"")}</p><a class="btn" href="watch.html?id=${encodeURIComponent(v.id)}">Watch on YouTube</a></div></article>`}).join("");
 empty.hidden=vids.length!==0;
}
function esc(s){return String(s).replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[m]))}
const cats=[...new Set(getVideos().map(v=>v.category).filter(Boolean))];cats.forEach(c=>{const o=document.createElement("option");o.value=c;o.textContent=c;category.appendChild(o)});
search.oninput=render;category.onchange=render;render();''',

"admin.js": r'''const KEY="viewtube_videos";
const form=document.querySelector("#videoForm"), list=document.querySelector("#adminList"), msg=document.querySelector("#message");
const $=id=>document.querySelector("#"+id);
function get(){try{return JSON.parse(localStorage.getItem(KEY))||[]}catch{return []}}
function set(v){localStorage.setItem(KEY,JSON.stringify(v))}
function ytId(url){try{const u=new URL(url);if(u.hostname.includes("youtu.be"))return u.pathname.slice(1).split("/")[0];if(u.searchParams.get("v"))return u.searchParams.get("v");const m=u.pathname.match(/\/(?:shorts|embed)\/([^/?]+)/);return m?m[1]:null}catch{return null}}
function esc(s){return String(s).replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[m]))}
function render(){const vids=get();list.innerHTML=vids.length?vids.map(v=>`<div class="admin-item"><div><b>${esc(v.title)}</b><div class="muted">${esc(v.category||"General")}</div></div><div class="row"><button onclick="editVideo('${v.id}')">Edit</button><button class="secondary" onclick="deleteVideo('${v.id}')">Delete</button></div></div>`).join(""):"<p class='muted'>No videos yet.</p>"}
form.onsubmit=e=>{e.preventDefault();const title=$("title").value.trim(),url=$("url").value.trim();if(!ytId(url)){msg.textContent="Please enter a valid YouTube URL.";return}let vids=get(),id=$("editId").value||crypto.randomUUID();const item={id,title,url,category:$("cat").value.trim()||"General",description:$("desc").value.trim()};const i=vids.findIndex(v=>v.id===id);if(i>=0)vids[i]=item;else vids.unshift(item);set(vids);reset();render();msg.textContent="Saved."}
function editVideo(id){const v=get().find(x=>x.id===id);if(!v)return;$("editId").value=v.id;$("title").value=v.title;$("url").value=v.url;$("cat").value=v.category||"";$("desc").value=v.description||"";scrollTo({top:0,behavior:"smooth"})}
function deleteVideo(id){if(confirm("Delete this video?")){set(get().filter(v=>v.id!==id));render()}}
function reset(){form.reset();$("editId").value=""}
$("cancel").onclick=reset;render();''',

"watch.js": r'''const KEY="viewtube_videos";
function get(){try{return JSON.parse(localStorage.getItem(KEY))||[]}catch{return []}}
function idFromUrl(url){try{const u=new URL(url);if(u.hostname.includes("youtu.be"))return u.pathname.slice(1).split("/")[0];if(u.searchParams.get("v"))return u.searchParams.get("v");const m=u.pathname.match(/\/(?:shorts|embed)\/([^/?]+)/);return m?m[1]:null}catch{return null}}
function esc(s){return String(s).replace(/[&<>"']/g,m=>({"&":"&amp;","<":"&lt;",">":"&gt;",'"':"&quot;","'":"&#039;"}[m]))}
const id=new URLSearchParams(location.search).get("id"),v=get().find(x=>x.id===id),root=document.querySelector("#watch");
if(!v){root.innerHTML="<div class='card'><h1>Video not found</h1><a class='btn' href='index.html'>Back home</a></div>"}else{const vid=idFromUrl(v.url);root.innerHTML=`<div class="player"><iframe src="https://www.youtube.com/embed/${encodeURIComponent(vid)}?rel=0" title="${esc(v.title)}" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe></div><h1 class="watch-title">${esc(v.title)}</h1><div class="meta">${esc(v.category||"General")}</div><p class="muted">${esc(v.description||"")}</p><div class="notice">This player uses YouTube's official embed. Valid views are determined by YouTube; this site does not create artificial views.</div>`}''',

"README.md": r'''# ViewTube — Real YouTube Views Website

A lightweight YouTube video portal. Visitors watch videos through YouTube's official embed, so the website does **not** generate fake/bot views or manipulate YouTube metrics.

## Run locally

Because the frontend uses localStorage, you can simply open `index.html` in a browser. For best results, serve the folder with any static server:

```bash
python -m http.server 8080
