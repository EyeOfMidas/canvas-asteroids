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

Space isn't really space without some huge gray rocks floating through it. Since this is a short kata, we can make do with some gray circles. Much like the stars and ship, let's set up some functions to contain the asteroids.

* `initAsteroids()`
* `updateAsteroids()`
* `drawAsteroids(context)`

and call them in the appropriate spots.

Inside the `initAsteroids` function, let's make a loop like we did with starsNear, and add a few asteroid objects to an array. We want to give these asteroid objects a size, a position, and velocity so they can drift around.

    function initAsteroids() {
        asteroids = [];
        for (let i = 0; i < 2; i++) {
            let position = getRandomPositionOnScreen();
            let velocity = { x: rand(-1, 2), y: rand(-1, 2) };
            let size = { width: 200, height: 200 };
            let asteroid = {};
            asteroid.x = position.x;
            asteroid.y = position.y;
            asteroid.width = size.width;
            asteroid.height = size.height;
            asteroid.velocity = {};
            asteroid.velocity.x = velocity.x;
            asteroid.velocity.y = velocity.y;
            asteroids.push(asteroid);
        }
    }

The drawing function will be similar to both the stars and ship drawing function; we want to center the context on the asteroid and draw it as a circle.

    function drawAsteroids(context) {
        context.strokeStyle = "lightgray";
        context.fillStyle = "gray";
        for (let i = 0; i < asteroids.length; i++) {
            let asteroid = asteroids[i];
            context.save();
            context.translate(asteroid.x, asteroid.y);
            context.beginPath();
            context.arc(0, 0, asteroid.width / 2, 0, 2 * Math.PI);
            context.fill();
            context.stroke();
            context.restore();
        }
    }

And now we've got some big gray circles on the screen, adding the position update loop for each asteroid will make them the best kind of floaty.

    function updateAsteroids() {
        for (let i = 0; i < asteroids.length; i++) {
            let asteroid = asteroids[i];
            asteroid.x += asteroid.velocity.x;
            asteroid.y += asteroid.velocity.y;
            wrapAround(asteroid); // they should remain in the play field
        }
    }

At this point, you should have two asteroids drifting randomly across your starfield. Truly a space to behold!

## Collisions

But what good are giant rocks if you can't crash into them? We can use a really simple formula to simulate collision logic.
In the file there's already a function called `doesCollide()` that uses the distance formula to determine if two objects are too close to each other. This isn't nearly as accurate as a full physics engine since it treats all objects like circles, but... all of the objects we can crash into are circles!

Inside the updateShip function, after we manipulate the ship and wrap it to the screen, add a loop that checks over all the asteroids and determines if the ship is intersecting them.

    for (let j = 0; j < asteroids.length; j++) {
        let asteroid = asteroids[j];
        if (doesCollide(ship, asteroid)) {
             // crash into this asteroid!
            break;
        }
    }

