Welcome to the Canvas Asteroids kata! (A kata is a practice of something to help you learn/memorize the steps.)

This is built using Vanilla JS (ES5) and HTML5/CSS3 and as a result has no dependencies. The two additional JS files are:
 * rAF.js - a cross-browser polyfill for requestAnimationFrame
 * KeyCode.js - an ASCII keycode constant to use human-readable values instead of ASCII codes
Neither of these are absolutely necessary, but make it convenient while developing/debugging.

# I don't care about all this! Just let me see the finished result!
[Play Here!](https://eyeofmidas.github.io/canvas-asteroids/asteroids.html)

# Setup

To start, create a folder on your computer (Mine is called canvas-asteroids) and in it put these three files:

* https://raw.githubusercontent.com/EyeOfMidas/canvas-asteroids/start-gamedev/asteroids.html
* https://raw.githubusercontent.com/EyeOfMidas/canvas-asteroids/start-gamedev/KeyCode.js
* https://raw.githubusercontent.com/EyeOfMidas/canvas-asteroids/start-gamedev/rAF.js

Then you can open your browser and navigate to the file directly. You can double-click on the asteroids.html file or go to:

    file:///PATH_TO_YOUR_FOLDER/canvas-asteroids/asteroids.html

which will open the local file in your preferred browser.

You will know it worked because your browser will display a completely black page. If it's showing a white page, something is missing or mis-copied. (If there's a bug, please report it via the [Issues section on Github](https://github.com/EyeOfMidas/canvas-asteroids/issues))

# The Kata
At the bottom of the asteroids.html file, you can see three functions with comments in them:
* `init()`
* `update()`
* `draw(context)`

Init is called once to setup and start the game: in this function you should populate the arrays and objects with the values they should have at the start of the game.

Update is called once per frame, just before the draw happens. In this function you should keep all your game logic: movement, physics, collisions. DO NOT MIX logic and drawing code; debugging any issues later will be a lot easier.

Draw takes in the canvas context object, and is called once per frame to draw the game objects. Draw should always be called after update, and will display the current frame of the game on the canvas.

## Starfield
We can't have an asteroids game without setting it in space! So let's draw some random stars on the screen. Create three new functions:
initStars()
updateStars()
drawStars(context)

and call them in their respective game-loop functions. (For clarity, put `initStars()` at the bottom of the `init()` function, etc).

We will need to populate the `starsNear` array with some star objects. In the initStars function, write a for loop from 0-50 and push in stars into the starsNear array.

    starsNear = [];
    for(let i = 0; i < 50; i++) {
        let star = {};
        //This function picks a random coordinate within the bounds of the window
        let randomPosition = getRandomPositionOnScreen();
        star.x = randomPosition.x;
        star.y = randomPosition.y;
        //stars are circles, so their width and height are the same size
        star.width = rand(2, 4);
        star.height = star.width;
        starsNear.push(star);
    }

Now we've got a nice populated starfield, but nothing will show up until we draw it. inside the drawStars(context) function, create a for loop to iterate through the starsNear array and draw some circles.

    //this will be the color of all the stars
    context.fillStyle = "lightgray";
    for (let i = 0; i < starsNear.length; i++) {
        let star = starsNear[i];
        //saving and restoring the context like this allows us to draw 0,0 centered images, which makes later transformations easier.
        context.save();
        //moves the canvas matrix to be centered on the star's screen position
        context.translate(star.x, star.y);
        //star drawing a vector shape
        context.beginPath();
        //plots out a cirle at 0,0: (x, y, radius, startAngle, endAngle)
        context.arc(0, 0, star.width / 2, 0, 2 * Math.PI);
        //draws the shape as a filled, lightgray circle
        context.fill();
        context.restore();
    }

At this point, you should see a smattering of random dots all across your screen. But it doesn't feel very "space-y".

## Parallax
Space has movement and depth, and our random dots have neither of these things. Let's duplicate the actions we took to populate the starsNear array and also make a starsFar array we could update separately.
The init loop should now have two for loops in it, with the one difference being starsFar stars will be more dense, but smaller sized.

    for (let i = 0; i < 300; i++) {
        ...
        star.width = rand(0.1, 2);
        star.height = star.width;
        starsFar.push(star);
    }

You'll draw the starsFar array exactly the same way, but to give us a neat drifting parallax effect, let's make the two star arrays move at slightly different rates.

Inside the updateStars() function, put in two for loops, one for starsNear and starsFar. These two loops will be nearly identical, except that the starsFar loop will move the stars much slower than starsNear.

    for (let i = 0; i < starsNear.length; i++) {
        let star = starsNear[i];
        star.x += 0.15; //these stars a closer so they'll appear to move faster
        star.y += 0.15;
        //this function checks the position against the width/height and will wrap the objects around the screen.
        wrapAround(star);
    }

    for (let i = 0; i < starsFar.length; i++) {
        let star = starsFar[i];
        star.x += 0.05; //far-away stars will move much more slowly
        star.y += 0.05;
        wrapAround(star);
    }

## Ship

## Movement

## Thrust Tail

## Asteroids

## Collisions

## Bullets

## Bullet Collisions

## Asteroid Splitting

## Scoring and UI

## Game Over

