import pygame
import sys

# Initialize Pygame
pygame.init()

# Constants
SCREEN_WIDTH, SCREEN_HEIGHT = 800, 600
FPS = 70

# Colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)

# Game Settings
background_image_path = "background.jpg"  # Your background image file path
player_image_path = "player.png"  # Your character sprite file path
background_music_path = "background_music.mp3"  # Background music file path

# Set up screen
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Simple RPG Adventure")

# Load images
background = pygame.image.load(background_image_path)
player_img = pygame.image.load(player_image_path)

# Adjust the size of the character sprite 
player_img = pygame.transform.scale(player_img, (90, 90))  

# Create player rect for collision and positioning
player_rect = player_img.get_rect(center=(SCREEN_WIDTH // 2, SCREEN_HEIGHT // 2))

# Initialize sound effects and music
pygame.mixer.init()  # Initialize the mixer for sound
pygame.mixer.music.load(background_music_path)  # Load background music
pygame.mixer.music.set_volume(1.0)  # Set the background music volume (0.0 to 1.0)

# Play background music
pygame.mixer.music.play(-1, 0.0)  # -1 means loop forever, 0.0 means start immediately

# Font for button
font = pygame.font.SysFont("Arial", 30)

# Button Class
class Button:
    def __init__(self, text, x, y, width, height, action=None):
        self.text = text
        self.rect = pygame.Rect(x, y, width, height)
        self.action = action

    def draw(self, screen):
        pygame.draw.rect(screen, (0, 0, 0), self.rect)
        label = font.render(self.text, True, (255, 255, 255))
        screen.blit(label, (self.rect.x + (self.rect.width - label.get_width()) // 2, 
                            self.rect.y + (self.rect.height - label.get_height()) // 2))

    def is_pressed(self, event):
        if self.rect.collidepoint(event.pos):
            if self.action:
                self.action()

# Player Class
class Player:
    def __init__(self):
        self.rect = player_rect
        self.speed = 5

    def move(self, keys):
        if keys[pygame.K_LEFT]:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT]:
            self.rect.x += self.speed
        if keys[pygame.K_UP]:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN]:
            self.rect.y += self.speed

# Function to draw the repeating background
def draw_background(camera_x, camera_y):
    # Background image width and height
    bg_width, bg_height = background.get_width(), background.get_height()

    # Draw the background image multiple times to cover the entire screen
    for x in range(0, SCREEN_WIDTH, bg_width):
        for y in range(0, SCREEN_HEIGHT, bg_height):
            screen.blit(background, (x - camera_x, y - camera_y))

# Main game loop
def game_loop():
    clock = pygame.time.Clock()
    player = Player()
    camera_x, camera_y = 0, 0

    # Start button
    start_button = Button("Start Game", 300, 250, 200, 50, start_game)

    # Main loop
    while True:
        screen.fill(WHITE)
        
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()

            if event.type == pygame.MOUSEBUTTONDOWN:
                start_button.is_pressed(event)

        # Draw start button in the menu screen
        start_button.draw(screen)

        # If game has started, draw the game screen
        if game_state == "playing":
            # Move the player
            keys = pygame.key.get_pressed()
            player.move(keys)

            # Draw the repeating background to simulate infinite scrolling
            draw_background(camera_x, camera_y)

            # Draw player in the center of the screen
            screen.blit(player_img, (SCREEN_WIDTH // 2 - player.rect.width // 2, SCREEN_HEIGHT // 2 - player.rect.height // 2))

            # Camera follows the player
            camera_x = player.rect.centerx - SCREEN_WIDTH // 2
            camera_y = player.rect.centery - SCREEN_HEIGHT // 2

        # Update display
        pygame.display.flip()
        clock.tick(FPS)

# Start game
def start_game():
    global game_state
    game_state = "playing"
    game_loop()

# Start the game
game_state = "menu"
game_loop()
