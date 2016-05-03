## ...using jQuery and CSS

**The problem:**
We have an over-sized image that we want to scale down and fit inside of a <div> without any distortion.

 We then need to center that image with said <div>. We also need to take into account whether the image is portrait or landscape. While we're at it, let's make it responsive.


**The solution:**
First, I've set up a responsive, square container that will contain our image. I'll be using the [Foundation](http://foundation.zurb.com/) framework to set the playing field.

```html
<div class="row">
  <div class="small-12 column">
    <div class="img-container">
      <!--<img src="img/puppy.jpg" alt="puppy"/> -->
    </div>
  </div>
</div>
```

```sass
.img-container
  width: 80%
  padding-bottom: 80%
  border: 1px dashed #000
```

_(Border included to verify a square div exists and that you aren't hallucinating)_

As soon as I uncomment the img tag, the img-container will unfortunately resize to accommodate the puppy image. To keep things "square" and prevent resizing, give the container relative positioning and the image absolute positioning.

Also giving the image a _max-width_ of 'none' allows the image to use it's natural width and not be limited by the container.

We also give the _.img-container_ an _overflow:hidden_ so that only the portion of the photo contained within that div will show:

```sass
.img-container
  overflow: hidden
  width: 80%
  padding-bottom: 80%
  position: relative
  img
    position: absolute
    max-width: none
```

The puppy image was taken in portrait mode and needs to fit into this square div:

![Husky puppy](http://i.imgur.com/8zepEXGm.jpg)
![square](http://i.imgur.com/9Twik1pm.png)

So how will the image be resized without any distortion? With a little JS of course. First, I'm going to retrieve the geometric data that we need:

```javascript
function scaleImg() {
  img = $(".img-container > img")[0];
  imgWidth = img.naturalWidth;
  imgHeight = img.naturalHeight;
  targetDim = $('.img-container').width();
. . .
```
_naturalHeight_ and _naturalWidth_ will give us the original dimensions of the image before being manipulated by any styling.
Selecting by 'img' with jQuery returns an array of objects, so **img[0]** targets the first image found. Our target dimension is the width (or height since we're using a square) of the container.

To scale down the image, we need to include a condition statement for landscape or portrait images. Then scale down accordingly with the correct ratio:

```javascript
// Landscape photo
  if (imgWidth > imgHeight) {
    scale = targetDim / imgHeight;
    imgWidth *= scale;
    imgHeight = targetDim;

    $(img).css('width', imgWidth);
    $(img).css('height', imgHeight);

// Portrait photo
  } else {
    scale = targetDim / imgWidth;
    imgHeight *= scale;
    imgWidth = targetDim;

    $(img).css('width', imgWidth);
    $(img).css('height', imgHeight);
  }
}
```

To summarize, we take the greater of the width or height and scale that side down by the correct ratio. So in the case of a portrait,

**imgHeight = imgHeight * (targetDim / imgWidth)**

 Since we know that the target dimension is the width of the container, we can just set the smaller image dimension to that container width.


![puppy-scaled](http://i.imgur.com/QrTDD29m.png)

Sweet, our photo scaled as intended...but wait, it isn't quite centered.
To fix this, we just need to add one line for each condition:

```javascript
function scaleImg() {
  . . .
  // Landscape photo
  if (imgWidth > imgHeight) {
    . . .
    $(img).css('left', (($(img).width() - targetDim) / 2) * -1);

  // Portrait photo  
  } else {
    . . .
    $(img).css('top', (($(img).height() - targetDim) / 2) * -1);
  }
}
```
We take the image width or height, whichever one was longer, and subtract the container width to get the length of 'extra' photo and divide that in half to center the image on the div.

Reposition with **.css** using _'top'_ and _'left'_ accordingly and multiply the value by -1 so that we can adjust the image from it's default position, which is the top left corner of the container.



![puppy-final-step](http://i.imgur.com/2h5poVz.png)

One last thing: if we resize the window, the image will not rescale with the window size. This is simply a matter of calling _scaleImg()_ on widow resize.

There's also an issue where script will run before the CSS has loaded, which will affect our results. _(window).load_ solves this.

```javascript
$(function() {
  scaleImg();
});

$(window).resize(function() {
  scaleImg();
});

$(window).load(function() {
  scaleImg();
});
```

Congrats. Your image is now scaled, centered, and responsive. You could also just use this [jQuery plugin](https://github.com/karacas/imgLiquid), but where's the fun in that?
