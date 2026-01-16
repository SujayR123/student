---
layout: post
title: About
permalink: /about/
comments: true
---

## As a conversation Starter

Here are some places I have lived.

<comment>
Flags are made using Wikipedia images
</comment>

<style>
    .grid-container {
        display: grid;
        grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
        gap: 10px;
    }
    .grid-item {
        text-align: center;
    }
    .grid-item img {
        width: 100%;
        height: 120px;
        object-fit: cover;
    }
    .grid-item p {
        margin: 5px 0;
    }

    .image-gallery {
        display: flex;
        flex-wrap: nowrap;
        overflow-x: auto;
        gap: 10px;
    }

    .image-gallery img {
        max-height: 150px;
        object-fit: cover;
        border-radius: 5px;
    }
</style>


<div class="grid-container" id="grid_container"></div>


<h2>My Favorite Songs</h2>
<div class="grid-container" id="songs_grid_container"></div>

<script>
  
    var container = document.getElementById("grid_container");

    var http_source = "https://upload.wikimedia.org/wikipedia/commons/";
    var living_in_the_world = [
        {"flag": "0/01/Flag_of_California.svg", "description": "California - forever"},
        {"flag": "b/b9/Flag_of_Minnesota.svg", "description": "Minnesota - Born"}
    ];

    for (const location of living_in_the_world) {
        var gridItem = document.createElement("div");
        gridItem.className = "grid-item";

        var img = document.createElement("img");
        img.src = http_source + location.flag;
        img.alt = location.description + " Flag";

        var description = document.createElement("p");
        description.textContent = location.description;

        gridItem.appendChild(img);
        gridItem.appendChild(description);
        container.appendChild(gridItem);
    }

    
    var songsContainer = document.getElementById("songs_grid_container");

    var favorite_songs = [
        {
            image: "https://upload.wikimedia.org/wikipedia/commons/6/6f/Yeat.png",
            title: "Breathe",
            artist: "Yeat"
        },
        {
            image: "https://upload.wikimedia.org/wikipedia/commons/1/14/Travis_Scott_-_Openair_Frauenfeld_2019_08.jpg",
            title: "SICKO MODE",
            artist: "Travis Scott"
        },
        {
            image: "https://upload.wikimedia.org/wikipedia/commons/1/14/Travis_Scott_-_Openair_Frauenfeld_2019_08.jpg",
            title: "Butterfly Effect",
            artist: "Travis Scott"
        },
        {
            image: "https://upload.wikimedia.org/wikipedia/commons/9/93/Don_Toliver_2017.png",
            title: "No Pole",
            artist: "Don Toliver"
        }
    ];

    for (const song of favorite_songs) {
        var gridItem = document.createElement("div");
        gridItem.className = "grid-item";

        var img = document.createElement("img");
        img.src = song.image;
        img.alt = song.artist + " photo";

        var title = document.createElement("p");
        title.textContent = song.title;
        title.style.fontWeight = "bold";

        var artist = document.createElement("p");
        artist.textContent = song.artist;
        artist.style.fontStyle = "italic";
        artist.style.opacity = "0.7";

        gridItem.appendChild(img);
        gridItem.appendChild(title);
        gridItem.appendChild(artist);

        songsContainer.appendChild(gridItem);
    }
</script>

### Journey through Life

- 🏫 Went to Montery Ridge Elementry School
- 🏫 Went to Oak Valley Middle School
- 🎓 Joined Cyber Aegis Club
- ⛪ Play Baseball for a travel team
- 💼 Started teching Math for a club
- 🎓 Graduated Middle School
- 💼 Went to DNHS (Current)

### Culture, Family, and Fun

<comment>
Gallery of Pics, scroll to the right for more ...
</comment>

<div class="image-gallery">
  <img src="{{site.baseurl}}/images/about/missionary.jpg">
  <img src="{{site.baseurl}}/images/about/john_tamara.jpg">
  <img src="{{site.baseurl}}/images/about/tamara_fam.jpg">
  <img src="{{site.baseurl}}/images/about/surf.jpg">
  <img src="{{site.baseurl}}/images/about/john_lora.jpg">
  <img src="{{site.baseurl}}/images/about/lora_fam.jpg">
  <img src="{{site.baseurl}}/images/about/lora_fam2.jpg">
  <img src="{{site.baseurl}}/images/about/pj_party.jpg">
  <img src="{{site.baseurl}}/images/about/trent_family.png">
  <img src="{{site.baseurl}}/images/about/claire.jpg">
  <img src="{{site.baseurl}}/images/about/grandkids.jpg">
  <img src="{{site.baseurl}}/images/about/farm.jpg">
</div>
