import pygame
import random
import math

pygame.init()

# -----------------------------
# SCREEN
# -----------------------------
WIDTH, HEIGHT = 480, 800
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Flappy Sky")

clock = pygame.time.Clock()

# -----------------------------
# COLORS
# -----------------------------
SKY = (120, 200, 255)
SKY2 = (190, 235, 255)
WHITE = (255, 255, 255)
BLACK = (25, 30, 40)
GREEN = (65, 190, 95)
GREEN_DARK = (40, 145, 70)
YELLOW = (255, 220, 60)
ORANGE = (255, 150, 40)
GROUND = (235, 205, 120)

# -----------------------------
# FONTS
# -----------------------------
font_big = pygame.font.SysFont("arial", 54, bold=True)
font_score = pygame.font.SysFont("arial", 42, bold=True)
font_medium = pygame.font.SysFont("arial", 28, bold=True)
font_small = pygame.font.SysFont("arial", 22)

# -----------------------------
# GAME SETTINGS
# -----------------------------
GRAVITY = 0.45
FLAP_POWER = -8.5
PIPE_SPEED = 3.2
PIPE_WIDTH = 75
GAP = 190

# -----------------------------
# BIRD
# -----------------------------
bird_x = 110
bird_y = HEIGHT // 2
bird_velocity = 0
bird_angle = 0

# -----------------------------
# GAME VARIABLES
# -----------------------------
pipes = []
particles = []

score = 0
best_score = 0

game_started = False
game_over = False

spawn_timer = 0

# -----------------------------
# CREATE PIPE
# -----------------------------
def create_pipe():
    gap_y = random.randint(190, HEIGHT - 230)

    pipes.append({
        "x": WIDTH + 30,
        "gap_y": gap_y,
        "passed": False
    })

# -----------------------------
# RESET GAME
# -----------------------------
def reset_game():
    global bird_y, bird_velocity
    global pipes, particles
    global score, game_started, game_over
    global spawn_timer

    bird_y = HEIGHT // 2
    bird_velocity = 0

    pipes.clear()
    particles.clear()

    score = 0
    spawn_timer = 0

    game_started = False
    game_over = False

# -----------------------------
# FLAP
# -----------------------------
def flap():
    global bird_velocity
    global game_started

    if not game_over:
        game_started = True
        bird_velocity = FLAP_POWER

        # particles
        for _ in range(7):
            particles.append({
                "x": bird_x - 10,
                "y": bird_y + 10,
                "vx": random.uniform(-2, -0.5),
                "vy": random.uniform(-1, 2),
                "life": 25
            })

# -----------------------------
# DRAW BIRD
# -----------------------------
def draw_bird():
    global bird_angle

    bird_angle = max(-25, min(70, -bird_velocity * 3))

    bird_surface = pygame.Surface((60, 50), pygame.SRCALPHA)

    # Body
    pygame.draw.ellipse(
        bird_surface,
        YELLOW,
        (8, 10, 42, 32)
    )

    # Wing
    pygame.draw.ellipse(
        bird_surface,
        ORANGE,
        (12, 25, 23, 13)
    )

    # Eye
    pygame.draw.circle(
        bird_surface,
        WHITE,
        (38, 18),
        8
    )

    pygame.draw.circle(
        bird_surface,
        BLACK,
        (40, 18),
        4
    )

    # Beak
    pygame.draw.polygon(
        bird_surface,
        ORANGE,
        [(47, 25), (60, 30), (47, 34)]
    )

    rotated = pygame.transform.rotate(
        bird_surface,
        bird_angle
    )

    rect = rotated.get_rect(
        center=(bird_x, int(bird_y))
    )

    screen.blit(rotated, rect)

