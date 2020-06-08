---
layout: post
tags: ["tech"]
---

This article is meant as a reminder to myself on how canvas work is done in javascript as I don't do it that often.

## Set canvas size

I just wasted some time figuring out that I was sizing my canvas the wrong way. If you resize the canvas using css, it will actually just scale the canvas, just like if you were to scale an image using css.

WRONG:

```html
<canvas style="position: relative;" id= "canvas" style="width: 600px; height: 300px"></canvas>
```

This will work when you do basic javascript updates, like drawing a line using the context object. But when using pixel manipulation which I will do in the following, it will actually stretch the result in an unintended way.

CORRECT Way:

```html
<canvas style="position: relative;" id= "canvas" width="600" height="300"></canvas>
```

## Pixel manipulation vs context drawing

When updating the canvas you can either use convenient drawing methods with a context object or you can do manual pixel manipulation. Both methods have pros and cons and whenever performance is not an issue, you may stick to context drawing for cleaner code.

However, when performance is an issue, using manual pixel manipulation will get you further.

### Using canvas 2D context

Drawing a line using context:

```html
<canvas style="position: relative;" id= "canvas_manual_drawing_demo" width="600" height="300">
</canvas>
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
```

Result:
<canvas style="position: relative;" id= "canvas_manual_drawing_demo" width="600" height="300"></canvas>
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

Say we have a 10 x 7 pixel canvas. Then the data array will be of size 10 x 7 x 4 = 280. The 'x 4' is because every pixel is represented as rgba data, having an int representing each value.

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
<canvas style="position: relative;" id= "canvas_draw_line_with_pixelmanipulation_demo" width="600" height="300">'
</canvas>
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
```

<canvas style="position: relative;" id= "canvas_draw_line_with_pixelmanipulation_demo" width="600" height="300">'
</canvas>
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

The event object 'e' contains layerX and layerY which is the position of the mouse.

> NOTE: You MUST add style="position: relative;" for this to work in firefox and edge. If not, the layerX and layerY will be the mouse position relative to the window.

A working example of drawing a circle at the mouse position will look like this:

```html
<canvas style="position: relative;" id= "canvas_mouse_move_demo" width="600" height="300">
</canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_mouse_move_demo");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            
            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){
                    var index = (y * width + x) * 4;

                    var distanceToMouse = Math.sqrt((mousePosition.x - x)*(mousePosition.x - x) + (mousePosition.y - y)*(mousePosition.y - y));
                    if(distanceToMouse < 50){
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
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

    
        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
            console.log(11);
        });
    })();
