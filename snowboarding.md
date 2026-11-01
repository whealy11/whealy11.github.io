---
layout: default
title: Snowboarding
permalink: /snowboarding/
---

{% include tabs.html %}

<div class="snowboarding-note" markdown="1">
You were one hell of an adventure, snowboarding. Thank you for all the great times and people.
</div>

<div class="photo-grid" id="photo-grid">
  {% assign photos = site.static_files | where_exp: "file", "file.path contains '/fav_snowboard_photos/'" %}
  {% for file in photos %}
    {% if file.extname == ".jpg" or file.extname == ".jpeg" or file.extname == ".png" or file.extname == ".JPG" or file.extname == ".PNG" %}
      <img src="{{ file.path }}" alt="Snowboarding">
    {% endif %}
  {% endfor %}
</div>

<script>
(function () {
  var grid = document.getElementById("photo-grid");
  if (!grid) return;

  var GAP = 8;
  var TARGET_HEIGHT = 150;
  var SHUFFLE_SEED = 8;
  var photos = Array.prototype.slice.call(grid.querySelectorAll("img"));
  grid.style.visibility = "hidden";

  function random() {
    SHUFFLE_SEED = SHUFFLE_SEED + 0x6D2B79F5 | 0;
    var t = Math.imul(SHUFFLE_SEED ^ SHUFFLE_SEED >>> 15, 1 | SHUFFLE_SEED);
    t = t + Math.imul(t ^ t >>> 7, 61 | t) ^ t;
    return ((t ^ t >>> 14) >>> 0) / 4294967296;
  }

  for (var i = photos.length - 1; i > 0; i--) {
    var j = Math.floor(random() * (i + 1));
    var tmp = photos[i];
    photos[i] = photos[j];
    photos[j] = tmp;
  }
  photos.forEach(function (photo) {
    grid.appendChild(photo);
  });

  function aspect(img) {
    if (!img.naturalWidth || !img.naturalHeight) return 1.5;
    return img.naturalWidth / img.naturalHeight;
  }

  function layout() {
    var containerWidth = grid.clientWidth;
    if (!containerWidth) return;

    grid.style.height = "auto";
    photos.forEach(function (img) {
      img.style.position = "";
      img.style.left = "";
      img.style.top = "";
    });

    var row = [];
    var rowAspect = 0;

    function flush(fillWidth) {
      if (!row.length) return;
      var gaps = GAP * (row.length - 1);
      var height = TARGET_HEIGHT;
      if (fillWidth) {
        height = (containerWidth - gaps) / rowAspect;
      }
      row.forEach(function (img) {
        img.style.height = height + "px";
        img.style.width = aspect(img) * height + "px";
      });
      row = [];
      rowAspect = 0;
    }

    photos.forEach(function (img) {
      var nextAspect = rowAspect + aspect(img);
      var nextWidth = nextAspect * TARGET_HEIGHT + GAP * row.length;
      if (row.length && nextWidth > containerWidth) {
        flush(true);
      }
      row.push(img);
      rowAspect += aspect(img);
    });
    flush(false);
    grid.style.visibility = "visible";
  }

  function whenLoaded() {
    return Promise.all(photos.map(function (img) {
      if (img.complete && img.naturalWidth) return Promise.resolve();
      return new Promise(function (resolve) {
        img.addEventListener("load", resolve, { once: true });
        img.addEventListener("error", resolve, { once: true });
      });
    }));
  }

  whenLoaded().then(layout);
  window.addEventListener("resize", layout);
})();
</script>