# -----------------------------
# DRAW PIPES
# -----------------------------
def draw_pipe(pipe):
    x = int(pipe["x"])
    gap_y = pipe["gap_y"]

    top_height = gap_y - GAP // 2
    bottom_y = gap_y + GAP // 2

    # Top pipe
    pygame.draw.rect(
        screen,
        GREEN_DARK,
        (x, 0, PIPE_WIDTH, top_height)
    )

    pygame.draw.rect(
        screen,
        GREEN,
        (x + 7, 0, PIPE_WIDTH - 14, top_height)
    )

    # Top cap
    pygame.draw.rect(
        screen,
        GREEN_DARK,
        (x - 5, top_height - 25, PIPE_WIDTH + 10, 25)
    )

    pygame.draw.rect(
        screen,
        GREEN,
        (x + 3, top_height - 22, PIPE_WIDTH - 6, 18)
    )

    # Bottom pipe
    pygame.draw.rect(
        screen,
        GREEN_DARK,
        (x, bottom_y, PIPE_WIDTH, HEIGHT - bottom_y - 90)
    )

    pygame.draw.rect(
        screen,
        GREEN,
        (x + 7, bottom_y, PIPE_WIDTH - 14, HEIGHT - bottom_y - 90)
    )

    # Bottom cap
    pygame.draw.rect(
        screen,
        GREEN_DARK,
        (x - 5, bottom_y, PIPE_WIDTH + 10, 25)
    )

    pygame.draw.rect(
        screen,
        GREEN,
        (x + 3, bottom_y + 3, PIPE_WIDTH - 6, 18)
    )

# -----------------------------
# DRAW CLOUDS
# -----------------------------
clouds = [
    [80, 120, 1],
    [330, 180, 0.8],
    [200, 300, 0.7],
    [420, 80, 0.6]
]

def draw_clouds():
    for cloud in clouds:
        x, y, scale = cloud

        pygame.draw.circle(
            screen,
            WHITE,
            (int(x), int(y)),
            int(25 * scale)
        )

        pygame.draw.circle(
            screen,
            WHITE,
            (int(x + 25 * scale), int(y + 5 * scale)),
            int(20 * scale)
        )

        pygame.draw.circle(
            screen,
            WHITE,
            (int(x - 25 * scale), int(y + 7 * scale)),
            int(18 * scale)
        )

# -----------------------------
# PARTICLES
# -----------------------------
def update_particles():
    for p in particles[:]:
        p["x"] += p["vx"]
        p["y"] += p["vy"]
        p["vy"] += 0.08
        p["life"] -= 1

        if p["life"] <= 0:
            particles.remove(p)

def draw_particles():
    for p in particles:
        pygame.draw.circle(
            screen,
            WHITE,
            (int(p["x"]), int(p["y"])),
            3
        )

# -----------------------------
# COLLISION
# -----------------------------
def check_collision():
    bird_rect = pygame.Rect(
        bird_x - 18,
        bird_y - 16,
        36,
        32
    )

    # Ground / ceiling
    if bird_y - 16 <= 0:
        return True

    if bird_y + 16 >= HEIGHT - 90:
        return True

    for pipe in pipes:

        pipe_x = pipe["x"]
        gap_y = pipe["gap_y"]

        top_rect = pygame.Rect(
            pipe_x,
            0,
            PIPE_WIDTH,
            gap_y - GAP // 2
        )

        bottom_rect = pygame.Rect(
            pipe_x,
            gap_y + GAP // 2,
            PIPE_WIDTH,
            HEIGHT
        )

        if bird_rect.colliderect(top_rect):
            return True

        if bird_rect.colliderect(bottom_rect):
            return True

    return False

# -----------------------------
# UPDATE GAME
# -----------------------------
def update_game():
    global bird_y, bird_velocity
    global spawn_timer, score
    global game_over, best_score

    if not game_started or game_over:
        return

    # Bird physics
    bird_velocity += GRAVITY
    bird_y += bird_velocity

    # Pipes
    spawn_timer += 1

    if spawn_timer > 100:
        create_pipe()
        spawn_timer = 0

    for pipe in pipes:
        pipe["x"] -= PIPE_SPEED

        # Score
        if not pipe["passed"] and pipe["x"] + PIPE_WIDTH < bird_x:
            pipe["passed"] = True
            score += 1

    # Remove old pipes
    pipes[:] = [
        p for p in pipes
        if p["x"] > -PIPE_WIDTH - 20
    ]

    # Collision
    if check_collision():
        game_over = True

        if score > best_score:
            best_score = score

# -----------------------------
# DRAW GROUND
# -----------------------------
ground_scroll = 0

