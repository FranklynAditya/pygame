import pygame
import sys
import random

# Inisialisasi Pygame
pygame.init()

# Ukuran layar
SCREEN_WIDTH = 400
SCREEN_HEIGHT = 600

# Warna
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
RED = (255, 0, 0)
GREEN = (0, 255, 0)
BLUE = (0, 0, 255)

# Membuat layar
screen = pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT))
pygame.display.set_caption("Flappy Bird")

# Frame rate
clock = pygame.time.Clock()
FPS = 60

# Variabel game
GRAVITY = 0.5
JUMP_STRENGTH = -10
PIPE_GAP = 150
PIPE_WIDTH = 50
PIPE_SPEED = 3

# Font
font = pygame.font.Font(None, 36)

def draw_text(text, font, color, surface, x, y):
    text_obj = font.render(text, True, color)
    text_rect = text_obj.get_rect()
    text_rect.topleft = (x, y)
    surface.blit(text_obj, text_rect)

# Kelas Burung
class Bird:
    def __init__(self):
        self.x = 50
        self.y = SCREEN_HEIGHT // 2
        self.radius = 15
        self.velocity = 0

    def update(self):
        self.velocity += GRAVITY
        self.y += self.velocity

    def jump(self):
        self.velocity = JUMP_STRENGTH

    def draw(self):
        pygame.draw.circle(screen, RED, (self.x, int(self.y)), self.radius)

# Kelas Pipa
class Pipe:
    def __init__(self, x):
        self.x = x
        self.top_height = random.randint(50, SCREEN_HEIGHT - PIPE_GAP - 50)
        self.bottom_height = SCREEN_HEIGHT - self.top_height - PIPE_GAP

    def update(self):
        self.x -= PIPE_SPEED

    def draw(self):
        pygame.draw.rect(screen, GREEN, (self.x, 0, PIPE_WIDTH, self.top_height))
        pygame.draw.rect(screen, GREEN, (self.x, SCREEN_HEIGHT - self.bottom_height, PIPE_WIDTH, self.bottom_height))

    def is_off_screen(self):
        return self.x + PIPE_WIDTH < 0

# Fungsi utama

def main():
    bird = Bird()
    pipes = [Pipe(SCREEN_WIDTH)]
    score = 0

    running = True
    while running:
        screen.fill(WHITE)

        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE:
                    bird.jump()

        # Update burung
        bird.update()

        # Update pipa
        for pipe in pipes:
            pipe.update()
            if pipe.x == bird.x:
                score += 1
        
        # Tambahkan pipa baru jika perlu
        if pipes[-1].x < SCREEN_WIDTH - 200:
            pipes.append(Pipe(SCREEN_WIDTH))

        # Hapus pipa yang keluar dari layar
        pipes = [pipe for pipe in pipes if not pipe.is_off_screen()]

        # Gambar burung dan pipa
        bird.draw()
        for pipe in pipes:
            pipe.draw()

        # Tabrakan
        for pipe in pipes:
            if (bird.x + bird.radius > pipe.x and bird.x - bird.radius < pipe.x + PIPE_WIDTH):
                if (bird.y - bird.radius < pipe.top_height or bird.y + bird.radius > SCREEN_HEIGHT - pipe.bottom_height):
                    running = False

        # Tabrakan dengan tanah atau langit
        if bird.y - bird.radius < 0 or bird.y + bird.radius > SCREEN_HEIGHT:
            running = False

        # Tampilkan skor
        draw_text(f"Score: {score}", font, BLACK, screen, 10, 10)

        # Perbarui layar
        pygame.display.flip()
        clock.tick(FPS)

if __name__ == "__main__":
    main()
