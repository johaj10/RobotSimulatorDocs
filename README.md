# Microbit

## Snake
```

points = [[2,2],[2,2],[2,2],[2,2]]
points_length = 4

def limit(number):
    if number<0:
        number=0
    if number>4:
        number=4
    return number


def move(points, right, down):
    new_point = [points[points_length-1][0],points[points_length-1][1]]
    new_point[0] = limit(new_point[0]+right)
    new_point[1] = limit(new_point[1]+down)

    points=points[1:]
    points.append(new_point)
    return points

def clear_grid():
    for y in range(5):
        for x in range(5):
            led.unplot(x, y)

def display(points):
    for point in points:
        led.plot(point[0], point[1])

while True:
    pitch = input.rotation(Rotation.PITCH)
    if pitch<0:
        right=-1
    elif pitch>0:
        right=1
    else:
        right=0

    roll = input.rotation(Rotation.ROLL)
    if roll<0:
        down=-1
    elif roll>0:
        down=1
    else:
        down=0

    points = move(points,down,right)
    clear_grid()
    display(points)
    basic.pause(100)

```


# Robot Simulator
How to use the simulator

## Initializing Your Robot
```
from controller import Robot
from controller import Compass
import math
robot = Robot()

timeStep = int(robot.getBasicTimeStep())

leftMotor = robot.getMotor("motor.left") #left wheel if in square path
rightMotor = robot.getMotor("motor.right") #right wheel if in square path

leftEncoder = leftMotor.getPositionSensor()
rightEncoder = rightMotor.getPositionSensor()

leftEncoder.enable(timeStep)
rightEncoder.enable(timeStep)

```
## Initialize Compass (not available on square challenge)
```
compass = robot.getCompass("compass")
compass.enable(timeStep)
def get_bearing_in_degrees():
  north = compass.getValues();
  rad = math.atan2(north[0], north[2])
  bearing = (rad - 1.5708) / math.pi * 180.0
  if (bearing < 0.0):
    bearing = bearing + 360.0
  return bearing
```
## Initialize Distance Sensors
```
# Get frontal distance sensors.
outerLeftSensor = robot.getDistanceSensor("prox.horizontal.0")
centralLeftSensor = robot.getDistanceSensor("prox.horizontal.1")
centralSensor = robot.getDistanceSensor("prox.horizontal.2")
centralRightSensor = robot.getDistanceSensor("prox.horizontal.3")
outerRightSensor = robot.getDistanceSensor("prox.horizontal.4")

# Enable distance sensors.
outerLeftSensor.enable(timeStep)
centralLeftSensor.enable(timeStep)
centralSensor.enable(timeStep)
centralRightSensor.enable(timeStep)
outerRightSensor.enable(timeStep)
```

## Moving the Robot
### Move the wheels to a position
```
target = 5  #target is 5 wheel rotations
leftMotor.setPosition(target*math.pi*2) #moves left motor 5 rotations
rightMotor.setPosition(target*math.pi*2)#moves right motor 5 rotations
```

### Move the wheels at a speed
```
target = 0.5  #target is to move at half a wheel rotation every second
leftMotor.setPosition(float('inf')) #allows us to control velocity instead of position
rightMotor.setPosition(float('inf')) #allows us to control velocity instead of position

time=0
while robot.step(timeStep) != -1:
    rightMotor.setVelocity(target*math.pi*2)
    leftMotor.setVelocity(target*math.pi*2)
    print(leftEncoder.getValue(),rightEncoder.getValue()) #print encoder readouts
    time+=1
```

### Turning to an angle

```
speed = 0.3  #target is to move at half a wheel rotation every second
target_angle = 180
leftMotor.setPosition(float('inf')) #allows us to control velocity instead of position
rightMotor.setPosition(float('inf')) #allows us to control velocity instead of position

while robot.step(timeStep) != -1:
    angle = get_bearing_in_degrees()
    if angle>target_angle:
        rightMotor.setVelocity(speed*math.pi*2)
        leftMotor.setVelocity(-speed*math.pi*2)
    if angle<target_angle:
        rightMotor.setVelocity(-speed*math.pi*2)
        leftMotor.setVelocity(speed*math.pi*2)
    if abs(target_angle-angle)<.1:
        print(abs(target_angle-angle))
        rightMotor.setVelocity(0)
        leftMotor.setVelocity(0)
        break
    
```