But what do we want to do? Create a function called `hitShip(asteroid)` and call it from inside our collision loop (where we have the comment _ // crash into this asteroid!_). Inside of `hitShip` we should do a few things; first, we should pick a new position for the ship to spawn, set the velocity and acceleration to 0, and force the thrust key to no longer trigger.

    function hitShip(asteroid) {
        // move to a new random location (hopefully not somewhere an asteroid already is)
        ship.x = rand(0, canvas.width);
        ship.y = rand(0, canvas.height);
        // quit moving the ship
        ship.velocity.x = 0;
        ship.velocity.y = 0;
        ship.acceleration.x = 0;
        ship.acceleration.y = 0;
        // if the user is still holding up, clear that key
        keys[KeyCode.Up] = false;
    }

Now when the ship touches an asteroid, it will vanish and appear somewhere else!

## Bullets

A game where you only can die is not really fun, so let's add a way to defend your ship. We can spawn bullets that can collide with asteroids and break them apart. Like any other object that moves and is drawn, let's create some containing functions and call them in their appropriate places.

* `initBullets()`
* `updateBullets()`
* `drawBullets(context)`

Since we're not creating bullets right when the game starts, the `initBullets` function will just set the bullets array to empty, and get it ready to receive our bullet objects.

    function initBullets() {
        bullets = [];
    }

Since bullets won't happen unless the player shoots them, let's add in a way to add new bullets when they press a key. Create a new function called `addBullet()` and call it inside our `updateShip` function.

    if (keys[KeyCode.Space]) {
        addBullet();
    }

Inside the `addBullet` function, you'll have to create a bullet object and push it into the bullets array. Bullets should spawn from the front of the ship, and inherit the ship's velocity. Bullets should also have a size so they can hit things using our collision function.

    function addBullet() {
        let anglePosition = getShipAnglePosition();
        let bullet = {};
        bullet.x = ship.x + ((ship.height / 2) * anglePosition.x);
        bullet.y = ship.y + ((ship.height / 2) * anglePosition.y);
        bullet.width = 6;
        bullet.height = 6;
        bullet.velocity = {};
        bullet.velocity.x = ship.bulletSpeed * anglePosition.x + ship.velocity.x;
        bullet.velocity.y = ship.bulletSpeed * anglePosition.y + ship.velocity.y;
        bullets.push(bullet);
    }

With this change, pressing spacebar will create bullets and push them into our bullet array, but we can't see any because we're not drawing them!

Drawing bullets is almost identical to how we draw stars:

    function drawBullets(context) {
        context.fillStyle = "white";
        for (let i = 0; i < bullets.length; i++) {
            let bullet = bullets[i];
            context.save();
            context.translate(bullet.x, bullet.y);
            context.beginPath();
            context.arc(0, 0, bullet.width / 2, 0, 2 * Math.PI);
            context.fill();
            context.restore();
        }
    }

Playing the game at this point will reveal three important facts. We have no rate limit to how quickly bullets can spawn (you make one bullet per frame drawn when holding spacebar), bullets last forever on screen, and none of the bullets are moving!

Let's get the bullets moving first by updating them inside the `updateBullets` function. Bullets will also use the same physics logic that asteroids do, to drift around.

    function updateBullets() {
        for (let i = 0; i < bullets.length; i++) {
            let bullet = bullets[i];
            bullet.x += bullet.velocity.x;
            bullet.y += bullet.velocity.y;
            wrapAround(bullet);
        }
    }

This is really neat, actually; bullets spawn infinitely and last forever, so if you spend a few minutes playing your screen will end up a hailstorm of random bullets. Let's add a lifetime so they go away after a few ticks.

When we `addBullet` let's give the bullet a lifetime value as well.

    bullet.lifespan = 75;

In the `updateBullets` function, deduct 1 from each bullets' lifespan when we update them.

    bullet.lifespan--;

After we have updated all the bullets (outside the for loop), we'll have to go through the bullets array and filter out any bullets that have a lifespan less than or equal to 0.

    bullets = bullets.filter(bullet => bullet.lifespan > 0);

Now when we try it out, the bullets still spawn very quickly, but will vanish shortly. Let's slow down the rate that we add bullets, and add a shoot delay. Inside `addBullet` set the `shootDelay` variable to a reasonable number.

    shootDelay = 15;

Now when we spawn the bullets, we need to check if we are within the delay time (just like how we do the lifespan for bullets) and only add a bullet when we've waited long enough.

    ...
    shootDelay--;
    if (keys[KeyCode.Space]) {
        if (shootDelay <= 0) {
            addBullet();
        }
    }
    ...

Don't forget to deduct 1 from the shoot delay each update loop, or you'll only be able to shoot one bullet ever!

## Bullet Collisions

Next, we'll need to compare the positions of the bullets with the asteroids to determine if they're colliding. Inside the `updateBullets` function, while we're looping through the bullets, add a check against the asteroids to see if the bullet collides.

    ...
    wrapAround(bullet);
    bullet.lifespan--;

    // check each asteroid against this bullet
    for (let j = 0; j < asteroids.length; j++) {
        let asteroid = asteroids[j];
        if (doesCollide(bullet, asteroid)) {
            hitAsteroid(bullet, asteroid);
            break;
        }
    }
    ...

Create the `hitAsteroid(bullet, asteroid)` function and we can destroy both the bullet and asteroid when they collide.

    function hitAsteroid(bullet, asteroid) {
        bullet.lifespan = 0;
        let asteroidIndex = asteroids.indexOf(asteroid);
        asteroids.splice(asteroidIndex, 1);
    }

## Asteroid Splitting

A single bullet destroying each asteroid is pretty boring, so let's make each asteroid instead spawn two more smaller asteroids. Before we do that we'll need to refactor some of the code to make the asteroid construction easier to reuse without having to copy-and-paste. 

In the `initAsteroid` function, pull out the asteroid-creating into a separate function, called `addAsteroid`. Replace the lines that built and added the asteroid to the asteroids array with a call to the refactored function. Notice how we didn't actually change anything here; a refactor is a code change that alters the structure, but does not alter the functionality of the code.

    function initAsteroids() {
        asteroids = [];
        for (let i = 0; i < 2; i++) {
            let position = getRandomPositionOnScreen();
            let velocity = { x: rand(-1, 2), y: rand(-1, 2) };
            let size = { width: 200, height: 200 };
            addAsteroid(position, velocity, size); // <-- this is now a function
        }
    }

    function addAsteroid(position, velocity, size) { // but this is still doing the same thing
        let asteroid = {};
        asteroid.x = position.x;
        asteroid.y = position.y;
        asteroid.width = size.width;
        asteroid.height = size.height;
        asteroid.velocity = {};
        asteroid.velocity.x = velocity.x;
        asteroid.velocity.y = velocity.y;
        asteroids.push(asteroid);
    }

With the ability to add asteroids easily reusable, we should add a few lines to the end of the `hitAsteroid` function. Let's create two new asteroids that are half the size of the original one, and add in a little bit of velocity randomness. We still want the new asteroids moving in the same general direction as the original one, so we'll inherit some of it's velocity.

        ...
        asteroids.splice(asteroidIndex, 1);

        // this size object is going to be half the size of the asteroid this bullet is colliding with
        let newSize = { width: asteroid.width / 2, height: asteroid.height / 2 };

        let velocityVariation = {};

        // a slightly random velocity, based on the parent asteroid
        velocityVariation.x = asteroid.velocity.x + rand(-1, 2);
        velocityVariation.y = asteroid.velocity.y + rand(-1, 2);
        addAsteroid(asteroid, velocityVariation, newSize);

        // a slightly different random velocity, so they drift apart
        velocityVariation.x = asteroid.velocity.x + rand(-1, 2);
        velocityVariation.y = asteroid.velocity.y + rand(-1, 2);
        addAsteroid(asteroid, velocityVariation, newSize);
    }

