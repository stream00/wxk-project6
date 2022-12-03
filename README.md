# wxk-project6
```java

import controlP5.*;
import peasy.*;
PeasyCam cam;
ControlP5 bar;

FlowField flowfield;
ArrayList<Vehicle> vehicles;
int fn=10,in=150;
float bs=1,bh=1,jl=35,sd=1,rw=1,max=1;
void setup() {
  size(800, 800);
  fill(0,20);
  background(0);
  
  flowfield = new FlowField(fn);
  vehicles = new ArrayList<Vehicle>();
    
    
    cam = new PeasyCam(this, 800);
  UI();
  
  
  for (int i = 0; i < in; i++) {
    vehicles.add(new Vehicle(new PVector(width/2, height/2), random(0.5, 5), random(0.1, 0.5), random(255), random(4)));
  }
}

void draw() {
 UIShow();
 
  flowfield.update();

  for (Vehicle v : vehicles) {
    v.follow(flowfield);
    v.separate(vehicles);
    v.run();
  }
}


class FlowField {

  PVector[][] field;
  int cols, rows; 
  int resolution; 
  float zoff = 0.0; 

  FlowField(int r) {
    resolution = r;
    cols = 720/resolution;
    rows = 720/resolution;
    field = new PVector[cols][rows];
    update();
  }

  void update() {
    float xoff = 0;
    for (int i = 0; i < cols; i++) {
      float yoff = 0;
      for (int j = 0; j < rows; j++) {
        float x = bh*(i*resolution-width/2);
        float y = bh*(j*resolution-height/2);
        float r = sqrt((x*x)+ (y*y))*bs;
        PVector v = new PVector(sin(r-zoff), cos(r-zoff));
        v.normalize();
        field[i][j] = v; 
        yoff += 0.1*sd;
      }
      xoff += 0.1*sd;
    }
    zoff += 0.1*sd;
  }

  PVector lookup(PVector lookup) {
    int column = int(constrain(lookup.x/resolution, 0, cols-1));
    int row = int(constrain(lookup.y/resolution, 0, rows-1));
    return field[column][row].get();
  }
}



class Vehicle {

  PVector location, velocity, acceleration;
  float maxforce, maxspeed;
  float r, c;

  Vehicle(PVector l, float ms, float mf, float c_, float r_) {
    location = l.get();
    acceleration = new PVector(0, 0);
    velocity = new PVector(0, 0);    
    maxforce = mf;
    maxspeed = ms;
    r = r_;    
    c = c_;
  }

  void run() {
    update();
    display();
  }

  void follow(FlowField flow) {
    PVector desired = flow.lookup(location);
    desired.mult(maxspeed*max);
    PVector steer = PVector.sub(desired, velocity);
    steer.limit(maxforce);  
    applyForce(steer);
  }

  void applyForce(PVector force) {
    acceleration.add(force);
  }

  void update() {
    velocity.add(acceleration);
    velocity.limit(maxspeed);
    location.add(velocity);
    acceleration.mult(0);
  }

  void separate (ArrayList<Vehicle> vehicles) {
    float desiredseparation = r*2;
    PVector sum = new PVector();
    int count = 0;
    for (Vehicle other : vehicles) {
      float d = PVector.dist(location, other.location);  
      if (d < jl) { 
        stroke(255, jl-d);
        line(location.x, location.y, other.location.x, other.location.y);
      }
      if ((d > 0) && (d < desiredseparation)) {
        PVector diff = PVector.sub(location, other.location);
        diff.normalize();
        diff.div(d);        
        sum.add(diff);
        count++;            
      }
    }
    if (count > 0) {
      sum.div(count);
      sum.normalize();
      sum.mult(maxspeed);
      PVector steer = PVector.sub(sum, velocity);
      steer.limit(maxforce);
      applyForce(steer);
    }
  }

  void display() {
    fill(0);
    noStroke();
    pushMatrix();
    translate(location.x, location.y);
    ellipse(0, 0, r*rw, r*rw);
    popMatrix();
  }
}

int canvasLeftCornerX = 30;
int canvasLeftCornerY = 60;



void s1() {
  size(800, 800);
 cam = new PeasyCam(this,800);
  UI();
}

void d1() {
  background(51);

  UIShow();
}



void UI() {
  bar = new ControlP5(this, createFont("微软雅黑", 14));
  int barSize = 200;
  int barHeight = 20;
  int barInterval = barHeight + 10;
  bar.addSlider("max", 1, 20,1, canvasLeftCornerX, canvasLeftCornerY, barSize, barHeight).setLabel("速度");
  bar.addSlider("sd", 1, 10, 1, canvasLeftCornerX, canvasLeftCornerY+barInterval, barSize, barHeight).setLabel("凝固度");
  bar.addSlider("jl", 1, 150, 35, canvasLeftCornerX, canvasLeftCornerY+barInterval*3, barSize, barHeight).setLabel("粒子距离");
  bar.addSlider("rw", 1, 10, 1, canvasLeftCornerX, canvasLeftCornerY+barInterval*4, barSize, barHeight).setLabel("半径");
  bar.addSlider("fn", 0, 50, 10, canvasLeftCornerX, canvasLeftCornerY+barInterval*5, barSize, barHeight).setLabel("场域变化");
  bar.addSlider("bs", 0, 10, 1,canvasLeftCornerX, canvasLeftCornerY+barInterval*6, barSize, barHeight).setLabel("混乱度");
    bar.addSlider("bh", 0, 200, 1,canvasLeftCornerX, canvasLeftCornerY+barInterval*7, barSize, barHeight).setLabel("位置变化");
  bar.setAutoDraw(false);
}



void UIShow() {
  cam.beginHUD();
 
  bar.draw();
  cam.endHUD();

}

```
