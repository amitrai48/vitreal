# vitreal
MEAN stack


'use strict';

window.requestAnimationFrame = window.requestAnimationFrame ||
								window.mozRequestAnimationFrame||
								window.webkitRequestAnimationFrame ||
								window.oRequestAnimationFrame ||
								window.msRequestAnimationFrame ||
								function(callback){
									window.setTimeout(callback,1000/60)
								};

function Vector2(x,y){
	this.x = typeof x!=='undefined' ? x:0;
	this.y = typeof y!=='undefined' ? y:0;
}

Vector2.prototype.copy = function(){
	return new Vector2(this.x,this.y);
};

Vector2.prototype.equals = function(obj){
	return this.x === obj.x && this.y === obj.y;
}

Vector2.prototype.add = function(num){
	if(num.constructor === Vector2){
		this.x = this.x + num.x;
		this.y = this.y + num.y;
	}
	else if(num.constructor === Number){
		this.x = this.x + num;
		this.y = this.y + num;
	}
	return this;
}

Vector2.prototype.subtract = function(num){
	if(num.constructor === Vector2){
		this.x = this.x - num.x;
		this.y = this.y - num.y;
	}

	else if(num.constructor === Number){
		this.x = this.x - num;
		this.y = this.y - num;
	}

	return this;
}

Vector2.prototype.multiply = function(num){
	if(num.constructor === Vector2){
		this.x = this.x * num.x;
		this.y = this.y * num.y;
	}

	else if(num.constructor === Number){
		this.x = this.x * num;
		this.y = this.y * num;
	}

	return this;
}

Vector2.prototype.divide = function(num){
	if(num.constructor === Vector2){
		this.x = this.x / num.x;
		this.y = this.y / num.y;
	}

	else if(num.constructor === Number){
		this.x = this.x / num;
		this.y = this.y / num;
	}

	return this;
}

var Mouse ={
	position:{
		'x':0,
		'y':0
	},
	leftButtonPressed:false,
	leftDown:false
};

Mouse.reset = function(){
	Mouse.leftButtonPressed=false;
}

Mouse.mouseDown = function(evt){
	if(evt.which===1){
		if(!Mouse.leftDown)
			Mouse.leftButtonPressed = true;
		Mouse.leftDown=true;
	}
}

Mouse.mouseUp = function(evt){
	if(evt.which===1){
		Mouse.leftDown = false;
	}
}

var Keys = {
	'R':82,
	'G':71,
	'B':66
}

var Keyboard ={
	keyDown:-1
}

Keyboard.handleKeyDown = function(evt){
	Keyboard.keyDown = evt.keyCode
}

Keyboard.handleKeyUp = function(evt){
	Keyboard.keyDown = -1
}

Mouse.handleInput = function(evt){
	Mouse.position.x = evt.pageX;
	Mouse.position.y = evt.pageY;
}

var painterGameWorld={};
painterGameWorld.isOutsideWorld = function(position){
	return position.x<0 || position.x>sprite.background.width || position.y>sprite.background.height;
}

function Ball(){
	this.position= new Vector2();
	this.origin= new Vector2();
	this.velocity= new Vector2();
	this.isShooting = false,
	this.currentColor = sprite.canon_red
}


Ball.prototype.handleInput = function(delta){
	if(Mouse.leftButtonPressed && !this.shooting){
		this.shooting = true;
		this.velocity.x = (Mouse.position.x - this.position.x)*1.6;
		this.velocity.y = (Mouse.position.y - this.position.y)*1.6;
	}
};

