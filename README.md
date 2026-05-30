import pygame
import sys
import random

# Ініціалізація Pygame
pygame.init()

# Налаштування екрану
SCREEN_WIDTH = 800
SCREEN_HEIGHT = 500
screen = pygame.display.set_surface(pygame.display.set_mode((SCREEN_WIDTH, SCREEN_HEIGHT)))
pygame.display.set_caption("Geometry Dash Clone (Python)")

# Кольори
BG_COLOR = (20, 20, 40)      # Темно-синій фон
NEON_BLUE = (0, 191, 255)    # Колір гравця
NEON_GREEN = (57, 255, 20)   # Колір підлоги
NEON_RED = (255, 7, 58)      # Колір шипів
WHITE = (255, 255, 255)

# Фізика та ігрові параметри
GRAVITY = 0.8
FLOOR_Y = SCREEN_HEIGHT - 100
GAME_SPEED = 7

class Player:
    def __init__(self):
        self.size = 40
        self.x = 150
        self.y = FLOOR_Y - self.size
        self.velocity_y = 0
        self.is_jumping = False
        self.angle = 0

    def jump(self):
        if not self.is_jumping:
            self.velocity_y = -14
            self.is_jumping = True

    def update(self):
        # Гравітація
        self.velocity_y += GRAVITY
        self.y += self.velocity_y

        # Перевірка зіткнення з підлогою
        if self.y >= FLOOR_Y - self.size:
            self.y = FLOOR_Y - self.size
            self.velocity_y = 0
            self.is_jumping = False
            # Вирівнювання кута при приземленні (кратне 90 градусам)
            self.angle = (self.angle + 45) // 90 * 90

        # Анімація обертання під час стрибка
        if self.is_jumping:
            self.angle -= 5  # Обертання в повітрі

    def draw(self, surface):
        # Створення ефекту обертання куба
        cube_surface = pygame.Surface((self.size, self.size), pygame.SRCALPHA)
        pygame.draw.rect(cube_surface, NEON_BLUE, (0, 0, self.size, self.size))
        pygame.draw.rect(cube_surface, WHITE, (5, 5, self.size - 10, self.size - 10), 3) # Внутрішній квадрат
        
        rotated_surface = pygame.transform.rotate(cube_surface, self.angle)
        new_rect = rotated_surface.get_rect(center=(self.x + self.size // 2, self.y + self.size // 2))
        surface.blit(rotated_surface, new_rect.topleft)


class Obstacle:
    def __init__(self, x):
        self.width = 40
        self.height = 40
        self.x = x
        self.y = FLOOR_Y

    def update(self):
        self.x -= GAME_SPEED

    def draw(self, surface):
        # Малюємо шип (трикутник)
        points = [
            (self.x, self.y),
            (self.x + self.width, self.y),
            (self.x + self.width // 2, self.y - self.height)
        ]
        pygame.draw.polygon(surface, NEON_RED, points)

    def get_hitbox(self):
        # Хітбокс трохи менший за графіку для чесності гри
        return pygame.Rect(self.x + 5, self.y - self.height + 5, self.width - 10, self.height - 5)


def main():
    clock = pygame.time.Clock()
    player = Player()
    obstacles = []
    score = 0

    # Таймер для генерації перешкод
    obstacle_timer = 0

    running = True
    while running:
        clock.tick(60) # 60 FPS
        screen.fill(BG_COLOR)

        # Обробка подій (Кліки / Кнопки)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                running = False
                pygame.quit()
                sys.exit()
            if event.type == pygame.KEYDOWN:
                if event.key == pygame.K_SPACE or event.key == pygame.K_UP:
                    player.jump()

        # Оновлення гравця
        player.update()

        # Генерація перешкод (випадкові інтервали)
        obstacle_timer -= 1
        if obstacle_timer <= 0:
            if random.random() < 0.3:  # Шанс появи
                obstacles.append(Obstacle(SCREEN_WIDTH + random.randint(0, 200)))
                obstacle_timer = 90  # Затримка між шипами
            else:
                obstacle_timer = 20

        # Оновлення та перевірка перешкод
        player_rect = pygame.Rect(player.x, player.y, player.size, player.size)
        
        for obstacle in obstacles[:]:
            obstacle.update()
            
            # Перевірка на програш (колізія)
            if player_rect.colliderect(obstacle.get_hitbox()):
                # Рестарт гри при зіткненні
                player = Player()
                obstacles.clear()
                score = 0
                break

            # Видалення шипів, які вилетіли за екран
            if obstacle.x < -obstacle.width:
                obstacles.remove(obstacle)
                score += 1

        # Малюємо підлогу
        pygame.draw.line(screen, NEON_GREEN, (0, FLOOR_Y), (SCREEN_WIDTH, FLOOR_Y), 5)
        pygame.draw.rect(screen, (10, 10, 20), (0, FLOOR_Y + 3, SCREEN_WIDTH, SCREEN_HEIGHT - FLOOR_Y))

        # Малюємо об'єкти
        player.draw(screen)
        for obstacle in obstacles:
            obstacle.draw(screen)

        # Виведення рахунку
        font = pygame.font.SysFont("Arial", 24)
        score_text = font.render(f"Score: {score}", True, WHITE)
        screen.blit(score_text, (20, 20))

        pygame.display.flip()

if __name__ == "__main__":
    main()
