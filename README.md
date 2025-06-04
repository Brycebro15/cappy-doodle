import pygame
import sys
import random
import os

# Set working directory to script location
script_dir = os.path.dirname(os.path.abspath(sys.argv[0]))
os.chdir(script_dir)

pygame.init()

# Screen setup
WIDTH, HEIGHT = 400, 600
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Flappy Doodle")
clock = pygame.time.Clock()
font = pygame.font.SysFont(None, 40)

# Game variables
gravity = 0.25
bird_movement = 0
bird_rect = pygame.Rect(100, HEIGHT // 2, 34, 24)
floor_y = HEIGHT - 100
pipe_gap = 150
pipe_speed = 3
pipe_list = []
passed_pipes = []
score = 0
best_score = 0
game_active = True
game_over_screen = False
floor_x = 0

# Load images
bg_img = pygame.image.load("bg.png").convert()
ground_img = pygame.image.load("ground.png").convert()
pipe_up_img = pygame.image.load("pipe_up.png").convert_alpha()
pipe_down_img = pygame.image.load("pipe_down.png").convert_alpha()

# Load character images
doodle_img = pygame.image.load("doodle.png").convert_alpha()
doodle_img = pygame.transform.scale(doodle_img, (50, 40))

capybara_img = pygame.image.load("bird.png").convert_alpha()
capybara_img = pygame.transform.scale(capybara_img, (50, 40))

selected_character_img = None
character_selected = False

# Resize if needed
ground_img = pygame.transform.scale(ground_img, (WIDTH, 100))
pipe_width = pipe_up_img.get_width()

# Timer
SPAWNPIPE = pygame.USEREVENT
pygame.time.set_timer(SPAWNPIPE, 1200)

# Speed scaling
speed_increment_interval = 10
speed_increment_amount = 1
last_speed_update_score = 0

def draw_floor(x_pos):
    screen.blit(ground_img, (x_pos, floor_y))
    screen.blit(ground_img, (x_pos + WIDTH, floor_y))

def character_selection_screen():
    screen.fill((0, 0, 0))
    font = pygame.font.SysFont(None, 50)
    text = font.render("Select your character", True, (255, 255, 255))
    screen.blit(text, (WIDTH // 2 - text.get_width() // 2, 50))

    # Draw the characters to select
    screen.blit(doodle_img, (WIDTH // 4 - 25, HEIGHT // 2 - 20))
    screen.blit(capybara_img, (3 * WIDTH // 4 - 25, HEIGHT // 2 - 20))

    # Label below each character
    font_small = pygame.font.SysFont(None, 30)
    doodle_label = font_small.render("Doodle (Press 1)", True, (255, 255, 255))
    capybara_label = font_small.render("Capybara (Press 2)", True, (255, 255, 255))

    screen.blit(doodle_label, (WIDTH // 4 - doodle_label.get_width() // 2, HEIGHT // 2 + 40))
    screen.blit(capybara_label, (3 * WIDTH // 4 - capybara_label.get_width() // 2, HEIGHT // 2 + 40))

    pygame.display.update()

def create_pipe():
    height = random.randint(200, 400)
    
    bottom_pipe = {
        "image": pipe_up_img,
        "rect": pipe_up_img.get_rect(midtop=(WIDTH + 50, height))
    }

    top_pipe = {
        "image": pipe_down_img,
        "rect": pipe_down_img.get_rect(midbottom=(WIDTH + 50, height - pipe_gap))
    }

    return {"bottom": bottom_pipe, "top": top_pipe, "passed": False}

def move_pipes(pipes):
    for pipe_pair in pipes:
        pipe_pair["bottom"]["rect"].centerx -= pipe_speed
        pipe_pair["top"]["rect"].centerx -= pipe_speed
    return pipes

def draw_pipes(pipes):
    for pipe_pair in pipes:
        screen.blit(pipe_pair["bottom"]["image"], pipe_pair["bottom"]["rect"])
        screen.blit(pipe_pair["top"]["image"], pipe_pair["top"]["rect"])

def check_collision(pipes):
    for pipe_pair in pipes:
        if bird_rect.colliderect(pipe_pair["bottom"]["rect"]) or bird_rect.colliderect(pipe_pair["top"]["rect"]):
            return False
    if bird_rect.top <= -50 or bird_rect.bottom >= floor_y:
        return False
    return True

def show_score(score, best, pos=(10, 10)):
    score_surface = font.render(f"Score: {score}  Best: {best}", True, (255, 255, 255))
    screen.blit(score_surface, pos)

def show_game_over_screen():
    screen.fill((60, 100, 100))
    title = font.render("Bryce's Game", True, (255, 255, 255))
    score_text = font.render(f"Score: {score}", True, (255, 255, 255))
    best_text = font.render(f"Best Score: {best_score}", True, (255, 255, 255))
    restart = font.render("Press SPACE to Play Again", True, (0, 0, 0))
    reselect = font.render("Press C to Reselect Character", True, (0, 0, 0))
    screen.blit(title, (WIDTH // 2 - 100, 100))
    screen.blit(score_text, (WIDTH // 2 - 70, 200))
    screen.blit(best_text, (WIDTH // 2 - 90, 250))
    screen.blit(restart, (WIDTH // 2 - 160, 350))
    screen.blit(reselect, (WIDTH // 2 - 180, 400))
    pygame.display.update()

# Main game loop
# Main game loop
while True:
    for event in pygame.event.get():
        if event.type == pygame.QUIT:
            pygame.quit()
            sys.exit()

        if not character_selected:
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_1:
                    selected_character_img = doodle_img
                    character_selected = True
                    game_active = True
                elif event.key == pygame.K_2:
                    selected_character_img = capybara_img
                    character_selected = True
                    game_active = True

        else:
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    if game_active:
                        bird_movement = 0
                        bird_movement -= 6
                    elif game_over_screen:
                        # Restart game
                        game_active = True
                        game_over_screen = False
                        pipe_list.clear()
                        bird_rect.center = (100, HEIGHT // 2)
                        bird_movement = 0
                        score = 0
                        pipe_speed = 3
                        passed_pipes.clear()
                        last_speed_update_score = 0

                elif event.key == pygame.K_c and game_over_screen:
                    # Reselect character
                    character_selected = False
                    game_over_screen = False
                    game_active = False
                    pipe_list.clear()
                    bird_rect.center = (100, HEIGHT // 2)
                    bird_movement = 0
                    score = 0
                    pipe_speed = 3
                    passed_pipes.clear()
                    last_speed_update_score = 0
                    selected_character_img = None

            if event.type == SPAWNPIPE and game_active:
                pipe_list.append(create_pipe())

    if not character_selected:
        character_selection_screen()

    elif game_active:
        screen.blit(bg_img, (0, 0))

        # Bird
        bird_movement += gravity
        bird_rect.centery += bird_movement
        screen.blit(selected_character_img, bird_rect)

        # Pipes
        pipe_list = move_pipes(pipe_list)
        draw_pipes(pipe_list)

        # Collision
        game_active = check_collision(pipe_list)

        # Scoring
        for pipe_pair in pipe_list:
            if pipe_pair['bottom']['rect'].right < bird_rect.left and not pipe_pair['passed']:
                score += 1
                pipe_pair['passed'] = True

        # Speed increase
        if score >= last_speed_update_score + speed_increment_interval:
            pipe_speed += speed_increment_amount
            last_speed_update_score = score

        # Floor
        floor_x -= 1
        if floor_x <= -WIDTH:
            floor_x = 0
        draw_floor(floor_x)

        # Score
        show_score(score, best_score)

        pygame.display.update()
        clock.tick(60)

    else:  # Game over screen
        if not game_over_screen:
            if score > best_score:
                best_score = score
            game_over_screen = True
        show_game_over_screen()
