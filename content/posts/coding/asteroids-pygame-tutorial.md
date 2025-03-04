+++
title = "Re-create Asteroids in PyGame"
tags = ["coding", "python", "pygame"]
date = "2022-01-01"
draft = false
+++

# Introduction

In this tutorial, we will guide you through creating a classic Asteroids game using Python and the Pygame library. We'll cover setting up the game, handling user input, managing game objects, and implementing game logic.

### Project Structure

Here's the structure of the project:

```
asteroids-pygame/
│
├── main.py
├── README.md
```

### Step 1: Setting Up the Game

First, let's set up the basic structure of the game. We'll initialize Pygame, create the game window, and set up the main game loop.

```python
import pygame
import time
import numpy as np
import sys
import random
from loguru import logger
import math

# Initialize Pygame
pygame.init()
pygame.font.init()
my_font = pygame.font.SysFont('Segoe UI', 13)

# Set up the screen
screen_x, screen_y = 1200, 600
screen = pygame.display.set_mode((screen_x, screen_y))
done = False
clock = pygame.time.Clock()
fps = 20

# Delta time initialization
last_time = time.time()

# Ectagon model for asteroids
ect_x = [0, 25, 75, 100, 100, 75, 25, 0]
ect_y = [0, -25, -25, 0, 25, 50, 50, 25]
```

### Step 2: Creating the SpaceObject Class

The `SpaceObject` class will represent all moving objects in the game, including the player's spaceship, asteroids, and bullets.

```python
class SpaceObject:
    def __init__(self, x, y, dx, dy, size, angle, color, mx, my, label=None):
        self.x = x
        self.y = y
        self.dx = dx
        self.dy = dy
        self.size = size
        self.angle = angle
        self.color = color
        self.mx = mx
        self.my = my
        self.sx = np.zeros(len(self.mx))
        self.sy = np.zeros(len(self.my))
        self.collided = False
        self.label = label

    def wrap_coordinates(self, screen_x, screen_y):
        ox = self.x
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

    def rotate_left(self, dt, intensity):
        self.angle -= intensity * dt

    def rotate_right(self, dt, intensity):
        self.angle += intensity * dt

    def thrust_up(self, dt, intensity):
        self.dx += np.sin(self.angle) * intensity * dt
        self.dy += -np.cos(self.angle) * intensity * dt

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
        self.x, self.y = self.wrap_coordinates(screen_x, screen_y)
        self.poly_points = []
        for i in range(0, len(self.sx)):
            self.poly_points.append([int(self.sx[i]), int(self.sy[i])])

        self.bb = bounding_box(self.poly_points)
        self.bb_width = math.hypot(self.bb[0][0]-self.bb[1][0], self.bb[0][1]-self.bb[1][1])
        self.bb_height = math.hypot(self.bb[1][0]-self.bb[2][0], self.bb[1][1]-self.bb[2][1])
        self.surface = pygame.Surface((int(self.bb_width), int(self.bb_height)))
        self.hit_box = self.surface.get_rect(topleft=(self.bb[0][0], self.bb[0][1]))

        pygame.draw.polygon(screen, self.color, self.poly_points, 1)

    def update_and_render(self, dt, screen, screen_x, screen_y):
        self.update_rotation()
        self.update_translation()
        self.update_position(dt)
        self.draw(screen, screen_x, screen_y)
```

### Step 3: Handling User Input

Next, we'll handle user input to control the spaceship. The player will be able to rotate the spaceship, accelerate, and shoot bullets.

