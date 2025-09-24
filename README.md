# rasberry-

sudo apt update
sudo apt install -y python3-gpiozero python3-pip
sudo pip3 install keyboard


script file


#!/usr/bin/env python3
import time
import keyboard  # requires sudo on Linux
from gpiozero import OutputDevice

# --- Pin mapping (BCM) ---
ENA = OutputDevice(25)   # Enable (we'll hold it HIGH to enable)
IN1 = OutputDevice(23)   # Left/Ch A input 1  (backward for that side)
IN2 = OutputDevice(24)   # Left/Ch A input 2  (forward for that side)
IN3 = OutputDevice(27)   # Right/Ch B input 1 (backward for that side)
IN4 = OutputDevice(22)   # Right/Ch B input 2 (forward for that side)

def all_low():
    IN1.off(); IN2.off(); IN3.off(); IN4.off()

def stop():
    all_low()

def forward():
    # INA2 + INA4 = forward
    IN1.off(); IN3.off()
    IN2.on();  IN4.on()

def backward():
    # INA1 + INA3 = backward
    IN2.off(); IN4.off()
    IN1.on();  IN3.on()

def turn_left():
    # Left wheel backward (IN1), Right wheel forward (IN4)
    IN2.off(); IN3.off()
    IN1.on();  IN4.on()

def turn_right():
    # Left wheel forward (IN2), Right wheel backward (IN3)
    IN1.off(); IN4.off()
    IN2.on();  IN3.on()

pressed = {"w": False, "a": False, "s": False, "d": False,
           "up": False, "down": False, "left": False, "right": False, "space": False}

def update_motion():
    # Priority: space (stop) > direction keys.
    if pressed["space"]:
        stop()
        return

    if pressed["w"] or pressed["up"]:
        forward();  return
    if pressed["s"] or pressed["down"]:
        backward(); return
    if pressed["a"] or pressed["left"]:
        turn_left(); return
    if pressed["d"] or pressed["right"]:
        turn_right(); return

    # If nothing is held, stop.
    stop()

def on_press(e):
    key = e.name
    if key in pressed:
        pressed[key] = True
        update_motion()
    elif key == 'q':
        raise KeyboardInterrupt
    elif key == 'space':
        pressed["space"] = True
        update_motion()

def on_release(e):
    key = e.name
    if key in pressed:
        pressed[key] = False
        update_motion()
    elif key == 'space':
        pressed["space"] = False
        update_motion()

def main():
    print("Keyboard Robot Driver (hold keys): W/A/S/D or arrows, Space=stop, Q=quit")
    print("Wiring (BCM): ENA=25, IN1=23, IN2=24, IN3=27, IN4=22")
    ENA.on()        # enable the driver
    stop()          # start safe

    # Register handlers
    keyboard.on_press(on_press)
    keyboard.on_release(on_release)

    try:
        while True:
            time.sleep(0.1)  # idle; events drive motion
    except KeyboardInterrupt:
        pass
    finally:
        print("Stopping motors and exiting…")
        stop()
        ENA.off()

if __name__ == "__main__":
    main()





sudo python3 drive_bot_hold.py


