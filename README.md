# Tizen-App-Barrage
###概述
Barrage基于web project开发，本篇主要描述Barrage的核心算法。
###核心算法
根据游戏需求，抽象出三个类，分别是：
* Bullet.js
* Plain.js
* Global.js

####Bullet.js
<pre>
/**
 * 子弹类 
 * */
function Bullet(belong,x,y,angle,xspeed,yspeed,aspeed,speed){
	base(this,LSprite,[]);
	var self = this;
	//子弹所属
	self.belong = belong;
	//出现位置
	self.x = x;
	self.y = y;
	//角度
	self.angle = angle;
	//移动速度
	self.speed = speed;
	//xy轴速度
	self.xspeed = xspeed;
	self.yspeed = yspeed;
	//旋转角度加成
	self.aspeed = aspeed;
	//子弹图片
	var bitmapdata,bitmap;
	bitmapdata = new LBitmapData(imglist["item1"]);
	bitmap = new LBitmap(bitmapdata);
	self.bitmap = bitmap;
	//显示
	self.addChild(bitmap);
}

/**
 * 循环
 * @param 子弹序号
 * */
Bullet.prototype.onframe = function (index){
	var self = this;

	//子弹移动
	self.x += self.xspeed;
	self.y += self.yspeed;
	
	//子弹角度变更
	if(self.aspeed != 0){
		self.angle += self.aspeed;
		//子弹角度变更后，重新计算xy轴速度
		self.xspeed = self.speed*Math.sin(self.angle * Math.PI / 180);
		self.yspeed = self.speed*Math.cos(self.angle * Math.PI / 180);
	}
	//子弹位置检测
	if(self.x < 0 || self.x > LGlobal.width || self.y < 0 || self.y > LGlobal.height){
		//从屏幕移除
		backLayer.removeChild(self);
		//从子弹数组移除
		barrage.splice(index,1);
	}else{
		self.hitTest(index);
	}
	
};
</pre>
子弹碰撞监测
<pre>
  /**
   * 子弹碰撞检测
   * */
  Bullet.prototype.hitTest = function (index){
  	var self = this;
  	var disx,disy,sw,ew,obj,i;
  	if(self.belong == player.belong){
  		//自机子弹
  		for(i=0;i < enemys.length;i++){
  			obj = enemys[i];
  			sw = self.bitmap.getWidth()/2;
  			ew = obj.bitmap.getWidth()/2;
  			disx = self.x+sw - (obj.x + ew);
  			disy = self.y+self.bitmap.getHeight()/2 - (obj.y + obj.bitmap.getHeight()/2);
  			//距离检测
  			if(disx*disx + disy*disy < ew*ew){
  				obj.hp--;
  				if(obj.hp == 0){
  					point += 1;
  					pointText.text = point;
  					//从屏幕移除
  					backLayer.removeChild(obj);
  					//从敌机数组移除
  					enemys.splice(i,1);
  					if(obj.name == "boss"){
  						gameclear = true;
  					}
  				}
  				//从屏幕移除
  				backLayer.removeChild(self);
  				//从子弹数组移除
  				barrage.splice(index,1);
  			}
  		}
  	}else{
  		//敌机子弹
  		obj = player;
  		sw = self.bitmap.getWidth()/2;
  		ew = obj.bitmap.getWidth()/2;
  		disx = self.x+sw - (obj.x + ew);
  		disy = self.y+self.bitmap.getHeight()/2 - (obj.y + obj.bitmap.getHeight()/2);
  		//距离检测
  		if(disx*disx + disy*disy < ew*ew - 10){			
  	    obj.hp--;	
  	    if(obj.hp <=0){
  			  obj.visible = false;	
  			  gameover = true;
  		  }
  			//从屏幕移除
  			backLayer.removeChild(self);
  			//从子弹数组移除
  			barrage.splice(index,1);
  		}
  	}
  };
</pre>
#### Plain.js
<pre>
/**
 * 飞机类
 * */
