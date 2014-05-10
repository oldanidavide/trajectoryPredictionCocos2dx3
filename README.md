### Predict Trajectory with Cocos2dx 3 and Box2d

How to predict a trajectory with cocos2d-x and Box2d

Create new cocos2d-x project with:

	cocos new MyGame -p com.MyCompany.MyGame -l cpp -d ~/MyCompany

for more details check that page:

	http://cocos2d-x.org/wiki/How_to_Start_A_New_Cocos2D-X_Game
	
<b>NOTE: When you create a project with cocos-console, it make in your project dir, the folder `cocos2d`, i don't have upload it because the size in MB it's to big</b>

### Resources<br/>

Download the `ball.png` and `dot.png` and copy it in the Resources folder

<img src="https://raw.githubusercontent.com/oldanidavide/trajectoryPredictionCocos2dx3/master/Example/MyCompany/MyGame/Resources/ball.png" />
<br/><br/>
<img src="https://raw.githubusercontent.com/oldanidavide/trajectoryPredictionCocos2dx3/master/Example/MyCompany/MyGame/Resources/dot.png" />

### Linux Version<br/>
open your project folder and in the file `CMakeLists.txt` add on line 161 ( on target link libraries ) the box2d

	box2d

the result looks like that:

	target_link_libraries(${APP_NAME}
		  ui
		  network
		  storage
		  spine
		  cocostudio
		  cocosbuilder
		  extensions
		  audio
		  cocos2d
		  box2d
	  )

### Windows Version

Open your `proj.win32` `.SLN` file and add `Box2D` dependency

Right click on Solution not project name then click on `add`, `add exist project`, search on external resources the `BOX2D` project and add it

Now right click on the solution, click property, select `Project Dependencies` and flag the `libBox2D` to compile.

Now right click on your project (inside the solution), and select `reference`, on the bottom, click `ADD NEW REFERENCE` then flag `libBox2D`, and click `OK`.

Click `OK` to close the windows


### IT'S TIME TO CODE:

In `HelloWorldScene.h` file, on line 4 under `#include "cocos2d.h"` add:

    USING_NS_CC; 

and under public add:

    bool onTouchBegan(Touch* touch, Event* event);
    void onTouchMoved(Touch* touch, Event* event);
    void onTouchEnded(Touch* touch, Event* event);

On the `INIT` main function add mouse event listener (example: `HelloWorldScene.cpp` )

    //SET MOUSE LISTENER
    auto listener = EventListenerTouchOneByOne::create();
    listener->setSwallowTouches(true);
    
    listener->onTouchBegan = CC_CALLBACK_2(HelloWorld::onTouchBegan, this);
    listener->onTouchMoved = CC_CALLBACK_2(HelloWorld::onTouchMoved, this);
    listener->onTouchEnded = CC_CALLBACK_2(HelloWorld::onTouchEnded, this);
    
    _eventDispatcher->addEventListenerWithSceneGraphPriority(listener, this);
    //END MOUSE LISTENER


In the main CPP file (`HelloWorldScene.cpp`) add the following function;	


	bool HelloWorld::onTouchBegan(Touch* touch, Event* event)
	{
	
	    return true;
	}
	
	void HelloWorld::onTouchMoved(Touch* touch, Event* event)
	{
	
	   
	}
	
	void HelloWorld::onTouchEnded(Touch* touch, Event* event)
	{
	
	} 

Inside `INIT` function add:

    //CREATE A BALL
    dragOffsetStartX = 0;    
    dragOffsetEndX = 0;    
    dragOffsetStartY = 0;    
    dragOffsetEndY = 0;    
    existBall= false;    
    ballX = 500;
    ballY = 200;    
    ball =Sprite::create("ball.png");
    ball->setPosition(CCPoint(ballX,ballY));
    this->addChild(ball); 

