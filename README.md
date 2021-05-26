<!DOCTYPE html>
<html>
    <head>
        <meta charset="utf-8">
        <title>HTML Platformer (Need Help)</title>
        <style>
            body {
                overflow: hidden;
            }
            #game-canvas {
                margin: -8px;
            }
        </style>
    </head>
    <body>
        <canvas id="game-canvas"></canvas>
        <script>
            /*
            Credits
            Help From:
            Bluish
            https://www.khanacademy.org/profile/kaid_168759064112144461843068
            */
            /******  CANVAS SETUP  ******/
            var gameCanvas = document.getElementById("game-canvas");
            var ctx = gameCanvas.getContext("2d");
            gameCanvas.width = "400";
            gameCanvas.height = "400";
            
            
            /******  ARROW KEY SETUPS  ******/
            var leftPressed = false;
            var upPressed = false;
            var rightPressed = false;
            document.onkeydown = keyPressed;
            document.onkeyup = keyReleased;
            function keyPressed(e) {
                e = e || window.event;
                if (e.keyCode == '37') {
                   leftPressed = true;
                } else if (e.keyCode == '38') {
                    upPressed = true;
                } else if (e.keyCode == '39') {
                   rightPressed = true;
                }
            }
            function keyReleased(e) {
                e = e || window.event;
                if (e.keyCode == '37') {
                   leftPressed = false;
                } else if (e.keyCode == '38') {
                    upPressed = false;
                } else if (e.keyCode == '39') {
                   rightPressed = false;
                }
            }
            
            
            /******  THE ACTUAL GAME  ******/
            var blockSize = gameCanvas.height / 15;
            var Level = 1;
            var l = Level - 1;
            var Levels = [
                [
                    "bbbbbbbbbbbbbbb",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b             b",
                    "b  p       bbbb",
                    "b             b",
                    "b     ^^^    @b",
                    "bbbbbbbbbbbbbbb",
                ],
            ];
            var blocks = [];
            var players = [];
            var spikes = [];
            var portals = [];
            
            function Collide(thing1, thing2) {
                return thing1.x - thing2.x < thing2.w &&
                    thing2.x - thing1.x < thing1.w &&
                    thing1.y - thing2.y < thing2.h &&
                    thing2.y - thing1.y < thing1.h;
            }
            
            function Block(config) {
                this.x = config.x;
                this.y = config.y;
                this.w = config.w;
                this.h = config.h;
                this.draw = function() {
                    ctx.fillStyle = "rgb(50, 50, 50)";
                    ctx.strokeStyle = ctx.fillStyle;
                    ctx.fillRect(this.x, this.y, this.w, this.h);
                    ctx.strokeRect(this.x, this.y, this.w, this.h)
                };
            }
            function Player(config) {
                this.x = config.x;
                this.y = config.y;
                this.w = config.w;
                this.h = config.h;
                this.xVel = 0;
                this.yVel = 0;
                this.g = 0;
                this.draw = function() {
                    ctx.fillStyle = "rgb(255, 50, 50)";
                    ctx.strokeStyle = ctx.fillStyle;
                    ctx.fillRect(this.x, this.y, this.w, this.h);
                    ctx.strokeRect(this.x, this.y, this.w, this.h);
                };
                this.moveX = function() {
                    if (leftPressed) {
                        this.xVel = -3;
                    } else if (rightPressed) {
                        this.xVel = 3;
                    } else {
                        this.xVel = 0;
                    }
                    this.x += this.xVel;
                };
                this.moveY = function() {
                    if (upPressed && this.yVel < 0.1 && this.yVel > -0.1 && this.g < 0.1 && this.g > -0.1) {
                        this.yVel = -5;
                    }
                    if (this.yVel < 8) {
                        this.g = 0.2;
                    } else {
                        this.g = 0;
                    }
                    if (this.yVel > 8) {
                        this.yVel = 8;
                    }
                    if (this.yVel < -8) {
                        this.yVel = -8;
                    }
                    this.y += this.yVel;
                    this.yVel += this.g;
                };
                this.collideX = function() {
                    for (var i = 0; i < blocks[l].length; i++) {
                        if (Collide(this, blocks[l][i])) {
                            if (this.x < blocks[l][i].x) {
                                this.x = blocks[l][i].x - this.w;
                                this.xVel = 0;
                            } else {
                                this.x = blocks[l][i].x + blocks[l][i].w;
                                this.xVel = 0;
                            }
                        }
                    }
                    for (var i = 0; i < spikes[l].length; i++) {
                        if (Collide(this, spikes[l][i])) {
                            if (this.x < spikes[l][i].x) {
                                this.x = spikes[l][i].x - this.w;
                                this.xVel = 0;
                            } else {
                                this.x = spikes[l][i].x + spikes[l][i].w;
                                this.xVel = 0;
                            }
                        }
                    }
                };
                this.collideY = function() {
                    for (var i = 0; i < blocks[l].length; i++) {
                        if (Collide(this, blocks[l][i])) {
                            if (this.y < blocks[l][i].y) {
                                this.y = blocks[l][i].y - this.h;
                                this.yVel = 0;
                                this.g = 0;
                            } else {
                                this.y = blocks[l][i].y + blocks[l][i].h;
                                this.yVel *= -1;
                            }
                        }
                    }
                    for (var i = 0; i < spikes[l].length; i++) {
                        if (Collide(this, spikes[l][i])) {
                            if (this.y < spikes[l][i].y) {
                                this.y = spikes[l][i].y - this.h;
                                this.yVel *= -1;
                                this.yVel += 0.4;
                                this.g = 0;
                            } else {
                                this.y = spikes[l][i].y + spikes[l][i].h;
                                this.yVel *= -1;
                            }
                        }
                    }
                };
                this.update = function() {
                    this.collideY();
                    this.moveX();
                    this.collideX();
                    this.moveY();
                };
            }
            function Spike(config) {
                this.x = config.x;
                this.y = config.y;
                this.w = config.w;
                this.h = config.h;
                this.draw = function() {
                    ctx.fillStyle = "rgb(200, 200, 200)";
                    ctx.strokeStyle = ctx.fillStyle;
                    ctx.beginPath();
                    ctx.moveTo(this.x + (this.w / 2), this.y);
                    ctx.lineTo(this.x + this.w, this.y + this.h);
                    ctx.lineTo(this.x, this.y + this.h);
                    ctx.closePath();
                    ctx.fill();
                };
            }
            function Portal(config) {
                this.x = config.x;
                this.y = config.y;
                this.w = config.w;
                this.h = config.h;
                this.draw = function() {
                    ctx.fillStyle = "rgb(50, 255, 50)";
                    ctx.strokeStyle = ctx.fillStyle;
                    ctx.fillRect(this.x, this.y, this.w, this.h);
                    ctx.strokeRect(this.x, this.y, this.w, this.h);
                };
            }
            
            
            function UpdateLevel() {
                blocks[l] = [];
                players[l] = [];
                spikes[l] = [];
                portals[l] = [];
                for (var i = 0; i < Levels[l].length; i++) {
                    for (var j = 0; j < Levels[l][i].length; j++) {
                        switch(Levels[l][i][j]) {
                            case "b":
                                blocks[l].push(new Block({
                                    x: j * blockSize,
                                    y: i * blockSize,
                                    w: blockSize,
                                    h: blockSize
                                }));
                            break;
                            case "p":
                                players[l].push(new Player({
                                    x: j * blockSize,
                                    y: i * blockSize,
                                    w: blockSize,
                                    h: blockSize
                                }));
                            break;
                            case "^":
                                spikes[l].push(new Spike({
                                    x: j * blockSize,
                                    y: i * blockSize,
                                    w: blockSize,
                                    h: blockSize
                                }));
                            break;
                            case "@":
                                portals[l].push(new Portal({
                                    x: j * blockSize,
                                    y: i * blockSize,
                                    w: blockSize,
                                    h: blockSize
                                }));
                            break;
                        }
                    }
                }
            }
            UpdateLevel();
            
            function draw() {
                ctx.clearRect(0, 0, gameCanvas.width, gameCanvas.height);
                for (var i = 0; i < blocks[l].length; i++) {
                    blocks[l][i].draw();
                }
                for (var i = 0; i < players[l].length; i++) {
                    players[l][i].draw();
                    players[l][i].update();
                }
                for (var i = 0; i < spikes[l].length; i++) {
                    spikes[l][i].draw();
                }
                for (var i = 0; i < portals[l].length; i++) {
                    portals[l][i].draw();
                }
            }
            window.setInterval(draw, 1000 / 60);
        </script>
    </body>
</html>
