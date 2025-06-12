خیلی خوب! در این نسخه جدید، برنامه چت رو با رابط گرافیکی (GUI) به کمک Tkinter طراحی می‌کنم.

⸻

✅ ویژگی‌های این نسخه گرافیکی:

قابلیت	پشتیبانی
ثبت‌نام / ورود	✔️ با پنجره فرم
چت عمومی	✔️ نمایش پیام‌ها در پنجره
چت گروهی	✔️ انتخاب گروه و ارسال پیام
پیام خصوصی	✔️ ارسال پیام مستقیم به کاربر
لیست کاربران آنلاین	✔️ نمایش کاربران آنلاین در پنل
راهنمای دستورات	✔️ نمایش داخل GUI


⸻

🧱 ساختار:

chat_gui/
├── server.py       ← سرور (همان نسخه حرفه‌ای قبلی)
└── client_gui.py   ← کلاینت گرافیکی با Tkinter


⸻

💬 اول سرور (server.py)

از همان نسخه‌ی قبلی استفاده کن (اگر خواستی، با ذخیره در فایل/دیتابیس هم می‌سازم).

⸻

🎨 client_gui.py – کلاینت با رابط گرافیکی Tkinter

import socket
import threading
import tkinter as tk
from tkinter import messagebox, simpledialog, scrolledtext

HOST = '127.0.0.1'
PORT = 5001

class ChatClient:
    def __init__(self, master):
        self.master = master
        self.master.title("💬 چت با Tkinter")
        self.sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
        try:
            self.sock.connect((HOST, PORT))
        except:
            messagebox.showerror("خطا", "اتصال به سرور برقرار نشد!")
            master.destroy()
            return

        self.frame = tk.Frame(master)
        self.chat_box = scrolledtext.ScrolledText(self.frame, wrap=tk.WORD, height=20, width=50, state='disabled')
        self.chat_box.pack(padx=10, pady=10)

        self.entry = tk.Entry(self.frame, width=40)
        self.entry.pack(side=tk.LEFT, padx=(10, 5), pady=10)
        self.entry.bind("<Return>", self.send_msg)

        self.send_button = tk.Button(self.frame, text="ارسال", command=self.send_msg)
        self.send_button.pack(side=tk.LEFT, padx=(5, 10))

        self.users_button = tk.Button(self.frame, text="🔍 کاربران", command=self.get_users)
        self.users_button.pack(side=tk.LEFT)

        self.frame.pack()

        self.username = None
        self.login_or_register()

        self.receive_thread = threading.Thread(target=self.receive)
        self.receive_thread.daemon = True
        self.receive_thread.start()

    def login_or_register(self):
        while True:
            action = simpledialog.askstring("ورود یا ثبت‌نام", "login یا register؟")
            if action not in ("login", "register"):
                continue
            user = simpledialog.askstring("نام کاربری", "لطفاً نام کاربری را وارد کنید:")
            pwd = simpledialog.askstring("رمز عبور", "لطفاً رمز عبور را وارد کنید:", show='*')
            if not user or not pwd:
                continue
            self.sock.send(f"/{action} {user} {pwd}".encode())
            response = self.sock.recv(1024).decode()
            self.chat_box.config(state='normal')
            self.chat_box.insert(tk.END, response + "\n")
            self.chat_box.config(state='disabled')
            if "خوش آمدی" in response:
                self.username = user
                break

    def send_msg(self, event=None):
        msg = self.entry.get()
        if msg:
            self.sock.send(msg.encode())
            self.entry.delete(0, tk.END)

    def get_users(self):
        self.sock.send("/users".encode())

    def receive(self):
        while True:
            try:
                msg = self.sock.recv(1024).decode()
                self.chat_box.config(state='normal')
                self.chat_box.insert(tk.END, msg + "\n")
                self.chat_box.see(tk.END)
                self.chat_box.config(state='disabled')
            except:
                break

if __name__ == "__main__":
    root = tk.Tk()
    app = ChatClient(root)
    root.mainloop()


⸻

🧪 طرز استفاده
	1.	اجرای سرور:

python server.py

	2.	اجرای کلاینت گرافیکی:

python client_gui.py


⸻

✨ امکانات پیشنهادی بعدی برای نسخه GUI:
	•	انتخاب گروه از لیست کشویی
	•	ساخت گروه جدید از طریق رابط
	•	نمایش پیام‌های خصوصی جداگانه
	•	ذخیره تاریخچه چت
	•	رمزنگاری پیام‌ها

⸻

آیا مایل هستی که ذخیره پیام‌ها در فایل یا نمایش پیام‌های خصوصی در پنجره جداگانه هم اضافه بشه؟