in the header (`HelloWorldScene.h`) file add:
	
    Sprite *ball;
    bool existBall;
    float ballX;
    float ballY;    
    int dragOffsetStartX;
    int dragOffsetEndX;
    int dragOffsetStartY;
    int dragOffsetEndY;    
    b2Body *ballBody;    
    b2CircleShape ballShape;
    b2BodyDef ballBodyDef;  
    void defineBall();


### ADD A PHYSICS	
	
Include in the header file the box2d library:
	
     #include <Box2D/Box2D.h>
	
Add the b2ContactListener changing:

	class HelloWorld : public cocos2d::Layer

to:

	class HelloWorld : public cocos2d::Layer, public b2ContactListener

then add :

    b2World *world;
    float deltaTime;

Add in the `INIT` function the following line of code:

    b2Vec2 gravity = b2Vec2(0.0f, -10.0f);
    world = new b2World(gravity);    

Where the -10.0f indicate the gravity on the y axis.

Now, we need to add a `SCALE_RATIO`. then define it at the top of the source with:
	 
    #define SCALE_RATIO 32.0
	
`SCALE_RATIO` indicate the value to convert pixel in meter, because BOX2D work with METERS.

In main source code (`HelloWorldScene.cpp`) add:

	void HelloWorld::defineBall(){
	    ballShape.m_radius = 45 / SCALE_RATIO;
	    b2FixtureDef ballFixture;
	    ballFixture.density=10;
	    ballFixture.friction=0.8;
	    ballFixture.restitution=0.6;
	    ballFixture.shape=&ballShape;
	
	    ballBodyDef.type= b2_dynamicBody;
	    ballBodyDef.userData=ball;
	
	    ballBodyDef.position.Set(ball->getPosition().x/SCALE_RATIO,ball->getPosition().y/SCALE_RATIO);
	
	    ballBody = world->CreateBody(&ballBodyDef);
	    ballBody->CreateFixture(&ballFixture);
	    ballBody->SetGravityScale(10);
	}

Call the function just after create a ball sprite, under the `this->addChild(ball);`

    HelloWorld::defineBall();

Add an event to update the physics

In the header file add:

    void update(float dt);

And in main cpp source add:
	
	//Simulate Physics
	void HelloWorld::update(float dt){
	   int positionIterations = 10;  
	   int velocityIterations = 10;
	   
	   deltaTime = dt;
	   world->Step(dt, velocityIterations, positionIterations);  
	  
	   for (b2Body *body = world->GetBodyList(); body != NULL; body = body->GetNext())   
	     if (body->GetUserData()) 
	     {  
	       CCSprite *sprite = (CCSprite *) body->GetUserData();  
	       sprite->setPosition(ccp(body->GetPosition().x * SCALE_RATIO,body->GetPosition().y * SCALE_RATIO));  
	       sprite->setRotation(-1 * CC_RADIANS_TO_DEGREES(body->GetAngle())); 
	
	
	
	     }  
	    world->ClearForces();
	    world->DrawDebugData();        
	}  	

Call the `scheduleUpdate` in the init function with:

	scheduleUpdate();

If you try to start your application, you see the ball fall down

Now add a wall, to make our ball bouncing on it..

in the header file add:
	
	void addWall(float w,float h,float px,float py);

in the main cpp file add:

	void HelloWorld::addWall(float w,float h,float px,float py) {

	    b2PolygonShape floorShape;

	    floorShape.SetAsBox(w/ SCALE_RATIO,h/ SCALE_RATIO);
	    b2FixtureDef floorFixture;
	    floorFixture.density=0;
	    floorFixture.friction=10;
	    floorFixture.restitution=0.5;
	    floorFixture.shape=&floorShape;
	    b2BodyDef floorBodyDef;

	    floorBodyDef.position.Set(px/ SCALE_RATIO,py/ SCALE_RATIO);
	    b2Body *floorBody = world->CreateBody(&floorBodyDef);

	    floorBody->CreateFixture(&floorFixture);

	}		

