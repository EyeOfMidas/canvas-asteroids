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
* `initStars()`
* `updateStars()`
* `drawStars(context)`

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

The ship is set up and drawn the same way as the stars are, but there's only one. First, create 3 new functions:
* `initShip()`
* `updateShip()`
* `drawShip(context)`

and call them in their respective game-loop functions. (For clarity, put `initShip()` at the bottom of the `init()` function, etc).

First, set up the ship with it's default values. There is already a `ship` object defined, so go ahead and copy the initialization and also put it in the `initShip()` function.

    function initShip() {
        ship = { 
            x: 0,
            y: 0,
            angle: -90,
            acceleration: { x: 0, y: 0 },
            velocity: { x: 0, y: 0 },
            width: 10,
            height: 15,
            rotationSpeed: 3,
            thrust: 0.2,
            bulletSpeed: 2,
            life: 3,
            score: 0,
        };
    }

To draw the ship as a triangle, I created a convenient function called `shipShape` that will draw it nicely. Just like drawing the stars, we're going to translate to the ship position, and draw everything centered at 0,0 to make things like rotations easy to calculate.

    function drawShip(context) {
        context.save();
        // the styleshere are css color values: try putting any HEX code, rgb() or color word here!
        context.strokeStyle = "red";
        context.fillStyle = "crimson";
        context.translate(ship.x, ship.y);

        context.beginPath();
        shipShape(context); // this is just a convenient function for drawing a triangle
        context.fill();
        context.stroke();

        context.restore();
    }

Now you should be drawing a neat red triangle ship out in space! ... way off in the corner. Let's tweak some of the initial values of the ship so it starts in the center of the screen. Inside the `initShip` function, add the following lines:

    ship.x = canvas.width / 2;
    ship.y = canvas.height / 2;

This will set the ship starting position to be the center of the viewable window, however big it might be.

The ship seems a little small, so we can adjust the width and height here too, and the `shipShape` function will take care of drawing the properly sized triangle. Let's change the width to 15 and the height to 20, so the ship is a bit bigger.

    ship = {
        ...
        width: 15,
        height: 20,
        ...
    }


## Movement

This is a pretty spiffy triangle, but it would be a lot cooler if we could make it move around in space. I've added a few _event listeners_ to handle the details, but essentially I'm setting up an array of keys and keeping track of if they're pressed or not. We can use the `keys` array in the update loop to check if a key is currently held down, and make the ship do things based on that.

Let's add in some turning, first. Inside `updateShip()` add the following two `if` checks:

    function updateShip() {
        if (keys[KeyCode.Left]) {
            ship.angle -= ship.rotationSpeed;
        }

        if (keys[KeyCode.Right]) {
            ship.angle += ship.rotationSpeed;
        }
    }

This will detect if the left or right arrow keys are pressed, and change the `angle` on the ship by a small amount. But we'll need to change the way we draw the ship to reflect the proper rotation. Inside `drawShip`, after the `context.translate` but before we `beginPath` you'll need to add a context rotation.

    function drawShip(context) {
        ...
        context.translate(ship.x, ship.y);
        context.rotate((ship.angle - 90) * (Math.PI / 180)); // this will rotate the entire context around the ship.position
        context.beginPath();
        ...
    }

Neat! The ship will turn around. We'll need to add some thrust and space physics to really make this feel like a space ship though. Typical physics rules in 2D games use acceleration to affect velocity, and velocity to affect position.

At the top of the `updateShip` function, let's zero-out the ship acceleration each update loop.

    ship.acceleration.x = 0;
    ship.acceleration.y = 0;

and right below that, let's add a key listener for up, to apply accleration (thrust) in the direction the ship is facing.

    if (keys[KeyCode.Up]) {
        // the getShipAnglePosition uses sin and cos to determine which way the ship is facing
        let position = getShipAnglePosition();

        //then we apply the maximum thrust value to those values to move the ship in that direction
        ship.acceleration.x = ship.thrust * position.x;
        ship.acceleration.y = ship.thrust * position.y;
    }

Now that we've turned on directional accleration, all that's left to do is to apply it to the velocity and the position. Below the turning logic, add the following calculations:

    ship.velocity.x += ship.acceleration.x;
    ship.velocity.y += ship.acceleration.y;

    ship.x += ship.velocity.x;
    ship.y += ship.velocity.y;

And since we want the ship to stay where we can see it, use the `wrapAround` function that we used for the stars as well.

        wrapAround(ship); // don't get lost in space!
    }

## Thrust Tail

We've got a zippy little ship flying around space, but something is missing. Games are always about fun feedback, so let's take a bit of a detour to add in some polish: a flame tail!

There's not much to it; we'll use similar drawing logic for the shipShape and instead draw a little orange triangle at the base. We'll only draw the flame when the thrust key is held down, so the player can get immediate feedback about what they're doing.

Inside the `drawShip` function, below where we draw the ship, but still inside the ship coordinate space (you know, put before the `context.restore()`) you can add this.

    if (keys[KeyCode.Up]) {
        context.fillStyle = "orange";
        context.beginPath();
        context.moveTo(-ship.width / 4, -ship.height / 2);
        context.lineTo(ship.width / 4, -ship.height / 2);
        context.lineTo(0, -1.2 * ship.height); // fiddle with the -1.2 value here to change how big the flame is
        context.lineTo(-ship.width / 4, -ship.height / 2);
        context.fill();
    }

Now when you press the Up key, a little jet of rocket flame will shoot out the back of your ship and send you zooming through space!

## Asteroids

## Collisions

## Bullets

## Bullet Collisions

## Asteroid Splitting

## Scoring and UI

## Game Over
