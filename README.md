import tkinter as tk
from tkinter import ttk, messagebox, font, filedialog
from datetime import datetime

class ChatApp:
    def __init__(self, host='127.0.0.1', port=5555):
        self.server_addr = (host, port)
        self.sock = None
        self.sockfile = None

        self.conversations = {}
        self.current_chat_user = None

        self.root = tk.Tk()
        self.root.title("online chat")
        self.root.geometry("1000x700")  # ← تم تصغير الشاشة
        self.root.minsize(800, 600)

        self.custom_font = font.Font(family="Helvetica", size=12)
        self.title_font = font.Font(family="Helvetica", size=16, weight="bold")
        self.time_font = font.Font(family="Helvetica", size=9)

        self.setup_styles()
        self.setup_login_ui()
        self.root.mainloop()

    def setup_styles(self):
        st = ttk.Style()
        st.configure("TButton", font=self.custom_font, padding=10)
        st.configure("TEntry", font=self.custom_font, padding=8)
        st.configure("TLabel", font=self.custom_font)
        st.configure("TFrame", background="#f9f9f9")
        st.configure("Striped.Treeview", font=self.custom_font,
                    background="#ffffff", fieldbackground="#ffffff", rowheight=40)
        st.map("Treeview",
            background=[('selected', '#4CAF50')],
            foreground=[('selected', 'white')])

    def setup_login_ui(self):
        self.login_frame = ttk.Frame(self.root, padding=40)
        self.login_frame.place(relx=0.5, rely=0.5, anchor="center")

        ttk.Label(self.login_frame, text="online chat", font=self.title_font).grid(row=0, column=0, columnspan=2, pady=(0, 30))

        ttk.Label(self.login_frame, text="user name:").grid(row=1, column=0, sticky="e")
        self.username_entry = ttk.Entry(self.login_frame)
        self.username_entry.grid(row=1, column=1, pady=5, ipady=5)

        ttk.Label(self.login_frame, text="Password:").grid(row=2, column=0, sticky="e")
        self.password_entry = ttk.Entry(self.login_frame, show="*")
        self.password_entry.grid(row=2, column=1, pady=5, ipady=5)

        ttk.Button(self.login_frame, text="Log In", command=self.login).grid(row=3, column=0, columnspan=2, sticky="we", pady=15)

    def setup_chat_ui(self):
        self.login_frame.destroy()

        self.users_frame = ttk.LabelFrame(self.root, text="Online", width=250)
        self.users_frame.pack(side="left", fill="y", padx=10, pady=10)
        self.users_frame.columnconfigure(0, weight=1)
        self.users_frame.rowconfigure(1, weight=1)

        tree_frame = ttk.Frame(self.users_frame)
        tree_frame.grid(row=1, column=0, sticky="nsew", padx=5, pady=5)

        self.user_list = ttk.Treeview(tree_frame, columns=("username",), show="headings", style="Striped.Treeview")
        self.user_list.heading("username", text="Chat")
        self.user_list.column("username", anchor="center", width=150)

        scrollbar_u = ttk.Scrollbar(tree_frame, orient="vertical", command=self.user_list.yview)
        self.user_list.configure(yscrollcommand=scrollbar_u.set)
        self.user_list.pack(side="left", fill="both", expand=True)
        scrollbar_u.pack(side="right", fill="y")

        self.chat_frame = ttk.Frame(self.root)
        self.chat_frame.pack(side="right", expand=True, fill="both", padx=10, pady=10)
        self.chat_frame.rowconfigure(1, weight=1)
        self.chat_frame.columnconfigure(0, weight=1)

        self.chat_header = ttk.Label(self.chat_frame, text="", font=self.title_font, anchor="center", background="#ebedef")
        self.chat_header.grid(row=0, column=0, columnspan=3, sticky="ew")

        self.chat_canvas = tk.Canvas(self.chat_frame, bg="#ebedef", bd=0, highlightthickness=0)
        self.chat_canvas.grid(row=1, column=0, columnspan=3, sticky="nsew")
        scrollbar = ttk.Scrollbar(self.chat_frame, orient="vertical", command=self.chat_canvas.yview)
        scrollbar.grid(row=1, column=3, sticky="ns")
        self.chat_canvas.configure(yscrollcommand=scrollbar.set)

        self.chat_container = tk.Frame(self.chat_canvas, bg="#ebedef")
        self.chat_canvas.create_window((0, 0), window=self.chat_container, anchor="nw")
        self.chat_container.bind("<Configure>", lambda e: self.chat_canvas.configure(scrollregion=self.chat_canvas.bbox("all")))

        self.send_frame = ttk.Frame(self.chat_frame)
        self.send_frame.grid(row=2, column=0, columnspan=3, sticky="ew", pady=5)
        self.send_frame.columnconfigure(0, weight=1)

        self.msg_entry = ttk.Entry(self.send_frame)
        self.msg_entry.grid(row=0, column=0, sticky="ew", ipady=8, padx=(5, 2))

        send_button = tk.Button(
            self.send_frame,
            text="➤",
            font=(None, 14),
            bd=0,
            bg="#25D366",
            fg="white",
            activebackground="#128C7E",
            activeforeground="white",
            command=self.send_message
        )
        send_button.grid(row=0, column=1, sticky="e", padx=(2, 5))

        file_button = tk.Button(
            self.send_frame,
            text="📎",
            font=(None, 14),
            bd=0,
            bg="#3b5998",
            fg="white",
            activebackground="#2d4373",
            activeforeground="white",
            command=self.send_file
        )
        file_button.grid(row=0, column=2, sticky="e", padx=(2, 5))

        self.msg_entry.bind("<Return>", self.send_message)
        self.msg_entry.focus_set()

        self.update_users(["Amr", "Joumana", "Basmala", "mariam", "Doaa", "Asmaa"])
        
        
        
        
        
        file_button = tk.Button(
            self.send_frame,
            text="📎",
            font=(None, 14),
            bd=0,
            bg="#3b5998",
            fg="white",
            activebackground="#2d4373",
            activeforeground="white",
            command=self.send_file
)
        file_button.grid(row=0, column=2, sticky="e", padx=(2, 5))
        self.msg_entry.bind("<Return>", self.send_message)
        self.msg_entry.focus_set()
        self.update_users(["Amr", "Joumana", "Basmala", "mariam", "Doaa", "Asmaa"])
        
        

    def login(self):
        username = self.username_entry.get().strip()
        password = self.password_entry.get().strip()

        allowed_users = ["Amr", "Joumana", "Basmala", "mariam", "Doaa", "Asmaa"]
        allowed_password = "1234"

        if username not in allowed_users or password != allowed_password:
            messagebox.showerror("تم الرفض", "بيانات الدخول غير صحيحة")
            return

        self.username = username
        self.setup_chat_ui()

    def update_users(self, users):
        self.user_list.delete(*self.user_list.get_children())
        for idx, user in enumerate(users):
            tag = 'evenrow' if idx % 2 == 0 else 'oddrow'
            self.user_list.insert("", "end", values=(user,), tags=(tag,))
        self.user_list.bind("<ButtonRelease-1>", self.on_user_select)

    def on_user_select(self, event):
        item_id = self.user_list.focus()
        if not item_id:
            return
        selected_user = self.user_list.item(item_id)['values'][0]
        if selected_user == self.username:
            return
        self.current_chat_user = selected_user
        self.display_chat_with_user(selected_user)

    def display_chat_with_user(self, user):
        self.chat_header.config(text=f" Chat with {user}")
        for widget in self.chat_container.winfo_children():
            widget.destroy()
        messages = self.conversations.get(user, [])
        for sender, content, time in messages:
            self.display_message(sender, content, time, refresh=False)
        self.chat_container.update_idletasks()
        self.chat_canvas.yview_moveto(1.0)

    def display_message(self, sender, content, time, refresh=True):
        target = self.current_chat_user if sender == self.username else sender
        self.conversations.setdefault(target, []).append((sender, content, time))
        if refresh and target == self.current_chat_user:
            self.display_chat_with_user(self.current_chat_user)

    def send_message(self, event=None):
        text = self.msg_entry.get().strip()
        if text and self.current_chat_user:
            now = datetime.now().strftime("%H:%M")
            self.display_message(self.username, text, now)
            self.msg_entry.delete(0, "end")

    def send_file(self):
        file_path = filedialog.askopenfilename()
        if file_path and self.current_chat_user:
            now = datetime.now().strftime("%H:%M")
            self.display_message(self.username, f"[ملف] {file_path.split('/')[-1]}", now)

if __name__ == "__main__":
    ChatApp()