### Abstraction


We don't want to write that code every time we want to turn to an angle. Instead, we can write a reusable function:
```
def turn(target_angle, speed, tolerance):
  leftMotor.setPosition(float('inf')) #allows us to control velocity instead of position
  rightMotor.setPosition(float('inf')) #allows us to control velocity instead of position
  while robot.step(timeStep) != -1:
      angle = get_bearing_in_degrees()
      if angle>target_angle:
          rightMotor.setVelocity(speed*math.pi*2)
          leftMotor.setVelocity(-speed*math.pi*2)
      if angle<target_angle:
          rightMotor.setVelocity(-speed*math.pi*2)
          leftMotor.setVelocity(speed*math.pi*2)
      if abs(target_angle-angle)<tolerance:
          print(abs(target_angle-angle))
          rightMotor.setVelocity(0)
          leftMotor.setVelocity(0)
          break
    
```

```
def move(target_distance, speed, tolerance):
    leftMotor.setPosition(float('inf')) #allows us to control velocity instead of position
    rightMotor.setPosition(float('inf')) #allows us to control velocity instead of position
    while robot.step(timeStep) != -1:
        distance = (leftEncoder.getValue()+rightEncoder.getValue())/2
        if distance<target_distance:
            rightMotor.setVelocity(speed*math.pi*2)
            leftMotor.setVelocity(speed*math.pi*2)
        if distance>target_distance:
            rightMotor.setVelocity(-speed*math.pi*2)
            leftMotor.setVelocity(-speed*math.pi*2)
        if abs(distance-target_distance)<tolerance:
            rightMotor.setVelocity(0)
            leftMotor.setVelocity(0)
            break
            
```

And then when we want to turn, we can call it by typing:

```
turn(180, 0.3, .1)
```

## Getting Distance Sensor Readings (starts at zero, increases when object is close)
```
outerLeftSensorValue = outerLeftSensor.getValue() / 360
centralLeftSensorValue = centralLeftSensor.getValue() / 360
centralSensorValue = centralSensor.getValue() / 360
centralRightSensorValue = centralRightSensor.getValue() / 360
outerRightSensorValue = outerRightSensor.getValue() / 360
```

## Simplified Obstacle Avoidance (Part 0).
```
sensor_conversion_constant = 3600
for segment in range(2000):
    move(segment*.8,.95,.1) #distance speed tolerance
    outerLeftSensorValue = outerLeftSensor.getValue() / sensor_conversion_constant
    centralLeftSensorValue = centralLeftSensor.getValue() / sensor_conversion_constant
    centralSensorValue = centralSensor.getValue() / sensor_conversion_constant
    centralRightSensorValue = centralRightSensor.getValue() / sensor_conversion_constant
    outerRightSensorValue = outerRightSensor.getValue() / sensor_conversion_constant
    if centralSensorValue > 0.1 or outerLeftSensorValue > 0.1 or centralLeftSensorValue > 0.1 or centralRightSensorValue > 0.1 or outerRightSensorValue > 0.1:
        break
 ```
 
## Simplified Obstacle Avoidance (Part 1)
```
sensor_conversion_constant = 3600
for segment in range(2000):
    move(segment*.8,.95,.1) #distance speed tolerance
    outerLeftSensorValue = outerLeftSensor.getValue() / sensor_conversion_constant
    centralLeftSensorValue = centralLeftSensor.getValue() / sensor_conversion_constant
    centralSensorValue = centralSensor.getValue() / sensor_conversion_constant
    centralRightSensorValue = centralRightSensor.getValue() / sensor_conversion_constant
    outerRightSensorValue = outerRightSensor.getValue() / sensor_conversion_constant
    if centralSensorValue > 0.1 or outerLeftSensorValue > 0.1 or centralLeftSensorValue > 0.1:
        turn(130, 0.95,5)
    if centralRightSensorValue > 0.1 or outerRightSensorValue > 0.1:
        turn(40, 0.95,5)
 ```
 
 
