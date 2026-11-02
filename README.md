
# =========================================================
# FPS AIM TRAINER
# Processing 4 - Python Mode
# A 30-second shooting game with randomly placed targets,
# hit detection, accuracy tracking and shooting effects.
# All graphics are created directly through code.
# No external image files are required.
# =========================================================


# A list storing target positions as [x, y] coordinates.
targets = []

# Player statistics and target settings.
score = 0
misses = 0
target_count = 5
target_radius = 25

# The game has three states: start, playing and gameover.
game_state = "start"

game_length = 30
start_time = 0

# Camera movement and shooting animation variables.
look_x = 0
look_y = 0

shake_x = 0
shake_y = 0
shake_frames = 0

last_shot_time = -1000

# Stores temporary hit effects as [x, y, creation_time].
hit_effects = []


# =========================================================
# GAME SETUP
# =========================================================

def settings():
    size(1000, 700)


def setup():
    # Hide the default mouse cursor.
    # A custom crosshair will be drawn instead.
    noCursor()

    # Create five targets when the program starts.
    create_targets()


# =========================================================
# MAIN DRAW LOOP
# =========================================================

def draw():
    # The main loop updates animations and draws the game
    # according to its current state.

    update_camera()
    update_shake()

    draw_range()

    if game_state == "start":
        draw_targets()
        draw_crosshair()
        draw_start_screen()

    elif game_state == "playing":
        update_game()
        draw_targets()
        draw_hit_effects()
        draw_hud()
        draw_crosshair()

    elif game_state == "gameover":
        draw_targets()
        draw_crosshair()
        draw_game_over()


# =========================================================
# CAMERA MOVEMENT
# =========================================================

def update_camera():
    global look_x
    global look_y

    # Map mouse coordinates to a small camera offset.
    wanted_x = map(mouseX, 0, width, -30, 30)
    wanted_y = map(mouseY, 0, height, -18, 18)

    # Gradually move 8% towards the desired position
    # each frame to create smooth camera movement.
    look_x += (wanted_x - look_x) * 0.08
    look_y += (wanted_y - look_y) * 0.08


# =========================================================
# GAME BACKGROUND
# =========================================================

def draw_range():
    # Create a simple shooting range using Processing shapes.
    # This replaces the original external background image.

    background(40, 45, 55)

    # Draw a darker upper wall.
    noStroke()
    fill(32, 37, 47)
    rect(0, 0, width, 250)

    # Draw the main shooting range wall.
    fill(55, 62, 72)
    rect(0, 250, width, 280)

    # Draw the floor.
    fill(35, 40, 48)
    rect(0, 530, width, 170)

    # Draw horizontal lines to create depth.
    stroke(85, 95, 105)
    strokeWeight(2)

    line(0, 250, width, 250)
    line(0, 530, width, 530)

    # Draw a simple frame around the target area.
    noFill()
    stroke(105, 115, 125)
    strokeWeight(3)
    rect(130, 260, 740, 220)

    noStroke()


# =========================================================
# TARGET GENERATION
# =========================================================

def create_targets():
    # Clear previous targets before generating a new set.
    del targets[:]

    # Use a loop to create the required number of targets.
    for i in range(target_count):
        add_target()


def add_target():
    attempts = 0

    # Keep generating positions until a valid one is found.
    # The attempt limit prevents an infinite loop.
    while attempts < 100:
        x = random(170, 830)
        y = random(300, 440)

        valid = True

        # Check the distance from all existing targets
        # to avoid placing them too close together.
        for other in targets:
            distance = dist(x, y, other[0], other[1])

            if distance < 90:
                valid = False
                break

        if valid:
            targets.append([x, y])
            return

        attempts += 1


def move_target(target):
    attempts = 0

    # Generate a new position after the target is hit.
    while attempts < 100:
        new_x = random(170, 830)
        new_y = random(300, 440)

        valid = True

        for other in targets:
            if other is not target:
                distance = dist(
                    new_x,
                    new_y,
                    other[0],
                    other[1]
                )

                if distance < 90:
                    valid = False
                    break

        if valid:
            # Update the existing target's coordinates
            # rather than creating another target.
            target[0] = new_x
            target[1] = new_y
            return

        attempts += 1


# =========================================================
# TARGET DRAWING
# =========================================================

def draw_targets():
    pushMatrix()

    # Apply camera movement to all targets.
    translate(
        -look_x + shake_x,
        -look_y + shake_y
    )

    # Draw each target using coordinates from the list.
    for target in targets:
        draw_target(target[0], target[1])

    popMatrix()