and in the `INIT` main function add:

    addWall(visibleSize.width ,10,(visibleSize.width / 2) ,0); //CEIL
    addWall(10 ,visibleSize.height ,0,(visibleSize.height / 2) ); //LEFT
    addWall(10 ,visibleSize.height ,visibleSize.width,(visibleSize.height / 2) ); //RIGHT


### ADD POINT FOR TRAJECTORY

in the header file add:

    Sprite *points[32];

and in the `INIT` main function add:

    for (int i = 1 ; i <= 31; i++){
	points[i] =CCSprite::create("dot.png");
	this->addChild(points[i]); 
    }

### ADD CONTROL

Remove the `HelloWorld::defineBall();` calling previously in `INIT` method

Now inside the method `onTouchBegan` add:

    dragOffsetStartX = touch->getLocation().x;
    dragOffsetStartY = touch->getLocation().y;

    
    CCPoint touchLocation = touch->getLocation();

    ballX = touchLocation.x;
    ballY = touchLocation.y;

    if (existBall){        
	world->DestroyBody(ballBody);
    }

    ball->setPosition(ccp(ballX ,ballY));

Inside the method `onTouchMoved` add:

    CCPoint touchLocation = touch->getLocation();

    dragOffsetEndX = touchLocation.x;
    dragOffsetEndY = touchLocation.y;

    float dragDistanceX = dragOffsetStartX - dragOffsetEndX;
    float dragDistanceY = dragOffsetStartY - dragOffsetEndY;

    HelloWorld::simulateTrajectory(b2Vec2((dragDistanceX )/SCALE_RATIO,(dragDistanceY )/SCALE_RATIO));

Now we need to create the function `simulateTrajectory` function

Add in the header file:

	void simulateTrajectory(b2Vec2 coord);

And in the main cpp source add:

	void HelloWorld::simulateTrajectory(b2Vec2 coord){

	    //define ball physicis
	    HelloWorld::defineBall();

	    ballBody->SetLinearVelocity(b2Vec2(coord.x,coord.y));
	    for (int i = 1; i <= 31; i++){
		world->Step(deltaTime,10,10);
		points[i]->setPosition(CCPoint(ballBody->GetPosition().x*SCALE_RATIO,ballBody->GetPosition().y*SCALE_RATIO));
		world->ClearForces();
			
	    }

	    world->DestroyBody(ballBody);
	}
	
If you try the code, now we have a ball, in the "center" of the screen, and if you try drag the mouse, make a trajectory starting from ball.
		
### THROW THE BALL
	
Now we need to add an `onTouchEnded`, go to the method and add:

    existBall = true;

    HelloWorld::defineBall();

    CCPoint touchLocation = touch->getLocation();

    dragOffsetEndX = touchLocation.x;
    dragOffsetEndY = touchLocation.y;

    float dragDistanceX = dragOffsetStartX - dragOffsetEndX;
    float dragDistanceY = dragOffsetStartY - dragOffsetEndY;

    ballBody->SetLinearVelocity(b2Vec2((dragDistanceX)/SCALE_RATIO,(dragDistanceY)/SCALE_RATIO));	


### ADD SIMPLE POWER
	
For add power at shooting just add a coefficient multiplier, add it on the header file:

	float powerMultiplier;

set the power when create the ball adding:

	powerMultiplier = 10;

then change the line:

    HelloWorld::simulateTrajectory(b2Vec2((dragDistanceX )/SCALE_RATIO,(dragDistanceY )/SCALE_RATIO));

To:

    HelloWorld::simulateTrajectory(b2Vec2((dragDistanceX * powerMultiplier)/SCALE_RATIO,(dragDistanceY * powerMultiplier)/SCALE_RATIO));

And the line:
	
    ballBody->SetLinearVelocity(b2Vec2((dragDistanceX)/SCALE_RATIO,(dragDistanceY)/SCALE_RATIO));

To:
	
    ballBody->SetLinearVelocity(b2Vec2((dragDistanceX * powerMultiplier)/SCALE_RATIO,(dragDistanceY * powerMultiplier)/SCALE_RATIO));
		

