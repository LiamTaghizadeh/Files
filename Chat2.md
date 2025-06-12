حتماً! در ادامه نسخه‌ی کامل و حرفه‌ای برنامه چت با سوکت رو می‌نویسم که همه‌ی قابلیت‌هایی که گفتی رو داره. این نسخه بدون نیاز به پایگاه داده کار می‌کنه، ولی در آینده می‌شه به راحتی به SQLite یا فایل اضافه کرد.

⸻

✅ لیست قابلیت‌ها:

قابلیت	توضیح
ثبت‌نام و ورود	کاربران با نام کاربری و رمز ثبت‌نام یا وارد می‌شوند.
چت عمومی	ارسال پیام به همه کاربران آنلاین
چت گروهی	عضویت در گروه و ارسال پیام در گروه خاص
پیام خصوصی	ارسال پیام مستقیم به کاربر دیگر
مشاهده کاربران آنلاین	لیست تمام کاربران آنلاین
راهنمای دستورات	دستور /help برای راهنمایی کاربر


⸻

📁 ساختار پروژه

chat_advanced/
├── server.py
└── client.py


⸻

🔧 server.py – سرور پیشرفته

import socket
import threading

HOST = '127.0.0.1'
PORT = 5001

clients = {}         # {sock: username}
user_sockets = {}    # {username: sock}
credentials = {}     # {username: password}
groups = {}          # {group_name: set(usernames)}

def broadcast(msg, exclude=None, group=None):
    for client_sock, username in clients.items():
        if client_sock != exclude:
            if group:
                if username in groups.get(group, set()):
                    try:
                        client_sock.send(msg.encode())
                    except:
                        pass
            else:
                try:
                    client_sock.send(msg.encode())
                except:
                    pass

def handle_client(sock):
    sock.send("📋 لطفاً وارد شوید یا ثبت‌نام کنید:\n/login USER PASS\n/register USER PASS\n".encode())
    username = None

    while not username:
        data = sock.recv(1024).decode()
        if data.startswith("/login "):
            try:
                _, user, pwd = data.strip().split()
                if credentials.get(user) == pwd:
                    username = user
                    sock.send(f"✅ خوش آمدی {user}\n".encode())
                else:
                    sock.send("❌ اطلاعات ورود اشتباهه.\n".encode())
            except:
                sock.send("❌ فرمت اشتباه. مثال: /login ali 1234\n".encode())
        elif data.startswith("/register "):
            try:
                _, user, pwd = data.strip().split()
                if user in credentials:
                    sock.send("❌ این نام کاربری قبلاً ثبت شده.\n".encode())
                else:
                    credentials[user] = pwd
                    username = user
                    sock.send(f"✅ ثبت‌نام موفق. خوش آمدی {user}\n".encode())
            except:
                sock.send("❌ فرمت اشتباه. مثال: /register ali 1234\n".encode())
        else:
            sock.send("❗ لطفاً اول وارد شوید یا ثبت‌نام کنید.\n".encode())

    clients[sock] = username
    user_sockets[username] = sock
    broadcast(f"📢 {username} آنلاین شد.", exclude=sock)
    sock.send("✅ دستور `/help` برای لیست دستورات در دسترس است.\n".encode())

    while True:
        try:
            msg = sock.recv(1024).decode()
            if msg.lower() == "exit":
                break

            if msg.startswith("/help"):
                help_text = (
                    "\n🛠️ دستورات:\n"
                    "/group GROUP_NAME               → عضویت در گروه\n"
                    "/msg GROUP_NAME MESSAGE         → ارسال پیام به گروه\n"
                    "/pm USER MESSAGE                → ارسال پیام خصوصی\n"
                    "/users                          → نمایش کاربران آنلاین\n"
                    "/help                           → راهنمای دستورات\n"
                    "exit                            → خروج\n"
                )
                sock.send(help_text.encode())

            elif msg.startswith("/group "):
                _, gname = msg.split(maxsplit=1)
                groups.setdefault(gname, set()).add(username)
                sock.send(f"✅ شما عضو گروه {gname} شدید.\n".encode())

            elif msg.startswith("/msg "):
                parts = msg.split()
                if len(parts) < 3:
                    sock.send("❌ فرمت اشتباه. مثال: /msg group_name پیام شما\n".encode())
                    continue
                gname = parts[1]
                text = ' '.join(parts[2:])
                if username not in groups.get(gname, set()):
                    sock.send("⛔ شما عضو این گروه نیستید.\n".encode())
                else:
                    broadcast(f"[{username} in {gname}]: {text}", exclude=sock, group=gname)

            elif msg.startswith("/pm "):
                parts = msg.split()
                if len(parts) < 3:
                    sock.send("❌ فرمت اشتباه. مثال: /pm user پیام\n".encode())
                    continue
                target_user = parts[1]
                pm_text = ' '.join(parts[2:])
                target_sock = user_sockets.get(target_user)
                if target_sock:
                    target_sock.send(f"[📩 پیام خصوصی از {username}]: {pm_text}".encode())
                    sock.send("✅ پیام خصوصی ارسال شد.\n".encode())
                else:
                    sock.send("❌ کاربر مورد نظر آنلاین نیست.\n".encode())

            elif msg.startswith("/users"):
                online = ', '.join(user_sockets.keys())
                sock.send(f"🟢 کاربران آنلاین: {online}\n".encode())

            else:
                broadcast(f"[{username}]: {msg}", exclude=sock)

        except:
            break

    # Cleanup on disconnect
    print(f"❌ {username} قطع اتصال.")
    broadcast(f"📴 {username} از چت خارج شد.", exclude=sock)
    del clients[sock]
    del user_sockets[username]
    sock.close()

def start():
    server = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    server.bind((HOST, PORT))
    server.listen()
    print(f"🚀 سرور در حال اجرا روی {HOST}:{PORT}")

    while True:
        client_sock, _ = server.accept()
        threading.Thread(target=handle_client, args=(client_sock,), daemon=True).start()

if __name__ == "__main__":
    start()


⸻

💬 client.py – کلاینت CLI حرفه‌ای

import socket
import threading

HOST = '127.0.0.1'
PORT = 5001

def receive(sock):
    while True:
        try:
            data = sock.recv(1024).decode()
            if data:
                print("\n" + data + "\n> ", end='')
        except:
            print("❌ اتصال با سرور قطع شد.")
            break

def main():
    sock = socket.socket(socket.AF_INET, socket.SOCK_STREAM)
    sock.connect((HOST, PORT))

    threading.Thread(target=receive, args=(sock,), daemon=True).start()

    while True:
        msg = input("> ")
        if msg.strip().lower() == "exit":
            sock.send(b"exit")
            break
        sock.send(msg.encode())

    sock.close()

if __name__ == "__main__":
    main()


⸻

🎮 نحوه اجرا:
	1.	اجرای سرور در ترمینال اول:

python server.py

	2.	اجرای کلاینت در چند ترمینال دیگر:

python client.py


⸻

✨ مثال‌هایی از استفاده در کلاینت:

/register ali 1234
/login ali 1234
/group devs
/msg devs سلام بچه‌ها
/pm reza سلام خصوصی!
/users
/help
exit


⸻

اگر بخوای مرحله بعدی رو هم انجام بدیم (مثل ذخیره کاربران و پیام‌ها توی فایل یا دیتابیس، یا ساخت رابط گرافیکی) فقط بگو تا گام بعدی رو برات پیاده‌سازی کنم.
