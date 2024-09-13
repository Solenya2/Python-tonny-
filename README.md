"# Python-tonny-" 

import pygame
import random

 Initialize Pygame
pygame.init()

 Screen dimensions
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 600
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Platformer with Enemies, Collectibles, and Victory")

 Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
BLUE = (0, 0, 255)
GREEN = (0, 255, 0)
RED = (255, 0, 0)
YELLOW = (255, 255, 0)

 Clock
clock = pygame.time.Clock()

 Player settings
player_width = 40
player_height = 60
player_color = BLUE
playerX = 100
playerY = SCREEN_HEIGHT - player_height
playerX_change = 0
playerY_change = 0
player_speed = 5
jump_height = -15
gravity = 1
on_ground = False

 Platform settings
platform_width = 100
platform_height = 20
platform_color = GREEN

 List of platforms (x, y, width, height)
platforms = [
    (50, 500, platform_width, platform_height),
    (200, 400, platform_width, platform_height),
    (400, 300, platform_width, platform_height),
    (600, 200, platform_width, platform_height),
]

 Enemy settings
enemy_width = 40
enemy_height = 40
enemy_color = RED
enemy_speed = 2
enemies = [
    {"x": 150, "y": 480, "direction": 1},
    {"x": 300, "y": 380, "direction": -1},
]

 Collectible settings (coins)
coin_width = 20
coin_height = 20
coin_color = YELLOW
coins = [
    {"x": 70, "y": 470},
    {"x": 220, "y": 370},
    {"x": 420, "y": 270},
    {"x": 620, "y": 170},
]

 Score
score = 0
total_coins = len(coins)  # Total number of coins in the game
font = pygame.font.Font(None, 36)
victory_font = pygame.font.Font(None, 72)

 Function to draw the player
def draw_player(x, y):
    pygame.draw.rect(screen, player_color, (x, y, player_width, player_height))

 Function to draw platforms
def draw_platforms(platforms):
    for platform in platforms:
        pygame.draw.rect(screen, platform_color, platform)

 Function to draw enemies
def draw_enemies(enemies):
    for enemy in enemies:
        pygame.draw.rect(screen, enemy_color, (enemy["x"], enemy["y"], enemy_width, enemy_height))

 Function to move enemies back and forth
def move_enemies(enemies):
    for enemy in enemies:
        enemy["x"] += enemy["direction"] * enemy_speed
        if enemy["x"] <= 0 or enemy["x"] >= SCREEN_WIDTH - enemy_width:
            enemy["direction"] *= -1  # Reverse direction

 Function to draw collectibles (coins)
def draw_coins(coins):
    for coin in coins:
        pygame.draw.rect(screen, coin_color, (coin["x"], coin["y"], coin_width, coin_height))

 Function to check for collisions between the player and platforms
def check_collision(player_rect, platforms):
    for platform in platforms:
        platform_rect = pygame.Rect(platform)
        if player_rect.colliderect(platform_rect) and playerY_change >= 0:
            return platform_rect.top
    return None

 Function to check for collisions between the player and enemies
def check_enemy_collision(player_rect, enemies):
    for enemy in enemies:
        enemy_rect = pygame.Rect(enemy["x"], enemy["y"], enemy_width, enemy_height)
        if player_rect.colliderect(enemy_rect):
            return True
    return False

 Function to check for collisions between the player and coins (collectibles)
def check_coin_collision(player_rect, coins):
    global score
    for coin in coins[:]:  # Use a copy of the list to modify it while iterating
        coin_rect = pygame.Rect(coin["x"], coin["y"], coin_width, coin_height)
        if player_rect.colliderect(coin_rect):
            coins.remove(coin)
            score += 10  # Add points for each collected coin

 Function to show the score on the screen
def show_score():
    score_text = font.render(f"Score: {score}", True, BLACK)
    screen.blit(score_text, (10, 10))

 Function to show the victory message
def show_victory():
    victory_text = victory_font.render("YOU WIN!", True, BLACK)
    screen.blit(victory_text, (SCREEN_WIDTH // 2 - 150, SCREEN_HEIGHT // 2 - 50))

 Main Game Loop
running = True
victory = False

while running:
    
    screen.fill(WHITE)   Fill background

    if victory:
         Show the victory screen
        show_victory()
        pygame.display.update()
        continue  # Skip further game logic once the player wins

    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            running = False

         Keydown for movement
        if event.type == pygame.KEYDOWN:
            if event.key == pygame.K_LEFT:
                playerX_change = -player_speed
            if event.key == pygame.K_RIGHT:
                playerX_change = player_speed
            if event.key == pygame.K_SPACE and on_ground:
                playerY_change = jump_height
                on_ground = False

        #Keyup to stop movement
        if event.type == pygame.KEYUP:
            if event.key == pygame.K_LEFT or event.key == pygame.K_RIGHT:
                playerX_change = 0

     Apply gravity
    playerY_change += gravity
    playerY += playerY_change
    playerX += playerX_change

     Prevent player from falling off the screen
    if playerY > SCREEN_HEIGHT - player_height:
        playerY = SCREEN_HEIGHT - player_height
        playerY_change = 0
        on_ground = True

     Prevent player from moving out of screen horizontally
    if playerX < 0:
        playerX = 0
    elif playerX > SCREEN_WIDTH - player_width:
        playerX = SCREEN_WIDTH - player_width

     Create player rectangle for collision detection
    player_rect = pygame.Rect(playerX, playerY, player_width, player_height)

     Check for collisions with platforms
    platform_top = check_collision(player_rect, platforms)
    if platform_top is not None:
        playerY = platform_top - player_height
        playerY_change = 0
        on_ground = True

     Check for collisions with enemies
    if check_enemy_collision(player_rect, enemies):
        print("Game Over!")
        running = False

     Check for collisions with coins
    check_coin_collision(player_rect, coins)

     Check if all coins are collected
    if len(coins) == 0:
        victory = True

     Move enemies
    move_enemies(enemies)

     Draw everything
    draw_player(playerX, playerY)
    draw_platforms(platforms)
    draw_enemies(enemies)
    draw_coins(coins)
    show_score()

     Update the display
    pygame.display.update()

     Frame rate
    clock.tick(60)


 Quit the game
pygame.quit()
