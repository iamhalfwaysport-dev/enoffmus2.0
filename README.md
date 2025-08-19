# enoffmus2.0<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1">
  <title>Enoffmus — All Channel Videos</title>
  <style>
    body { font-family: Arial, sans-serif; background:#111; color:#eee; margin:0; }
    header { text-align:center; padding:20px; }
    h1 { margin:0; font-size:2rem; }
    .container { max-width:1000px; margin:0 auto; padding:20px; }
    .grid { display:grid; grid-template-columns: repeat(auto-fill, minmax(250px,1fr)); gap:16px; }
    .video { background:#222; border-radius:10px; overflow:hidden; text-decoration:none; color:inherit; cursor:pointer; }
    .video:hover { background:#333; }
    .thumb img { width:100%; display:block; }
    .meta { padding:10px; }
    .meta h3 { margin:0 0 5px; font-size:1rem; }
    .meta .muted { font-size:0.85rem; color:#aaa; }
    #loadMore { display:block; margin:20px auto; padding:10px 20px; background:#444; color:#fff; border:none; border-radius:5px; cursor:pointer; }
    #loadMore:hover { background:#666; }
    #player { width:100%; height:500px; background:#000; margin-bottom:20px; border:none; border-radius:10px; }
  </style>
</head>
<body>
  <header>
    <h1>Enoffmus — All Videos</h1>
    <p>Browse every upload from <a href="https://www.youtube.com/@Enoffmus" style="color:#6ee7ff;">@Enoffmus</a>.</p>
  </header>
  <div class="container">
    <!-- Embedded video player -->
    <iframe id="player" src="" allowfullscreen></iframe>

    <div id="videos" class="grid"></div>
    <button id="loadMore">Load More</button>
  </div>

  <script>
    const API_KEY = "AIzaSyBf0Zu1QcjY32XzbIMH4wpHj0iFQz6OcW4"; // Your API key
    const CHANNEL_ID = "UCBFnihm8Q7LtFzTHvFIhoNw"; // @Enoffmus
    let nextPageToken = "";

    async function fetchUploads() {
      try {
        // Step 1: Get Uploads playlist ID
        const channelRes = await fetch(`https://www.googleapis.com/youtube/v3/channels?part=contentDetails&id=${CHANNEL_ID}&key=${API_KEY}`);
        const channelData = await channelRes.json();
        const uploadsPlaylistId = channelData.items[0].contentDetails.relatedPlaylists.uploads;

        // Step 2: Get videos from the uploads playlist
        const playlistRes = await fetch(`https://www.googleapis.com/youtube/v3/playlistItems?part=snippet&maxResults=12&playlistId=${uploadsPlaylistId}&key=${API_KEY}&pageToken=${nextPageToken}`);
        const playlistData = await playlistRes.json();
        nextPageToken = playlistData.nextPageToken || "";

        const videosDiv = document.getElementById("videos");
        playlistData.items.forEach((item, index) => {
          const vid = item.snippet.resourceId.videoId;
          const title = item.snippet.title;
          const thumb = item.snippet.thumbnails.medium.url;
          const date = new Date(item.snippet.publishedAt).toLocaleDateString();

          // Create clickable video card
          const videoCard = document.createElement("div");
          videoCard.className = "video";
          videoCard.innerHTML = `
            <div class="thumb"><img src="${thumb}" alt="${title}"></div>
            <div class="meta">
              <h3>${title}</h3>
              <div class="muted">Uploaded ${date}</div>
            </div>
          `;

          // Play inside iframe on click
          videoCard.addEventListener("click", () => {
            document.getElementById("player").src = `https://www.youtube.com/embed/${vid}?autoplay=1`;
            window.scrollTo({ top: 0, behavior: "smooth" });
          });

          videosDiv.appendChild(videoCard);

          // Auto-load first video into player
          if (!document.getElementById("player").src && index === 0) {
            document.getElementById("player").src = `https://www.youtube.com/embed/${vid}`;
          }
        });

        // Hide button if no more pages
        if (!nextPageToken) {
          document.getElementById("loadMore").style.display = "none";
        }
      } catch (err) {
        console.error("Error fetching videos", err);
        document.getElementById("videos").innerHTML = "<p>Could not load videos. Check your API key.</p>";
      }
    }

    document.getElementById("loadMore").addEventListener("click", fetchUploads);
    fetchUploads();
  </script>
</body>
</html>
