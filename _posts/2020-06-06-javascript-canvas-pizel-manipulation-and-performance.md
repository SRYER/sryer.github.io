---
layout: post
tags: ["tech"]
---

This article is meant as a reminder to myself on how canvas work is done in javascript as I don't do it that often.

## Set canvas size

I just wasted some time figuring out that I was sizing my canvas the wrong way. If you resize the canvas using css, it will actually just scale the canvas, just like if you were to scale an image using css.

WRONG:

```html
<canvas id="canvas" style="width: 600px; height: 300px"></canvas>
```

This will work when you do basic javascript updates, like drawing a line using the context object. But when using pixel manipulation which I will do in the following, it will actually stretch the result in an unintended way.

CORRECT Way:

```html
<canvas id="canvas" width="600" height="300"></canvas>
```

## Update on mousemove

Updating the canvas is done on mousemove event this way:

```js
canvas.addEventListener("mousemove", function (e) {
    var mousePosition = {x: e.layerX, y: e.layerY};
    updateCanvas(mousePosition);
});

var updateCanvas = function(mousePosition){
    // Some update logic
};

```

## Pixel manipulation vs context drawing

When updating the canvas you can either use convenient drawing methods with a context object or you can do manual pixel manipulation. Both methods has pros and cons and whenever performance is not an issue, you may stick to context drawing instead for cleaner code.

However, when performance is an issue, using manual pixel manipulation will get you further.

### Using canvas 2D context

Drawing a line using context:

```html
<canvas id="canvas_manual_drawing_demo" width="600" height="300"></canvas>
```
```js
var canvas = document.getElementById("canvas_manual_drawing_demo");
var ctx = canvas.getContext("2d");

// Fill rect
ctx.fillStyle = "rgb(80,144,80)";
ctx.fillRect(0, 0, canvas.width, canvas.height);

// Draw line
ctx.strokeStyle = "rgb(144,80,80)";
ctx.beginPath();
ctx.moveTo(0, 0);
ctx.lineTo(300, 300);
ctx.stroke();
```

Result:
<canvas id="canvas_manual_drawing_demo" width="600" height="300"></canvas>
<script>
var canvas = document.getElementById("canvas_manual_drawing_demo");
var ctx = canvas.getContext("2d");
// Fill rect
ctx.fillStyle = "rgb(80,144,80)";
ctx.fillRect(0, 0, canvas.width, canvas.height);

// Draw line
ctx.strokeStyle = "rgb(144,80,80)";
ctx.beginPath();
ctx.moveTo(0, 0);
ctx.lineTo(300, 300);
ctx.stroke();
</script>

(Sorry about the color picking)

### Using pixel manipulation

Instead of using the context, you can get an array of pixel information. The flow is as follows:

> 1. Get an array of pixel data, either the existing pixels on the canvas or a blank sheet
> 2. Manipulate the array using you own custom logic
> 3. Update the canvas with the updated pixel data

Getting the pixel data from the canvas is done this way:

```js 
var canvas = document.getElementById("myCanvas");
var ctx = canvas.getContext("2d");
var imgData = ctx.getImageData(0, 0, width, height);
var data = imgData.data; // The actual array
```

Say we have a 10 x 7 pixel canvas. Then the data array will be 10 x 7 x 4 pixels big. Then 'x 4' is because every pixel is represented as rgba data, having an int representing each value.

#### Pixel data format

The array will contain data in this format:

```js 
data[0]; // Pixel 0,0,RED
data[1]; // Pixel 0,0,GREEN
data[2]; // Pixel 0,0,BLUE
data[3]; // Pixel 0,0,ALPHA

data[4]; // Pixel 1,0,RED
data[5]; // Pixel 1,0,GREEN
data[6]; // Pixel 1,0,BLUE
data[7]; // Pixel 1,0,ALPHA

data[8]; // Pixel 2,0,RED
data[9]; // Pixel 2,0,GREEN
data[10];// Pixel 2,0,BLUE
data[11];// Pixel 2,0,ALPHA

...

data[44];// Pixel 0,1,RED
data[45];// Pixel 0,1,GREEN
data[46];// Pixel 0,1,BLUE
data[47];// Pixel 0,1,ALPHA

```

Converting from x,y to a specific position in the array is done using the following operation:

```js 

var width = 10;
var height = 7;

var x = 5;
var y = 2;

var index = (y * width + x) * 4;

data[index + 0] // Red value of the x,y pixel (+0 is done for pure readability)
data[index + 1] // Green value of the x,y pixel
data[index + 2] // Blue value of the x,y pixel
data[index + 3] // Alpha (transparency) value of the x,y pixel. 0 = invisible, 255 is solid color.

```

#### Drawing using pixel manipulation

The following code will reproduce the line-drawing from previous, but this time using pixel manipulation:

```html
<canvas id="canvas_draw_line_with_pixelmanipulation_demo" width="600" height="300"></canvas>
```
```js 
<script>
// Grab the pixel data
var width = canvas.width;
var height = canvas.height;

var canvas = document.getElementById("canvas_draw_line_with_pixelmanipulation_demo");
var ctx = canvas.getContext("2d");
var imgData = ctx.getImageData(0, 0, width, height);
var data = imgData.data;                        

// Update the data array
for(var x = 0; x < width; x++){

    for(var y = 0; y < height; y++){
        var index = (y * width + x) * 4;

         // This is the actual decision on whether green or red should be drawn on point (x,y)
        var isPointOnLine = x == y;

        if(isPointOnLine){
            data[index + 0] = 144; // Red
            data[index + 1] = 90; // Green
            data[index + 2] = 90; // Blue
            data[index + 3] = 255; // Alpha (transparency)
        }else{
            data[index + 0] = 90; // Red
            data[index + 1] = 144; // Green
            data[index + 2] = 90; // Blue
            data[index + 3] = 255; // Alpha (transparency)
        }
    }
}

// Overwrite the canvas with the updated data array
ctx.putImageData(imgData, 0, 0);

</script>
```

<canvas id="canvas_draw_line_with_pixelmanipulation_demo" width="600" height="300"></canvas>
<script>
// Grab the pixel data
var width = canvas.width;
var height = canvas.height;

var canvas = document.getElementById("canvas_draw_line_with_pixelmanipulation_demo");
var ctx = canvas.getContext("2d");
var imgData = ctx.getImageData(0, 0, width, height);
var data = imgData.data;                        


// Update the data array

for(var x = 0; x < width; x++){

    for(var y = 0; y < height; y++){
        var index = (y * width + x) * 4;

        var isPointOnLine = x == y;
        if(isPointOnLine){
            data[index + 0] = 144; // Red
            data[index + 1] = 90; // Green
            data[index + 2] = 90; // Blue
            data[index + 3] = 255; // Alpha (transparency)
        }else{
            data[index + 0] = 90; // Red
            data[index + 1] = 144; // Green
            data[index + 2] = 90; // Blue
            data[index + 3] = 255; // Alpha (transparency)
        }
    }
}

// Overwrite the canvas with the updated data array
ctx.putImageData(imgData, 0, 0);

</script>