Ball.prototype.update = function(delta){
	if(this.shooting){
		this.velocity.x = this.velocity.x*0.99;
		this.velocity.y +=6;
		console.log(this.position);
		/*this.position.x += this.velocity.x*delta;
		this.position.y += this.velocity.y*delta;*/
		this.position.add(this.velocity.multiply(delta));
	}
	else{
		if(Game.canon.currentColor === sprite.canon_red)
			this.currentColor = sprite.red_ball;
		else if(Game.canon.currentColor === sprite.canon_green)
			this.currentColor = sprite.green_ball;
		else if(Game.canon.currentColor === sprite.canon_blue)
			this.currentColor = sprite.blue_ball;
		this.position = Game.canon.ballPosition();
        this.position.x -= this.currentColor.width / 2;
        this.position.y -= this.currentColor.height / 2;
	}
	 if (painterGameWorld.isOutsideWorld(this.position))
        this.reset();
}

Ball.prototype.draw = function(){
	if(!this.shooting){
		return
	}
	Canvas2D.drawImage(this.currentColor,this.position,0,this.origin);
}

Ball.prototype.reset = function(){
	this.position = {x:0,y:0};
	this.shooting=false;
}


function PaintCan(positionOffset){
	this.currentColor = sprite.red_can;
	this.position = {'x':0,'y':0};
	this.velocity = {'x':0,'y':0};
	this.origin = {'x':0,'y':0};
	this.positionOffset = positionOffset;
	this.reset();
}

PaintCan.prototype.reset = function(){
	this.moveToTop();
	this.minVelocity=30;
}

PaintCan.prototype.moveToTop = function(){
	this.position={'x':this.positionOffset,'y':-200};
	this.velocity={'x':0,'y':0};
}

PaintCan.prototype.update = function(delta){
	this.position.x += this.velocity.x * delta;
	this.position.y += this.velocity.y * delta;
	 if (this.velocity.y === 0 && Math.random() < 0.01) {
        this.velocity = this.calculateRandomVelocity();
        this.currentColor = this.calculateRandomColor();
    }

    if (painterGameWorld.isOutsideWorld(this.position))
        this.moveToTop();
    this.minVelocity += 0.01;
}

PaintCan.prototype.draw = function(delta){
	Canvas2D.drawImage(this.currentColor,this.position,0,this.origin);
}

PaintCan.prototype.calculateRandomVelocity = function(){
	return {'x':0,'y':Math.random()*30 + this.minVelocity};
};

PaintCan.prototype.calculateRandomColor = function(){
	var random = Math.floor(Math.random()*3);
	if(random == 0)
		return sprite.red_can;
	else if(random == 1)
		return sprite.green_can;
	else if(random == 2)
		return sprite.blue_can;

}

var Canvas2D = {
	'canvas':undefined,
	'canvasContext':undefined
};

Canvas2D.initialize = function(canvasName){
	Canvas2D.canvas = document.getElementById(canvasName);
	Canvas2D.canvasContext = Canvas2D.canvas.getContext('2d');
}

Canvas2D.drawImage = function(sprite,position,rotation,origin){
	Canvas2D.canvasContext.save();
	Canvas2D.canvasContext.translate(position.x,position.y);
	Canvas2D.canvasContext.rotate(rotation);
	Canvas2D.canvasContext.drawImage(sprite,0,0,sprite.width,sprite.height,-origin.x,-origin.y,sprite.width,sprite.height);
	Canvas2D.canvasContext.restore();
};

Canvas2D.clear = function(){
	Canvas2D.canvasContext.clearRect(0,0,Canvas2D.canvas.width,Canvas2D.canvas.height);
}

var sprite={};
function Canon(){
		this.position={
		'x':72,
		'y':405
	};
	this.origin={
		'x':34,
		'y':34
	},
	this.rotation = 0;
	this.currentColor = sprite.canon_red;
	this.colorPosition = {x:55,y:388};
	this.colorOrigin = {x:0,y:0};
}


Canon.prototype.draw = function(){
	Canvas2D.drawImage(sprite.canon,this.position,this.rotation,this.origin);
	Canvas2D.drawImage(this.currentColor,this.colorPosition,0,this.colorOrigin)
}

