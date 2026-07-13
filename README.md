아니 이거 왜 자꾸 이상하게 나오는거야 진짜 
/그냥 코드 눌러서 코드 복사하세요
/제미나이한테 질문해도 해결이 안돼요



"Seriously, what is going on? The way they're working is really weird. / Just copy the extraction code. / Even asking Gemini about the code doesn't solve the problem."




import tkinter as tk
import random
import math

class MicroGameApp:
    def __init__(self, root):
        self.root = root
        self.root.title("저그본능 바보")
        
        self.root.attributes('-fullscreen', True)
        self.root.update_idletasks()
        self.WIDTH = self.root.winfo_screenwidth()
        self.HEIGHT = self.root.winfo_screenheight()
        self.root.bind("<Escape>", lambda e: self.root.destroy())
        
        self.canvas = tk.Canvas(root, width=self.WIDTH, height=self.HEIGHT, bg="#1e1e1e", highlightthickness=0)
        self.canvas.pack()
        
        self.volume = 50
        self.state = "MENU"
        self.timer_job = None
        self.game_active = False
        self.current_game_id = None
        self.game_pool = []
        
        self.show_main_menu()

    def clear(self):
        self.canvas.delete("all")
        if self.timer_job:
            self.root.after_cancel(self.timer_job)
            self.timer_job = None

    def unbind_all(self):
        self.root.unbind("<Key>")
        self.root.unbind("<space>")
        self.root.unbind("<Motion>")
        self.root.unbind("<KeyPress>")
        self.root.unbind("<KeyRelease>")
        self.canvas.unbind("<Button-1>")

    # --- 메인 메뉴 ---
    def show_main_menu(self):
        self.clear()
        self.unbind_all()
        self.state = "MENU"
        
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/3, text="저그본능 바보", fill="white", font=("Arial", 80, "bold"))
        
        self.btn_play = self.create_button(self.WIDTH/2 - 150, self.HEIGHT/2, 300, 80, "플레이", self.start_game_sequence)
        self.btn_settings = self.create_button(self.WIDTH/2 - 150, self.HEIGHT/2 + 100, 300, 80, "설정", self.show_settings)
        
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT - 50, text="게임을 종료하려면 ESC 키를 누르세요", fill="gray", font=("Arial", 20))
        self.root.bind("<Button-1>", self.handle_menu_click)

    def show_settings(self):
        self.clear()
        self.state = "SETTINGS"
        
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/3, text="설정", fill="white", font=("Arial", 80, "bold"))
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 50, text=f"소리 조절: {self.volume}%", fill="white", font=("Arial", 30))
        
        vx1, vy1, vx2, vy2 = self.WIDTH/2 - 300, self.HEIGHT/2, self.WIDTH/2 + 300, self.HEIGHT/2 + 20
        self.vol_slider_coords = (vx1, vy1, vx2, vy2)
        
        self.canvas.create_rectangle(vx1, vy1, vx2, vy2, fill="darkgray")
        fill_x = vx1 + (self.volume / 100) * (vx2 - vx1)
        self.canvas.create_rectangle(vx1, vy1, fill_x, vy2, fill="#00ff00")
        
        self.btn_back = self.create_button(self.WIDTH/2 - 150, self.HEIGHT/2 + 150, 300, 80, "뒤로 가기", self.show_main_menu)

    def create_button(self, x, y, w, h, text, action):
        self.canvas.create_rectangle(x, y, x+w, y+h, fill="#444444", outline="white", width=3, tags="btn")
        self.canvas.create_text(x+w/2, y+h/2, text=text, fill="white", font=("Arial", 30, "bold"), tags="btn")
        return {"action": action, "coords": (x, y, x+w, y+h)}
        
    def handle_menu_click(self, event):
        if self.state == "MENU":
            for btn in [self.btn_play, self.btn_settings]:
                x1, y1, x2, y2 = btn["coords"]
                if x1 <= event.x <= x2 and y1 <= event.y <= y2:
                    btn["action"]()
        elif self.state == "SETTINGS":
            x1, y1, x2, y2 = self.btn_back["coords"]
            if x1 <= event.x <= x2 and y1 <= event.y <= y2:
                self.btn_back["action"]()
            
            vx1, vy1, vx2, vy2 = self.vol_slider_coords
            if vx1 <= event.x <= vx2 and vy1 - 20 <= event.y <= vy2 + 20:
                self.volume = max(0, min(100, int((event.x - vx1) / (vx2 - vx1) * 100)))
                self.show_settings()

    def start_game_sequence(self):
        self.root.unbind("<Button-1>")
        self.stage = 1
        self.lives = 3
        self.speed = 1.0 
        self.score = 0
        self.state = "PLAYING"
        
        self.game_pool = [
            self.start_click_target, self.start_mash_key, self.start_avoid, 
            self.start_count, self.start_catch, self.start_timing, 
            self.start_find_different, self.start_math, self.start_dont_move,
            self.start_spam_click, self.start_type_word, self.start_laser,
            self.start_hidden_target, self.start_clean_screen
        ]
        random.shuffle(self.game_pool)
        self.show_stage_intro()

    # --- 시스템 및 타이머 ---
    def show_stage_intro(self):
        self.clear()
        self.unbind_all()
        self.game_active = False
        
        if self.lives <= 0:
            self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 120, text="GAME OVER", fill="red", font=("Arial", 80, "bold"))
            self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 20, text=f"Final Score: {self.score}", fill="white", font=("Arial", 40))
            
            btn_w, btn_h = 400, 100
            bx, by = self.WIDTH/2 - btn_w/2, self.HEIGHT/2 + 60
            self.canvas.create_rectangle(bx, by, bx+btn_w, by+btn_h, fill="white", outline="gray", width=5)
            self.canvas.create_text(self.WIDTH/2, by + btn_h/2, text="메인 화면으로", fill="black", font=("Arial", 30, "bold"))
            
            def on_restart(event):
                if bx <= event.x <= bx+btn_w and by <= event.y <= by+btn_h:
                    self.show_main_menu()
            self.canvas.bind("<Button-1>", on_restart)
            return

        if self.stage > 1 and (self.stage - 1) % 4 == 0:
            self.speed += 0.3
            self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="SPEED UP!", fill="yellow", font=("Arial", 100, "bold"))
            self.root.after(1500, self.ready_screen)
        else:
            self.ready_screen()

    def ready_screen(self):
        self.clear()
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 60, text=f"Stage {self.stage}", fill="white", font=("Arial", 60, "bold"))
        
        # 하트로 체력 표시
        hearts = "♥ " * self.lives + "♡ " * (3 - self.lives)
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 + 30, text=f"내 체력: {hearts}", fill="red", font=("Arial", 40, "bold"))
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 + 100, text=f"Speed: {self.speed:.1f}x", fill="gray", font=("Arial", 30))
        
        self.root.after(1500, self.choose_minigame)

    def choose_minigame(self):
        if self.stage == 15:
            self.show_boss_intro()
        elif self.game_pool:
            game_func = self.game_pool.pop()
            game_func()
        else:
            self.show_game_clear()

    def show_boss_intro(self):
        self.clear()
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="보스 스테이지!!", fill="red", font=("Arial", 100, "bold"))
        self.root.after(2500, self.start_boss)

    def update_rope_ui(self):
        self.canvas.delete("rope_ui")
        if 0 < self.time_left <= 4.0:
            burn_ratio = self.time_left / 4.0
            current_x = self.WIDTH * burn_ratio
            self.canvas.create_line(0, self.HEIGHT - 20, self.WIDTH, self.HEIGHT - 20, fill="#333333", width=15, tags="rope_ui")
            self.canvas.create_line(0, self.HEIGHT - 20, current_x, self.HEIGHT - 20, fill="#8B4513", width=15, tags="rope_ui")
            self.canvas.create_oval(current_x - 15, self.HEIGHT - 35, current_x + 15, self.HEIGHT - 5, fill="orange", outline="yellow", width=2, tags="rope_ui")
            
            bx, by = 60, self.HEIGHT - 60
            self.canvas.create_oval(bx-40, by-40, bx+40, by+40, fill="black", tags="rope_ui")
            self.canvas.create_text(bx, by, text="공겜", fill="white", font=("Arial", 16, "bold"), tags="rope_ui")

    def tick_timer(self):
        if not self.game_active: return
        self.time_left -= 0.1
        self.canvas.delete("timer_text")
        # 게임 중에는 남은 시간만 보여줍니다냥♡
        self.canvas.create_text(150, 80, text=f"Time: {max(0, self.time_left):.1f}", fill="white", font=("Arial", 30, "bold"), tags="timer_text")
        self.update_rope_ui()
        
        if self.time_left <= 0:
            self.game_active = False
            self.show_explosion()
            # 살아남아야 이기는 게임 목록
            survive_games = ["avoid", "dont_move", "laser"]
            self.root.after(600, lambda: self.end_minigame(self.current_game_id in survive_games))
        else:
            self.timer_job = self.root.after(100, self.tick_timer)

    def show_explosion(self):
        self.canvas.delete("rope_ui")
        bx, by = 60, self.HEIGHT - 60
        self.canvas.create_oval(bx-80, by-80, bx+80, by+80, fill="orange", outline="red", width=5, tags="explosion_ui")
        self.canvas.create_text(bx, by, text="쾅!", fill="red", font=("Arial", 40, "bold"), tags="explosion_ui")

    def end_minigame(self, success):
        self.game_active = False
        self.current_game_id = None
        self.clear()
        self.unbind_all()
        
        if success:
            self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="SUCCESS!", fill="#00ff00", font=("Arial", 80, "bold"))
            self.score += 1
        else:
            self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="FAIL...", fill="#ff0000", font=("Arial", 80, "bold"))
            self.lives -= 1
            
        if self.stage == 15 and self.lives > 0:
            self.root.after(1500, self.show_game_clear)
        elif self.lives <= 0:
            self.root.after(1500, self.show_stage_intro)
        else:
            self.stage += 1
            self.root.after(1500, self.show_stage_intro)

    def show_game_clear(self):
        self.clear()
        self.unbind_all()
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 120, text="GAME CLEAR!", fill="#00ff00", font=("Arial", 80, "bold"))
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 20, text=f"Final Score: {self.score}", fill="white", font=("Arial", 40))
        
        btn_w, btn_h = 400, 100
        bx, by = self.WIDTH/2 - btn_w/2, self.HEIGHT/2 + 60
        self.canvas.create_rectangle(bx, by, bx+btn_w, by+btn_h, fill="white", outline="gray", width=5)
        self.canvas.create_text(self.WIDTH/2, by + btn_h/2, text="메인 화면으로", fill="black", font=("Arial", 30, "bold"))
        
        def on_restart(event):
            if bx <= event.x <= bx+btn_w and by <= event.y <= by+btn_h:
                self.show_main_menu()
        self.canvas.bind("<Button-1>", on_restart)

    def set_time(self):
        self.time_left = max(3.5, 7.0 - (self.speed - 1.0))
        self.tick_timer()

    # --- 미니게임 1: 타겟 클릭하기 ---
    def start_click_target(self):
        self.clear(); self.game_active = True; self.current_game_id = "click"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="클릭해라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))
        
        x, y = random.randint(100, self.WIDTH - 150), random.randint(150, self.HEIGHT - 250)
        self.canvas.create_rectangle(x, y, x + 100, y + 100, fill="#0099ff", outline="")
        self.canvas.bind("<Button-1>", lambda e: self.end_minigame(True) if (x <= e.x <= x+100 and y <= e.y <= y+100) else None)
        self.set_time()

    # --- 미니게임 2: 무작위 키 연타 ---
    def start_mash_key(self):
        self.clear(); self.game_active = True; self.current_game_id = "mash"
        keys_pool = [('A', 'a'), ('S', 's'), ('D', 'd'), ('F', 'f'), ('스페이스바', 'space')]
        d_name, self.target_keysym = random.choice(keys_pool)
        self.clicks = 0
        target = int(10 + (self.speed * 3))
        
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 150, text=f"'{d_name}' 연타해라!", fill="yellow", font=("Arial", 60, "bold"))
        cx, cy, bar_w = self.WIDTH / 2, self.HEIGHT / 2 + 50, 600
        self.canvas.create_rectangle(cx - bar_w/2, cy, cx + bar_w/2, cy + 60, fill="#444444", outline="white")
        gauge_id = self.canvas.create_rectangle(cx - bar_w/2, cy, cx - bar_w/2, cy + 60, fill="#00ff00", outline="")
        
        def on_key(event):
            if event.keysym.lower() == self.target_keysym:
                self.clicks += 1
                self.canvas.coords(gauge_id, cx - bar_w/2, cy, cx - bar_w/2 + min(bar_w, (self.clicks/target)*bar_w), cy + 60)
                if self.clicks >= target: self.end_minigame(True)
        self.root.bind("<Key>", on_key)
        self.set_time()

    # --- 미니게임 3: 피해라! (5개로 증가) ---
    def start_avoid(self):
        self.clear(); self.game_active = True; self.current_game_id = "avoid"
        self.enemy_active = False 
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 150, text="피해라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))

        self.enemies = []
        for _ in range(5): # 방해물 5개로 늘림냥♡
            sx, sy = random.randint(100, self.WIDTH-100), random.randint(100, self.HEIGHT-100)
            eid = self.canvas.create_rectangle(sx-30, sy-30, sx+30, sy+30, outline="red", dash=(5, 5), width=3, fill="")
            self.enemies.append({"id": eid, "dx": random.choice([-15, 15]), "dy": random.choice([-15, 15])})
        
        def activate_enemy():
            if not self.game_active: return
            self.enemy_active = True
            for en in self.enemies: self.canvas.itemconfig(en["id"], fill="red", dash=(), outline="black")
        self.root.after(500, activate_enemy)
        
        def on_motion(event):
            if not self.enemy_active: return
            for en in self.enemies:
                c = self.canvas.coords(en["id"])
                if c and c[0] <= event.x <= c[2] and c[1] <= event.y <= c[3]:
                    self.end_minigame(False)
        self.root.bind("<Motion>", on_motion)

        def move_enemy():
            if not self.game_active: return
            for en in self.enemies:
                c = self.canvas.coords(en["id"])
                if not c: continue
                if c[0] + en["dx"] < 0 or c[2] + en["dx"] > self.WIDTH: en["dx"] *= -1
                if c[1] + en["dy"] < 0 or c[3] + en["dy"] > self.HEIGHT: en["dy"] *= -1
                self.canvas.move(en["id"], en["dx"], en["dy"])
            self.root.after(max(15, int(40 / self.speed)), move_enemy)
        move_enemy()
        self.set_time()

    # --- 미니게임 4: 몇 개? (도형들이 움직임) ---
    def start_count(self):
        self.clear(); self.game_active = True; self.current_game_id = "count"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="빨간 공은 몇 개?", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))
        
        target = random.randint(3, 7)
        self.moving_objects = []
        
        for _ in range(random.randint(5, 12)):
            x, y = random.randint(100, self.WIDTH - 150), random.randint(150, self.HEIGHT - 400)
            rid = self.canvas.create_rectangle(x, y, x+50, y+50, fill="#2b5cff", outline="")
            self.moving_objects.append({"id": rid, "dx": random.choice([-5, 5]), "dy": random.choice([-5, 5])})
            
        for _ in range(target):
            x, y = random.randint(100, self.WIDTH - 150), random.randint(150, self.HEIGHT - 400)
            oid = self.canvas.create_oval(x, y, x+50, y+50, fill="#ff2b2b", outline="white")
            self.moving_objects.append({"id": oid, "dx": random.choice([-5, 5]), "dy": random.choice([-5, 5])})
            
        choices = [target]
        while len(choices) < 3:
            w = random.randint(1, 10)
            if w not in choices: choices.append(w)
        random.shuffle(choices)
        
        self.boxes = []
        bw, bh, sp = 200, 100, (self.WIDTH - 600) // 4
        for i, val in enumerate(choices):
            cx, cy = sp * (i + 1) + bw * i, self.HEIGHT - 250
            rid = self.canvas.create_rectangle(cx, cy, cx+bw, cy+bh, fill="white", outline="gray", width=5)
            self.canvas.create_text(cx + bw/2, cy + bh/2, text=str(val), font=("Arial", 50, "bold"), fill="black")
            self.boxes.append((rid, val))
            
        def move_objects():
            if not self.game_active or self.current_game_id != "count": return
            for obj in self.moving_objects:
                c = self.canvas.coords(obj["id"])
                if not c: continue
                if c[0] + obj["dx"] < 0 or c[2] + obj["dx"] > self.WIDTH: obj["dx"] *= -1
                if c[1] + obj["dy"] < 120 or c[3] + obj["dy"] > self.HEIGHT - 280: obj["dy"] *= -1
                self.canvas.move(obj["id"], obj["dx"], obj["dy"])
            self.root.after(30, move_objects)
        move_objects()
            
        def on_click(e):
            for rid, val in self.boxes:
                c = self.canvas.coords(rid)
                if c and c[0] <= e.x <= c[2] and c[1] <= e.y <= c[3]:
                    self.end_minigame(val == target)
        self.canvas.bind("<Button-1>", on_click)
        self.set_time()

    # --- 미니게임 5: 받아라! ---
    def start_catch(self):
        self.clear(); self.game_active = True; self.current_game_id = "catch"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="받아라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))
        
        pw = 150
        self.pad_id = self.canvas.create_rectangle(self.WIDTH//2 - pw/2, self.HEIGHT - 120, self.WIDTH//2 + pw/2, self.HEIGHT - 90, fill="cyan")
        ix = random.randint(100, self.WIDTH-100)
        self.itm_id = self.canvas.create_oval(ix-20, 50, ix+20, 90, fill="orange")
        
        self.root.bind("<Motion>", lambda e: self.canvas.coords(self.pad_id, e.x - pw/2, self.HEIGHT - 120, e.x + pw/2, self.HEIGHT - 90))
            
        def drop_item():
            if not self.game_active: return
            self.canvas.move(self.itm_id, 0, 15 * self.speed)
            c, pc = self.canvas.coords(self.itm_id), self.canvas.coords(self.pad_id)
            if c and pc:
                if c[3] >= pc[1] and pc[0] <= c[2] and c[0] <= pc[2]: return self.end_minigame(True)
                if c[3] >= self.HEIGHT: return self.end_minigame(False)
            self.root.after(max(15, int(30/self.speed)), drop_item)
        drop_item()
        self.set_time()

    # --- 미니게임 6: 리듬에 맞춰 멈춰라! ---
    def start_timing(self):
        self.clear(); self.game_active = True; self.current_game_id = "timing"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 150, text="스페이스바로 초록색에 멈춰라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))
        
        tz_w = max(40, 150 - (self.speed * 10))
        tx1, tx2 = self.WIDTH/2 - tz_w, self.WIDTH/2 + tz_w
        self.canvas.create_rectangle(tx1, self.HEIGHT/2 - 60, tx2, self.HEIGHT/2 + 60, fill="green", outline="white", width=4)
        
        self.bar_id = self.canvas.create_rectangle(0, self.HEIGHT/2 - 80, 20, self.HEIGHT/2 + 80, fill="red")
        self.bar_pos, self.bar_dir = 0, 1
        self.bar_spd = min(40, 20 * self.speed)
        
        def move_bar():
            if not self.game_active: return
            self.bar_pos += self.bar_dir * self.bar_spd
            if self.bar_pos > self.WIDTH or self.bar_pos < 0: self.bar_dir *= -1
            self.canvas.coords(self.bar_id, self.bar_pos - 10, self.HEIGHT/2 - 80, self.bar_pos + 10, self.HEIGHT/2 + 80)
            self.root.after(30, move_bar)
            
        def on_space(e):
            if e.keysym.lower() == 'space': self.end_minigame(tx1 <= self.bar_pos <= tx2)
        self.root.bind("<Key>", on_space)
        move_bar()
        self.set_time()

    # --- 미니게임 7: 다른 색 찾기 ---
    def start_find_different(self):
        self.clear(); self.game_active = True; self.current_game_id = "find"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 300, text="다른 색을 찾아라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        
        grid = min(8, 4 + int(self.stage / 3)) 
        cw, pad = 50, 10
        tw = grid * cw + (grid - 1) * pad
        sx, sy = self.WIDTH/2 - tw/2, self.HEIGHT/2 - tw/2 + 50
        
        r, g, b = random.randint(50, 200), random.randint(50, 200), random.randint(50, 200)
        diff = max(8, int(30 / self.speed))
        c1 = f"#{r:02x}{g:02x}{b:02x}"
        c2 = f"#{min(255,r+diff):02x}{min(255,g+diff):02x}{min(255,b+diff):02x}"
        
        tr, tc = random.randint(0, grid-1), random.randint(0, grid-1)
        self.circles = []
        for row in range(grid):
            for col in range(grid):
                x, y = sx + col * (cw + pad), sy + row * (cw + pad)
                color = c2 if (row == tr and col == tc) else c1
                cid = self.canvas.create_oval(x, y, x+cw, y+cw, fill=color, outline="")
                self.circles.append((cid, row == tr and col == tc))
                
        def on_click(e):
            for cid, is_tgt in self.circles:
                c = self.canvas.coords(cid)
                if c and c[0] <= e.x <= c[2] and c[1] <= e.y <= c[3]: return self.end_minigame(is_tgt)
        self.canvas.bind("<Button-1>", on_click)
        self.set_time()

    # --- 미니게임 8: 수학 계산 ---
    def start_math(self):
        self.clear(); self.game_active = True; self.current_game_id = "math"
        a, b = random.randint(1, 4), random.randint(1, 4)
        ans = str(a + b)
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 100, text="계산해라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 + 50, text=f"{a} + {b} = ?", fill="white", font=("Arial", 100, "bold"))
        
        def on_key(e):
            if e.char.isdigit(): self.end_minigame(e.char == ans)
        self.root.bind("<Key>", on_key)
        self.set_time()

    # --- 미니게임 9: 가만히 있어라! ---
    def start_dont_move(self):
        self.clear(); self.game_active = True; self.current_game_id = "dont_move"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="마우스를 건드리지 마라!", fill="yellow", font=("Arial", 80, "bold"))
        self.start_x, self.start_y = self.root.winfo_pointerx(), self.root.winfo_pointery()
        
        def check_move():
            if not self.game_active: return
            cx, cy = self.root.winfo_pointerx(), self.root.winfo_pointery()
            if abs(cx - self.start_x) > 20 or abs(cy - self.start_y) > 20: return self.end_minigame(False)
            self.root.after(50, check_move)
        self.root.after(500, check_move) 
        self.set_time()

    # --- 미니게임 10: 마구 클릭해라! ---
    def start_spam_click(self):
        self.clear(); self.game_active = True; self.current_game_id = "spam"
        target = int(12 + (self.speed * 3))
        self.clicks = 0
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 150, text="마구 클릭해라!", fill="yellow", font=("Arial", 60, "bold"))
        bw, bh = 400, 200
        cx, cy = self.WIDTH/2 - bw/2, self.HEIGHT/2
        btn = self.canvas.create_rectangle(cx, cy, cx+bw, cy+bh, fill="purple", outline="white", width=5)
        txt = self.canvas.create_text(cx+bw/2, cy+bh/2, text=f"0 / {target}", fill="white", font=("Arial", 50, "bold"))
        
        def on_click(e):
            if cx <= e.x <= cx+bw and cy <= e.y <= cy+bh:
                self.clicks += 1
                self.canvas.itemconfig(txt, text=f"{self.clicks} / {target}")
                if self.clicks >= target: self.end_minigame(True)
        self.canvas.bind("<Button-1>", on_click)
        self.set_time()

   # --- 미니게임 11: 단어 입력 (견고하게 수정됨) ---
    def start_type_word(self):
        self.clear()
        self.game_active = True
        self.current_game_id = "type"
        
        # 단어 생성
        word = "".join(random.choices("ABCDEFGHIJKLMNOPQRSTUVWXYZ", k=3))
        self.typed = ""
        
        # 지시어 표시
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 100, text="키보드로 쳐라!", fill="yellow", font=("Arial", 60, "bold"))
        
        # 단어 표시 (텍스트 객체를 확실하게 생성)
        txt_id = self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 + 50, text=word, fill="white", font=("Arial", 100, "bold"))
        
        def on_key(e):
            if not self.game_active: return
            char = e.char.upper()
            
            # 입력값 검증 (알파벳만 통과)
            if not char or not char.isalpha() or len(char) != 1: return
            
            if char == word[len(self.typed)]:
                self.typed += char
                # 남은 글자 표시
                remaining = word[len(self.typed):]
                self.canvas.itemconfig(txt_id, text=self.typed + remaining, fill="green")
                
                if len(self.typed) == len(word):
                    self.end_minigame(True)
            else:
                self.end_minigame(False)
                
        self.root.bind("<Key>", on_key)
        self.set_time()

    # --- 미니게임 12: 레이저 피하기 (WASD 조작 도입) ---
    def start_laser(self):
        self.clear(); self.game_active = True; self.current_game_id = "laser"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 250, text="WASD로 피하라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))
        
        px, py = self.WIDTH/2, self.HEIGHT/2
        player = self.canvas.create_rectangle(px-20, py-20, px+20, py+20, fill="cyan", outline="white", width=2, tags="player")
        
        self.keys_pressed = set()
        def key_press(e):
            self.keys_pressed.add(e.keysym.lower())
        def key_release(e):
            self.keys_pressed.discard(e.keysym.lower())
        self.root.bind("<KeyPress>", key_press)
        self.root.bind("<KeyRelease>", key_release)
        
        def move_player():
            if not self.game_active or self.current_game_id != "laser": return
            dx, dy = 0, 0
            if 'w' in self.keys_pressed: dy -= 15 * self.speed
            if 's' in self.keys_pressed: dy += 15 * self.speed
            if 'a' in self.keys_pressed: dx -= 15 * self.speed
            if 'd' in self.keys_pressed: dx += 15 * self.speed
            
            c = self.canvas.coords(player)
            if c:
                if c[0] + dx < 0 or c[2] + dx > self.WIDTH: dx = 0
                if c[1] + dy < 0 or c[3] + dy > self.HEIGHT: dy = 0
                self.canvas.move(player, dx, dy)
            self.root.after(30, move_player)
        move_player()
        
        self.lasers = []
        def spawn_laser():
            if not self.game_active or self.current_game_id != "laser": return
            is_horiz = random.choice([True, False])
            if is_horiz:
                y = random.randint(100, self.HEIGHT-100)
                lid = self.canvas.create_rectangle(0, y-25, self.WIDTH, y+25, fill="", outline="red", dash=(10,10), width=3)
                self.lasers.append({"id": lid, "active": False})
            else:
                x = random.randint(100, self.WIDTH-100)
                lid = self.canvas.create_rectangle(x-25, 0, x+25, self.HEIGHT, fill="", outline="red", dash=(10,10), width=3)
                self.lasers.append({"id": lid, "active": False})
                
            self.root.after(max(400, int(1200/self.speed)), lambda: activate_laser(lid))
            self.root.after(max(400, int(1500/self.speed)), spawn_laser)
            
        def activate_laser(lid):
            if not self.game_active: return
            self.canvas.itemconfig(lid, fill="red", dash=(), outline="")
            for l in self.lasers:
                if l["id"] == lid:
                    l["active"] = True
            self.root.after(300, lambda: remove_laser(lid))
            
        def remove_laser(lid):
            if not self.game_active: return
            self.canvas.delete(lid)
            self.lasers = [l for l in self.lasers if l["id"] != lid]
            
        def check_collision():
            if not self.game_active: return
            pc = self.canvas.coords(player)
            if pc:
                for l in self.lasers:
                    if l["active"]:
                        lc = self.canvas.coords(l["id"])
                        if lc and not (pc[2] < lc[0] or pc[0] > lc[2] or pc[3] < lc[1] or pc[1] > lc[3]):
                            return self.end_minigame(False)
            self.root.after(50, check_collision)
            
        spawn_laser()
        check_collision()
        self.set_time()

    # --- 미니게임 13: 기억해라! ---
    def start_hidden_target(self):
        self.clear(); self.game_active = True; self.current_game_id = "hidden"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2 - 250, text="위치를 기억해라!", fill="yellow", font=("Arial", 60, "bold"))
        
        x, y = random.randint(100, self.WIDTH-150), random.randint(150, self.HEIGHT-250)
        box = self.canvas.create_rectangle(x, y, x+150, y+150, fill="#0099ff", outline="")
        
        def hide():
            if self.game_active: self.canvas.itemconfig(box, fill="#1e1e1e", outline="#1e1e1e")
        self.root.after(max(400, int(1500/self.speed)), hide) 
        
        def on_click(e):
            if x <= e.x <= x+150 and y <= e.y <= y+150: self.end_minigame(True)
            else: self.end_minigame(False)
        self.canvas.bind("<Button-1>", on_click)
        self.set_time()

    # --- 미니게임 14: 화면 닦기 ---
    def start_clean_screen(self):
        self.clear(); self.game_active = True; self.current_game_id = "clean"
        self.canvas.create_text(self.WIDTH/2, self.HEIGHT/2, text="마우스로 닦아라!", fill="yellow", font=("Arial", 60, "bold"), tags="inst")
        self.root.after(max(200, int(1000/self.speed)), lambda: self.canvas.delete("inst"))
        
        self.dirts = []
        for _ in range(5):
            x, y = random.randint(100, self.WIDTH-100), random.randint(100, self.HEIGHT-150)
            did = self.canvas.create_oval(x-40, y-40, x+40, y+40, fill="#3b2b1a", outline="")
            self.dirts.append(did)
            
        def on_motion(e):
            for did in self.dirts[:]:
                c = self.canvas.coords(did)
                if c and c[0] <= e.x <= c[2] and c[1] <= e.y <= c[3]:
                    self.canvas.delete(did)
                    self.dirts.remove(did)
            if not self.dirts: self.end_minigame(True)
        self.root.bind("<Motion>", on_motion)
        self.set_time()

    # --- 15 스테이지 보스전 리메이크 (난이도 완화) ---
    def start_boss(self):
        self.clear()
        self.game_active = True
        self.current_game_id = "boss"
        self.time_left = 20.0
        self.boss_hits = 0
        self.boss_size = 200
        
        self.canvas.create_text(self.WIDTH/2, 80, text="BOSS! Click 10 Times! (총알 닿으면 사망)", fill="red", font=("Arial", 40, "bold"))
        self.hits_txt = self.canvas.create_text(self.WIDTH/2, 130, text="Hits: 0 / 10", fill="white", font=("Arial", 30))
        
        self.boss_x, self.boss_y = self.WIDTH/2, self.HEIGHT/2
        self.boss_id = self.canvas.create_oval(self.boss_x - self.boss_size/2, self.boss_y - self.boss_size/2, 
                                                self.boss_x + self.boss_size/2, self.boss_y + self.boss_size/2, fill="#880000", outline="red", width=5)
        
        self.bullets = []
        
        def on_click(e):
            if not self.game_active: return
            c = self.canvas.coords(self.boss_id)
            if c and c[0] <= e.x <= c[2] and c[1] <= e.y <= c[3]:
                self.boss_hits += 1
                self.canvas.itemconfig(self.hits_txt, text=f"Hits: {self.boss_hits} / 10")
                self.canvas.itemconfig(self.boss_id, fill="white")
                self.root.after(50, lambda: self.canvas.itemconfig(self.boss_id, fill="#880000"))
                
                if self.boss_hits >= 10:
                    self.end_minigame(True)
                else:
                    self.boss_x = random.randint(200, self.WIDTH-200)
                    self.boss_y = random.randint(250, self.HEIGHT-250)
                    self.canvas.coords(self.boss_id, self.boss_x - self.boss_size/2, self.boss_y - self.boss_size/2, 
                                       self.boss_x + self.boss_size/2, self.boss_y + self.boss_size/2)
                    
        self.canvas.bind("<Button-1>", on_click)
        
        def on_motion(e):
            if not self.game_active: return
            for b in self.bullets:
                c = self.canvas.coords(b["id"])
                if c and c[0] <= e.x <= c[2] and c[1] <= e.y <= c[3]:
                    self.end_minigame(False)
        self.root.bind("<Motion>", on_motion)

        def update_boss():
            if not self.game_active or self.current_game_id != "boss": return
            
            if random.random() < 0.05 * self.speed:
                ang = random.uniform(0, math.pi * 2)
                spd = random.randint(10, 15) * self.speed
                bid = self.canvas.create_oval(self.boss_x-10, self.boss_y-10, self.boss_x+10, self.boss_y+10, fill="orange", outline="")
                self.bullets.append({"id": bid, "dx": math.cos(ang)*spd, "dy": math.sin(ang)*spd})
            
            for b in self.bullets[:]:
                self.canvas.move(b["id"], b["dx"], b["dy"])
                c = self.canvas.coords(b["id"])
                if c and (c[2] < 0 or c[0] > self.WIDTH or c[3] < 0 or c[1] > self.HEIGHT):
                    self.canvas.delete(b["id"])
                    self.bullets.remove(b)
                    
            self.root.after(30, update_boss)

        update_boss()
        self.tick_timer()

if __name__ == "__main__":
    root = tk.Tk()
    app = MicroGameApp(root)
    root.mainloop()
