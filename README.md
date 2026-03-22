# Ping-pong-game
from pygame import *
from random import randint
from time import time as timer

#font and caption
font.init()
font1 = font.Font(None,80)
font2 = font.Font(None,36)
win = font1.render("YOU WIN", 1 ,(0,255,0))
lose = font1.render("YOU LOSE", 1 ,(255,0,0))

#background music
mixer.init()
mixer.music.load('space.ogg')
mixer.music.play()
fire_sound = mixer.Sound('fire.ogg')

#create following image
img_bullet = 'bullet.png'
img_enemy = 'ufo.png'
img_background = 'galaxy.jpg'
img_astro = 'hero.png'

#create value
score = 0
lost = 0
max_lost = 3
goal = 10
live = 3


#create background
win_width = 700
win_height = 500
display.set_caption("Shooter")
window = display.set_mode((win_width,win_height))
background = transform.scale(image.load(img_background),(win_width,win_height))

class GameSprite(sprite.Sprite):
    def __init__(self, player_image, player_x, player_y, size_x, size_y, player_speed):
        sprite.Sprite.__init__(self)
        self.image = transform.scale(image.load(player_image), (size_x, size_y))
        self.speed = player_speed
        self.rect = self.image.get_rect()
        self.rect.x = player_x
        self.rect.y = player_y
    def reset(self):
        window.blit(self.image, (self.rect.x, self.rect.y))
class Player(GameSprite):
    def update(self):
      keys = key.get_pressed()
      if keys[K_LEFT] and self.rect.x > 5:
        self.rect.x -= self.speed
      if keys[K_RIGHT] and self.rect.x < win_width - 100:
        self.rect.x += self.speed
    def fire(self):
       bullet = Bullet(img_bullet, self.rect.centerx, self.rect.top,25,20,-15)
       bullets.add(bullet)
class Enemy(GameSprite):
   def update(self):
      self.rect.y += self.speed
      global lost
      if self.rect.y > win_height:
         self.rect.x = randint(80,win_width-80)
         self.rect.y = 0
         lost = lost + 1
class Bullet(GameSprite):
   def update(self):
      self.rect.y += self.speed
      if self.rect.y < 0:
         self.kill()

bullets = sprite.Group()
monsters = sprite.Group()
asteroids = sprite.Group()
for i in range(1,6):
   monster = Enemy(img_enemy,randint(80,win_width-80),-40,80,50,randint(1,5))
   monsters.add(monster)
for i in range(1,3):
   asteroid = Enemy(img_astro,randint(30,win_width-30),-40,80,50,randint(1,7))
   asteroids.add(asteroid)
rocket = Player("rocket.png", 5, win_height - 100, 80, 100, 10)

num_file = 0
real_time = False

finish = False
run = True
while run:
    for e in event.get():
      if e.type == QUIT:
        run = False
      elif e.type == KEYDOWN:
         if e.key == K_SPACE:
            if num_file < 5 and real_time == False:
               num_file += 1
               fire_sound.play()
               rocket.fire()
            if num_file >= 5 and real_time == False:
               last_time = timer()
               real_time = True
    if finish != True:
      window.blit(background,(0,0))
      rocket.update()
      rocket.reset()
      asteroids.update()
      asteroids.draw(window)
      monsters.update()
      monsters.draw(window)
      bullets.update()
      bullets.draw(window)
    if real_time == True:
         now_time = timer()
         if now_time - last_time < 3:
            reload = font2.render('wait reload',1,(150,0,0))
            window.blit(reload,(260,460))
         else:
            now_time = 0
            real_time = False
    collides = sprite.groupcollide(monsters,bullets,True,True)
      for c in collides:
         score += 1
         monster = Enemy(img_enemy,randint(80,win_width-80),-40,80,50,randint(1,5))
         monsters.add(monster)
      if sprite.spritecollide(rocket,monsters,False) or sprite.spritecollide(rocket,asteroids,False):
         sprite.spritecollide(rocket,monsters,True)
         sprite.spritecollide(rocket,asteroids,True)
         live -= 1
      if live == 0 or lost >= max_lost:
         finish = True
         window.blit(lose,(200,200))
      if score >= goal:
         finish = True
         window.blit(win,(200,200))
      textlost = font2.render('lost: ' + str(lost), 1,(255,255,255))
      textscore = font2.render('score: ' + str(score), 1,(255,255,255))
      window.blit(textscore,(10,20))
      window.blit(textlost,(10,50))
      if live == 3:
         live_color = (0,150,0)
      if live == 2:
         live_color = (150,150,0)
      if live == 1:
         live_color = (150,0,0)
      text_live = font1.render(str(live), 1, live_color)
      window.blit(text_live,(650,10))
      display.update()
    else:
      finish = False
      score = 0
      lost = 0
      num_file = 0
      live = 3
      for a in asteroids:
         a.kill()
      for b in bullets:
         b.kill()
      for m in monsters:
         m.kill()
      time.delay(3000)
      for i in range(1,6):
         monster = Enemy(img_enemy,randint(80,win_width-80),-40,80,50,randint(1,5))
         monsters.add(monster)
      for i in range(1,3):
         asteroid = Enemy(img_astro,randint(30,win_width-30),-40,80,50,randint(1,7))
         asteroids.add(asteroid)
    time.delay(50)
