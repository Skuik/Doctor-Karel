"""
DOCTOR KAREL has to get rid of all the virus.
Use the arrow keys to move right or left
With the space bar you shoot your enemies.
The virus are randomly generated same as their direction and rotation.
You have 3 lives for the game represented by 3 little items on the top right corner
of the screen.
Your life bar is on the top left. Every time you got hit by a virus you loose some life points

Enjoy the game !
"""

import pygame
import random
from os import path


# Constants for the Screen
WIDTH = 800
HEIGHT = 600

# Frame speed per second
FPS = 30

# Power up time in milliseconds (5s)
POWERUP_TIME = 5000

# define colors
WHITE = (255, 255, 255)
BLACK = (0, 0, 0)
GREEN = (0, 255, 0)


######### INITIALIZE PYGAME AND CREATE WINDOW #############
pygame.init()
pygame.mixer.init()
screen = pygame.display.set_mode((WIDTH, HEIGHT))
pygame.display.set_caption("Go Karel!")
clock = pygame.time.Clock()


############### CREATING CLASSES FOR THE GAME ##########################

### PLAYER ###
class Player(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image = pygame.transform.scale(player_img, (78, 79))
        self.image.set_colorkey()
        self.rect = self.image.get_rect()
        self.radius = 20
        self.rect.centerx = WIDTH / 2
        self.rect.bottom = HEIGHT - 10
        self.speedx = 0
        self.shield = 100
        self.shoot_delay = 250
        self.last_shot = pygame.time.get_ticks()
        self.lives = 3
        self.hidden = False
        self.hide_timer = pygame.time.get_ticks()
        self.power = 1
        self.power_time = pygame.time.get_ticks()

    def update(self):
        # timeout for powerups
        if self.power >= 2 and pygame.time.get_ticks() - self.power_time > POWERUP_TIME:
            self.power -= 1
            self.power_time = pygame.time.get_ticks()

        # unhide if hidden
        if self.hidden and pygame.time.get_ticks() - self.hide_timer > 1000:
            self.hidden = False
            self.rect.centerx = WIDTH / 2
            self.rect.bottom = HEIGHT - 10

        self.speedx = 0
        keystate = pygame.key.get_pressed()
        if keystate[pygame.K_LEFT]:
            self.speedx = -8
        if keystate[pygame.K_RIGHT]:
            self.speedx = 8
        if keystate[pygame.K_SPACE]:
            self.shoot()
        self.rect.x += self.speedx
        if self.rect.right > WIDTH:
            self.rect.right = WIDTH
        if self.rect.left < 0:
            self.rect.left = 0

    def powerup(self):
        self.power += 1
        self.power_time = pygame.time.get_ticks()

    def shoot(self):
        now = pygame.time.get_ticks()
        if now - self.last_shot > self.shoot_delay:
            self.last_shot = now
            if self.power == 1:
                bullet = Bullet(self.rect.centerx, self.rect.top)
                all_sprites.add(bullet)
                bullets.add(bullet)
                shoot_sound.play()
            if self.power >= 2:
                bullet1 = Bullet(self.rect.left, self.rect.centery)
                bullet2 = Bullet(self.rect.right, self.rect.centery)
                all_sprites.add(bullet1)
                all_sprites.add(bullet2)
                bullets.add(bullet1)
                bullets.add(bullet2)
                shoot_sound.play()

    def hide(self):
        # hide the player temporarily
        self.hidden = True
        self.hide_timer = pygame.time.get_ticks()
        self.rect.center = (WIDTH / 2, HEIGHT + 200)


### VIRUS ###
class Virus(pygame.sprite.Sprite):
    def __init__(self):
        pygame.sprite.Sprite.__init__(self)
        self.image_orig = random.choice(virus_images)
        self.image_orig.set_colorkey()
        self.image = self.image_orig.copy()
        self.rect = self.image.get_rect()
        self.radius = int(self.rect.width * .85 / 2)
        self.rect.x = random.randrange(WIDTH - self.rect.width)
        self.rect.bottom = random.randrange(-80, -20)
        self.speedy = random.randrange(1, 8)
        self.speedx = random.randrange(-3, 3)
        self.rot = 0
        self.rot_speed = random.randrange(-8, 8)
        self.last_update = pygame.time.get_ticks()

    def rotate(self):
        now = pygame.time.get_ticks()
        if now - self.last_update > 50:
            self.last_update = now
            self.rot = (self.rot + self.rot_speed) % 360
            new_image = pygame.transform.rotate(self.image_orig, self.rot)
            old_center = self.rect.center
            self.image = new_image
            self.rect = self.image.get_rect()
            self.rect.center = old_center

    def update(self):
        self.rotate()
        self.rect.x += self.speedx
        self.rect.y += self.speedy
        if self.rect.top > HEIGHT + 10 or self.rect.left < -100 or self.rect.right > WIDTH + 100:
            self.rect.x = random.randrange(WIDTH - self.rect.width)
            self.rect.y = random.randrange(-100, -40)
            self.speedy = random.randrange(1, 8)


### BULLETS ###
class Bullet(pygame.sprite.Sprite):
    def __init__(self, x, y):
        pygame.sprite.Sprite.__init__(self)
        self.image = bullet_img
        self.rect = self.image.get_rect()
        self.rect.bottom = y
        self.rect.centerx = x
        self.speedy = -10

    def update(self):
        self.rect.y += self.speedy
        # kill if it moves off the top of the screen
        if self.rect.bottom < 0:
            self.kill()


### SPECIAL POWER (BONUS) ###
class Power(pygame.sprite.Sprite):
    def __init__(self, center):
        pygame.sprite.Sprite.__init__(self)
        self.type = random.choice(['mask', 'gel'])
        self.image = powerup_images[self.type]
        self.image.set_colorkey()
        self.rect = self.image.get_rect()
        self.rect.center = center
        self.speedy = 5

    def update(self):
        self.rect.y += self.speedy
        # kill if it moves off the top of the screen
        if self.rect.top > HEIGHT:
            self.kill()


### EXPLOSIONS ###
class Explosion(pygame.sprite.Sprite):
    def __init__(self, center, size):
        pygame.sprite.Sprite.__init__(self)
        self.size = size
        self.image = explosion_anim[self.size][0]
        self.rect = self.image.get_rect()
        self.rect.center = center
        self.frame = 0
        self.last_update = pygame.time.get_ticks()
        self.frame_rate = 75

    def update(self):
        now = pygame.time.get_ticks()
        if now - self.last_update > self.frame_rate:
            self.last_update = now
            self.frame += 1
            if self.frame == len(explosion_anim[self.size]):
                self.kill()
            else:
                center = self.rect.center
                self.image = explosion_anim[self.size][self.frame]
                self.rect = self.image.get_rect()
                self.rect.center = center


######### FONCTIONS FOR OUR GAME #############
# every time we'll hit a virus, we'll create a new one
def new_virus():
    v = Virus()
    all_sprites.add(v)
    virus.add(v)


# CREATE LIFE BAR
def draw_life_bar(surf, x, y, pct):
    if pct < 0:
        pct = 0
    BAR_LENGTH = 100
    BAR_HEIGHT = 10
    fill = (pct / 100) * BAR_LENGTH
    outline_rect = pygame.Rect(x, y, BAR_LENGTH, BAR_HEIGHT)
    fill_rect = pygame.Rect(x, y, fill, BAR_HEIGHT)
    pygame.draw.rect(surf, GREEN, fill_rect)
    pygame.draw.rect(surf, WHITE, outline_rect, 2)


# DRAW LIVES ICONS ON THE SCREEN
def draw_lives(surf, x, y, lives, img):
    for i in range(lives):
        img_rect = img.get_rect()
        img_rect.x = x + 30 * i
        img_rect.y = y
        surf.blit(img, img_rect)


############# TEXT FOR THE INTRO ##############
font_name = pygame.font.match_font('comicsansms')

def draw_text(surf, text, size, x, y):
    font = pygame.font.Font(font_name, size)
    text_surface = font.render(text, True, BLACK)
    text_rect = text_surface.get_rect()
    text_rect.midtop = (x, y)
    surf.blit(text_surface, text_rect)


def show_go_screen():
    screen.blit(background, background_rect)
    draw_text(screen, "DOCTOR KAREL!", 50, WIDTH / 2, 10)
    draw_text(screen, "Get rid of the virus spread", 25, WIDTH / 2, 70)
    draw_text(screen, "Use Arrows to move, Space to fire", 22, WIDTH / 2, HEIGHT * 5/8)
    draw_text(screen, "Press any key to begin", 18, WIDTH / 2, HEIGHT -50)
    pygame.display.flip()
    waiting = True
    while waiting:
        clock.tick(FPS)
        for event in pygame.event.get():
            if event.type == pygame.QUIT:
                pygame.quit()
            if event.type == pygame.KEYUP:
                waiting = False


###################  LOADING ALL GRAPHICS FOR THE GAME   #####################
# To avoid any problem while downloading we call the path
img_dir = path.join(path.dirname(__file__), 'img')
snd_dir = path.join(path.dirname(__file__), 'snd')

# Background
background = pygame.image.load(path.join(img_dir, "bg800.png")).convert()
background_rect = background.get_rect()

# Player
player_img = pygame.image.load(path.join(img_dir, "300K.png")).convert_alpha()
player_mini_img = pygame.transform.scale(player_img, (20, 19))

# Bullets
original_bullet_img = pygame.image.load(path.join(img_dir, "syringe.png")).convert_alpha()
bullet_img = pygame.transform.scale(original_bullet_img, (22, 46))

# Powers
original_mask_img = pygame.image.load(path.join(img_dir, 'mask.png')).convert_alpha()
mask_img = pygame.transform.scale(original_mask_img, (70, 40))

original_gel_img = pygame.image.load(path.join(img_dir, 'alcohol-gel.png')).convert_alpha()
gel_img = pygame.transform.scale(original_gel_img, (23, 58))

powerup_images = {}
powerup_images['mask'] = mask_img
powerup_images['gel'] = gel_img

# Virus
red = pygame.image.load(path.join(img_dir, 'red_virus.png')).convert_alpha()
red_img = pygame.transform.scale(red, (54, 41))

blue = pygame.image.load(path.join(img_dir, 'blue_virus.png')).convert_alpha()
blue_img = pygame.transform.scale(blue, (48, 48))

fun = pygame.image.load(path.join(img_dir, 'funfun.png')).convert_alpha()
fun_img = pygame.transform.scale(fun, (50, 44))

coro = pygame.image.load(path.join(img_dir, 'Coronavirus-Germs-PNG-File.png')).convert_alpha()
coro_img = pygame.transform.scale(coro, (37, 39))

bad = pygame.image.load(path.join(img_dir, 'badbad.png')).convert_alpha()
bad_img = pygame.transform.scale(bad, (47, 46))

# Create a list with Virus sprites
virus_images = []
virus_list = [red_img, blue_img, fun_img, bad_img, coro_img]
for img in virus_list:
    virus_images.append(img)

# Explosions
explosion_anim = {}
explosion_anim['lg'] = []
explosion_anim['sm'] = []
explosion_anim['player'] = []
for i in range(9):
    filename = 'regularExplosion0{}.png'.format(i)
    img = pygame.image.load(path.join(img_dir, filename)).convert()
    img.set_colorkey(BLACK)
    img_lg = pygame.transform.scale(img, (75, 75))
    explosion_anim['lg'].append(img_lg)
    img_sm = pygame.transform.scale(img, (32, 32))
    explosion_anim['sm'].append(img_sm)
    filename = 'sonicExplosion0{}.png'.format(i)
    img = pygame.image.load(path.join(img_dir, filename)).convert()
    img.set_colorkey(BLACK)
    explosion_anim['player'].append(img)


################### LOADING SOUND EFFECT ########################

shoot_sound = pygame.mixer.Sound(path.join(snd_dir, 'Laser Blasts-SoundBible.com-108608437.wav'))

shield_sound = pygame.mixer.Sound(path.join(snd_dir, 'Lightsaber Turn On-SoundBible.com-1637663395.wav'))

power_sound = pygame.mixer.Sound(path.join(snd_dir, 'R2D2-SoundBible.com-460754772.wav'))

expl_sounds = []
for snd in ['Blast.wav', 'expl6.wav']:
    expl_sounds.append(pygame.mixer.Sound(path.join(snd_dir, snd)))

player_die_sound = pygame.mixer.Sound(path.join(snd_dir, 'wilhem.wav'))


# pygame.mixer.music.load(path.join(snd_dir, 'your_sound_file.ogg'))
# pygame.mixer.music.set_volume(0.3)
# pygame.mixer.music.play(loops=-1)

################ GAME LOOP ####################
game_over = True
running = True
while running:
    if game_over:
        show_go_screen()
        game_over = False
        all_sprites = pygame.sprite.Group()
        virus = pygame.sprite.Group()
        bullets = pygame.sprite.Group()
        powerups = pygame.sprite.Group()
        player = Player()
        all_sprites.add(player)
        for i in range(6):
            new_virus()
        score = 0

    # to keep loop running at the right speed
    clock.tick(FPS)
    # Process input (events)
    for event in pygame.event.get():
        # check for closing window
        if event.type == pygame.QUIT:
            running = False

# Update the screen
    all_sprites.update()

# check to see if a bullet hit a virus
    hits = pygame.sprite.groupcollide(virus, bullets, True, True)
    for hit in hits:
        score += 50 - hit.radius
        random.choice(expl_sounds).play()
        expl = Explosion(hit.rect.center, 'lg')
        all_sprites.add(expl)
        if random.random() > 0.9:
            pow = Power(hit.rect.center)
            all_sprites.add(pow)
            powerups.add(pow)
        new_virus()

# check to see if a virus hit the player
    hits = pygame.sprite.spritecollide(player, virus, True, pygame.sprite.collide_circle)
    for hit in hits:
        player.shield -= hit.radius * 2
        expl = Explosion(hit.rect.center, 'sm')
        all_sprites.add(expl)
        new_virus()
        if player.shield <= 0:
            player_die_sound.play()
            death_explosion = Explosion(player.rect.center, 'player')
            all_sprites.add(death_explosion)
            player.hide()
            player.lives -= 1
            player.shield = 100

# check to see if player hit a bonus power
    hits = pygame.sprite.spritecollide(player, powerups, True)
    for hit in hits:
        if hit.type == 'mask':
            player.shield += random.randrange(10, 30)
            shield_sound.play()
            if player.shield >= 100:
                player.shield = 100
        if hit.type == 'gel':
            player.powerup()
            power_sound.play()

# if the player died and the explosion has finished playing
    if player.lives == 0 and not death_explosion.alive():
        game_over = True

### Screen render ###
    screen.fill(BLACK)
    screen.blit(background, background_rect)
    all_sprites.draw(screen)
    draw_text(screen, str(score), 18, WIDTH / 2, 10)
    draw_life_bar(screen, 5, 5, player.shield)
    draw_lives(screen, WIDTH - 100, 5, player.lives, player_mini_img)

    pygame.display.flip()

pygame.quit()
