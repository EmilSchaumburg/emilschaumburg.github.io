---
layout: default
title: Emil Schaumburg
---

# Recent Projects

## Computational Robotic Design (Sung Robotics Lab)
<div style="max-width:640px; position:relative;">
  <img src="assets/img/hand.png" style="width:100%; display:block;" id="slide2" />
  <button onclick="prevSlide(2)" style="position:absolute;top:50%;left:0;">❮</button>
  <button onclick="nextSlide(2)" style="position:absolute;top:50%;right:0;">❯</button>
</div>

[More](projects/robodesign.html)

## Rescue Bot
<iframe width="640" height="360" src="https://www.youtube.com/embed/cdshrarlcyE" frameborder="0" allowfullscreen> </iframe>

[More](projects/rescuebot.html)

## Hot Wheels Launcher
<iframe width="640" height="360" src="https://www.youtube.com/embed/c7rVqEytUR0" frameborder="0" allowfullscreen> </iframe>

[More](projects/hotwheels.html)


<script>
  const slideshows = {
    1: {
      images: ["assets/img/dataserver.png", "assets/img/dashboard.png"],
      current: 0,
      elementId: "slide1"
    },
    2: {
      images: ["assets/img/hand.png", "assets/img/hexapod.png", "assets/img/optimizedhexapod.png"],
      current: 0,
      elementId: "slide2"
    }
  };

  function showSlide(slideshowNumber, index) {
    const slideshow = slideshows[slideshowNumber];
    slideshow.current = (index + slideshow.images.length) % slideshow.images.length;
    document.getElementById(slideshow.elementId).src = slideshow.images[slideshow.current];
  }

  function nextSlide(slideshowNumber) {
    showSlide(slideshowNumber, slideshows[slideshowNumber].current + 1);
  }

  function prevSlide(slideshowNumber) {
    showSlide(slideshowNumber, slideshows[slideshowNumber].current - 1);
  }
</script>
