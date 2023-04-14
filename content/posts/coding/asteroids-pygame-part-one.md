
---
title : "Creating Asteroids in python, part 1 of 2" 
tags : ["coding", "python", "game-develompment"] 
date : "2023-04-14" 
draft : true

---


# Python Tutorial: Building Asteroids Game with Pygame

In this tutorial, we will be building the classic arcade game Asteroids using pyGame, a popular python library for creating 2D games. The goal of the game is to navigate through an asteroid field while avoiding collisions with the asteroids and enemy spacecraft. The game will include the following features:

- A player spacecraft that can rotate, thrust, and fire weapons
- Asteroids that break upon collision
- Scoring system based on the number of asteroids destroyed

## Setting Up the Game Window

First, we need to set up the game window. We will import the Pygame library and create a window with a specified size.

```python
import pygame

# Initialize Pygame
pygame.init()

# Set the window size
screen_x = 800
screen_y = 600
screen = pygame.display.set_mode((screen_x, screen_y))
```

## Defining Space Objects

Next, we will define the SpaceObject class, which will be the base class for all game objects. This class will define the object's position, velocity, size, angle, color, and label. We will also define methods for rotating, thrusting, updating the object's position, and drawing the object on the screen.

```python
import numpy as np
import random
from loguru import logger
import math

class SpaceObject:
    def __init__(self, x, y, dx, dy, size, angle, color, mx, my, label=None):
        self.x = x # initial position x
        self.y = y
        self.dx = dx # velocity x
        self.dy = dy # velocity y
        self.size = size # [0,0]
        self.angle = angle # initial angle
        self.color = color # (255,255,255)
        self.mx = mx # array x coords
        self.my = my # array y coords
        self.sx = np.zeros(len(self.mx))
        self.sy = np.zeros(len(self.my))
        self.collided = False
        self.label = label

    def rotate_left(self, dt, intensity):
        self.angle -= intensity * dt

    def rotate_right(self, dt, intensity):
        self.angle += intensity * dt

    def thrust_up(self, dt, intensity):
        self.dx += np.sin(self.angle) * intensity * dt # acceleration changes velocity (with respect to dt)
        self.dy += -np.cos(self.angle) * intensity * dt # -cos() because screen is flipped

    def update_rotation(self):
        for i in range(0, len(self.mx)):
            self.sx[i] = self.mx[i] * np.cos(self.angle) - self.my[i] * np.sin(self.angle)
            self.sy[i] = self.mx[i] * np.sin(self.angle) + self.my[i] * np.cos(self.angle)

    def update_translation(self):
        for i in range(0, len(self.sx)):
            self.sx[i] = self.sx[i] + self.x
            self.sy[i] = self.sy[i] + self.y

    def update_position(self, dt):
        self.x += self.dx * dt
        self.y += self.dy * dt

    def draw(self, screen, screen_x, screen_y):
        pygame.draw.polygon(screen, self.color, self.poly_points, 1)

    def update_and_render(self, dt, screen, screen_x, screen_y):

        self.update_rotation()
        self.update_translation()
        self.update_position(dt)
        self.draw(screen, screen_x, screen_y)
```

## Wrapping Object Coordinates

To ensure that game objects wrap around the screen, we will define a `wrap_coordinates` method that checks the object's position and wraps it around to the opposite side of the screen if necessary.

```python
    def wrap_coordinates(self, screen_x, screen_y):
        ox = self.x  # if no conditions met return input value
        oy = self.y
        if self.x < 0:
            ox = self.x + screen_x
            return ox, oy
        if self.x >= screen_x:
            ox = self.x - screen_x
            return ox, oy
        if self.y < 0:
            oy = self.y + screen_y
        if self.y >= screen_y:
            oy = self.y - screen_y
            return ox, oy
        else:
            return ox, oy
```

## Drawing Objects on the Screen

To draw objects on the screen, we will define a `draw` method that uses Pygame's `draw.polygon` method to draw the object's shape on the screen. We will also define a `bounding_box` method that calculates the object's bounding box, which will be used for collision detection.

