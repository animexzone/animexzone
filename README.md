i<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">

<title>Anime X Zone</title>

<style>
*{
  margin:0;
  padding:0;
  box-sizing:border-box;
  font-family:Arial,sans-serif;
}

body{
  background:#08080c;
  color:#fff;
}

header{
  background:linear-gradient(135deg,#11111a,#1d0710);
  padding:22px 15px;
  text-align:center;
  border-bottom:2px solid #e50914;
}

.logo{
  font-size:30px;
  font-weight:bold;
  color:#ff1744;
}

.logo span{
  color:white;
}

.tagline{
  color:#aaa;
  margin-top:6px;
  font-size:13px;
}

.search-box{
  padding:18px 15px;
  text-align:center;
  background:#0e0e14;
}

#search{
  width:100%;
  max-width:600px;
  padding:14px 18px;
  border:1px solid #333;
  border-radius:30px;
  background:#191922;
  color:white;
  outline:none;
  font-size:15px;
}

.container{
  max-width:1200px;
  margin:auto;
  padding:10px 15px 40px;
}

.section-title{
  margin:25px 0 15px;
  padding-left:12px;
  border-left:4px solid #ff1744;
  font-size:21px;
}

.grid{
  display:grid;
  grid-template-columns:repeat(2,1fr);
  gap:14px;
}

.card{
  background:#14141c;
  border-radius:10px;
  overflow:hidden;
  cursor:pointer;
  transition:.25s;
  border:1px solid #20202a;
}

.card:hover{
  transform:translateY(-5px);
  border-color:#ff1744;
}

.card img{
  width:100%;
  height:245px;
  object-fit:cover;
  display:block;
}

.card-info{
  padding:10px;
}

.card h3{
  font-size:15px;
  white-space:nowrap;
  overflow:hidden;
  text-overflow:ellipsis;
}

.card p{
  color:#999;
  font-size:12px;
  margin-top:6px;
}

.loading{
  text-align:center;
  padding:30px;
  color:#aaa;
}

button{
  border:0;
  cursor:pointer;
}

#loadMore{
  display:block;
  margin:25px auto;
  padding:12px 25px;
  border-radius:25px;
  background:#e50914;
  color:white;
  font-weight:bold;
}

/* Modal */

.modal{
  display:none;
  position:fixed;
  z-index:100;
  inset:0;
  background:rgba(0,0,0,.88);
  overflow:auto;
  padding:30px 15px;
}

.modal-box{
  max-width:800px;
  margin:auto;
  background:#15151d;
  border-radius:14px;
  overflow:hidden;
  border:1px solid #333;
}

.close{
  float:right;
  font-size:30px;
  padding:10px 18px;
  color:#fff;
  background:transparent;
}

.details{
  padding:20px;
}

.details img{
  width:150px;
  height:220px;
  object-fit:cover;
  border-radius:8px;
  float:left;
  margin:0 20px 15px 0;
}

.details h2{
  color:#ff1744;
  margin-bottom:10px;
}

.details p{
  color:#bbb;
  line-height:1.6;
  font-size:14px;
}

.clear{
  clear:both;
}

.episodes{
  margin-top:25px;
}

.episode{
  display:inline-block;
  padding:9px 13px;
  margin:4px;
  background:#252530;
  border-radius:6px;
  color:#fff;
  font-size:13px;
}

footer{
  background:#111118;
  padding:25px;
  text-align:center;
  color:#777;
  font-size:13px;
  border-top:1px solid #222;
}

@media(min-width:600px){
  .grid{
    grid-template-columns:repeat(4,1fr);
  }

  .card img{
    height:280px;
  }
}

@media(min-width:1000px){
  .grid{
    grid-template-columns:repeat(6,1fr);
  }
}
</style>
</head>

<body>

<header>
  <div class="logo">🔥 ANIME <span>X ZONE</span></div>
  <div class="tagline">Discover Anime • Sub • Dub • Movies</div>
</header>

<div class="search-box">
  <input
    id="search"
    type="text"
    placeholder="🔎 Search anime..."
    onkeyup="searchAnime()"
  >
</div>

<main class="container">

  <h2 class="section-title">🔥 Latest Anime</h2>
  <div id="latest" class="grid">
    <div class="loading">Loading anime...</div>
  </div>

  <h2 class="section-title">⭐ Top Rated</h2>
  <div id="top" class="grid">
    <div class="loading">Loading anime...</div>
  </div>

  <h2 class="section-title">🎬 Anime Movies</h2>
  <div id="movies" class="grid">
    <div class="loading">Loading movies...</div>
  </div>

  <button id="loadMore" onclick="loadMore()">Load More</button>

</main>

<!-- Anime Details -->

<div id="modal" class="modal">

  <div class="modal-box">

    <button class="close" onclick="closeModal()">×</button>

    <div id="details" class="details">
      Loading...
    </div>

  </div>

</div>

<footer>
  © 2026 Anime X Zone
  <br>
  Anime information powered by Jikan API
</footer>


<script>

const API = "https://api.jikan.moe/v4";

let currentPage = 1;
let searchTimer = null;


/* Create Anime Card */

function createCard(anime){

  const title =
    anime.title_english ||
    anime.title ||
    "Unknown Anime";

  const image =
    anime.images?.jpg?.large_image_url ||
    anime.images?.jpg?.image_url ||
    "";

  const type =
    anime.type || "Anime";

  const score =
    anime.score ? "⭐ " + anime.score : "No rating";

  return `
    <div class="card" onclick="showAnime(${anime.mal_id})">

      <img
        src="${image}"
        alt="${escapeHtml(title)}"
        loading="lazy"
      >

      <div class="card-info">

        <h3>${escapeHtml(title)}</h3>

        <p>${type} • ${score}</p>

      </div>

    </div>
  `;
}


