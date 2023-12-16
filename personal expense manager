import tkinter as tk
from tkinter import ttk
from tkinter import messagebox
import sqlite3

# Establish connection to the database
conn = sqlite3.connect('budget_manager.db')
cur = conn.cursor()

cur.execute('''CREATE TABLE IF NOT EXISTS users (
                id INTEGER PRIMARY KEY,
                username TEXT NOT NULL,
                password TEXT NOT NULL)''')

# Create transactions table if not exists
cur.execute('''CREATE TABLE IF NOT EXISTS transactions (
                id INTEGER PRIMARY KEY,
                user_id INTEGER,
                date TEXT NOT NULL,
                description TEXT,
                category TEXT,
                amount REAL,
                FOREIGN KEY(user_id) REFERENCES users(id))''')

def show_main():
    login_frame.pack_forget()
    registration_frame.pack_forget()
    dashboard_frame.pack_forget()
    main_frame.pack()

def show_login():
    show_main()
    login_frame.pack()

def show_registration():
    show_main()
    registration_frame.pack()

def show_transactions():
    main_frame.pack_forget()
    login_frame.pack_forget()
    registration_frame.pack_forget()
    dashboard_frame.pack()

def login():
    global logged_in_user_id
    username = username_entry.get()
    password = password_entry.get()

    cur.execute("SELECT * FROM users WHERE username=? AND password=?", (username, password))
    user = cur.fetchone()

    if user:
        messagebox.showinfo("Success", f"Welcome, {username}!")
        logged_in_user_id = user[0]  # Store the logged-in user's ID for transactions
        show_transactions()  # Display transaction-related interface after login
        display_transactions()  # Display existing transactions after login
    else:
        messagebox.showerror("Error", "Invalid credentials")

def register():
    username = username_entry_reg.get()
    password = password_entry_reg.get()

    if username and password:
        cur.execute("INSERT INTO users(username, password) VALUES (?, ?)", (username, password))
        conn.commit()
        messagebox.showinfo("Success", "Registration successful! Please log in.")
        show_login()
    else:
        messagebox.showwarning("Warning", "Please enter both username and password.")

def add_transaction():
    # Ensure logged_in_user_id is accessible
    global logged_in_user_id
    
    # Retrieve transaction details from the entries
    date = date_entry.get()
    description = description_entry.get()
    category = category_entry.get()
    amount = amount_entry.get()

    # Insert fetched transactions into Treeview
    transactions_tree.insert('', 'end', values=(date, description, category, amount))

    # Insert transaction into the database
    cur.execute("INSERT INTO transactions(user_id, date, description, category, amount) VALUES (?, ?, ?, ?, ?)",
                (logged_in_user_id, date, description, category, amount))
    conn.commit()

def display_transactions():
    # Clear existing entries in the Treeview
    for item in transactions_tree.get_children():
        transactions_tree.delete(item)

    # Fetch transactions from the database for the logged-in user
    cur.execute("SELECT date, description, category, amount FROM transactions WHERE user_id=?", (logged_in_user_id,))
    transactions = cur.fetchall()

    # Insert fetched transactions into Treeview
    for transaction in transactions:
        transactions_tree.insert('', 'end', values=transaction)

def delete_transaction():
    selected_item = transactions_tree.selection()[0]
    transactions_tree.delete(selected_item)

    # Delete the selected transaction from the database
    cur.execute("DELETE FROM transactions WHERE user_id=? AND date=? AND description=? AND category=? AND amount=?",
                transactions_tree.item(selected_item, 'values') + (logged_in_user_id,))
    conn.commit()

def update_transaction():
    selected_item = transactions_tree.selection()[0]
    old_values = transactions_tree.item(selected_item, 'values')

    # Get new values from entry fields or another form and update in the Treeview
    new_values = (date_entry.get(), description_entry.get(), category_entry.get(), amount_entry.get())

    transactions_tree.item(selected_item, values=new_values)

    # Update the transaction in the database
    cur.execute("UPDATE transactions SET date=?, description=?, category=?, amount=? WHERE user_id=? AND date=? AND description=? AND category=? AND amount=?",
                new_values + (logged_in_user_id,) + old_values)
    conn.commit()

def clear_transactions():
    # Remove all entries from the Treeview
    for item in transactions_tree.get_children():
        transactions_tree.delete(item)

    # Remove all transactions of the logged-in user from the database
    cur.execute("DELETE FROM transactions WHERE user_id=?", (logged_in_user_id,))
    conn.commit()