function Plain(name,belong,x,y,bullets){
	base(this,LSprite,[]);
	var self = this;
	//飞机名称
	self.name = name;
	//飞机位置
	self.x = x;
	self.y = y;
	//飞机所属
	self.belong = belong;
	//子弹数组
	self.bullets = bullets;
	//初始子弹
	self.bullet = self.bullets[Math.floor(Math.random()*self.bullets.length)];
	self.shootspeed = Global.bulletList[self.bullet].shootspeed;
	//枪口旋转角度
	self.sspeed = 0;
	//射击频率控制
	self.shootctrl = 0;
	//获取飞机属性
	self.list = Global.getPlainStatus(self);
	//飞机图片
	self.bitmap = self.list[0];
	//显示
	self.addChild(self.bitmap);
	//枪口位置
	self.shootx = self.list[1];
	self.shooty = self.list[2];
	//移动速度
	self.speed = self.list[3];
	//飞机hp
	self.hp = self.list[4];
	//移动方向
	self.move = [0,0];
	//发射子弹数
	self.shootcount = 0;
	//是否发射子弹
	self.canshoot = true;
	if(name=="player")self.canshoot = false;
}

/**
 * 循环
 * */
Plain.prototype.onframe = function (){
	var self = this;
	//移动
	self.x += self.move[0]*self.speed;
	self.y += self.move[1]*self.speed;
	
	switch (self.name){
		case "player":
			//自机移动位置限制
			if(self.x < 0)self.x = 0;
			else if(self.x + self.bitmap.getWidth() > LGlobal.width)self.x = LGlobal.width-self.bitmap.getWidth();
			if(self.y < 0)self.y = 0;
			else if(self.y + self.bitmap.getHeight() > LGlobal.height)self.y = LGlobal.height-self.bitmap.getHeight();
			break;
		case "boss":
			//敌机BOSS移动
			if(self.y < 0){
				self.y = 0;
				self.move[1] = 1;
			}else if(self.y + self.bitmap.getHeight() > LGlobal.height){
				self.y = LGlobal.height-self.bitmap.getHeight() - 200;
				self.move[1] = -1;
			}
			//碰撞检测
			self.hitTest();
			break;
		case "enemy":
		default:
			//碰撞检测
			self.hitTest();
	}
	//射击
	if(self.canshoot)self.shoot();
};

/**
 * 碰撞检测
 * */
Plain.prototype.hitTest = function (){
	var self = this;
	var disx,disy,sw,ew;
	sw = (self.bitmap.getWidth() + self.bitmap.getHeight())/4;
	ew = (player.bitmap.getWidth() + player.bitmap.getHeight())/4;
	disx = self.x+sw - (player.x + ew);
	disy = self.y+self.bitmap.getHeight()/2 - (player.y + player.bitmap.getHeight()/2);
	if(disx*disx + disy*disy < (sw+ew)*(sw+ew)){
	    self.hp--;
	    player.hp--;
	    if(player.hp <= 0){
			player.visible = false;	
			gameover = true;		}
	}
};
/**
 * 射击
 * */
Plain.prototype.shoot = function (){
	var self = this;
	if(self.shootctrl++ < self.shootspeed)return;
	self.shootctrl = 0;
	if(self.name == "boss"){
		if(self.shootcount++ % 40 > 5)return;
	}else{
		if(self.shootcount++ % 10 > 5)return;
	}
	Global.setBullet(self);
	if(self.name == "boss"){
		if(self.shootcount % 40 < 5)return;
	}else{
		if(self.shootcount % 10 < 5)return;
	}
	if(self.bullets.length <= 1)return;
	self.bullet = self.bullets[Math.floor(Math.random()*self.bullets.length)];
	self.shootspeed = Global.bulletList[self.bullet].shootspeed;
};
</pre>

####Global.js
<pre>
/**
 * 共同类
 * */
var Global = function (){};
/**
 * 获取飞机属性
 * @param 飞机
 * */
Global.getPlainStatus = function(plainObject){
	var list,bitmapdata,bitmap;
	bitmapdata = new LBitmapData(imglist[plainObject.name]);
	bitmap = new LBitmap(bitmapdata);
	switch (plainObject.name){
		case "player":
			list = [bitmap,35,0,10,5];
			break;
		case "boss":
			list = [bitmap,138,240,2,50];
			break;
		case "enemy":
		default:
			list = [bitmap,25,45,2,1];
	}
	return list;
};

