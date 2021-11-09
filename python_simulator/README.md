Assignment 1 - Robot Simulator
===============================

Assignment 1 - Robot Simulator is a smart solution for a given assignment thought for the Research Track I Robotics Engineering study course in order to stimulate student's problem solving capabilities and programming skills.
The assignment is about a portable robot simulator developed by [Student Robotics](https://studentrobotics.org) which drives autonomously in a given arena following a predetermined behavior which is further explained.
This solution is developed by Francesco Ferrazzi 5262829.



Installing and running
----------------------

The simulator requires a Python 2.7 installation, the [pygame](http://pygame.org/) library, [PyPyBox2D](https://pypi.python.org/pypi/pypybox2d/2.1-r331), and [PyYAML](https://pypi.python.org/pypi/PyYAML/).

Pygame, unfortunately, can be tricky (though [not impossible](http://askubuntu.com/q/312767)) to install in virtual environments. If you are using `pip`, you might try `pip install hg+https://bitbucket.org/pygame/pygame`, or you could use your operating system's package manager. Windows users could use [Portable Python](http://portablepython.com/). PyPyBox2D and PyYAML are more forgiving, and should install just fine using `pip` or `easy_install`.

## Exercise
-----------------------------

To run one or more scripts in the simulator, use `run.py`, passing it the file names.

The exercise is structured in order to let students acquire experience in the use of python programming language, develop skills of problem solving and innovative and creative thinking.
The aim of the assignment is to write a script that allows a robot, which is equipped with motors, vision system and a grabber, to drive independently around a given arena achieving a predetermined behavior, which is:
- constantly drive the robot around the given circuit following a counter-clockwise direction;
- avoiding touching the golden boxes;
- when the robot is close to a silver box, it should grab it, and move it behind itself.

To run the program is sufficient to type the following command in the terminal:

```bash
$ python run.py assignment.py
```

Robot API
-------------------------------------

The API for controlling a simulated robot is designed to be as similar as possible to the [SR API][sr-api]. The API are used in order to achieve the wanted behavior previously explained.

### Motors ###

The simulated robot has two motors configured for skid steering, connected to a two-output [Motor Board](https://studentrobotics.org/docs/kit/motor_board). The left motor is connected to output `0` and the right motor to output `1`.

The Motor Board API is identical to [that of the SR API](https://studentrobotics.org/docs/programming/sr/motors/), except that motor boards cannot be addressed by serial number. So, to turn on the spot at one quarter of full power, one might write the following:

```python
R.motors[0].m0.power = 25
R.motors[0].m1.power = -25
```

### The Grabber ###

The robot is equipped with a grabber, capable of picking up a token which is in front of the robot and within 0.4 metres of the robot's centre. To pick up a token, call the `R.grab` method:

```python
success = R.grab()
```

The `R.grab` function returns `True` if a token was successfully picked up, or `False` otherwise. If the robot is already holding a token, it will throw an `AlreadyHoldingSomethingException`.

To drop the token, call the `R.release` method.

Cable-tie flails are not implemented.

### Vision ###

To help the robot find tokens and navigate, each token has markers stuck to it, as does each wall. The `R.see` method returns a list of all the markers the robot can see, as `Marker` objects. The robot can only see markers which it is facing towards.

Each `Marker` object has the following attributes:

* `info`: a `MarkerInfo` object describing the marker itself. Has the following attributes:
  * `code`: the numeric code of the marker.
  * `marker_type`: the type of object the marker is attached to (either `MARKER_TOKEN_GOLD`, `MARKER_TOKEN_SILVER` or `MARKER_ARENA`).
  * `offset`: offset of the numeric code of the marker from the lowest numbered marker of its type. For example, token number 3 has the code 43, but offset 3.
  * `size`: the size that the marker would be in the real game, for compatibility with the SR API.
* `centre`: the location of the marker in polar coordinates, as a `PolarCoord` object. Has the following attributes:
  * `length`: the distance from the centre of the robot to the object (in metres).
  * `rot_y`: rotation about the Y axis in degrees.
* `dist`: an alias for `centre.length`
* `res`: the value of the `res` parameter of `R.see`, for compatibility with the SR API.
* `rot_y`: an alias for `centre.rot_y`
* `timestamp`: the time at which the marker was seen (when `R.see` was called).

For example, the following code lists all of the markers the robot can see:

```python
markers = R.see()
print "I can see", len(markers), "markers:"

for m in markers:
    if m.info.marker_type in (MARKER_TOKEN_GOLD, MARKER_TOKEN_SILVER):
        print " - Token {0} is {1} metres away".format( m.info.offset, m.dist )
    elif m.info.marker_type == MARKER_ARENA:
        print " - Arena marker {0} is {1} metres away".format( m.info.offset, m.dist )
```

Robot Functions
---------------------------------

In order to achieve the wanted behavior, a python script was developed.
For improving the clarity, the structure and the understandability, the code has been divided into functions, leaving in the `main` function only the clauses and loops that are necessary to let the robot obtain the objective for which it was designed.
All the function which are explained below are defined before the `main` function is called. This allows to use all of them inside the `main`.

### Drive ###

Since the robot is equipped with motors and wheels, this function uses the API `Motors` for setting a linear velocity.
The function gets as arguments `speed` and `seconds`.
The motors of both wheels are setted at the same speed which is kept for a given time, expressed in seconds. Once the time is over, the motors reset to `0`.

```
**function drive**  (speed, seconds)

right.motor set to speed
left.motor set to speed
sleep for seconds
right.motor set to 0
left.motor set to 0
```

### Drive Circle Left and Drive Circle Right ###

Once a golden token is avoided, the robot keeps driving around the arena. The `drive_circle_left` and `drive_circle_right` functions are very similar and are thought to let the robot drive further away from a golden token once it has been avoided.
The function gets as arguments, equally to `drive` function, `speed` and `seconds`.
The difference is that, one of the two motors is setted to have a higher speed that the other one. For example, in the case of `drive_circle_left`, the left motor speed is multiplied by a `1.2` factor, allowing the left wheel to go faster and make the robot drive straight and turn away from the golden token at the same time.
In this way, the possibility of collide in the same row of the golden token just avoided decreases.

```
**function drive_circle_left**  (speed, seconds)

right.motor set to speed
left.motor set to 1.2*speed
sleep for seconds
right.motor set to 0
left.motor set to 0
```

Trivial for the `drive_circle_right` function.

### Turn ###

This function is similar to the previous ones explained.
It gets as arguments `speed` and `seconds`.
Differently from the `Drive` function, the `speed` of the motors is setted to be equal in module but opposite in direction. In this way the robot stays in the same position while it rotates.

```
**function turn**  (speed, seconds)

right.motor set to speed
left.motor set to -speed
sleep for seconds
right.motor set to 0
left.motor set to 0
```

### Find Silver Token ###

While moving around the arena the robot needs to detect silver tokens which are in his way.
In order to find all silver tokens along the circuit, the `find_silver_token` function was developed.
It does not take anything as arguments and returns two values:
- `dist` which is the distance of the closest silver token from the center of the robot.
- `rot_y` which is the angle between the robot and the closest silver token.
The silver tokens, in order to be detected, need to be inside a chosen range, decided at the beginning of the program which is in front of the robot inside a certain distance threshold.
In my solution the linear distance threshold (given beginning_dist) is setted to `100` and the angular region detected (angular_threshold) goes from `-60` to `+60` degrees

```
**function find_silver_token** ()

giving a beginning_dist
FOR token in the filed of view:
      IF token.dist is less than beginning_dist AND token.rot_y is inside the angular_threshold AND token is silver:
            dist set to dist_silver_from_robot
            rot_y set to angle_silver_in_respect_to_robot
IF dist is the same as beginning_dist:
      return is -1 AND -1
ELSE:
      return is dist_silver_from_robot AND angle_silver_in_respect_to_robot
```

### Find Closest Golden Token Front ###

This function is very similar to the previous one explained. In fact, the only difference is that the previous one detects silver tokens, whereas this function detects golden tokens.
The aim is the same: detect all the golden token in a given range in front of the robot.

```
**function find_closest_golden_token_front** ()

giving a beginning_dist
FOR token in the filed of view:
      IF token.rot_y is inside the angular_threshold AND token is gold:
            IF token.dist is less than beginning_dist AND token is gold:
                  dist set to dist_gold_from_robot
                  rot_y set to angle_gold_in_respect_to_robot
IF dist is the same as beginning dist:
      return is -1 AND -1
ELSE:
      return is dist_gold_from_robot AND angle_gold_in_respect_to_robot
```

### Find Closest Golden Left and Find Closest Golden Right ###

The aim of the functions is to find the closest token on the left side of the robot and the right side respectively.
The function are similar to the previous described one, except for the region detected (angular_threshold) that goes from `-130` to `-55` degrees on the left, and `+55` to `+130` degrees on the right.

The pseudocode is exactly the same as reported above in the `find_closest_golden_token_front` function.

### Find Closest Golden Token Collide ###

The objective of the function is to let the robot understand if there is a token in front of him that could actually lead to a collision and, therefore avoid the contact.
No arguments are needed and the return value is a boolean, which is called `out`.
The return value of the function is `True` if a golden token is detected in front of the robot inside a given range. `False` if there are no golden markers in front of the robot that could lead to a collision.
The range detected from the function is narrow in front of the robot and close to it. This is due to the fact that we need to be sure that a token in this range would lead to a collision. In my solution the linear distance threshold (given beginning_dist) is setted to `1.1` and the angular region detected (angular_threshold) goes from `-25` to `+25` degrees

```
**function find_closest_golden_token_collide** ()

giving a beginning_dist
out set to False
FOR token in the filed of view:
      IF token.dist is less than beginning_dist AND token.rot_y is inside the angular_threshold AND token is gold:
                  out set to True
                  return out
ELSE:
      return out //which is false
```

### Main Function ###

The main function is inside a while infinite loop that will keep running forever until the user closes the exercise.
The `Main` is quite easy and thanks to this is easily understandable, has good performance and respect the requirements.
In fact has three key points that, using the functions defined above, allows the robot to respect the wanted behavior.
The three main points are:
1. The robot is close to a golden token.
2. The robot is close to a silver token.
3. The robot does not have anything in his way.

At each cycle of the while loop, before checking anything, the main calls the `find_silver_token`, `find_closest_golden_token_front` and `find_closest_golden_token_collide` functions in order to have all the informations needed in the next points

1. Using the `find_closest_golden_token_front` and `find_closest_golden_token_collide` the robot detects that he is too close to a golden token that could lead to a collision. If the answer is true, the robot makes a decision and it turn left or right in order to avoid the golden token. The decision is made calling the `find_closest_golden_token_left` and `find_closest_golden_token_right`. If the distance detected from the robot to the closest marker on the left is greater than the distance detected from the robot to the closest marker on the right, the robot should turn left because there is more space for him to go. Vice-versa, the robot should turn right. The robot should turn until in front of him there are no golden markers that are inside a given linear threshold.

```
IF distance from golden token in front is less than a linear threshold AND find_closest_golden_token_collide is True:
      find_closest_golden_token_left // call the functions to have the distance from tokens to the left and right
      find_closest_golden_token_right
      IF distance from left is grater than distance from right:
            turn left set to true
            WHILE turn left is true:
                  turn left
                  IF distance from golden token in front is grater than a linear threshold:
                        drive a bit on the left
                        turn left set to False
      ELSE:
            turn right set to true
            WHILE turn right is true:
                  turn right
                  IF distance from golden token in front is grater than a linear threshold:
                        drive a bit on the right
                        turn right set to False

     ```

2. 