## Full Obstacle Avoidance Example. Not the fastest, can you make a faster one?
 
 ```
from controller import Robot
from controller import Compass
import math
robot = Robot()
compass = robot.getCompass("compass")

timeStep = int(robot.getBasicTimeStep())

leftMotor = robot.getMotor("motor.left")
rightMotor = robot.getMotor("motor.right")

leftEncoder = leftMotor.getPositionSensor()
rightEncoder = rightMotor.getPositionSensor()

# Get frontal distance sensors.
outerLeftSensor = robot.getDistanceSensor("prox.horizontal.0")
centralLeftSensor = robot.getDistanceSensor("prox.horizontal.1")
centralSensor = robot.getDistanceSensor("prox.horizontal.2")
centralRightSensor = robot.getDistanceSensor("prox.horizontal.3")
outerRightSensor = robot.getDistanceSensor("prox.horizontal.4")

# Enable distance sensors.
outerLeftSensor.enable(timeStep)
centralLeftSensor.enable(timeStep)
centralSensor.enable(timeStep)
centralRightSensor.enable(timeStep)
outerRightSensor.enable(timeStep)

leftEncoder.enable(timeStep)
rightEncoder.enable(timeStep)
compass.enable(timeStep)

def get_bearing_in_degrees():
  north = compass.getValues();
  rad = math.atan2(north[0], north[2])
  bearing = (rad - 1.5708) / math.pi * 180.0
  if (bearing < 0.0):
    bearing = bearing + 360.0
  return bearing


def move(target_distance, speed, tolerance):
    leftMotor.setPosition(float('inf')) #allows us to control velocity instead of position
    rightMotor.setPosition(float('inf')) #allows us to control velocity instead of position
    while robot.step(timeStep) != -1:
        distance = (leftEncoder.getValue()+rightEncoder.getValue())/2
        if distance<target_distance:
            rightMotor.setVelocity(speed*math.pi*2)
            leftMotor.setVelocity(speed*math.pi*2)
        if distance>target_distance:
            rightMotor.setVelocity(-speed*math.pi*2)
            leftMotor.setVelocity(-speed*math.pi*2)
        if abs(distance-target_distance)<tolerance:
            rightMotor.setVelocity(0)
            leftMotor.setVelocity(0)
            break
    
def turn(target_angle, speed, tolerance):
  leftMotor.setPosition(float('inf')) #allows us to control velocity instead of position
  rightMotor.setPosition(float('inf')) #allows us to control velocity instead of position
  while robot.step(timeStep) != -1:
      angle = get_bearing_in_degrees()
      if angle>target_angle:
          rightMotor.setVelocity(speed*math.pi*2)
          leftMotor.setVelocity(-speed*math.pi*2)
      if angle<target_angle:
          rightMotor.setVelocity(-speed*math.pi*2)
          leftMotor.setVelocity(speed*math.pi*2)
      if abs(target_angle-angle)<tolerance:
          rightMotor.setVelocity(0)
          leftMotor.setVelocity(0)
          break
      
      
total_left_sensed = 0
total_right_sensed = 0
rebound_constant = 0.9 #making it higher will make the robot steer away from the obtacle for a longer time
sensor_conversion_constant = 3600
side_responsiveness_constant = .95

def clamp(n, minn, maxn):
    return max(min(maxn, n), minn)

for segment in range(2000):
    move(segment*.75,.95,.1)
    
    current_angle = (90-get_bearing_in_degrees())
    outerLeftSensorValue = outerLeftSensor.getValue() / sensor_conversion_constant
    centralLeftSensorValue = centralLeftSensor.getValue() / sensor_conversion_constant
    centralSensorValue = centralSensor.getValue() / sensor_conversion_constant
    centralRightSensorValue = centralRightSensor.getValue() / sensor_conversion_constant
    outerRightSensorValue = outerRightSensor.getValue() / sensor_conversion_constant
    
    total_left_sensed = (outerLeftSensorValue*side_responsiveness_constant + centralLeftSensorValue) + rebound_constant * total_left_sensed
    total_right_sensed = (centralRightSensorValue + outerRightSensorValue*side_responsiveness_constant) + rebound_constant * total_right_sensed
    
    if total_left_sensed>total_right_sensed:
        total_left_sensed += centralSensorValue
    else:
        total_right_sensed += centralSensorValue
    
    target_angle = 90+90*clamp(total_left_sensed - total_right_sensed, -1.4, 1.4)    

    turn(target_angle if target_angle>=0 else target_angle+360, .95, 5)


```

