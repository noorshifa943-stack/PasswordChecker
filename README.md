#!/usr/bin/env python3
import re
import tkinter as tk
from tkinter import ttk

MIN_LENGTH = 8

def is_length_ok(p: str) -> bool:
    return len(p) >= MIN_LENGTH

def has_upper(p: str) -> bool:
    return bool(re.search(r'[A-Z]', p))

def has_lower(p: str) -> bool:
    return bool(re.search(r'[a-z]', p))

def has_digit(p: str) -> bool:
    return bool(re.search(r'\d', p))

def has_special(p: str) -> bool:
    return bool(re.search(r'[^A-Za-z0-9]', p))

class PasswordChecker(tk.Tk):
    def __init__(self):
        super().__init__()
        self.title("Password Strength Checker")
        self.resizable(False, False)
        self.configure(padx=12, pady=12)

        header = ttk.Label(self, text="Password Strength Checker", font=("TkDefaultFont", 14, "bold"))
        header.grid(row=0, column=0, columnspan=3, pady=(0, 8))

        self.pw_var = tk.StringVar()
        self.entry = ttk.Entry(self, textvariable=self.pw_var, show="*", width=32)
        self.entry.grid(row=1, column=0, padx=(0,8))
        self.entry.bind("<KeyRelease>", lambda e: self.evaluate())

        self.toggle_btn = ttk.Button(self, text="Show", width=6, command=self.toggle_show)
        self.toggle_btn.grid(row=1, column=1)

        # Checklist items (label and status)
        self.items = [
            ("At least {} characters".format(MIN_LENGTH), is_length_ok),
            ("Uppercase letter (A-Z)", has_upper),
            ("Lowercase letter (a-z)", has_lower),
            ("Number (0-9)", has_digit),
            ("Special character (!@#$...)", has_special),
        ]

        self.status_labels = []
        for i, (text, _) in enumerate(self.items, start=2):
            lbl = ttk.Label(self, text="✗ " + text, foreground="red")
            lbl.grid(row=i, column=0, columnspan=2, sticky="w", pady=2)
            self.status_labels.append(lbl)

        self.result_label = ttk.Label(self, text="Enter a password to check", font=("TkDefaultFont", 11, "bold"))
        self.result_label.grid(row=7, column=0, columnspan=2, pady=(8,0))

        # Evaluate initially
        self.evaluate()

    def toggle_show(self):
        if self.entry.cget("show") == "":
            self.entry.config(show="*")
            self.toggle_btn.config(text="Show")
        else:
            self.entry.config(show="")
            self.toggle_btn.config(text="Hide")

    def evaluate(self):
        p = self.pw_var.get()
        checks = [fn(p) for _, fn in self.items]

        for ok, lbl in zip(checks, self.status_labels):
            if ok:
                lbl.config(text="✓ " + lbl.cget("text")[2:], foreground="green")
            else:
                lbl.config(text="✗ " + lbl.cget("text")[2:], foreground="red")

        if not p:
            self.result_label.config(text="Enter a password to check", foreground="black")
        else:
            strong = all(checks)
            if strong:
                self.result_label.config(text="Strong Password", foreground="green")
            else:
                self.result_label.config(text="Weak Password", foreground="red")

if __name__ == "__main__":
    app = PasswordChecker()
    app.mainloop()
