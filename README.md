# jeu-de-l-oie
voici le prompt envoyer à l'iag chatgpt :
Fais un jeu de l’oie selon mes conditions : nom : oie-pixel, case pour saisir le nombre d'équipes jouant (entre 2 et 10) puis un bouton pour proposer de lancer le jeu. afficher la position des pions des joueurs sur le plateau et leur déplacements, afficher un ecran perdu lors d’une defaite et gagné lors d’une victoire, ajouter un compteur de score pour chaque joueurs, Pour windows, pour serveur web ,en python, couleurs : vertes bleu couleur retro, style retro gaming, le jeu doit comporter 63 cases avec des cases aléatoires comportantes : oie :avance de 2 cases, piège : recule de 3 cases et perd 1 point, crâne : retour au depart et defaite immediate, boost : +3 points, systeme de dès tour à tour equipe par equipe, ajouter le classement des joueurs au moment de l’écran fin


        x = MARGIN + col * CELL
        y = MARGIN + (6 - row_from_bottom) * CELL
        return x, y

    def draw_board(self):
        self.board.delete("all")
        self.board.create_text(BOARD_W/2, 8, text="★  START → FINISH  ★", fill=GREEN,
                               font=("Consolas", 9, "bold"), anchor="n")
        special = {}
        for n in [5, 9, 14, 18, 23, 27, 32, 36, 41, 45, 50, 54, 59]:
            special[n] = ("OIE", GREEN)
        for n in [12, 29, 47]:
            special[n] = ("BOOST", BLUE)
        for n in [19, 38, 56]:
            special[n] = ("PIÈGE", GOLD)
        for n in [31, 52]:
            special[n] = ("CRÂNE", RED)

        for n in range(1, FINISH + 1):
            x, y = self.cell_xy(n)
            x1, y1, x2, y2 = x + 3, y + 3, x + CELL - 3, y + CELL - 3
            label, color = special.get(n, (str(n), "#21434a"))
            fill = "#103b32" if label == "OIE" else ("#102f49" if label == "BOOST" else
                   "#403b21" if label == "PIÈGE" else "#452631" if label == "CRÂNE" else "#102a32")
            self.board.create_rectangle(x1, y1, x2, y2, fill=fill, outline=color, width=2)
            if n == FINISH:
                label, color = "🏁 FIN", GOLD
            self.board.create_text((x1+x2)/2, y1+14, text=label, fill=color,
                                   font=("Consolas", 8 if len(label) > 3 else 10, "bold"))
            if n == 1:
                self.board.create_text((x1+x2)/2, y2-10, text="DÉPART", fill=PALE,
                                       font=("Consolas", 7, "bold"))
        # Positions des pions
        positions = {}
        for i, p in enumerate(self.players):
            if p["pos"] > 0:
                positions.setdefault(p["pos"], []).append((i, p))
        for pos, items in positions.items():
            x, y = self.cell_xy(pos)
            for j, (i, p) in enumerate(items):
                offsets = [(-9, 7), (9, 7), (-9, 22), (9, 22), (0, 0),
                           (-15, 14), (15, 14), (0, 25), (-15, 25), (15, 25)]
                dx, dy = offsets[j % len(offsets)]
                cx, cy = x + CELL/2 + dx, y + CELL/2 + dy
                r = 7
                self.board.create_oval(cx-r, cy-r, cx+r, cy+r, fill=p["color"],
                                       outline="#ffffff", width=1)
                self.board.create_text(cx, cy, text=str(i+1), fill=BG,
                                       font=("Consolas", 7, "bold"))
        # Marqueur départ des joueurs non encore partis
        for i, p in enumerate(self.players):
            if p["pos"] == 0:
                self.board.create_text(MARGIN + 24 + (i % 5)*8, BOARD_H - 5,
                                       text=str(i+1), fill=p["color"], font=("Consolas", 7, "bold"), anchor="s")

    def refresh_scores(self):
        for i, p in enumerate(self.players):
            marker = "▶ " if i == self.turn and not self.finished else "  "
            self.score_labels[i].config(text=f"{marker}{p['name']}\n   CASE {p['pos']:02d}   SCORE {p['score']:03d}",
                                        fg=p["color"] if i == self.turn else PALE)

    def refresh_turn(self):
        if not self.players:
            return
        p = self.players[self.turn]
        self.turn_label.config(text=f"AU TOUR DE\n{p['name']}", fg=p["color"])
        self.message_label.config(text=self.last_message)
        self.refresh_scores()

    def roll_dice(self):
        if self.finished:
            return
        p = self.players[self.turn]
        roll = random.randint(1, 6)
        self.dice_value = roll
        self.dice_label.config(text=f"⚄  {roll}")
        old = p["pos"]
        target = old + roll
        if target > FINISH:
            target = FINISH - (target - FINISH)  # rebond sur la dernière case
        p["pos"] = target
        p["score"] += roll
        msg = f"{p['name']} lance {roll} : case {old} → case {target}."
        # Événements sur cases spéciales
        if target in [5, 9, 14, 18, 23, 27, 32, 36, 41, 45, 50, 54, 59]:
            before = p["pos"]
            p["pos"] = min(FINISH, p["pos"] + 2)
            p["score"] += 2
            msg += f"\nOIE ! Avance à la case {p['pos']} (+2 points)."
        elif target in [12, 29, 47]:
            p["score"] += 3
            msg += "\nBOOST ! +3 points bonus."
        elif target in [19, 38, 56]:
            p["pos"] = max(0, p["pos"] - 3)
            p["score"] = max(0, p["score"] - 1)
            msg += f"\nPIÈGE ! Recule à la case {p['pos']} (-1 point)."
        elif target in [31, 52]:
            p["pos"] = 0
            p["score"] = max(0, p["score"] - 5)
            msg += "\nCRÂNE ! Retour au départ (-5 points)."
        self.last_message = msg
        self.draw_board()
        self.refresh_scores()
        if p["pos"] >= FINISH:
            self.finished = True
            self.show_end(winner=p, defeated=False)
            return
        # Les cases Crâne déclenchent un écran de défaite.
        if target in [31, 52]:
            self.finished = True
            self.show_end(winner=None, defeated=True, player=p)
            return
        self.turn = (self.turn + 1) % len(self.players)
        self.refresh_turn()

    def show_end(self, winner=None, defeated=False, player=None):
        self.roll_button.config(state="disabled")
        overlay = tk.Frame(self.root, bg=BG, highlightbackground=RED if defeated else GREEN,
                           highlightthickness=5)
        overlay.place(relx=0.5, rely=0.5, anchor="center", width=560, height=350)
        if defeated:
            title, color = "GAME OVER", RED
            subtitle = f"{player['name']} a perdu tous ses points !"
        else:
            title, color = "VICTOIRE !", GREEN
            subtitle = f"{winner['name']} atteint la case 63 !"
        tk.Label(overlay, text="▓▒░  OIE-PIXEL  ░▒▓", bg=BG, fg=BLUE,
                 font=("Consolas", 13, "bold")).pack(pady=(30, 14))
        tk.Label(overlay, text=title, bg=BG, fg=color,
                 font=("Consolas", 32, "bold")).pack(pady=4)
        tk.Label(overlay, text=subtitle, bg=BG, fg=PALE,
                 font=("Consolas", 13, "bold"), wraplength=480).pack(pady=12)
        if winner:
            tk.Label(overlay, text=f"SCORE FINAL : {winner['score']} POINTS",
                     bg=BG, fg=GOLD, font=("Consolas", 14, "bold")).pack(pady=4)
        tk.Button(overlay, text="REJOUER", command=self.start_game, bg=GREEN, fg=BG,
                  font=("Consolas", 12, "bold"), relief="flat", padx=20, pady=9).pack(side="left", padx=(90, 10), pady=20)
        tk.Button(overlay, text="MENU", command=self.show_setup, bg=BLUE, fg=BG,
                  font=("Consolas", 12, "bold"), relief="flat", padx=20, pady=9).pack(side="left", padx=10, pady=20)


if __name__ == "__main__":
    root = tk.Tk()
    app = GooseGame(root)
    root.mainloop()