At this point, you've got a decent game. You can fly around and shoot asteroids until they're teeny tiny impossible-to-hit specks that insta-kill you... yea we need to put a limit on how small they're allowed to get before they're just destroyed.

We can use an early return conditional to check if the size of the asteroid is too small, and just not create the two new ones. Above the creation of the two new asteroids, put in a check on the current asteroid's size, and early return if it's below a certain threshold.

    if (asteroid.width <= 25) {
        return;
    }

Now we can play, and shoot all the asteroids until there are none left!

## Scoring and UI

If you've played your game for a little while, you realize there's some things missing. You can die any number of times with no real penalty, and once all the asteroids are gone, it's pretty empty and pointless. So let's add some arbitrary numbers to influence your emotions! Add a new function called `drawUI(context)` and add it at the end of our `draw` function.

Since UI always goes over top of anything else on the screen, it has to be drawn last.

    function drawUI(context) {
        context.font = "30px Arial";
        context.textAlign = "left";
        context.fillText(`Lives: ${ship.life}`, 10, 50);
        context.textAlign = "right";
        context.fillText(`Score: ${ship.score}`, canvas.width - 10, 50);
    }

Since we've kept game data, update logic and drawing logic separate, it's really easy to ask the appropriate objects about their values and just draw them on the screen. But we're not actually adding points, or subtracting lives. So let's find some appropriate places to adjust those.

In the `hitShip` function seems to be a pretty reasonable place to subtract a life; at the top of that function just add

    ship.life--;

and watch it count down as you crash over and over. Similarly, adding to your score makes the most sense when a bullet hits an asteroid, so in the `hitAsteroid` function, add

    ship.score += asteroid.width * 10;

and watch your score skyrocket as you blast those space rocks.




## Game Over
