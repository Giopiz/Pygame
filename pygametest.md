import pygame
import time
import random

pygame.font.init()
pygame.mixer.init()

# Constants
WIDTH, HEIGHT = 1000, 800
WIN = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Enhanced Game")

# Load assets
BG = pygame.image.load("background.png").convert()
BG_HEIGHT = BG.get_height()
PLAYER_IMG = pygame.image.load("player.png")
PLAYER_WIDTH, PLAYER_HEIGHT = 100, 200
PLAYER_IMG = pygame.transform.scale(PLAYER_IMG, (PLAYER_WIDTH, PLAYER_HEIGHT))
GOBLIN_IMG = pygame.image.load("goblin.png")
GOBLIN_WIDTH, GOBLIN_HEIGHT = 100, 100
GOBLIN_IMG = pygame.transform.scale(GOBLIN_IMG, (GOBLIN_WIDTH, GOBLIN_HEIGHT))
pygame.mixer.music.load("background_music.mp3")
pygame.mixer.music.set_volume(0.5)

# Fonts
FONT = pygame.font.SysFont("comicsans", 30)
HIT_FONT = pygame.font.SysFont("comicsans", 40)

# Player and Goblin speeds
PLAYER_VEL = 15
GOBLIN_VEL = 10

def draw(player, elapsed_time, goblins, lives, score, bg_y, hit_message=None):
    # Draw scrolling background
    bg_y = bg_y % HEIGHT  # Wrap the background position
    WIN.blit(BG, (0, bg_y))
    WIN.blit(BG, (0, bg_y - HEIGHT))  # Draw the second part seamlessly

    # Draw player and goblins
    WIN.blit(PLAYER_IMG, (player.x, player.y))
    for goblin in goblins:
        WIN.blit(GOBLIN_IMG, (goblin.x, goblin.y))

    # Display score, elapsed time, and lives
    score_text = FONT.render(f"Score: {score}", True, "white")
    WIN.blit(score_text, (10, 10))
    time_text = FONT.render(f"Time: {round(elapsed_time)}s", True, "white")
    WIN.blit(time_text, (10, 50))
    lives_text = FONT.render(f"Lives: {lives}", True, "white")
    WIN.blit(lives_text, (10, 90))

    # Display hit message
    if hit_message:
        message_text = HIT_FONT.render(hit_message, True, "red")
        WIN.blit(message_text, (WIDTH // 2 - message_text.get_width() // 2, HEIGHT // 2))

    pygame.display.update()

def draw_game_over(score):
    WIN.fill("black")
    game_over_text = HIT_FONT.render("Game Over!", True, "red")
    score_text = FONT.render(f"Final Score: {score}", True, "white")
    WIN.blit(game_over_text, (WIDTH // 2 - game_over_text.get_width() // 2, HEIGHT // 3))
    WIN.blit(score_text, (WIDTH // 2 - score_text.get_width() // 2, HEIGHT // 2))
    pygame.display.update()
    pygame.time.delay(3000)  # Pause for 3 seconds

def main():
    pygame.mixer.music.play(-1)

    # Game variables
    run = True
    player = pygame.Rect(200, HEIGHT - PLAYER_HEIGHT - 20, PLAYER_WIDTH, PLAYER_HEIGHT)
    clock = pygame.time.Clock()
    start_time = time.time()
    elapsed_time = 0
    score = 0
    lives = 3

    goblin_add_increment = 2000
    goblin_count = 0
    goblins = []
    hit_message = None
    hit_message_time = 0
    bg_y = 0

    while run:
        goblin_count += clock.tick(60)
        elapsed_time = time.time() - start_time
        bg_y += 5  # Scrolling background speed
        if bg_y >= BG_HEIGHT:
            bg_y = 0

        # Spawn goblins
        if goblin_count > goblin_add_increment:
            for _ in range(3):
                new_goblin_x = random.randint(0, WIDTH - GOBLIN_WIDTH)
                new_goblin = pygame.Rect(new_goblin_x, -GOBLIN_HEIGHT, GOBLIN_WIDTH, GOBLIN_HEIGHT)
                goblins.append(new_goblin)

            goblin_add_increment = max(500, goblin_add_increment - 100)
            goblin_count = 0

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                run = False

        # Player movement
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT] and player.x - PLAYER_VEL >= 0:
            player.x -= PLAYER_VEL
        if keys[pygame.K_RIGHT] and player.x + PLAYER_VEL + player.width <= WIDTH:
            player.x += PLAYER_VEL
        if keys[pygame.K_UP] and player.y - PLAYER_VEL >= 0:
            player.y -= PLAYER_VEL
        if keys[pygame.K_DOWN] and player.y + PLAYER_VEL + player.height <= HEIGHT:
            player.y += PLAYER_VEL

        # Update goblins and check for collisions
        for goblin in goblins[:]:
            goblin.y += GOBLIN_VEL
            if goblin.y > HEIGHT:
                goblins.remove(goblin)
                score += 1  # Increase score for dodging
            elif goblin.colliderect(player):
                goblins.remove(goblin)
                lives -= 1  # Reduce lives
                hit_message = "You got hit!"
                hit_message_time = time.time()

        # Remove hit message after 2 seconds
        if hit_message and time.time() - hit_message_time > 2:
            hit_message = None

        # Check for game over
        if lives <= 0:
            draw_game_over(score)
            break

        draw(player, elapsed_time, goblins, lives, score, bg_y, hit_message)

    pygame.mixer.music.stop()
    pygame.quit()

if __name__ == "__main__":
    main()