```python
def bounding_box(points):
    # adapted from:
    # https://stackoverflow.com/questions/46335488/how-to-efficiently-find-the-bounding-box-of-a-collection-of-points
    x, y = zip(*points)
    return [(min(x), min(y)),\
             (max(x), min(y)),\
             (max(x), max(y)),\
             (min(x), max(y))]  # all points (anti-clockwise)

class SpaceObject:
    ...

    def draw(self, screen, screen_x, screen_y):
        self.x, self.y = self.wrap_coordinates(screen_x, screen_y)
        self.poly_points = []
        for i in range(0, len(self.sx)):
            self.poly_points.append([int(self.sx[i]), int(self.sy[i])])

        self.bb = bounding_box(self.poly_points)
        self.bb_width = math.hypot(self.bb[0][0]-self.bb[1][0], self.bb[0][1]-self.bb[1][1])
        self.bb_height = math.hypot(self.bb[1][0]-self.bb[2][0], self.bb[1][1]-self.bb[2][1])
        self.surface = pygame.Surface((int(self.bb_width), int(self.bb_height)))
        self.hit_box = self.surface.get_rect(topleft=(self.bb[0][0], self.bb[0][1])) # remember to set top_left in the update cycle!

        pygame.draw.polygon(screen, self.color, self.poly_points, 1)
        # pygame.draw.rect(screen, (135,0,135), self.hit_box, 1) # it's useful to render bb for debugging
```

## Updating and Rendering Objects

Finally, we will define an `update_and_render` method that updates the object's position and rotation, and then renders the object on the screen.

```python
class SpaceObject:
    ...

    def update_and_render(self, dt, screen, screen_x, screen_y):

        self.update_rotation()
        self.update_translation()
        self.update_position(dt)
        self.draw(screen, screen_x, screen_y)
```

With these basic SpaceObject methods defined, we can now start building the game mechanics.


# Implementing Asteroids Game in Python using Pygame

Asteroids is a classic arcade game that was released in 1979 by Atari. The game involves controlling a spaceship that must destroy asteroids, while avoiding collisions with them. In this tutorial, we will be implementing a Python version of the Asteroids game using Pygame, a popular game development module in Python.

## Setting up the Game

So far we created the main class we are gonna use. There are still a couple things to setup the game window, like the font and size.

```python
pygame.init()  # init
pygame.font.init()
my_font = pygame.font.SysFont('Segoe UI', 13)
screen_x, screen_y = 1200, 600 
screen = pygame.display.set_mode((screen_x, screen_y))  # screen size
```

Next, we will set up the game loop using a while loop that will continue running until the user quits the game. We will also set up a clock object to control the frames per second (fps) of the game.

```python
done = False  # flag for quitting
clock = pygame.time.Clock()  # class for setting fps
fps = 20

# delta time init
last_time = time.time()
```

To process input from the user, we will use the `pygame.event.get()` method to get a list of all the events that the Pygame module has queued up. We will then loop through the events and check if the user has pressed the 'QUIT' button to exit the game.

```python
for event in pygame.event.get():
    if event.type == pygame.QUIT:
        done = True
```

## Generating Asteroids

In the Asteroids game, the player must destroy asteroids that are randomly generated throughout the game. We will create a function that generates a new asteroid with random attributes such as size, velocity, and position. This function will return a `SpaceObject` with these attributes.

```python

# ectagon model (in vertices)
ect_x = [0, 25, 75, 100, 100, 75, 25, 0]
ect_y = [0, -25, -25, 0, 25, 50, 50, 25]

def generate_asteroid():
    labels = ["Luce", "Gas", "Sigarette", "Preservativi rotti", "Lubrificante anale", "Cocaina", \
                "Pannloni", "Benzina", "Telefono", "Internet", "Condominio"]
    return SpaceObject(random.randint(50, screen_x/3), random.randint(50, screen_y), random.uniform(0.0, 1),
                random.uniform(0.0, 1), [10, 10],
                random.uniform(0, 0.5), (255, 255, 255), [i + random.uniform(0, 10) for i in ect_x],
                [i + random.uniform(0, 10) for i in ect_y], \
                       label=labels[random.randint(0, len(labels) - 1)])
```

We will then create a list of asteroids and add six randomly generated asteroids to this list.

```python
asteroids = []
for i in range (0,6):
    asteroids.append(generate_asteroid())
```

## Creating the Player

The player is represented by a spaceship that can rotate, thrust forward, and shoot bullets. We will create a `SpaceObject` for the player with the following attributes:

- Position in the center of the screen
- Velocity set to 0
- Size of [10, 10]
- Color of red
- Shape of an equilateral triangle with side length of 50 pixels

```python
player = SpaceObject(int(screen_x/2),int(screen_y/2),0,0,[10,10],0,(255,0,0),[0, -25, 25],[-30, 25, 25])
```

Finally, we initialize a couple variables to store important info just before the core of the program, the game loop.

```python
bullets = []
score = 0
life = 10
level = 1
```

In the next post we shall illustrate how to put all of this togheter and actually using player input to handle events in the game. 
