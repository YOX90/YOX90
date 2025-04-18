••YO X FREE ••

import winsound
import win32api
import win32con
import numpy as np
import random
import time 
import cv2
import mss

class CaptureSettings:
    def __init__(self):
        self.screen_width = 1920
        self.screen_height = 1080
        self.mid_x = self.screen_width // 2
        self.mid_y = self.screen_height // 2
        self.box_size = 300
        self.half_box = self.box_size // 2
        self.x_start = self.mid_x - self.half_box
        self.y_start = self.mid_y - self.half_box
        
        self.capture_area = {
            "top": self.y_start,
            "left": self.x_start,
            "width": self.box_size,
            "height": self.box_size
        }

def calculate_sensitivity():
    base_sens = 0.6
    mouse_sens = 0.55
    aim_sens = 1
    
    game_sens = mouse_sens * aim_sens
    movement_factor = 0.2
    return ((base_sens * game_sens) / 0.6) + movement_factor

def process_frame(frame, template, half_w, half_h, half_box, sens_multiplier):
    result = cv2.matchTemplate(frame, template, cv2.TM_CCOEFF_NORMED)
    _, confidence, _, position = cv2.minMaxLoc(result)
    
    if confidence >= 0.78:
        target_x = position[0] + half_w
        target_y = position[1] + half_h
        
        move_x = (-(half_box - target_x)) * sens_multiplier
        move_y = (-(half_box - target_y)) * sens_multiplier
        
        win32api.mouse_event(win32con.MOUSEEVENTF_MOVE, int(move_x), int(move_y), 0, 0)
        win32api.mouse_event(0x0002, 0, 0, 0, 0)
        time.sleep(random.uniform(0.01, 0.05))
        win32api.mouse_event(0x0004, 0, 0, 0, 0)

def main():
    settings = CaptureSettings()
    screen = mss.mss()
    
    target_img = cv2.imread("image.png", cv2.IMREAD_UNCHANGED)
    gray_target = cv2.cvtColor(target_img, cv2.COLOR_BGR2GRAY)
    width, height = gray_target.shape[::-1]
    
    half_width = width // 2
    half_height = height // 2
    half_box = settings.half_box
    sens = calculate_sensitivity()
    
    while True:
        time.sleep(0.001)
        screen_data = np.array(screen.grab(settings.capture_area))
        gray_frame = cv2.cvtColor(screen_data, cv2.COLOR_BGRA2GRAY)
        
        if win32api.GetAsyncKeyState(0x12) < 0:
            winsound.Beep(800, 5)
            break
        elif win32api.GetAsyncKeyState(0x02) < 0:
            process_frame(gray_frame, gray_target, half_width, half_height, half_box, sens)
    

if __name__ == "__main__":
    main()