/* Load Latest */

async function loadLatest(){

  try{

    const response =
      await fetch(`${API}/anime?order_by=popularity&sort=asc&limit=12`);

    const data = await response.json();

    document.getElementById("latest").innerHTML =
      data.data.map(createCard).join("");

  }catch(error){

    document.getElementById("latest").innerHTML =
      `<div class="loading">Unable to load anime.</div>`;
  }
}


/* Load Top */

async function loadTop(){

  try{

    const response =
      await fetch(`${API}/top/anime?limit=12`);

    const data = await response.json();

    document.getElementById("top").innerHTML =
      data.data.map(createCard).join("");

  }catch(error){

    document.getElementById("top").innerHTML =
      `<div class="loading">Unable to load anime.</div>`;
  }
}


/* Load Movies */

async function loadMovies(){

  try{

    const response =
      await fetch(`${API}/anime?type=movie&order_by=score&sort=desc&limit=12`);

    const data = await response.json();

    document.getElementById("movies").innerHTML =
      data.data.map(createCard).join("");

  }catch(error){

    document.getElementById("movies").innerHTML =
      `<div class="loading">Unable to load movies.</div>`;
  }
}


/* Search */

function searchAnime(){

  clearTimeout(searchTimer);

  const query =
    document.getElementById("search").value.trim();

  if(query.length < 2){

    loadLatest();

    return;
  }

  searchTimer = setTimeout(async()=>{

    try{

      const response =
        await fetch(
          `${API}/anime?q=${encodeURIComponent(query)}&limit=24`
        );

      const data = await response.json();

      document.getElementById("latest").innerHTML =
        data.data.length
        ? data.data.map(createCard).join("")
        : `<div class="loading">No anime found.</div>`;

    }catch(error){

      document.getElementById("latest").innerHTML =
        `<div class="loading">Search failed. Try again.</div>`;
    }

  },500);
}


/* Anime Details */

async function showAnime(id){

  const modal =
    document.getElementById("modal");

  const details =
    document.getElementById("details");

  modal.style.display = "block";

  details.innerHTML =
    `<div class="loading">Loading anime details...</div>`;

  try{

    const response =
      await fetch(`${API}/anime/${id}/full`);

    const result =
      await response.json();

    const anime = result.data;

    const title =
      anime.title_english ||
      anime.title ||
      "Unknown";

    const image =
      anime.images?.jpg?.large_image_url || "";

    const genres =
      anime.genres?.map(g => g.name).join(", ")
      || "N/A";

    const synopsis =
      anime.synopsis ||
      "No synopsis available.";

    details.innerHTML = `

      <img src="${image}" alt="${escapeHtml(title)}">

      <h2>${escapeHtml(title)}</h2>

      <p>
        <b>Japanese:</b> ${escapeHtml(anime.title_japanese || "N/A")}
      </p>

      <p>
        <b>Type:</b> ${anime.type || "N/A"}
      </p>

      <p>
        <b>Episodes:</b> ${anime.episodes || "Unknown"}
      </p>

      <p>
        <b>Status:</b> ${anime.status || "N/A"}
      </p>

      <p>
        <b>Score:</b> ⭐ ${anime.score || "N/A"}
      </p>

      <p>
        <b>Genres:</b> ${escapeHtml(genres)}
      </p>

      <br>

      <p>${escapeHtml(synopsis)}</p>

      <div class="clear"></div>

      <div class="episodes">

        <h3>📺 Episodes</h3>

        <div id="episodeList">
          Loading episodes...
        </div>

      </div>
    `;

    loadEpisodes(id);

  }catch(error){

    details.innerHTML =
      `<div class="loading">Could not load details.</div>`;
  }
}


/* Episodes */

async function loadEpisodes(id){

  const episodeList =
    document.getElementById("episodeList");

  try{

    const response =
      await fetch(`${API}/anime/${id}/episodes?limit=100`);

    const result =
      await response.json();

    if(!result.data.length){

      episodeList.innerHTML =
        `<p>No episode information available.</p>`;

      return;
    }

    episodeList.innerHTML =
      result.data.map(ep =>

        `<span class="episode">
          Episode ${ep.mal_id}
        </span>`

      ).join("");

  }catch(error){

    episodeList.innerHTML =
      `<p>Episode information unavailable.</p>`;
  }
}


/* Close Modal */

function closeModal(){

  document.getElementById("modal").style.display = "none";
}


/* Load More */

async function loadMore(){

  currentPage++;

  try{

    const response =
      await fetch(
        `${API}/anime?order_by=popularity&sort=asc&page=${currentPage}&limit=12`
      );

    const data =
      await response.json();

    document.getElementById("latest").innerHTML +=
      data.data.map(createCard).join("");

  }catch(error){

    alert("Could not load more anime.");
  }
}


/* Security */

function escapeHtml(text){

  return String(text)
    .replace(/&/g,"&amp;")
    .replace(/</g,"&lt;")
    .replace(/>/g,"&gt;")
    .replace(/"/g,"&quot;")
    .replace(/'/g,"&#039;");
}


/* Start */

loadLatest();
loadTop();
loadMovies();

</script>

</body>
</html>
