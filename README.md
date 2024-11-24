import tkinter as tk
import random

# シンボルのリスト（スロットのリールに表示されるもの）
symbols = ['７', '🍋', '🍊', '🍉', '？', '🍓']

class SlotMachineGame:
    def __init__(self, root):
        self.root = root
        self.root.title("スロットゲーム")
        self.root.configure(bg="lightgray")
        
        # 初期所持金と掛け金
        self.balance = 1000  # プレイヤーの所持金
        self.bet = 10  # 初期掛け金
        
        # 所持金表示ラベル
        self.balance_label = tk.Label(root, text=f"所持金: ¥{self.balance}", font=("Arial", 20), bg="lightgray")
        self.balance_label.grid(row=0, column=0, columnspan=3, pady=10)

        # リールのラベルを作成
        self.reels = [tk.Label(root, text="７", font=("Arial", 40), width=6, height=2, relief="solid", bg="white") for _ in range(3)]
        for i, reel in enumerate(self.reels):
            reel.grid(row=1, column=i, padx=20, pady=20)

        # スピンボタン
        self.spin_button = tk.Button(root, text="スピン!", font=("Arial", 24), width=10, height=2, bg="#4CAF50", fg="white", command=self.start_spin)
        self.spin_button.grid(row=2, column=0, columnspan=3, pady=20)
        
        # 各リールのストップボタン
        self.stop_buttons = []
        for i in range(3):
            stop_button = tk.Button(root, text=f"リール{i+1} ストップ", font=("Arial", 18), width=15, height=2, bg="#f44336", fg="white", command=lambda i=i: self.stop_reel(i))
            stop_button.grid(row=3, column=i, pady=10)
            stop_button.config(state="disabled")  # 初期状態で無効化
            self.stop_buttons.append(stop_button)
        
        # 再チャレンジボタン
        self.retry_button = tk.Button(root, text="再チャレンジ", font=("Arial", 18), width=15, height=2, bg="#009688", fg="white", command=self.reset_game)
        self.retry_button.grid(row=4, column=0, columnspan=3, pady=20)
        self.retry_button.config(state="disabled")  # 初期状態で無効化
        
        # 掛け金の設定ボタン
        self.bet_label = tk.Label(root, text=f"掛け金: ¥{self.bet}", font=("Arial", 20), bg="lightgray")
        self.bet_label.grid(row=5, column=0, columnspan=3, pady=10)
        
        self.increase_bet_button = tk.Button(root, text="掛け金 +¥10", font=("Arial", 18), command=self.increase_bet)
        self.increase_bet_button.grid(row=6, column=0, padx=10, pady=10)

        self.decrease_bet_button = tk.Button(root, text="掛け金 -¥10", font=("Arial", 18), command=self.decrease_bet)
        self.decrease_bet_button.grid(row=6, column=1, padx=10, pady=10)

        # 状態のフラグ
        self.is_spinning = False
        self.final_reels = []  # 最終的なリールの結果
        self.stopped_reels = [False, False, False]  # 各リールが停止したかどうかを追跡
        self.result_label = None  # 結果メッセージ用のラベルを初期化

    def start_spin(self):
        if self.is_spinning or self.balance < self.bet:
            return  # すでに回転中か、所持金が足りない場合は何もしない
        
        # 掛け金を所持金から差し引く
        self.balance -= self.bet
        self.balance_label.config(text=f"所持金: ¥{self.balance}")
        
        self.is_spinning = True
        self.stopped_reels = [False, False, False]  # 全リールを再度回転させる
        self.final_reels = []  # 最終的なリールの結果をリセット

        # リールのシンボルを順番に設定（周期的に繰り返す）
        self.reel_data = [[symbols[i % len(symbols)] for i in range(6)] for _ in range(3)]  # 6つのシンボルを繰り返し
        for i in range(3):
            self.reels[i].config(text=self.reel_data[i][0])  # 初期状態で1つ目のシンボルを表示

        # スピンの演出を開始
        self.spin_reels_animation()

        # スピン後に最終結果を決める
        self.root.after(2000, self.check_all_stopped)  # 2秒後に全てのリールが停止したか確認
        
        # スピンボタンを無効化
        self.spin_button.config(state="disabled")
        # ストップボタンを有効化
        for stop_button in self.stop_buttons:
            stop_button.config(state="normal")
        # 再チャレンジボタンを無効化
        self.retry_button.config(state="disabled")

    def spin_reels_animation(self):
        """リールが周期的に回転する演出を行う"""
        if self.is_spinning:
            for i in range(3):
                if not self.stopped_reels[i]:  # このリールがまだ停止していない場合
                    # リールのシンボルを1つずつ移動させる（周期的にスクロール）
                    self.reel_data[i].append(self.reel_data[i].pop(0))  # 最初のシンボルを末尾に移動
                    self.reels[i].config(text=self.reel_data[i][5])  # 最後のシンボルを表示
            self.root.after(1000, self.spin_reels_animation)  # 100ミリ秒ごとにシンボルを更新

    def stop_reel(self, reel_index):
        """特定のリールを停止する"""
        if self.is_spinning and not self.stopped_reels[reel_index]:
            self.stopped_reels[reel_index] = True  # リールを停止
            self.stop_buttons[reel_index].config(state="disabled")  # ストップボタンを無効化

            # 全てのリールが停止したかチェック
            self.check_all_stopped()

    def check_all_stopped(self):
        """全てのリールが停止したか確認し、停止後の処理を行う"""
        if all(self.stopped_reels):
            # 全てのリールが停止した場合
            self.final_reels = [self.reels[i].cget("text") for i in range(3)]  # 最終的なリールの結果
            self.check_win(self.final_reels)  # 勝敗をチェック
            self.spin_button.config(state="normal")  # スピンボタンを再度有効化
            self.retry_button.config(state="normal")  # 再チャレンジボタンを有効化
            self.is_spinning = False  # スピン中のフラグをリセット

    def check_win(self, reels):
        """リールの結果をチェックして、勝敗を決める"""
        # ジャックポットの条件（例：3つの🍒）
        if reels[0] == reels[1] == reels[2] == '７':
            jackpot = self.bet * 10  # 例えば掛け金の10倍の賞金
            self.balance += jackpot
            self.display_result(f"ジャックポット！７７７\n¥{jackpot}の賞金獲得！所持金: ¥{self.balance}")

        # ボーナスゲームの条件（例：🍇🍇🍇）
        elif reels[0] == reels[1] == reels[2] == '？':
            self.start_bonus_game()

        else:
            self.display_result("残念、ハズレ！")

    def start_bonus_game(self):
        """ボーナスゲームを開始"""
        bonus_amount = random.randint(50, 200)  # ランダムなボーナス賞金
        self.balance += bonus_amount
        self.display_result(f"ランダムなボーナス賞金を獲得！\n¥{bonus_amount}のボーナス賞金ゲット！所持金: ¥{self.balance}")

    def display_result(self, message):
        """ゲーム結果を画面に表示"""
        if self.result_label:
            self.result_label.grid_forget()  # 前の結果を消去
        self.result_label = tk.Label(self.root, text=message, font=("Arial", 18), bg="lightgray")
        self.result_label.grid(row=5, column=0, columnspan=3, pady=20)

    def reset_game(self):
        """ゲームをリセットして再チャレンジする"""
        self.stopped_reels = [False, False, False]
        for i in range(3):
            self.reels[i].config(text="７")
        self.spin_button.config(state="normal")
        for stop_button in self.stop_buttons:
            stop_button.config(state="disabled")
        self.retry_button.config(state="disabled")
        if self.result_label:
            self.result_label.grid_forget()  # 結果ラベルを消去
        self.balance = 1000  # 所持金をリセット
        self.balance_label.config(text=f"所持金: ¥{self.balance}")
        self.is_spinning = False  # スピン中のフラグをリセット

    def increase_bet(self):
        """掛け金を増やす"""
        if self.balance >= self.bet + 10:
            self.bet += 10
            self.bet_label.config(text=f"掛け金: ¥{self.bet}")

    def decrease_bet(self):
        """掛け金を減らす"""
        if self.bet > 10:
            self.bet -= 10
            self.bet_label.config(text=f"掛け金: ¥{self.bet}")

# Tkinterアプリケーションの作成
root = tk.Tk()
game = SlotMachineGame(root)
root.mainloop()