```python
def generate_asteroid():
    labels = ["Luce", "Gas", "Sigarette", "Preservativi rotti", "Lubrificante anale", "Cocaina",
                "Pannloni", "Benzina", "Telefono", "Internet", "Condominio"]
    return SpaceObject(random.randint(50, screen_x/3), random.randint(50, screen_y), random.uniform(0.0, 1),
                random.uniform(0.0, 1), [10, 10],
                random.uniform(0, 0.5), (255, 255, 255), [i + random.uniform(0, 10) for i in ect_x],
                [i + random.uniform(0, 10) for i in ect_y],
                       label=labels[random.randint(0, len(labels) - 1)])

asteroids = []
collision_time = 0

for i in range (0,6):
    asteroids.append(generate_asteroid())

player = SpaceObject(int(screen_x/2),int(screen_y/2),0,0,[10,10],0,(255,0,0),[0, -25, 25],[-30, 25, 25])
bullets = []
score = 0
life = 10
level = 1
while not done:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            done = True

    screen.fill((0, 0, 0))
    dt = time.time() - last_time
    dt *= 60
    last_time = time.time()

    pressed = pygame.key.get_pressed()

    if pressed[pygame.K_LEFT]:
        player.rotate_left(dt, 0.1)
    if pressed[pygame.K_RIGHT]:
        player.rotate_right(dt, 0.1)
    if pressed[pygame.K_UP] and player.collided == False:
        player.thrust_up(dt, 0.1)
    if pressed[pygame.K_DOWN] and player.collided == False:
        player.thrust_up(dt, -0.1)

    if pressed[pygame.K_SPACE]:
        bullets.append(SpaceObject(player.x,
                                   player.y,
                                   10*np.sin(player.angle),
                                   -10*np.cos(player.angle),
                                   [10,10],
                                   player.angle,
                                   (255,255,255),
                                   [0, 5, 5, 0],
                                   [0, 0, 5, 5]))

    player.update_and_render(dt, screen, screen_x, screen_y)

    for a in asteroids:
        a.update_and_render(dt, screen, screen_x, screen_y)

        if player.hit_box.colliderect(a.hit_box) and last_time >= collision_time + last_time%60/60+0.01:
            player.collided = True
            player.dx = player.dx * -1
            player.dy = player.dy * -1
            life -= 1
            collision_time = last_time
        else:
            player.collided = False

    if len(bullets) !=0:
        for b in bullets:
            if b.x > screen_x - 25 or b.y > screen_y - 25 or b.x < 20 or b.y < 25:
                bullets.remove(b)
            b.update_and_render(dt, screen, screen_x, screen_y)

            for a in asteroids:
                if b.hit_box.colliderect(a.hit_box):
                    try:
                        bullets.remove(b)
                        asteroids.remove(a)
                        score += 1
                    except:
                        pass

    if len(asteroids) <= 0:
        level += 1
        for a in range(0, 6 + level):
            asteroids.append(generate_asteroid())

    text_surface = my_font.render(f'Score: {score} Remaining ships: {life} Level: {level}',  False, (255, 255, 255))

    screen.blit(text_surface, (20, 20))

    clock.tick(fps)
    pygame.display.flip()
```

### Step 4: Adding Asteroids and Bullets

We'll create asteroids that the player must avoid or shoot. When an asteroid is hit by a bullet, it will be removed from the game.

```python
def generate_asteroid():
    labels = ["Luce", "Gas", "Sigarette", "Preservativi rotti", "Lubrificante anale", "Cocaina",
                "Pannloni", "Benzina", "Telefono", "Internet", "Condominio"]
    return SpaceObject(random.randint(50, screen_x/3), random.randint(50, screen_y), random.uniform(0.0, 1),
                random.uniform(0.0, 1), [10, 10],
                random.uniform(0, 0.5), (255, 255, 255), [i + random.uniform(0, 10) for i in ect_x],
                [i + random.uniform(0, 10) for i in ect_y],
                       label=labels[random.randint(0, len(labels) - 1)])

asteroids = []
collision_time = 0

for i in range (0,6):
    asteroids.append(generate_asteroid())
```

### Step 5: Handling Collisions

We'll detect collisions between the spaceship and asteroids, as well as between bullets and asteroids.

```python
if player.hit_box.colliderect(a.hit_box) and last_time >= collision_time + last_time%60/60+0.01:
    player.collided = True
    player.dx = player.dx * -1
    player.dy = player.dy * -1
    life -= 1
    collision_time = last_time
else:
    player.collided = False

if len(bullets) !=0:
    for b in bullets:
        if b.x > screen_x - 25 or b.y > screen_y - 25 or b.x < 20 or b.y < 25:
            bullets.remove(b)
        b.update_and_render(dt, screen, screen_x, screen_y)

        for a in asteroids:
            if b.hit_box.colliderect(a.hit_box):
                try:
                    bullets.remove(b)
                    asteroids.remove(a)
                    score += 1
                except:
                    pass
```


### Step 6: Adding Game Levels
We'll add levels to the game. As the player progresses, the number of asteroids will increase.

```python
if len(asteroids) <= 0:
    level += 1
    for a in range(0, 6 + level):
        asteroids.append(generate_asteroid())
```

### Conclusion

You've now created a basic Asteroids game using Pygame. You can further enhance the game by adding sounds, improving graphics, and implementing more advanced game mechanics. 

### Resources

- The code for this tutorial [on my GitHub](https://github.com/raphael2692/asteroids-pygame)