/**
 * 子弹类型数组
 * 【开始角度，增加角度，子弹速度，角度加速度，子弹总数，发动频率，枪口旋转】
 * */
Global.bulletList = new Array(
		{startAngle:0,angle:20,speed:5,aspeed:0,count:1,shootspeed:10,sspeed:0},//1发
		{startAngle:-20,angle:20,speed:5,aspeed:0,count:3,shootspeed:10,sspeed:0},//3发
		{startAngle:0,angle:20,speed:5,aspeed:0,count:1,shootspeed:1,sspeed:20},//1发旋转
		{startAngle:0,angle:20,speed:5,aspeed:0,count:18,shootspeed:3,sspeed:0},//环发
		{startAngle:0,angle:20,speed:5,aspeed:1,count:18,shootspeed:3,sspeed:0},//环发旋转
		{startAngle:180,angle:20,speed:5,aspeed:0,count:1,shootspeed:5,sspeed:0},//1发 up
		{startAngle:160,angle:20,speed:5,aspeed:0,count:3,shootspeed:5,sspeed:0}//3发 up
);
/**
 * 发射子弹
 * @param 飞机
 * */
Global.setBullet = function(plainObject){
	var i,j,obj,xspeed,yspeed,kaku;
	//获取子弹属性
	var bullet = Global.bulletList[plainObject.bullet];
	//设定枪口旋转
	plainObject.sspeed += bullet.sspeed;
	//开始发射
	for(i=0;i < bullet.count;i++){
		//发射角度
		kaku = i*bullet.angle + bullet.startAngle + plainObject.sspeed;
		//子弹xy轴速度
		xspeed = bullet.speed*Math.sin(kaku * Math.PI / 180);
		yspeed = barrageSpeed[0]*Math.cos(kaku * Math.PI / 180);
		//子弹实例化
		obj = new Bullet(plainObject.belong,plainObject.x+plainObject.shootx,plainObject.y+plainObject.shooty,kaku,xspeed,yspeed,bullet.aspeed,bullet.speed);
		//显示
		backLayer.addChild(obj);
		barrage.push(obj);
	}
};
</pre>
####Main.js
<pre>
/**
 * Main
 * */
//设定游戏速度，屏幕大小，回调函数

if(LGlobal.canTouch){
	LGlobal.stageScale = LStageScaleMode.EXACT_FIT;
	LSystem.screen(LStage.FULL_SCREEN);
}
init(30,"mylegend",320,508,main);

/**层变量*/
//显示进度条所用层
var loadingLayer;
//游戏最底层
var backLayer;
//控制层
var ctrlLayer;

/**int变量*/
//读取图片位置
var loadIndex = 0;
//贞数
var frames = 0;
//BOOS START
var boosstart = false;
//GAME OVER
var gameover = false;
//GAME CLEAR 
var gameclear = false;
//得分
var point = 0;
/**对象变量*/
//玩家
var player;
//得分
var pointText;

/**数组变量*/
//图片path数组
var imgData = new Array();
//读取完的图片数组
var imglist = {};
//子弹数组
var barrage = new Array();
//子弹速度数组
var barrageSpeed = [5,10];
//储存所有敌人飞机的数组
var enemys = new Array();

