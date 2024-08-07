# Expense-Tracker
import tkinter as tk
from tkinter import messagebox, ttk
import sqlite3
from datetime import datetime

class ExpenseTracker:
    def __init__(self, root):
        self.root = root
        self.root.title("Expense Tracker")
        self.create_db()
        
        # Create GUI components
        self.create_widgets()
        self.load_expenses()
        self.update_total_expense()

    def create_db(self):
        self.conn = sqlite3.connect('expenses.db')
        self.cursor = self.conn.cursor()
        self.cursor.execute('''CREATE TABLE IF NOT EXISTS expenses (
                            id INTEGER PRIMARY KEY,
                            amount REAL NOT NULL,
                            category TEXT NOT NULL,
                            date TEXT NOT NULL)''')
        self.conn.commit()

    def create_widgets(self):
        # Input Frame
        self.input_frame = tk.Frame(self.root)
        self.input_frame.pack(padx=10, pady=10, fill="x")

        tk.Label(self.input_frame, text="Amount:").grid(row=0, column=0, padx=5, pady=5, sticky='e')
        self.amount_entry = tk.Entry(self.input_frame)
        self.amount_entry.grid(row=0, column=1, padx=5, pady=5, sticky='w')

        tk.Label(self.input_frame, text="Category:").grid(row=1, column=0, padx=5, pady=5, sticky='e')
        self.category_var = tk.StringVar()
        self.category_combo = ttk.Combobox(self.input_frame, textvariable=self.category_var)
        self.category_combo['values'] = ("Food", "Transport", "Utilities", "Entertainment", "Other")
        self.category_combo.grid(row=1, column=1, padx=5, pady=5, sticky='w')

        self.add_expense_btn = tk.Button(self.input_frame, text="Add Expense", command=self.add_expense)
        self.add_expense_btn.grid(row=2, column=0, columnspan=2, pady=10)

        # Expense List Frame
        self.list_frame = tk.Frame(self.root)
        self.list_frame.pack(padx=10, pady=10, fill="both", expand=True)

        self.expense_listbox = tk.Listbox(self.list_frame)
        self.expense_listbox.pack(side="left", fill="both", expand=True)

        self.scrollbar = tk.Scrollbar(self.list_frame, orient="vertical", command=self.expense_listbox.yview)
        self.scrollbar.pack(side="right", fill="y")

        self.expense_listbox.config(yscrollcommand=self.scrollbar.set)

        self.delete_expense_btn = tk.Button(self.root, text="Delete Expense", command=self.delete_expense)
        self.delete_expense_btn.pack(pady=10)

        # Total Expenses Frame
        self.total_frame = tk.Frame(self.root)
        self.total_frame.pack(padx=10, pady=10, fill="x")

        self.total_label = tk.Label(self.total_frame, text="Total: $0.00")
        self.total_label.pack(padx=5, pady=5)

    def add_expense(self):
        amount = self.amount_entry.get()
        category = self.category_var.get()
        if amount and category:
            try:
                amount = float(amount)
                if amount <= 0:
                    raise ValueError("Amount must be positive.")
                date = datetime.now().strftime('%Y-%m-%d')
                self.cursor.execute('INSERT INTO expenses (amount, category, date) VALUES (?, ?, ?)', (amount, category, date))
                self.conn.commit()
                self.load_expenses()
                self.update_total_expense()
                self.amount_entry.delete(0, tk.END)
                self.category_var.set('')
            except ValueError as e:
                messagebox.showwarning("Warning", str(e))
        else:
            messagebox.showwarning("Warning", "You must enter an amount and select a category.")

    def delete_expense(self):
        try:
            selected_expense_index = self.expense_listbox.curselection()[0]
            selected_expense = self.expense_listbox.get(selected_expense_index)
            expense_id = selected_expense.split()[0]
            self.cursor.execute('DELETE FROM expenses WHERE id=?', (expense_id,))
            self.conn.commit()
            self.load_expenses()
            self.update_total_expense()
        except IndexError:
            messagebox.showwarning("Warning", "You must select an expense to delete.")

    def load_expenses(self):
        self.expense_listbox.delete(0, tk.END)
        self.cursor.execute('SELECT * FROM expenses')
        for expense in self.cursor.fetchall():
            self.expense_listbox.insert(tk.END, f"{expense[0]} {expense[3]} - ${expense[1]:.2f} - {expense[2]}")

    def update_total_expense(self):
        self.cursor.execute('SELECT SUM(amount) FROM expenses')
        total = self.cursor.fetchone()[0]
        total = total if total is not None else 0  # Handle the case where there are no expenses
        self.total_label.config(text=f"Total: ${total:.2f}")

if __name__ == "__main__":
    root = tk.Tk()
    app = ExpenseTracker(root)
    root.mainloop()

