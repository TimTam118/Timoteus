import numpy as np
from noise import pnoise2, pnoise3
import random

# World dimensions
width, length, height = 100, 100, 32

# Initialize world array (0=air, 1=stone, 2=grass, 3=dirt, 4=wood, 5=leaves)
world = np.zeros((width, length, height), dtype=int)

# Terrain generation (surface)
for x in range(width):
    for z in range(length):
        y = int(pnoise2(x / 30.0, z / 30.0, octaves=6, persistence=0.5, lacunarity=2.0) * 10 + (height // 2))
        y = max(5, min(height - 1, y))
        for h in range(y):
            if h == y - 1:
                world[x, z, h] = 2  # Grass
            elif h > y - 5:
                world[x, z, h] = 3  # Dirt
            else:
                world[x, z, h] = 1  # Stone

# Cave generation (Perlin Worms method)
num_caves = 30
for _ in range(num_caves):
    cx, cy, cz = random.randint(10, width-10), random.randint(5, height-10), random.randint(10, length-10)
    length_cave = random.randint(20, 40)
    theta, phi = random.uniform(0, 2*np.pi), random.uniform(0, np.pi)
    for i in range(length_cave):
        r = 2 + int(2 * np.sin(i / 5.0))  # Vary radius
        for dx in range(-r, r+1):
            for dy in range(-r, r+1):
                for dz in range(-r, r+1):
                    if dx*dx + dy*dy + dz*dz <= r*r:
                        x, y, z = int(cx+dx), int(cy+dy), int(cz+dz)
                        if 0 <= x < width and 0 <= y < height and 0 <= z < length:
                            world[x, z, y] = 0  # Carve air
        # Move the cave "worm"
        cx += int(np.cos(theta) * np.sin(phi))
        cy += int(np.cos(phi))
        cz += int(np.sin(theta) * np.sin(phi))
        theta += random.uniform(-0.2, 0.2)
        phi += random.uniform(-0.1, 0.1)

# Tree generation (simple procedural trees)
for _ in range(70):  # Number of trees
    x, z = random.randint(2, width-3), random.randint(2, length-3)
    # Find ground
    for y in range(height-2, 0, -1):
        if world[x, z, y] == 2:
            # Trunk
            for t in range(4):
                world[x, z, y+t+1] = 4
            # Leaves (simple sphere)
            for dx in range(-2, 3):
                for dz in range(-2, 3):
                    for dy in range(3, 6):
                        if dx*dx + dz*dz + (dy-4)**2 <= 5:
                            if 0 <= x+dx < width and 0 <= z+dz < length and 0 <= y+dy < height:
                                if world[x+dx, z+dz, y+dy] == 0:
                                    world[x+dx, z+dz, y+dy] = 5
            break