function main(){
	LMouseEventContainer.set(LMouseEvent.MOUSE_DOWN,true);
	LMouseEventContainer.set(LMouseEvent.MOUSE_UP,true);
	LMouseEventContainer.set(LMouseEvent.MOUSE_MOVE,true);
	//准备读取图片
	imgData.push({type:"js",path:"./js/Global.js"});
	imgData.push({type:"js",path:"./js/Bullet.js"});
	imgData.push({type:"js",path:"./js/Plain.js"});
	imgData.push({name:"back",path:"./images/back.jpg"});
	imgData.push({name:"enemy",path:"./images/e.png"});
	imgData.push({name:"player",path:"./images/player.png"});
	imgData.push({name:"boss",path:"./images/boss.png"});
	imgData.push({name:"ctrl",path:"./images/ctrl.png"});
	imgData.push({name:"item1",path:"./images/1.png"});
	
	loadingLayer = new LoadingSample1();
	addChild(loadingLayer);	
	LLoadManage.load(
		imgData,
		function(progress){
			loadingLayer.setProgress(progress);
		},
		function(result){
			imglist = result;
			removeChild(loadingLayer);
			loadingLayer = null;
			gameInit();
		}
	);
}
function gameInit(event){
	//游戏底层实例化
	backLayer = new LSprite();
	addChild(backLayer);
	ctrlLayer = new LSprite();
	addChild(ctrlLayer);
	//添加游戏背景
	bitmapdata = new LBitmapData(imglist["back"]);
	bitmap = new LBitmap(bitmapdata);
	backLayer.addChild(bitmap);
}
var monseIsDown = false;
function onup(event){	
	monseIsDown = false;
	player.move = [0,0];
	player.canshoot = false;
	player.shootctrl = player.shootspeed;
}
function ondown(event){
	monseIsDown = true;	
	player.shootcount = 0;	
	player.shootctrl = player.shootspeed+1;	
	player.canshoot = true;
}
/**
 * 循环
 * */
function onframe(){
	if(gameover){//游戏结束
		backLayer.die();
		var txtOver = new LTextField();
		txtOver.text = "GAME OVER";
		txtOver.color = "#ffffff";
		txtOver.x = 100;
		txtOver.y = 200;
		txtOver.size = 40;
		backLayer.addChild(txtOver);
	}else if(gameclear){//游戏通关
		backLayer.die();
		var txtOver = new LTextField();
		txtOver.text = "GAME CLEAR";
		txtOver.color = "#ffffff";
		txtOver.x = 100;
		txtOver.y = 200;
		txtOver.size = 40;
		backLayer.addChild(txtOver);
	}
	if(monseIsDown){		
		if(player.x + 30 - player.speed > LGlobal.offsetX){//left
			player.move[0] = -1;		
		}else if(player.x + 30 + player.speed < LGlobal.offsetX){//right
			player.move[0] = 1;		
		}else{			
			player.move[0] = 0;		
		}		
		if(player.y + 30 - player.speed > LGlobal.offsetY){//up
			player.move[1] = -1;		
		}else if(player.y + 30 + player.speed < LGlobal.offsetY){//down
			player.move[1] = 1;		
		}else{			
			player.move[1] = 0;		
		}	
	}	
	var i;
	//循环子弹
	for(i=0;i<barrage.length;i++){
		barrage[i].onframe(i);
	}
	//循环敌机
	for(i=0;i<enemys.length;i++){
		enemys[i].onframe();
	}
	//自机循环
	player.onframe();
	//添加敌机
	addEnemy();
}
/**
 * 添加敌机
 * */
function addEnemy(){
	if(boosstart)return;
	var plain;
	if(point >= 10){//得到10分的话，添加boss
		//加入一个boss敌人
		plain = new Plain("boss",1,100,0,[2,3,4]);
		plain.move = [0,1];
		enemys.push(plain);
		backLayer.addChild(plain);
		boosstart = true;
		return;
	}
	if(frames++ % 100 > 0)return;//限制敌人出现频率
	var rand = Math.random();
	var b;
	if(rand < 0.5){
		if(rand < 0.3){
			b=0;
		}else{
			b=1;
		}
		//左边加入一个敌人
		plain = new Plain("enemy",1,0,100*Math.random(),[b]);
		plain.move = [0.6,1];
	}else{
		if(rand < 0.8){
			b=0;
		}else{
			b=1;
		}
		//右边加入一个敌人
		plain = new Plain("enemy",1,520,100*Math.random(),[b]);
		plain.move = [-0.6,1];
	}
	enemys.push(plain);
	backLayer.addChild(plain);
}
window.addEventListener('tizenhwkey', function onTizenHwKey(e) {
    if (e.keyName === 'back') {
        try {
            tizen.application.getCurrentApplication().exit();
        } catch (err) {
            console.log('Error: ', err);
        }
    }
},false);
</pre>