def select_item(event):
    selected_item = transactions_tree.focus()

    values = transactions_tree.item(selected_item, 'values')
    date_entry.delete(0, tk.END)
    date_entry.insert(0, values[0])
    description_entry.delete(0, tk.END)
    description_entry.insert(0, values[1])
    category_entry.delete(0, tk.END)
    category_entry.insert(0, values[2])
    amount_entry.delete(0, tk.END)
    amount_entry.insert(0, values[3])

# Main window
root = tk.Tk()
root.title("Personal Budget Manager")

# Styling
style = ttk.Style()
style.theme_use('clam')  # Change theme to 'clam' for a modern look

# Main Screen Frame
main_frame = ttk.Frame(root)

login_button = ttk.Button(main_frame, text="Login", command=show_login)
login_button.pack(pady=10, padx=20, ipadx=10, ipady=5)

register_button = ttk.Button(main_frame, text="Register", command=show_registration)
register_button.pack(pady=10, padx=20, ipadx=10, ipady=5)

# Login Frame
login_frame = tk.Frame(root)

username_label = tk.Label(login_frame, text="Username:")
username_label.pack()
username_entry = tk.Entry(login_frame)
username_entry.pack()

password_label = tk.Label(login_frame, text="Password:")
password_label.pack()
password_entry = tk.Entry(login_frame, show="*")
password_entry.pack()

login_button = tk.Button(login_frame, text="Login", command=login)
login_button.pack()

# Registration Frame
registration_frame = tk.Frame(root)

username_label_reg = tk.Label(registration_frame, text="Username:")
username_label_reg.pack()
username_entry_reg = tk.Entry(registration_frame)
username_entry_reg.pack()

password_label_reg = tk.Label(registration_frame, text="Password:")
password_label_reg.pack()
password_entry_reg = tk.Entry(registration_frame, show="*")
password_entry_reg.pack()

register_button = tk.Button(registration_frame, text="Register", command=register)
register_button.pack()

# Dashboard Frame
dashboard_frame = ttk.Frame(root)

# Entry fields with Labels using ttk style
date_label = ttk.Label(dashboard_frame, text="Date:")
date_label.grid(row=0, column=0, padx=10, pady=5)
date_entry = ttk.Entry(dashboard_frame)
date_entry.grid(row=0, column=1, padx=10, pady=5)

description_label = ttk.Label(dashboard_frame, text="Description:")
description_label.grid(row=1, column=0, padx=10, pady=5)
description_entry = ttk.Entry(dashboard_frame)
description_entry.grid(row=1, column=1, padx=10, pady=5)

category_label = ttk.Label(dashboard_frame, text="Category:")
category_label.grid(row=2, column=0, padx=10, pady=5)
category_entry = ttk.Entry(dashboard_frame)
category_entry.grid(row=2, column=1, padx=10, pady=5)

amount_label = ttk.Label(dashboard_frame, text="Amount:")
amount_label.grid(row=3, column=0, padx=10, pady=5)
amount_entry = ttk.Entry(dashboard_frame)
amount_entry.grid(row=3, column=1, padx=10, pady=5)

add_transaction_button = ttk.Button(dashboard_frame, text="Add Transaction", command=add_transaction)
add_transaction_button.grid(row=4, column=0, columnspan=2, padx=10, pady=15)

# Buttons for delete and update actions
delete_button = ttk.Button(dashboard_frame, text="Delete", command=delete_transaction)
delete_button.grid(row=6, column=0, padx=10, pady=10)

update_button = ttk.Button(dashboard_frame, text="Save", command=update_transaction)
update_button.grid(row=6, column=1, padx=10, pady=10)

# Button for clearing transactions
clear_button = ttk.Button(dashboard_frame, text="Clear All", command=clear_transactions)
clear_button.grid(row=7, column=0, columnspan=2, padx=10, pady=10)

# Treeview widget for displaying transactions
transactions_tree = ttk.Treeview(dashboard_frame, columns=('Date', 'Description', 'Category', 'Amount'), show='headings')
transactions_tree.heading('Date', text='Date')
transactions_tree.heading('Description', text='Description')
transactions_tree.heading('Category', text='Category')
transactions_tree.heading('Amount', text='Amount')
transactions_tree.grid(row=5, column=0, columnspan=2, padx=10, pady=5)
transactions_tree.bind("<ButtonRelease-1>", select_item)  # Binding here

dashboard_frame.pack()
show_main()
root.mainloop()

# Close the connection when the application closes
conn.close()
