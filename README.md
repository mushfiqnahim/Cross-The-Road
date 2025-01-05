# Cross-The-Road
It's a 2D graphical game using OpenGL and Python, incorporating features like movement, collision detection, and power-ups (stars and shields).A controller(player) will have to pass some line to get to reach other side.There's some obstacle in the road, if the player collide with those obstacle the game will be over.The game has some level according to player's score.
from OpenGL.GL import *
from OpenGL.GLUT import *
from OpenGL.GLU import *
from OpenGL.GL import glColor3f
import random
import time

obstacles={'stop':False}
obs_flag=True
player_pos=[400,20] #x,y
lives=3
score=0
shield_active = False
shield_start_time = 0
shield_count = 2

#Green car variables
green_level = random.randint(1,10)

# Spike-related variables
spikes = []
spike_timer = time.time()
spike_stop = False
SPIKE_INTERVAL = 3  # Spikes appear every 3 seconds
SPIKE_DURATION = 3  # Spikes remain visible for 3 seconds
star = None  # Only one star at a time
last_star_time = 0  # Time of last star generation
star_duration = 5  # Time of last star generation

def draw_point(x, y):
    glBegin(GL_POINTS)
    glVertex2f(abs(x), abs(y))
    glEnd()

def controller_circle(x, y, r):
    zone1_points = midpoint_circle(r)
    whole_circle = create_whole_circle(zone1_points)
    for i in whole_circle:
        draw_point(i[0] + x, i[1] + y)
    #for i in whole_circle:
        i[0]+=x
        i[1]+=y
    return whole_circle

def midpoint_circle(r):
    d = 1 - r
    x = 0
    y = r
    intermediate = [[x, y]]
    
    while x <= y:
        if d >= 0:
            x += 1
            y -= 1
            intermediate.append([x, y])
            d = d + 2 * x - 2 * y + 5
        else:
            x += 1
            d = d + 2 * x + 3
            intermediate.append([x, y])
    
    return intermediate
def create_whole_circle(zone1_points):
    all_points = []
    for point in zone1_points:
        x, y = point
        all_points.append([x, y])
        all_points.append([-x, y])
        all_points.append([y, -x])
        all_points.append([-y, -x])
        all_points.append([-x, -y])
        all_points.append([x, -y])
        all_points.append([-y, x])
        all_points.append([y, x])
    return all_points