Canon.prototype.handleInput = function(){
	var opposite = Mouse.position.y - this.position.y;
	var adjacent = Mouse.position.x - this.position.x;
	this.rotation = Math.atan2(opposite,adjacent);
	if(Keyboard.keyDown === Keys.R)
		this.currentColor = sprite.canon_red;
	else if(Keyboard.keyDown === Keys.G)
		this.currentColor = sprite.canon_green;
	else if(Keyboard.keyDown === Keys.B)
		this.currentColor = sprite.canon_blue;
}

Canon.prototype.ballPosition = function() {
    var opposite = Math.sin(this.rotation) * sprite.canon.width * 0.6;
    var adjacent = Math.cos(this.rotation) * sprite.canon.width * 0.6;
    return { x : this.position.x + adjacent, y : this.position.y + opposite };
};

var Game={
	'stillLoadingAssets':0
}

Game.initialize = function(){
	Game.canon = new Canon();
	Game.ball = new Ball();
	Game.can1 = new PaintCan(450);
	Game.can2 = new PaintCan(575);
	Game.can3 = new PaintCan(700);
}

Game.loadAssets = function(imageName){
	var image = new Image();
	image.src = imageName;
	Game.stillLoadingAssets +=1;
	image.onload=function(){
		Game.stillLoadingAssets -=1;
	};
	return image;
}
Game.start = function(){
	Canvas2D.initialize('gameArea');
	document.onmousemove = Mouse.handleInput;
	document.onkeydown = Keyboard.handleKeyDown;
	document.onkeyup = Keyboard.handleKeyUp;
	document.onmousedown = Mouse.mouseDown;
	document.onmouseup = Mouse.mouseUp;
	var spriteFolder="../images/";
	sprite.background = Game.loadAssets(spriteFolder+"spr_background.jpg");
	sprite.canon = Game.loadAssets(spriteFolder+"spr_cannon_barrel.png");
	sprite.canon_red = Game.loadAssets(spriteFolder+"spr_cannon_red.png");
	sprite.canon_green = Game.loadAssets(spriteFolder+"spr_cannon_green.png");
	sprite.canon_blue = Game.loadAssets(spriteFolder+"spr_cannon_blue.png");
	sprite.red_ball = Game.loadAssets(spriteFolder+"spr_ball_red.png");
	sprite.green_ball = Game.loadAssets(spriteFolder+"spr_ball_green.png");
	sprite.blue_ball = Game.loadAssets(spriteFolder+"spr_ball_blue.png");
	sprite.red_can = Game.loadAssets(spriteFolder+"spr_can_red.png");
	sprite.green_can = Game.loadAssets(spriteFolder+"spr_can_green.png");
	sprite.blue_can = Game.loadAssets(spriteFolder+"spr_can_blue.png");
	Game.assetLoadingLoop();

}

Game.assetLoadingLoop = function(){
	if(Game.stillLoadingAssets>0){
		window.requestAnimationFrame(Game.assetLoadingLoop);
	}
	else{
		Game.initialize();
		Game.mainLoop();
	}
};

Game.handleInput = function(delta){
	Game.ball.handleInput(delta);
	Game.canon.handleInput();
}

Game.update = function(delta){
	Game.ball.update(delta);
	Game.can1.update(delta);
	Game.can2.update(delta);
	Game.can3.update(delta);
}

Game.draw = function(){
	Canvas2D.drawImage(sprite.background,{x:0,y:0},0,{x:0,y:0});
	Game.ball.draw();
	Game.canon.draw();	
	Game.can1.draw();
	Game.can2.draw();
	Game.can3.draw();
}

Game.mainLoop = function(){
	var delta = 1/60;
	Game.handleInput(delta);
	Game.update(delta);
	Canvas2D.clear();
	Game.draw(delta);
	Mouse.reset();
	window.requestAnimationFrame(Game.mainLoop);
}

document.addEventListener('DOMContentLoaded',function(evt){
	Game.start();
});