def draw_target(x, y):
    # Build a circular target from layered ellipses.
    # All target graphics are drawn directly through code.

    noStroke()

    # Target shadow.
    fill(0, 0, 0, 70)
    ellipse(
        x + 4,
        y + 5,
        target_radius * 2.6,
        target_radius * 2.6
    )

    # Outer white ring.
    fill(235, 235, 230)
    ellipse(
        x,
        y,
        target_radius * 2.4,
        target_radius * 2.4
    )

    # Dark ring.
    fill(55, 58, 60)
    ellipse(
        x,
        y,
        target_radius * 1.95,
        target_radius * 1.95
    )

    # Red ring.
    fill(205, 45, 40)
    ellipse(
        x,
        y,
        target_radius * 1.55,
        target_radius * 1.55
    )

    # Inner red circle.
    fill(125, 20, 20)
    ellipse(
        x,
        y,
        target_radius * 0.9,
        target_radius * 0.9
    )

    # Black centre.
    fill(20, 20, 20)
    ellipse(
        x,
        y,
        target_radius * 0.35,
        target_radius * 0.35
    )


# =========================================================
# CUSTOM CROSSHAIR
# =========================================================

def draw_crosshair():
    gap = 6
    arm = 12

    # Draw a custom crosshair that follows the mouse.
    stroke(80, 255, 120)
    strokeWeight(2)

    line(
        mouseX,
        mouseY - gap,
        mouseX,
        mouseY - gap - arm
    )

    line(
        mouseX,
        mouseY + gap,
        mouseX,
        mouseY + gap + arm
    )

    line(
        mouseX - gap,
        mouseY,
        mouseX - gap - arm,
        mouseY
    )

    line(
        mouseX + gap,
        mouseY,
        mouseX + gap + arm,
        mouseY
    )

    noStroke()
    fill(80, 255, 120)
    ellipse(mouseX, mouseY, 3, 3)


# =========================================================
# SCREEN SHAKE EFFECT
# =========================================================

def update_shake():
    global shake_x
    global shake_y
    global shake_frames

    # Apply random camera offsets for a few frames
    # after firing to simulate weapon impact.

    if shake_frames > 0:
        shake_x = random(-3, 3)
        shake_y = random(-2, 2)
        shake_frames -= 1

    else:
        shake_x = 0
        shake_y = 0


# =========================================================
# HIT EFFECTS
# =========================================================

def draw_hit_effects():
    pushMatrix()

    translate(
        -look_x + shake_x,
        -look_y + shake_y
    )

    # Iterate over a copy of the list so expired effects
    # can be removed safely during the loop.
    for effect in hit_effects[:]:
        x = effect[0]
        y = effect[1]
        created = effect[2]

        age = millis() - created

        if age > 350:
            hit_effects.remove(effect)

        else:
            # Fade the effect over time while expanding
            # the circle and moving the score text upward.

            alpha = map(age, 0, 350, 255, 0)

            noFill()
            stroke(255, 230, 150, alpha)
            strokeWeight(3)

            effect_size = 25 + age * 0.14
            ellipse(x, y, effect_size, effect_size)

            noStroke()
            fill(255, 255, 255, alpha)
            textAlign(CENTER)
            textSize(21)

            text(
                "+1",
                x,
                y - 32 - age * 0.05
            )

    popMatrix()


# =========================================================
# GAME HUD
# =========================================================

def draw_hud():
    total_clicks = score + misses

    # Calculate shooting accuracy as a percentage.
    # Check total_clicks first to avoid division by zero.
    if total_clicks > 0:
        accuracy = float(score) / total_clicks * 100

    else:
        accuracy = 0

    remaining = get_remaining_time()

    # Draw the HUD panel containing player statistics.
    noStroke()
    fill(15, 20, 28, 185)
    rect(625, 20, 350, 82)

    stroke(150, 190, 220, 100)
    strokeWeight(1)

    line(742, 34, 742, 90)
    line(858, 34, 858, 90)

    textAlign(CENTER)
    fill(160, 210, 245)
    textSize(13)

    text("SCORE", 685, 45)
    text("TIME", 800, 45)
    text("ACCURACY", 917, 45)

    fill(255)
    textSize(29)

    text(str(score), 685, 80)
    text("%.1f" % remaining, 800, 80)
    text("%.1f%%" % accuracy, 917, 80)

    # Draw the game title.
    textAlign(LEFT)
    fill(255)
    textSize(28)
    text("AIM TRAINER", 25, 42)

    fill(190, 225, 250)
    textSize(13)
    text("PISTOL PRACTICE", 27, 64)

    fill(255, 170, 150)
    textSize(14)
    text("MISSES: " + str(misses), 27, 89)


