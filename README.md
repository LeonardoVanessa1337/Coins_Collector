
from threading import Timer
import pygame
from random import randint
from time import sleep

import asyncio

pygame.init()
 
Screen_Height = 1000
Screen_Width = 1000
 
window = pygame.display.set_mode((Screen_Height, Screen_Width))
clock = pygame.time.Clock()
 
class Sprite():
    def __init__(self, x, y, height, width, img):
        self.img = pygame.transform.scale(pygame.image.load(img), (width, height))
        self.rect = self.img.get_rect() # get the rectangle area by the img
 
        self.rect.x = x
        self.rect.y = y
 
    def draw(self):
        window.blit(self.img, (self.rect.x, self.rect.y))
 
 
 
 
class Player(Sprite):
    def __init__(self, x, y, height, width, img):
        super().__init__(x, y, height, width, img)

    def move(self):

        self.speed = 5
    
        keys = pygame.key.get_pressed()
        if keys[pygame.K_LEFT]:
            self.rect.x -= self.speed
        if keys[pygame.K_RIGHT]:
            self.rect.x += self.speed
        if keys[pygame.K_UP]:
            self.rect.y -= self.speed
        if keys[pygame.K_DOWN]:
            self.rect.y += self.speed
# Border Collision
        if self.rect.x <= 0 :
            self.rect.x = 0
        if self.rect.x >= Screen_Width - self.rect.width:
            self.rect.x = Screen_Width - self.rect.width
        if self.rect.y <= 0 :
            self.rect.y = 0
        if self.rect.y >= Screen_Height - self.rect.height:
            self.rect.y = Screen_Height - self.rect.height
    
 
 
class Collectable(Sprite):
        
    def __init__(self, x, y, height, width, img):
        super().__init__(x, y, height, width, img)
        
    def random_pos(self):
        # if self.rect.colliderect(Fox):
            self.rect.x = randint(0, Screen_Width - self.rect.width)
            self.rect.y = randint(0, Screen_Height - self.rect.height)

    def Power_up(self):
        if self.rect.colliderect(Fox):
            self.rect.x = randint(0, Screen_Width - self.rect.width)
            self.rect.y = randint(0, Screen_Height - self.rect.height)
            

class Text():
    def __init__(self, x, y, height, width, bg_color, text):
        self.rect = pygame.Rect(x, y, height, width)
        self.bg_color = bg_color
        self.text = text
    def set_text(self, updated_text):
        self.text = updated_text

    def draw(self):
        pygame.draw.rect(window, self.bg_color, self.rect)
        txt_font =  pygame.font.Font(None, 70)
        window.blit(txt_font.render(self.text, True, (0, 0, 0)) ,(self.rect.x + 10, self.rect.y + 50))
 
 
        
 
    
 
Fox = Player(500, 500, 100, 100, 'fox.png')

Coin = Collectable(randint(0, Screen_Width - 70) ,randint(0, Screen_Height - 70) , 70, 70, 'coin.png')
 
Coin1 = Collectable(randint(0, Screen_Width - 70) ,randint(0, Screen_Height - 70) , 70, 70, 'coin.png')

Bomb = Collectable(randint(0, Screen_Width - 90) ,randint(0, Screen_Height - 90) , 90, 90, 'Pixel Bomb.png')

Trap = Collectable(randint(0, Screen_Width - 90) ,randint(0, Screen_Height - 70) , 40, 110, 'BearTrap.png')

SandGlass = Collectable(randint(0, Screen_Width - 90) ,randint(0, Screen_Height - 70) , 90, 55, 'sandglass.png')

Point = 0
Point_Text = Text(0, 0, 1, 1, (56, 155, 70), 'Score: ' + str(Point))

timer = 10
timer_text = Text(700, 0, 1, 1, (56, 155, 70), 'Timer' + str(timer))

coin_timeout = 100
coin1_timeout = 100
trap_timeout = 300
bomb_timeout = 400
SandGlass_timeout = 50
 
Pausing = 0

#gameloop
begin = 0
finish = 0

is_running = True