</script>
```

<canvas style="position: relative;" id= "canvas_mouse_move_demo" width="600" height="300">
</canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_mouse_move_demo");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            
            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){
                    var index = (y * width + x) * 4;

                    var distanceToMouse = Math.sqrt((mousePosition.x - x)*(mousePosition.x - x) + (mousePosition.y - y)*(mousePosition.y - y));
                    if(distanceToMouse < 50){
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
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

    
        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
            console.log(11);
        });
    })();
</script>

## Light effect #1

Instead of draw/dont draw depending on whether the distance exceeds 50 (indicating a radius of 50 for the circle), we can change the logic to fade out the longer the distance from a pixel to the mouse cursor:


```html
<canvas style="position: relative;" id= "canvas_light_demo" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var updateCanvas = function(mousePosition){
            
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            
            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){
                    var index = (y * width + x) * 4;

                    var distanceToMouse = Math.sqrt((mousePosition.x - x)*(mousePosition.x - x) + (mousePosition.y - y)*(mousePosition.y - y));
                    distanceToMouse = parseInt(distanceToMouse); // Round, throwing away all decimals
                    if(distanceToMouse < 1){
                        distanceToMouse = 1; // Avoid division by 0 here
                    }

                    var color = 255 / distanceToMouse;
                    color = parseInt(color); // Round again

                    data[index + 0] = color; // Red
                        data[index + 1] = color; // Green
                        data[index + 2] = color; // Blue
                        data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

    
        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>
```

<canvas style="position: relative;" id= "canvas_light_demo" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var updateCanvas = function(mousePosition){
            
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            
            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){
                    var index = (y * width + x) * 4;

                    var distanceToMouse = Math.sqrt((mousePosition.x - x)*(mousePosition.x - x) + (mousePosition.y - y)*(mousePosition.y - y));
                    distanceToMouse = parseInt(distanceToMouse); // Round, throwing away all decimals
                    if(distanceToMouse < 1){
                        distanceToMouse = 1; // Avoid division by 0 here
                    }

                    var color = 255 / distanceToMouse;
                    color = parseInt(color); // Round again

                    data[index + 0] = color; // Red
                        data[index + 1] = color; // Green
                        data[index + 2] = color; // Blue
                        data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

    
        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>

## Light effect #2 - changing looks of light

We can now make the light a little bigger by changing the logic to this:


```html
<canvas style="position: relative;" id= "canvas_light_demo_2" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_2");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            
            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){
                    var index = (y * width + x) * 4;

                    var distanceToMouse = Math.sqrt((mousePosition.x - x)*(mousePosition.x - x) + (mousePosition.y - y)*(mousePosition.y - y));
                    distanceToMouse = parseInt(distanceToMouse); // Round, throwing away all decimals
                    if(distanceToMouse < 1){
                        distanceToMouse = 1; // Avoid division by 0 here
                    }

                    var color = 100 / distanceToMouse / 4 * 255;
                    color = parseInt(color); // Round again

                    data[index + 0] = color; // Red
                        data[index + 1] = color; // Green
                        data[index + 2] = color; // Blue
                        data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>
```

<canvas style="position: relative;" id= "canvas_light_demo_2" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_2");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            
            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){
                    var index = (y * width + x) * 4;

                    var distanceToMouse = Math.sqrt((mousePosition.x - x)*(mousePosition.x - x) + (mousePosition.y - y)*(mousePosition.y - y));
                    distanceToMouse = parseInt(distanceToMouse); // Round, throwing away all decimals
                    if(distanceToMouse < 1){
                        distanceToMouse = 1; // Avoid division by 0 here
                    }

                    var color = 100 / distanceToMouse / 4 * 255;
                    color = parseInt(color); // Round again

                    data[index + 0] = color; // Red
                        data[index + 1] = color; // Green
                        data[index + 2] = color; // Blue
                        data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>



## Light effect #3 - multiple light sources

We can change the logic to be based on a list of light source; some static light source as well as a light source at the cursor position. We will then determine the pixel light based on the distance to the nearest light source:


```html
<canvas style="position: relative;" id= "canvas_light_demo_3" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_3");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = [{x: width/2, y: height/2}]; // Additional light sources, one at center, one a bottom right

        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var closestDistanceToAnyLight = 100000;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        if(distanceToLight < closestDistanceToAnyLight){
                            closestDistanceToAnyLight = distanceToLight;
                        }
					}

                    var index = (y * width + x) * 4;

                    var color = 100 / closestDistanceToAnyLight / 4 * 255;
                    color = parseInt(color); // Round again

                    data[index + 0] = color; // Red
                    data[index + 1] = color; // Green
                    data[index + 2] = color; // Blue
                    data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>
```

<canvas style="position: relative;" id= "canvas_light_demo_3" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_3");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = [{x: width/2, y: height/2}]; // Additional light sources, one at center, one a bottom right

        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var closestDistanceToAnyLight = 100000;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        if(distanceToLight < closestDistanceToAnyLight){
                            closestDistanceToAnyLight = distanceToLight;
                        }
					}

                    var index = (y * width + x) * 4;

                    var color = 100 / closestDistanceToAnyLight / 4 * 255;
                    color = parseInt(color); // Round again

                    data[index + 0] = color; // Red
                    data[index + 1] = color; // Green
                    data[index + 2] = color; // Blue
                    data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>




## Light effect #4 - enhancing logic to "sum"-logic

Now, instead of using the closest distance to determine a pixels light level, we will now give a pixel a score based on all distances to all light sources.


```html
<canvas style="position: relative;" id= "canvas_light_demo_4" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_4");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = [{x: width/2, y: height/2}]; // Additional light sources, one at center, one a bottom right

        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var score = 0;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        // Add value to score based on distance
                        score = score + 100 / distanceToLight;
					}

                    var index = (y * width + x) * 4;
                    var lightLevel = parseInt(score / 4 * 255);

                    data[index + 0] = lightLevel; // Red
                    data[index + 1] = lightLevel; // Green
                    data[index + 2] = lightLevel; // Blue
                    data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>