def midpoint_line(points):
    if points[0][0] >= points[1][0]:
        points[0], points[1] = points[1], points[0]  

    dx = points[1][0] - points[0][0]
    dy = points[1][1] - points[0][1]
    d = dy - (dx // 2)
    x = points[0][0]
    y = points[0][1]
    intermediate = [[x, y]]
    
    while x < points[1][0]:
        x += 1
        if d < 0:
            d += dy
        else:
            y += 1
            d += dy - dx
        intermediate.append([x, y])
    
    return intermediate

def controller_line(points):
    list1 = []
    square = [[-10, -10], [-10, 10], [10, 10], [10, -10]]  

    for i in range(len(points) - 1):
        zone = decide_zone([points[i], points[i + 1]])
        convert = converttozero([points[i], points[i + 1]], zone)
        interpoint = midpoint_line(convert)
        zonepoint = returntooriginal(interpoint, zone)
        
        list1.extend(zonepoint)


    zone = decide_zone([points[-1], points[0]])
    convert = converttozero([points[-1], points[0]], zone)
    interpoint = midpoint_line(convert)
    zonepoint = returntooriginal(interpoint, zone)
    list1.extend(zonepoint)

    for j in square:
        list1.append(j)
    for j in list1:
        draw_point(j[0], j[1])
    return points

def decide_zone(decide):
    dx=decide[0][0]-decide[1][0]
    dy=decide[0][1]-decide[1][1]
    if dx>0 and dy>=0 and abs(dx) >= abs(dy):
        return 0
    elif dx>=0 and dy>0 and abs(dx) < abs(dy):
        return 1
    elif dx<0 and dy>=0 and abs(dx) <= abs(dy):
        return 2
    elif dx<=0 and dy>0 and abs(dx) > abs(dy):
        return 3
    elif dx<0 and dy<=0 and abs(dx) >= abs(dy):
        return 4
    elif dx<=0 and dy<0 and abs(dx) < abs(dy):
        return 5

    elif dx>0 and dy<=0 and abs(dx) <= abs(dy):
        return 6

    elif dx>=0 and dy<0 and abs(dx) > abs(dy):
        return 7

def converttozero(zeroth,a):
    zeroth = [point[:] for point in zeroth] 
    if a==0:
        zeroth[0][0],zeroth[0][1]=zeroth[0][0],zeroth[0][1]
        zeroth[1][0],zeroth[1][1]=zeroth[1][0],zeroth[1][1]
        return zeroth
    
    elif a==1:
        zeroth[0][0],zeroth[0][1]=zeroth[0][1],zeroth[0][0]
        zeroth[1][0],zeroth[1][1]=zeroth[1][1],zeroth[1][0]
        return zeroth

    elif a==2:
        zeroth[0][0],zeroth[0][1]=zeroth[0][1],zeroth[0][0]*-1
        zeroth[1][0],zeroth[1][1]=zeroth[1][1],-1*zeroth[1][0]
        return zeroth

    elif a==3:
        zeroth[0][0],zeroth[0][1]=-1*zeroth[0][0],zeroth[0][1]
        zeroth[1][0],zeroth[1][1]=-1*zeroth[1][0],zeroth[1][1]
        return zeroth

    elif a==4:
        zeroth[0][0],zeroth[0][1]=-1*zeroth[0][0],-1*zeroth[0][1]
        zeroth[1][0],zeroth[1][1]=-1*zeroth[1][0],-1*zeroth[1][1]
        return zeroth

    elif a==5:
        zeroth[0][0],zeroth[0][1]=-1*zeroth[0][1],-1*zeroth[0][0]
        zeroth[1][0],zeroth[1][1]=-1*zeroth[1][1],-1*zeroth[1][0]
        return zeroth

    elif a==6:
        zeroth[0][0],zeroth[0][1]=-1*zeroth[0][1],zeroth[0][0]
        zeroth[1][0],zeroth[1][1]=-1*zeroth[1][1],zeroth[1][0]
        return zeroth

    elif a==7:
        zeroth[0][0],zeroth[0][1]=zeroth[0][0],-1*zeroth[0][1]
        zeroth[1][0],zeroth[1][1]=zeroth[1][0],-1*zeroth[1][1]
        return zeroth
    
def returntooriginal(points, zone):
    if zone == 0:
        return points  
    elif zone == 1:
        return [[point[1], point[0]] for point in points]

    elif zone == 2:
        return [[point[1], -point[0]] for point in points]

    elif zone == 3:
        return [[-point[0], point[1]] for point in points]

    elif zone == 4:
        return [[-point[0], -point[1]] for point in points]

    elif zone == 5:
        return [[-point[1], -point[0]] for point in points]

    elif zone == 6:
        return [[-point[1], point[0]] for point in points]

    elif zone == 7:
        return [[point[0], -point[1]] for point in points]
def generate_spikes():
    global spikes
    spikes = []
    for level in range(2, 10):
        if random.choice([True, False]):
            x = random.randint(0, 800)
            y = level * 40
            spikes.append({'x': x, 'y': y, 'level': level})

def draw_spikes():
    global spikes
    glColor3f(1, 1, 1)
    for spike in spikes:
        controller_line([[spike['x'], spike['y']], [spike['x']-10, spike['y']-20], [spike['x']+10, spike['y']-20]])

def keyboardListener(key, x, y):
    global player_pos, score, shield_active, shield_start_time, shield_count

    if not obstacles['stop']:
        if key == b'\033':
            print("Exiting...")
            glutLeaveMainLoop()

        if key == b's':
            if shield_count > 0:
                if not shield_active:
                    shield_active = True
                    shield_start_time = time.time()
                    shield_count -= 1
                    print("Shield activated!")
                    print(f"Shield count remaining: {shield_count}")
                else:
                    print("Shield is already active.")
            else:
                print("No shields available.")

    glutPostRedisplay()

def update_shield():
    global shield_active
    if shield_active and time.time() - shield_start_time > 2:
        shield_active = False
        print("Shield deactivated.")
def specialKeyListener(key, x, y):
    global player_pos, score
    if not obstacles['stop']:
        if key == GLUT_KEY_UP:  # Move up (arrow up key)
            player_pos[1] += 40
            score += 1
            print('Score:', score, 'Lives:', lives)

        if key == GLUT_KEY_DOWN:  # Move down (arrow down key)
            player_pos[1] -= 40
            score -= 1
            print('Score:', score, 'Lives:', lives)

        if key == GLUT_KEY_RIGHT:  # Move right (arrow right key)
            player_pos[0] += 40
            print('Score:', score, 'Lives:', lives)

        if key == GLUT_KEY_LEFT:  # Move left (arrow left key)
            player_pos[0] -= 40
            print('Score:', score, 'Lives:', lives)

    glutPostRedisplay()


def mouseListener(button, state, x, y):
    global obstacles,obs_flag,player_pos,lives, score, green_level, spike_stop
    
    #Restart
    # Restart
    if 10 <= x <= 50 and 470 <= -1 * (y - 500) <= 490: 
      if button == GLUT_LEFT_BUTTON and state == GLUT_DOWN:
        green_level = random.randint(1, 10)
        obstacles = {'stop': False}
        obs_flag = True
        player_pos = [400, 20]
        lives = 3
        score = 0
        shield_active = False
        shield_count = 2
        print('Game Restarted')

    
    #Pause
    elif 395<=x<=405 and 470<=-1*(y-500)<=490:
        if button==GLUT_LEFT_BUTTON: 
            if state == GLUT_DOWN:
                if spike_stop:
                    spike_stop = False
                else:
                    spike_stop = True
                if not obstacles['stop']:
                    obstacles['stop']=True
                else:
                    obstacles['stop']=False
    
    #Exit
    elif 770<=x<=790 and 460<=-1*(y-500)<=490:
        if button==GLUT_LEFT_BUTTON: 
            if(state == GLUT_DOWN):
                print("Goodbye")
                print("Final Score:", score, ' Live(s):', lives)
                glutLeaveMainLoop() 
    glutPostRedisplay()


def retry():
    global player_pos,obstacles,obs_flag
    obstacles={'stop':False}
    obs_flag=True
    player_pos=[400,20]
def check_collision(player):
    global player_pos, obstacles, lives, score, green_level, shield_active, shield_start_time
    
    for k, v in obstacles.items():
        if k != 'stop':
            val = v['points']
            
            if val[0][0] >= player_pos[0] + 10 >= val[3][0] or val[0][0] >= player_pos[0] - 10 >= val[3][0]:
                
                if val[0][1] <= player_pos[1] + 10 <= val[2][1] and k == green_level:
                    obstacles['stop'] = False
                    score += 3
                    player_pos[1] += 40
                    green_level = 0
                    print("Plus Points!")
                    print('Score:', score, 'Lives:', lives)
                    return
                
                elif val[0][1] <= player_pos[1] + 10 <= val[2][1]:
                    if shield_active and time.time() - shield_start_time <= 2:
                        print("Shield absorbed the collision!")
                        return
                    else:
                        obstacles['stop'] = True
                        lives -= 1
                        green_level = random.randint(1, 10)
                        print("Collision detected!")
                        print('Score:', score, 'Lives:', lives)
                        retry()
                        return 
    if score >= 100:
        print('You have Won!!!')
        print("Final Score:", score, ' Lives:', lives)
        glutLeaveMainLoop()

def check_spike_collision(player): 
    global spikes, score, shield_active, shield_start_time
    px, py = player_pos
    if shield_active and time.time() - shield_start_time <= 2:
        return
    
    for spike in spikes:
        sx, sy = spike['x'], spike['y']
        if sx - 20 <= px <= sx + 20 and sy - 25 <= py <= sy:
            score -= 3
            print("Spike collision! -3 points.")
            print('Score:', score, 'Lives:', lives)
            retry()
            break

def generate_star():
    global star, last_star_time
    if lives < 3:
        current_time = time.time()

        if current_time - last_star_time >= 5 and star is None:
            star_y = random.randint(1, 9) * 40 + 20
            star_x = random.randint(50, 750)
            star = {'x': star_x, 'y': star_y, 'time': current_time}
            last_star_time = current_time
def controller_star(x, y, r):
    glColor3f(1, 0, 0)
    left_circle = controller_circle(x - r // 2, y, r // 3)
    right_circle = controller_circle(x + r // 2, y, r // 3)
    line1 = [[x - r, y], [x - r // 2, y]]
    line2 = [[x + r // 2, y], [x + r, y]]
    line3 = [[x - r // 4, y + r // 2], [x + r // 4, y + r // 2]]
    line4 = [[x - r // 4, y - r // 2], [x + r // 4, y - r // 2]]
    lines = []
    lines.extend(midpoint_line(line1))
    lines.extend(midpoint_line(line2))
    lines.extend(midpoint_line(line3))
    lines.extend(midpoint_line(line4))
    for point in left_circle + right_circle:
        draw_point(point[0], point[1])
    for point in lines:
        draw_point(point[0], point[1])

def draw_star():
    global star
    if star:
        current_time = time.time()
        if current_time - star['time'] <= star_duration:
            glColor3f(1, 1, 0)
            controller_star(star['x'], star['y'], 10)
        else:
            star = None

def check_star_collision():
    global player_pos, star, lives
    if star:
        if (player_pos[0] <= star['x'] + 10 and player_pos[1] <= star['y'] + 10) and (star['x'] - 10 <= player_pos[0] and star['y'] - 10 <= player_pos[1]):
            lives += 1
            star = None

def animation():
    global obstacles, spike_timer, spike_stop, player_pos, score, lives
    generate_star()
    draw_star()
    check_star_collision()
    if not spike_stop:
        if time.time() - spike_timer > SPIKE_INTERVAL:
            generate_spikes()
            spike_timer = time.time()
        if time.time() - spike_timer > SPIKE_DURATION:
            spikes.clear()
    if not obstacles['stop']:
        for k, v in obstacles.items():
            if k != 'stop':
                point = v['points']
                if point[0][0] >= 800:
                    v['velocity'] = v['velocity'] * -1
                if point[3][0] <= 0:
                    v['velocity'] = v['velocity'] * -1
        for i, k in obstacles.items():
            if i != 'stop':
                if k['points'][0][0] <= 800 or k['points'][3][0] >= 0:
                    for point in k['points']:
                        point[0] += k["velocity"]
    if player_pos[1] > 400:
        player_pos = [400, 20]
    if player_pos[0] > 800:
        player_pos[0] = 400
    if player_pos[0] < 0:
        player_pos[0] = 250
    update_shield()
    glutPostRedisplay()

def generate_obstacle(level):
    global obstacles
    x = random.randint(50, 750)
    y = level * 40
    if score <= 10:
        velocity = 10
    elif score >= 30:
        velocity = score / 2
    elif 15 < score < 29:
        velocity = 15
    else:
        velocity = score
    obstacles[level] = {"points": [[x, y + 5], [x, y + 35], [x - 50, y + 35], [x - 50, y + 5]], "velocity": velocity}

def draw_text(x, y, text):
    glRasterPos2f(x, y)
    for c in text:
        glutBitmapCharacter(GLUT_BITMAP_HELVETICA_18, ord(c))

def draw_score_and_lives():
    glColor3f(1, 1, 1)
    draw_text(60, 470, f"Score: {score}")
    draw_text(700, 470, f"Lives: {lives}")
    draw_text(700, 450, f"Shields: {shield_count}")

def showScreen():
    global obstacles, obs_flag, player_pos, score, lives, green_level, spike_stop, shield_active
    glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT)
    glLoadIdentity()
    iteration()
    x = 40
    for level in range(1, 11):
        controller_line([[0, x], [800, x]])
        x += 40
        if obs_flag:
            generate_obstacle(level)
    obs_flag = False
    if lives > 0:
        for i, j in obstacles.items():
            if i != 'stop' and (i != green_level or green_level == 0):
                glColor3f(1, 1, 1)
                controller_line(j['points'])
            elif i != 'stop' and i == green_level:
                glColor3f(0, 1, 0)
                controller_line(j['points'])
        glColor3f(1, 1, 0)
        controller_circle(player_pos[0], player_pos[1], 10)
        if shield_active and time.time() - shield_start_time <= 2:
            glColor3f(0, 0, 1)
            controller_circle(player_pos[0], player_pos[1], 15)
        draw_spikes()
        draw_star()
        check_spike_collision(player_pos)
        check_collision(player_pos)
    else:
        glColor3f(1, 0, 0)
        draw_text(350, 250, "GAME OVER")
        draw_text(320, 220, f"FINAL SCORE: {score}")
        draw_text(300, 190, "Click Restart to play again!")
    glColor3f(0.1, 0.5, 0.7)
    controller_line([[30, 490], [10, 480], [30, 470], [10, 480], [50, 480], [10, 480]])
    glColor3f(1, 0.5, 0)
    if not obstacles['stop']:
        controller_line([[395, 490], [395, 470]])
        controller_line([[405, 490], [405, 470]])
    else:
        controller_line([[395, 490], [405, 480], [395, 470]])
    glColor3f(1, 0, 0)
    controller_line([[770, 490], [790, 470]])
    controller_line([[770, 470], [790, 490]])
    draw_score_and_lives()
    glutSwapBuffers()

def timer(value):
    animation()
    glutPostRedisplay()
    glutTimerFunc(16, timer, 0)

def iteration():
    glViewport(0, 0, 800, 500)
    glMatrixMode(GL_PROJECTION)
    glLoadIdentity()
    glOrtho(0.0, 800, 0.0, 500, 0.0, 1.0)
    glMatrixMode(GL_MODELVIEW)
    glLoadIdentity()

glutInit()
glutInitDisplayMode(GLUT_RGBA)
glutInitWindowSize(800, 500)
glutInitWindowPosition(100, 100)
wind = glutCreateWindow(b"Go Duck Go")
glutDisplayFunc(showScreen)
glutIdleFunc(animation)
glutKeyboardFunc(keyboardListener)
glutSpecialFunc(specialKeyListener)
glutMouseFunc(mouseListener)
glutMainLoop()
    
    
