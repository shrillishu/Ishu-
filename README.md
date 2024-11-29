# Codes of the pygame 
Python Based pong game 
import pygame
import sys
import random

# Initialize Pygame
pygame.init()

# Screen dimensions
WIDTH, HEIGHT = 800, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Pong Game - Cursor Control & Smarter Opponent")

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
BLUE = (0, 0, 255)
GREEN = (0, 255, 0)

# Game variables
ball_speed = [5, 5]
paddle_speed = 10
ball = pygame.Rect(WIDTH // 2 - 15, HEIGHT // 2 - 15, 30, 30)
player = pygame.Rect(WIDTH - 20, HEIGHT // 2 - 70, 10, 140)
opponent = pygame.Rect(10, HEIGHT // 2 - 70, 10, 140)

# Power-up variables
power_up_active = False
power_up_timer = 0
current_power_up = None
power_ups = [
    {"type": "speed", "color": RED},
    {"type": "slow", "color": BLUE},
    {"type": "reverse", "color": GREEN}
]

# Scores
player_score = 0
opponent_score = 0
font = pygame.font.Font(None, 50)

# Clock
clock = pygame.time.Clock()

# Show mouse cursor
pygame.mouse.set_visible(True)

def spawn_power_up():
    """Randomly spawns a power-up on the screen."""
    global current_power_up, power_up_active
    if not power_up_active:
        power_up = random.choice(power_ups)
        current_power_up = {
            "rect": pygame.Rect(random.randint(100, WIDTH - 100), random.randint(50, HEIGHT - 50), 20, 20),
            "type": power_up["type"],
            "color": power_up["color"]
        }
        power_up_active = True

def reset_ball():
    """Resets the ball to the center."""
    global ball_speed
    ball.center = (WIDTH // 2, HEIGHT // 2)
    ball_speed = [random.choice([-5, 5]), random.choice([-5, 5])]

def ball_movement():
    """Handles ball movement and collisions."""
    global player_score, opponent_score, ball_speed, power_up_active, power_up_timer

    ball.x += ball_speed[0]
    ball.y += ball_speed[1]

    # Bounce off top and bottom walls
    if ball.top <= 0 or ball.bottom >= HEIGHT:
        ball_speed[1] *= -1

    # Paddle collisions
    if ball.colliderect(player) or ball.colliderect(opponent):
        ball_speed[0] *= -1

    # Out of bounds
    if ball.left <= 0:
        player_score += 1
        reset_ball()
    if ball.right >= WIDTH:
        opponent_score += 1
        reset_ball()

    # Power-up collision
    if power_up_active and current_power_up and ball.colliderect(current_power_up["rect"]):
        power_up_active = False
        power_up_type = current_power_up["type"]

        if power_up_type == "speed":
            ball_speed[0] *= 1.5
            ball_speed[1] *= 1.5
        elif power_up_type == "slow":
            ball_speed[0] *= 0.5
            ball_speed[1] *= 0.5

        power_up_timer = pygame.time.get_ticks()

    # Reset effects after 5 seconds
    if pygame.time.get_ticks() - power_up_timer > 5000:
        ball_speed = [5 if x > 0 else -5 for x in ball_speed]

def player_movement():
    """Moves the player paddle based on mouse cursor."""
    mouse_y = pygame.mouse.get_pos()[1]  # Get the y-coordinate of the mouse
    player.centery = mouse_y  # Set the paddle's center to the mouse position

    # Prevent the paddle from moving out of bounds
    if player.top < 0:
        player.top = 0
    if player.bottom > HEIGHT:
        player.bottom = HEIGHT

def opponent_movement():
    """Improved AI for opponent paddle."""
    opponent_center = opponent.centery
    reaction_speed = 6  # Speed of the opponent paddle

    # Follow the ball only when it's moving toward the opponent
    if ball_speed[0] < 0:  # Ball moving towards opponent
        if opponent_center < ball.centery:
            opponent.y += reaction_speed
        elif opponent_center > ball.centery:
            opponent.y -= reaction_speed

    # Prevent the paddle from moving out of bounds
    if opponent.top < 0:
        opponent.top = 0
    if opponent.bottom > HEIGHT:
        opponent.bottom = HEIGHT

def draw_screen():
    """Draws all elements on the screen."""
    screen.fill(BLACK)
    pygame.draw.rect(screen, WHITE, player)
    pygame.draw.rect(screen, WHITE, opponent)
    pygame.draw.ellipse(screen, WHITE, ball)
    pygame.draw.aaline(screen, WHITE, (WIDTH // 2, 0), (WIDTH // 2, HEIGHT))

    # Draw scores
    player_text = font.render(f"{player_score}", True, WHITE)
    opponent_text = font.render(f"{opponent_score}", True, WHITE)
    screen.blit(player_text, (WIDTH // 2 + 20, 20))
    screen.blit(opponent_text, (WIDTH // 2 - 50, 20))

    # Draw power-up
    if power_up_active and current_power_up:
        pygame.draw.circle(screen, current_power_up["color"],
                           (current_power_up["rect"].centerx, current_power_up["rect"].centery),
                           current_power_up["rect"].width // 2)

# Main game loop
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

    player_movement()  # Paddle follows cursor
    opponent_movement()
    ball_movement()

    # Randomly spawn power-up
    if random.randint(1, 300) == 1:  # 1 in 300 chance per frame
        spawn_power_up()

    draw_screen()

    pygame.display.flip()
    clock.tick(60)