# =========================================================
# GAME TIMER
# =========================================================

def get_remaining_time():
    if game_state != "playing":
        return game_length

    # Convert elapsed milliseconds into seconds
    # and subtract them from the game duration.
    elapsed = (millis() - start_time) / 1000.0

    remaining = game_length - elapsed

    if remaining < 0:
        remaining = 0

    return remaining


def update_game():
    global game_state

    # End the round when the countdown reaches zero.
    if get_remaining_time() <= 0:
        game_state = "gameover"


# =========================================================
# GAME START AND RESET
# =========================================================

def start_game():
    global score
    global misses
    global start_time
    global game_state
    global shake_frames
    global shake_x
    global shake_y

    # Reset game statistics, effects and target positions.
    score = 0
    misses = 0

    shake_frames = 0
    shake_x = 0
    shake_y = 0

    del hit_effects[:]

    create_targets()

    # Record the starting time for the new round.
    start_time = millis()
    game_state = "playing"


# =========================================================
# MOUSE INPUT AND HIT DETECTION
# =========================================================

def mousePressed():
    global score
    global misses
    global game_state

    global last_shot_time
    global shake_frames

    # Clicking the start screen begins the game.
    if game_state == "start":
        start_game()
        return

    # Ignore clicks after the game ends.
    if game_state == "gameover":
        return

    if game_state == "playing":

        # Do not allow shots after time has expired.
        if get_remaining_time() <= 0:
            game_state = "gameover"
            return

        # Trigger the shooting animation.
        last_shot_time = millis()
        shake_frames = 3

        # Track whether this mouse click hits a target.
        hit = False

        for target in targets:

            # Convert target positions to screen coordinates
            # by including camera movement and shake.
            target_screen_x = target[0] - look_x + shake_x
            target_screen_y = target[1] - look_y + shake_y

            # Calculate the distance between the cursor
            # and the centre of the current target.
            distance = dist(
                mouseX,
                mouseY,
                target_screen_x,
                target_screen_y
            )

            # A hit occurs when the distance is within
            # the visible target radius.
            if distance <= target_radius * 1.2:
                score += 1

                # Store a temporary hit effect with its
                # location and creation time.
                hit_effects.append([
                    target[0],
                    target[1],
                    millis()
                ])

                # Relocate the target after a successful hit.
                move_target(target)

                hit = True
                break

        # Count a miss if no target was hit.
        if hit == False:
            misses += 1


# =========================================================
# START SCREEN
# =========================================================

def draw_start_screen():
    # Display a semi-transparent introduction screen.
    noStroke()
    fill(0, 0, 0, 120)
    rect(0, 0, width, height)

    textAlign(CENTER)

    fill(255)
    textSize(54)
    text("AIM TRAINER", width / 2, 245)

    fill(180, 220, 250)
    textSize(21)
    text("PISTOL RANGE", width / 2, 285)

    fill(235)
    textSize(16)
    text("30 SECOND PRACTICE", width / 2, 330)

    noStroke()
    fill(220, 150, 35, 230)
    rect(width / 2 - 115, 385, 230, 58)

    fill(255)
    textSize(19)
    text("CLICK TO START", width / 2, 421)


# =========================================================
# GAME OVER SCREEN
# =========================================================

def draw_game_over():
    total_clicks = score + misses

    # Calculate the final accuracy using hits and misses.
    if total_clicks > 0:
        accuracy = float(score) / total_clicks * 100

    else:
        accuracy = 0

    # Show the final score and performance statistics.
    noStroke()
    fill(0, 0, 0, 165)
    rect(0, 0, width, height)

    textAlign(CENTER)

    fill(185, 225, 250)
    textSize(22)
    text("TRAINING COMPLETE", width / 2, 205)

    fill(255)
    textSize(62)
    text(str(score), width / 2, 285)

    fill(200)
    textSize(15)
    text("FINAL SCORE", width / 2, 315)

    fill(255)
    textSize(20)

    text(
        "HITS         " + str(score),
        width / 2,
        375
    )

    text(
        "MISSES       " + str(misses),
        width / 2,
        410
    )

    text(
        "ACCURACY      %.1f%%" % accuracy,
        width / 2,
        445
    )

    fill(245, 185, 80)
    textSize(17)
    text("PRESS R TO RETRY", width / 2, 515)


# =========================================================
# KEYBOARD INPUT
# =========================================================

def keyPressed():
    # Allow the player to restart after the game ends.
    if game_state == "gameover":
        if key == 'r' or key == 'R':
            start_game()