async def main():
    global timer, begin, finish, Point, coin_timeout, coin1_timeout, trap_timeout, bomb_timeout, SandGlass_timeout, Pausing, is_running
    while True:
        if timer <= 1:
            is_running = False
            break
        else:
            timer -= finish - begin
            timer_text.set_text('Timer: ' + str(int(timer))) # set a new timer

        Point_Text.set_text('Score: ' + str(int(Point)))
        begin = pygame.time.get_ticks() / 1000 #in millisecond
        if Pausing == 0:
            # Coin timeout
            if coin_timeout <= 0: # It's time to change position
                Coin.random_pos()
                coin_timeout = 100 # Start a new timer
            else:
                coin_timeout -= 1

            # Coin1 timeout
            if coin1_timeout <= 0: # It's time to change position
                Coin1.random_pos()
                coin1_timeout = 100 # Start a new timer
            else:
                coin1_timeout -= 1

            # Trap timeout
            if trap_timeout <= 0: # It's time to change position
                Trap.random_pos()
                Bomb.random_pos()
                trap_timeout = 200 # Start a new timer
            else:
                trap_timeout -= 1
            
            # Bomb timeout
            if bomb_timeout <= 0: # It's time to change position
                Bomb.random_pos()
                bomb_timeout = 200 # Start a new timer
            else:
                bomb_timeout -= 1

            # SandGlass timeout
            if SandGlass_timeout <= 0: # It's time to change position
                SandGlass.random_pos()
                SandGlass_timeout = 50 # Start a new timer
            else:
                SandGlass_timeout -= 1

            for e in pygame.event.get():
                if e.type == pygame.QUIT:
                    pygame.quit()
                    exit()

            if Fox.rect.colliderect(Coin.rect):
                Point += 1
                Coin.random_pos()
                coin_timeout = 100

            # if Coin.rect.colliderect(Fox.rect):
            #     Point += 1
            if Coin1.rect.colliderect(Fox.rect):
                Coin1.random_pos()
                Point += 1
                coin1_timeout = 100
            if Bomb.rect.colliderect(Fox.rect):
                Bomb.random_pos()
                Point -= 5
                bomb_timeout = 400

            if SandGlass.rect.colliderect(Fox.rect):
                SandGlass.random_pos()
                timer += 1
                SandGlass_timeout = 50

            if Trap.rect.colliderect(Fox.rect):
                Pausing = 100
                # Fox.rect.x = Fox.rect.x - Fox.speed * 2
                # Fox.rect.y = Fox.rect.y - Fox.speed * 2
                Trap.random_pos()
                trap_timeout = 300
            Fox.move()
        else:
            Pausing -= 1
    
        window.fill((56, 155, 70))
        # Coin.random_pos()
        Coin.draw()
    
    
        # Coin1.random_pos()
        Coin1.draw()
    
        # Bomb.random_pos()
        Bomb.draw()

        # Trap.random_pos()
        Trap.draw()

        SandGlass.draw()

        Fox.draw()
        

        Point_Text.set_text('Score: ' + str(Point))
        Point_Text.draw()
        timer_text.draw()

        finish = pygame.time.get_ticks() / 1000 #in millisecond

        pygame.display.update()
        clock.tick(60)
        await asyncio.sleep(0)

    txt_font =  pygame.font.SysFont('comicsansms', 50)

    if Point > 3:
        window.fill((0, 255, 51))
        window.blit(txt_font.render('YOU WIN! Your score is ' + str(Point), True, (0, 0, 0)) ,(250, 240))
    else:
        window.fill((255, 0, 0))
        window.blit(txt_font.render('Close to winning! Your score is ' + str(Point), True, (0, 0, 0)) ,(150, 240))


    pygame.display.update()
    await asyncio.sleep(0)
    


asyncio.run(main())


#Result = ''

# while True:
#     for e in pygame.event.get():
#         if e.type == pygame.QUIT:
#             pygame.quit()
#             exit()
    

#     # Result (Win or Lose):

    

#     txt_font =  pygame.font.SysFont('comicsansms', 50)

#     if Point > 3:
#         window.fill((0, 255, 51))
#         window.blit(txt_font.render('YOU WIN! Your score is ' + str(Point), True, (0, 0, 0)) ,(250, 240))
#     else:
#         window.fill((255, 0, 0))
#         window.blit(txt_font.render('Close to winning! Your score is ' + str(Point), True, (0, 0, 0)) ,(150, 240))


#     pygame.display.update()

# #hand running: run  the code without the need of computer or in your mind