```
<canvas style="position: relative;" id= "canvas_light_demo_4" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_4");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = [{x: width/2, y: height/2}]; // Additional light sources, one at center, one a bottom right

        var updateCanvas = function(mousePosition){
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;            

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var score = 0;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        // Add value to score based on distance
                        score = score + 100 / distanceToLight;
					}

                    var index = (y * width + x) * 4;
                    var lightLevel = parseInt(score / 4 * 255);

                    data[index + 0] = lightLevel; // Red
                    data[index + 1] = lightLevel; // Green
                    data[index + 2] = lightLevel; // Blue
                    data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>



## Light effect #5 - switching back to binary drawing

A different, cool effect can be achieved by changing back logic to be draw/dont draw instead of fading light levels:


```html
<canvas style="position: relative;" id="canvas_light_demo_5" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_5");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = [{x: width/2, y: height/2}]; // Additional light sources, one at center, one a bottom right

        var updateCanvas = function(mousePosition){
        
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var score = 0;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        // Add value to score based on distance
                        score = score + 100 / distanceToLight;
					}

                    var index = (y * width + x) * 4;

                    // Binary logic
                    if(score > 4)
                    {
                        var lightLevel = 255;
                    }else{
                        var lightLevel = 0;
                    }
                    

                    data[index + 0] = lightLevel; // Red
                        data[index + 1] = lightLevel; // Green
                        data[index + 2] = lightLevel; // Blue
                        data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>

```
<canvas style="position: relative;" id="canvas_light_demo_5" width="600" height="300"></canvas>
<script>
    (function(){
        var canvas = document.getElementById("canvas_light_demo_5");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = [{x: width/2, y: height/2}]; // Additional light sources, one at center, one a bottom right

        var updateCanvas = function(mousePosition){
        
            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var score = 0;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        // Add value to score based on distance
                        score = score + 100 / distanceToLight;
					}

                    var index = (y * width + x) * 4;

                    // Binary logic
                    var lightLevel = 0;
                    if(score > 3.25)
                    {
                        lightLevel += 100;
                    }

                    if(score > 3.5)
                    {
                        lightLevel += 100;
                    }

                    if(score > 4)
                    {
                        lightLevel += 55;
                    }

                    data[index + 0] = lightLevel; // Red
                    data[index + 1] = lightLevel; // Green
                    data[index + 2] = lightLevel; // Blue
                    data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>

## Adding an fps counter

To keep track on performance, a good idea is to have an fps counter showing how many frames can be drawn per second (with highest better).

I have googled, and choose to use the first thing I found: [stats.js](https://github.com/mrdoob/stats.js/)

```script
npm install stats.js
```

<canvas style="position: relative;" id="canvas_fps_demo" width="600" height="300"></canvas>
<div id="stats_fps_demo"></div>
<script>
    (function(){
        var stats = new Stats();
        stats.showPanel( 1 ); // 0: fps, 1: ms, 2: mb, 3+: custom
        document.getElementById("stats_fps_demo").appendChild( stats.dom );

        var canvas = document.getElementById("canvas_fps_demo");
        canvas.style.cursor = 'none'; // Hide cursor

        var ctx = canvas.getContext("2d");

        var width = canvas.width;
        var height = canvas.height;
    
        var staticLightSources = []; // Additional light sources, one at center, one a bottom right
        var staticEntryCount = 4;
        for(var i = 0; i < staticEntryCount; i++){
            staticLightSources.push({x: width / staticEntryCount * i, y: 75});
        }

        var updateCanvas = function(mousePosition){
        
            stats.begin();

            var imgData = ctx.getImageData(0, 0, width, height);
            var data = imgData.data;

            // Create a single array containing all light sources
            var allLights = staticLightSources.slice();
            allLights.push(mousePosition);

            // Update the data array
            for(var x = 0; x < width; x++){

                for(var y = 0; y < height; y++){

                    var score = 0;
                    for(var i = 0; i < allLights.length; i++){
                        var currentLight = allLights[i];
                        var distanceToLight = Math.sqrt((currentLight.x - x)*(currentLight.x - x) + (currentLight.y - y)*(currentLight.y - y));
                        if(distanceToLight < 1){
                            distanceToLight = 1; // Avoid division by 0 here
                        }

                        // Add value to score based on distance
                        score = score + 100 / distanceToLight;
					}

                    var index = (y * width + x) * 4;

                    // Binary logic
                    if(score > 4)
                    {
                        var lightLevel = 255;
                    }else{
                        var lightLevel = 0;
                    }
                    

                    data[index + 0] = lightLevel; // Red
                        data[index + 1] = lightLevel; // Green
                        data[index + 2] = lightLevel; // Blue
                        data[index + 3] = 255; // Alpha (transparency)
                }
            }

            // Overwrite the canvas with the updated data array
            ctx.putImageData(imgData, 0, 0);

            stats.end();
        };

        updateCanvas({x: width/2, y: height/2});; // Ensures something drawn on load

        canvas.addEventListener("mousemove", function (e) {
            var mousePosition = {x: e.layerX, y: e.layerY};
            updateCanvas(mousePosition);
        });
    })();
</script>