def draw_ground():
    global ground_scroll

    ground_scroll -= PIPE_SPEED

    if ground_scroll <= -40:
        ground_scroll = 0

    pygame.draw.rect(
        screen,
        GROUND,
        (0, HEIGHT - 90, WIDTH, 90)
    )

    # Grass
    pygame.draw.rect(
        screen,
        GREEN,
        (0, HEIGHT - 90, WIDTH, 10)
    )

    # Moving pattern
    for x in range(-40, WIDTH + 40, 40):
        pygame.draw.rect(
            screen,
            (215, 180, 95),
            (
                int(x + ground_scroll),
                HEIGHT - 65,
                20,
                8
            )
        )

# -----------------------------
# UI
# -----------------------------
def draw_ui():

    if game_started and not game_over:
        text = font_score.render(
            str(score),
            True,
            WHITE
        )

        screen.blit(
            text,
            text.get_rect(
                center=(WIDTH // 2, 70)
            )
        )

    elif not game_started:

        title = font_big.render(
            "FLAPPY SKY",
            True,
            WHITE
        )

        screen.blit(
            title,
            title.get_rect(
                center=(WIDTH // 2, 250)
            )
        )

        info = font_medium.render(
            "TAP TO FLY",
            True,
            WHITE
        )

        screen.blit(
            info,
            info.get_rect(
                center=(WIDTH // 2, 330)
            )
        )

        info2 = font_small.render(
            "Avoid the pipes!",
            True,
            WHITE
        )

        screen.blit(
            info2,
            info2.get_rect(
                center=(WIDTH // 2, 370)
            )
        )

    if game_over:

        overlay = pygame.Surface(
            (WIDTH, HEIGHT),
            pygame.SRCALPHA
        )

        overlay.fill((0, 0, 0, 100))
        screen.blit(overlay, (0, 0))

        box = pygame.Rect(
            55,
            230,
            WIDTH - 110,
            300
        )

        pygame.draw.rect(
            screen,
            WHITE,
            box,
            border_radius=25
        )

        title = font_big.render(
            "GAME OVER",
            True,
            BLACK
        )

        screen.blit(
            title,
            title.get_rect(
                center=(WIDTH // 2, 290)
            )
        )

        score_text = font_medium.render(
            "Score: " + str(score),
            True,
            BLACK
        )

        best_text = font_medium.render(
            "Best: " + str(best_score),
            True,
            BLACK
        )

        screen.blit(
            score_text,
            score_text.get_rect(
                center=(WIDTH // 2, 350)
            )
        )

        screen.blit(
            best_text,
            best_text.get_rect(
                center=(WIDTH // 2, 390)
            )
        )

        restart = font_medium.render(
            "TAP TO RESTART",
            True,
            GREEN_DARK
        )

        screen.blit(
            restart,
            restart.get_rect(
                center=(WIDTH // 2, 465)
            )
        )

# -----------------------------
# MAIN LOOP
# -----------------------------
running = True

while running:

    for event in pygame.event.get():

        if event.type == pygame.QUIT:
            running = False

        # Keyboard
        if event.type == pygame.KEYDOWN:

            if event.key == pygame.K_SPACE:

                if game_over:
                    reset_game()
                    flap()
                else:
                    flap()

            if event.key == pygame.K_ESCAPE:
                running = False

        # Mouse / Touch
        if event.type == pygame.MOUSEBUTTONDOWN:

            if game_over:
                reset_game()
                flap()
            else:
                flap()

    # Background gradient
    for y in range(HEIGHT):
        ratio = y / HEIGHT

        r = int(SKY[0] * (1 - ratio) + SKY2[0] * ratio)
        g = int(SKY[1] * (1 - ratio) + SKY2[1] * ratio)
        b = int(SKY[2] * (1 - ratio) + SKY2[2] * ratio)

        pygame.draw.line(
            screen,
            (r, g, b),
            (0, y),
            (WIDTH, y)
        )

    draw_clouds()

    update_game()
    update_particles()

    for pipe in pipes:
        draw_pipe(pipe)

    draw_particles()
    draw_bird()
    draw_ground()
    draw_ui()

    pygame.display.flip()

    clock.tick(60)

pygame.quit()# flappy-bird-remake-
very interesting modern game 
