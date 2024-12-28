import tkinter as tk 
import mysql.connector
from tkinter import messagebox
from main_form import menu_form

def connect_to_db():
    try:
        return mysql.connector.connect(
            host = "localhost",
            user = "root",
            password = "",
            database = "gym_management")
    except mysql.connector.Error:
        messagebox.showerror(message="Failed to connect to database")
        return None

def on_close():
    window.destroy()
        
def login():
    user = entry_user.get()
    pas = entry_pass.get()
    
    conn = connect_to_db()
    if not conn:
        return None
    cursor = conn.cursor()
    query = "SELECT * FROM login WHERE user = %s AND pas = %s"
    cursor.execute(query, (user, pas))
    result = cursor.fetchone()
    conn.close()
    
    if result:
        messagebox.showinfo(message="Welcome to fithive gym management") 
        window.destroy()
        menu_form()
    else:
        messagebox.showerror(message="Invalid username and password")


def set_placeholder(entry, placeholder_text, is_password=False):
    entry.insert(0, placeholder_text)
    entry.config(fg="grey")
    entry.bind("<FocusIn>", lambda e: clear_placeholder(entry, placeholder_text, is_password))
    entry.bind("<FocusOut>", lambda e: restore_placeholder(entry, placeholder_text, is_password))

def clear_placeholder(entry, placeholder_text, is_password):
    if entry.get() == placeholder_text:
        entry.delete(0, tk.END)
        entry.config(fg="black")
        if is_password:
            entry.config(show="*")

def restore_placeholder(entry, placeholder_text, is_password):
    if not entry.get():
        entry.insert(0, placeholder_text)
        entry.config(fg="grey")
       
        if is_password:
            entry.config(show="")

window = tk.Tk()
window.config(bg="#EEEEEE")
window_width = 400
window_height = 400
screen_width = window.winfo_screenwidth()
screen_height = window.winfo_screenheight()
x_position = (screen_width - window_width) // 2
y_position = (screen_height - window_height) // 2
window.geometry(f"{window_width}x{window_height}+{x_position}+{y_position}")

header = tk.Label(window, bg="#F0F0F0",text="Login", font=("Aeonik Typface", 20, "bold"))
header.place(x=162, y=68)

global entry_user,entry_pass
entry_user = tk.Entry(window, width=25, font=("Aeonik Typface", 10))
set_placeholder(entry_user, "Username")
entry_user.place(x=110, y=148, height=35)
entry_pass = tk.Entry(window, width=25, font=("Aeonik Typface", 10))
set_placeholder(entry_pass, "Password", is_password=True)
entry_pass.place(x=110, y=195, height=35)

login_quote = tk.Label(window, bg="#F0F0F0",text="Exercise? I thought you say unli-rice", font=("Aeonik Typface", 7, "italic"))
login_quote.place(x=120, y=284)
login_button = tk.Button(window, text="Login", relief="raised", bd=2, font=("Aeonik Typface", 11), bg="black", fg="white", activebackground="grey", activeforeground="black", width=19, command=login)
login_button.place(x=108, y=315)

window.protocol("WM_DELETE_WINDOW", on_close)

window.mainloop()

import tkinter as tk
from tkinter import ttk  
from tkinter import messagebox
from dash import dash_board
import mysql.connector


def connect_to_db():
    try:
        return mysql.connector.connect(
            host = "localhost",
            user = "root",
            password = "",
            database = "gym_management")
    except mysql.connector.Error:
        messagebox.showerror(message="Failed to connect to database")
        return None
    
def delete_member(prim_id, tree):
    if messagebox.askyesno(message=f"Are you sure you want to delete member no. {prim_id}?"):
        try:
            conn = connect_to_db()
            cursor = conn.cursor()
            cursor.execute("DELETE FROM member WHERE prim_id = %s", (prim_id,))
            conn.commit()
            conn.close()

            messagebox.showinfo(message="Member successfully deleted")
            load_data_into_tree(tree)  
        except mysql.connector.Error as e:
            messagebox.showerror(message=f"Error: {e}")

    
def edit_member(prim_id, tree):
   
    member_data = get_member_data(prim_id)
    if member_data:
        open_edit_window(prim_id, member_data, tree)

        
def get_member_data(prim_id):
    conn = connect_to_db()
    if not conn:
        return None
    cursor = conn.cursor()
    cursor.execute("SELECT first_name, last_name, age, contact, gender FROM member WHERE prim_id = %s", (prim_id,))
    member_data = cursor.fetchone()
    conn.close()
    return member_data


def open_edit_window(prim_id, member_data, tree):
    edit_window = tk.Toplevel()
    edit_window.title("Edit member")
    edit_window_width = 400
    edit_window_height = 400
    screen_width = edit_window.winfo_screenwidth()
    screen_height = edit_window.winfo_screenheight()
    x_position = (screen_width - edit_window_width) // 2
    y_position = (screen_height - edit_window_height) // 2
    edit_window.geometry(f"{edit_window_width}x{edit_window_height}+{x_position}+{y_position}")
    
    header =tk.Label(edit_window, text="Edit data", font=("Canva Sans", 11, "bold"))
    header.pack(pady=23)
    
    
    tk.Label(edit_window, text="First Name:").place(x=20, y=100)
    fname_entry = tk.Entry(edit_window, width=25, font=("Canva Sans", 11))
    fname_entry.place(x=100, y=100)
    tk.Label(edit_window, text="Last Name:").place(x=20, y=130)
    lname_entry = tk.Entry(edit_window, width=25, font=("Canva Sans", 11))
    lname_entry.place(x=100, y=130)
    tk.Label(edit_window, text="Age:").place(x=20, y=160)
    age_entry = tk.Entry(edit_window, width=25, font=("Canva Sans", 11))
    age_entry.place(x=100, y=160)
    tk.Label(edit_window, text="Contact #:").place(x=20, y=190)
    contact_entry = tk.Entry(edit_window, width=25, font=("Canva Sans", 11))
    contact_entry.place(x=100, y=190)
    tk.Label(edit_window, text="Gender:").place(x=20, y=220)
    gender_entry = tk.Entry(edit_window, width=25, font=("Canva Sans", 11))
    gender_entry.place(x=100, y=220)
    
    tk.Label(edit_window, text="Plan:").place(x=20, y=250)
    plan = ["Monthly", "Weekly", "Daily"]
    type_plan = ttk.Combobox(edit_window, width=26, font=("Canva Sans", 10), values=plan, state="readonly")
    type_plan.place(x=100, y=250)
    
    
    button = tk.Button(edit_window, text="Save", font=("Canva Sans", 11), bg="#222831", activebackground="gray",  fg="white", activeforeground="white",  width=25, command=lambda: save_edit(prim_id, fname_entry, lname_entry, age_entry, contact_entry, gender_entry, type_plan, tree, edit_window))
    button.place(x=90, y=310)
    
def save_edit(prim_id, fname_entry, lname_entry, age_entry, contact_entry, gender_entry, type_plan, tree, edit_window):
    def calculate_cost(plan):
        if plan == "Monthly":
            return 1800.00
        elif plan == "Weekly":
            return 420.00
        elif plan == "Daily":
            return 60.00
        
    first_name = fname_entry.get()
    last_name = lname_entry.get()
    age = age_entry.get()
    contact = contact_entry.get()
    gender = gender_entry.get()
    ship_plan = type_plan.get()


    if not first_name or not last_name or not age or not contact or not gender or not ship_plan:
        messagebox.showerror(message="All fields are required")
        return None
    
    try:
        conn = connect_to_db()
        cursor = conn.cursor()
        cost = calculate_cost(ship_plan)

        query = """
        UPDATE member 
        SET first_name=%s, last_name=%s, age=%s, contact=%s, 
            gender=%s, ship_plan=%s, cost=%s 
        WHERE prim_id=%s
        """
        cursor.execute(query, (first_name, last_name, age, contact, gender, ship_plan, cost, prim_id))
        conn.commit()
        conn.close()

        messagebox.showinfo(message="Member successfully updated")
        load_data_into_tree(tree)
        edit_window.destroy()
    except mysql.connector.Error as e:
        messagebox.showerror(message=f"Error: {e}")


def load_data_into_tree(tree):
    for item in tree.get_children():
        tree.delete(item)  

    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    query = "SELECT prim_id, first_name, last_name, age, contact, gender, ship_plan, cost FROM member"
    cursor.execute(query)
    rows = cursor.fetchall()
    conn.close()
    
    for index, row in enumerate(rows):
        prim_id = row[0]
        custom_text_9 = "📝"
        custom_text_10 = "     🗑️"
        tag_name = "even" if index % 2 == 0 else "odd"
        tree.insert("", "end", values=row + (custom_text_9, custom_text_10), tags=(f"row_{prim_id}", tag_name,))
    
    tree.tag_configure("even")
    tree.tag_configure("odd", background="black", foreground="white") 

   

def search_results(result, root):
    if not result:
        messagebox.showwarning(message="No results found")
        return
        
    result_window = tk.Toplevel(root)
    result_window.title("Searching......")
    result_window.config(bg="#F0F0F0")
    result_window_width = 900
    result_window_height = 200
    screen_width = result_window.winfo_screenwidth()
    screen_height = result_window.winfo_screenheight()
    x_position = (screen_width - result_window_width) // 2
    y_position = (screen_height - result_window_height) // 2
    result_window.geometry(f"{result_window_width}x{result_window_height}+{x_position}+{y_position}")
    
    style = ttk.Style()
    style.configure("Custom.Treeview.Heading", background="grey", font=("Canva Sans", 10, "bold"))
    style.map("Custom.Treeview.Heading", background=[("active", "grey")])
    style.theme_use("default") 
    tree = ttk.Treeview(result_window, columns=("ID", "First_name", "Last_name", "Age", "Contact", "Gender", "Memship_Plan", "Cost"), show="headings")
    tree.heading("ID", text="No.")
    tree.heading("First_name", text="First_Name")
    tree.heading("Last_name", text="Last_Name")
    tree.heading("Age", text="Age")
    tree.heading("Contact", text="Contact")
    tree.heading("Gender", text="Gender")
    tree.heading("Memship_Plan", text="Membership_Plan")
    tree.heading("Cost", text="Cost")
    
    tree.column("ID", width=50, anchor="center")
    tree.column("First_name", width=150, anchor="center")
    tree.column("Last_name", width=150, anchor="center")
    tree.column("Age", width=60, anchor="center")
    tree.column("Contact", width=110, anchor="center")
    tree.column("Gender", width=100, anchor="center")
    tree.column("Memship_Plan", width=150, anchor="center")
    tree.column("Cost", width=60, anchor="center")
    tree.pack(pady=10, padx=10,fill=tk.BOTH, expand=True)
    
    for row in result:
        prim_id = row[0]
        tree.insert("", "end", values=row, tags=(f"row_{prim_id}",))
        
    def on_close():
        result_window.destroy()
    
    result_window.protocol("WM_DELETE_WINDOW", on_close)


def search(search_entry):
    query = search_entry.get()        
    if not query:
        messagebox.showerror(message="Please enter a name or ID")
        return
    conn = connect_to_db()
    if not conn:
        return
    try:
        cursor = conn.cursor()
        cursor.execute("SELECT prim_id, first_name, last_name, age, contact, gender, ship_plan, cost FROM member WHERE first_name LIKE %s OR prim_id=%s", (f"{query}%", query))
        result = cursor.fetchall()
        search_results(result, root)
    except mysql.connector.Error as err:
        messagebox.showerror(message=f"Error: {err}")
    finally:
        conn.close
        
def out(root):
    response = messagebox.askyesno(message="Are you sure you want to log out?")
    if response:
        root.destroy()
        from login import login
        login()
        
    else:
        pass


def show(tree):
    load_data_into_tree(tree)
 


def add(new_root, tree, fname_entry, lname_entry, age_entry, contact_entry, gender_entry, type_plan):
    def calculate_cost(plan):
        if plan == "Monthly":
            return 1800.00
        elif plan == "Weekly":
            return 420.00
        elif plan == "Daily":
            return 60.00
        
    first_name = fname_entry.get()
    last_name = lname_entry.get()
    age = age_entry.get()
    contact = contact_entry.get()
    gender = gender_entry.get()
    ship_plan = type_plan.get()
    
    
    if not first_name or not last_name or not age or not contact or not gender or not ship_plan:
        messagebox.showerror(message="All fields are required")
        return None
    try:
        conn = connect_to_db()
        cursor = conn.cursor()
        cost = calculate_cost(ship_plan)
        cursor.execute(
            "INSERT INTO member (first_name, last_name, age, contact, gender, ship_plan, cost) VALUES (%s, %s, %s, %s, %s, %s, %s)",
            (first_name, last_name, age, contact, gender, ship_plan, cost),
        )
        conn.commit()
        conn.close()
        
        messagebox.showinfo(message="Member successfully added")
        
        fname_entry.delete(0, tk.END)
        lname_entry.delete(0, tk.END)
        age_entry.delete(0, tk.END)
        contact_entry.delete(0, tk.END)
        gender_entry.delete(0, tk.END)
        type_plan.delete(0, tk.END)
        
        
        load_data_into_tree(tree)
        new_root.destroy()
    except mysql.connector.Error as e:
        messagebox.showerror(message=f"Error: {e}")

def set_placeholder(entry, placeholder_text, is_password=False):
    entry.insert(0, placeholder_text)
    entry.config(fg="grey")
    entry.bind("<FocusIn>", lambda e: clear_placeholder(entry, placeholder_text, is_password))
    entry.bind("<FocusOut>", lambda e: restore_placeholder(entry, placeholder_text, is_password))

def clear_placeholder(entry, placeholder_text, is_password):
    if entry.get() == placeholder_text:
        entry.delete(0, tk.END)
        entry.config(fg="black")
        if is_password:
            entry.config(show="*")

def restore_placeholder(entry, placeholder_text, is_password):
    if not entry.get():
        entry.insert(0, placeholder_text)
        entry.config(fg="grey")
        if is_password:
            entry.config(show="")
            
def open_add(root, tree):
    new_root = tk.Toplevel(root)
    new_root.title("Add member")
    new_root_width = 400
    new_root_height = 400
    screen_width = new_root.winfo_screenwidth()
    screen_height = new_root.winfo_screenheight()
    x_position = (screen_width - new_root_width) // 2
    y_position = (screen_height - new_root_height) // 2
    new_root.geometry(f"{new_root_width}x{new_root_height}+{x_position}+{y_position}")
    
    header =tk.Label(new_root, text="Fill in", font=("Canva Sans", 11, "bold"))
    header.pack(pady=23)
    
    

    tk.Label(new_root, text="First Name:").place(x=20, y=100)
    fname_entry = tk.Entry(new_root, width=25, font=("Canva Sans", 11))
    fname_entry.place(x=100, y=100)
    tk.Label(new_root, text="Last Name:").place(x=20, y=130)
    lname_entry = tk.Entry(new_root, width=25, font=("Canva Sans", 11))
    lname_entry.place(x=100, y=130)
    tk.Label(new_root, text="Age:").place(x=20, y=160)
    age_entry = tk.Entry(new_root, width=25, font=("Canva Sans", 11))
    age_entry.place(x=100, y=160)
    tk.Label(new_root, text="Contact #:").place(x=20, y=190)
    contact_entry = tk.Entry(new_root, width=25, font=("Canva Sans", 11))
    contact_entry.place(x=100, y=190)
    tk.Label(new_root, text="Gender:").place(x=20, y=220)
    gender_entry = tk.Entry(new_root, width=25, font=("Canva Sans", 11))
    gender_entry.place(x=100, y=220)
    
    tk.Label(new_root, text="Plan:").place(x=20, y=250)
    plan = ["Monthly", "Weekly", "Daily"]
    type_plan = ttk.Combobox(new_root, width=26, font=("Canva Sans", 10), values=plan, state="readonly")
    type_plan.place(x=100, y=250)

    
    
    button = tk.Button(new_root, text="Submit", font=("Canva Sans", 11), bg="#222831", activebackground="gray",  fg="white", activeforeground="white",  width=25, command=lambda: add(new_root, tree, fname_entry, lname_entry, age_entry, contact_entry, gender_entry, type_plan))
    button.place(x=85, y=310)
    
    return new_root
    

def menu_form():
    global root
    root = tk.Tk()
    root.title("Indomitable Zenith Gym Management System")
    root_width = 1024
    root_height = 530
    screen_width = root.winfo_screenwidth()
    screen_height = root.winfo_screenheight()
    x_position = (screen_width - root_width) // 2
    y_position = (screen_height - root_height) // 2
    root.geometry(f"{root_width}x{root_height}+{x_position}+{y_position}")
    
    
    header_frame = tk.Frame(root, bd=3, bg="#0308A1", height=40)
    header_frame.pack_propagate(False)
    header_frame.pack(fill="x")
    header = tk.Label(header_frame, text="INDOMITABLE ZENITH GYM FITNESS", bg="#0308A1", fg="white", font=("Aeonik Typface", 10, "bold"))
    header.place(x=43, y=7)
    
    
    member_window = tk.Frame(root)
    member_window.pack(fill="both", expand=True)
    
    footer_frame = tk.Frame(root, bd=3, bg="#0308A1", height=70)
    footer_frame.pack_propagate(False)
    footer_frame.pack(side="bottom", fill="x")
    
                  
    search_entry = tk.Entry(member_window)
    set_placeholder(search_entry, "Search...")
    search_entry.place(x=835, y=10, height=27)
    search_button = tk.Button(member_window, width=2, text="➤ ", font=("", 8), fg="white", bg="black", command=lambda: search(search_entry))   
    search_button.place(x=963, y=11) 
    
    menu_button = tk.Menubutton(header_frame, text="☰", fg="white", bg="#0308A1", activebackground="#0308A1", activeforeground="black", font=("", 15))
    menu_button.place(x=0, y=0)
    menu = tk.Menu(menu_button, tearoff=0)
    menu.add_command(label="👁 View list", command=lambda:show(tree))
    menu.add_command(label="📊 Dashboard", command=lambda: (root.withdraw(), dash_board()))
    menu.add_command(label="⬅️ Log out", command=lambda:out(root))
    menu_button.config(menu=menu)
    
    tree_frame = tk.Frame(member_window)
    tree_frame.pack_propagate(True)
    tree_frame.place(x=36, y=60)

    style = ttk.Style()
    style.configure("Custom.Treeview.Heading", relief="sunken", bd=3, background="grey", font=("Canva Sans", 10, "bold"))
    style.map("Custom.Treeview.Heading", background=[("active", "grey")])
    style.theme_use("default")  
    
    tree = ttk.Treeview(tree_frame, height=13, columns=("ID", "First_name", "Last_name", "Age", "Contact", "Gender", "Memship_Plan", "Cost", "Edit", "Delete"), show="headings")
    tree.heading("ID", text="No.")
    tree.heading("First_name", text="First_Name")
    tree.heading("Last_name", text="Last_Name")
    tree.heading("Age", text="Age")
    tree.heading("Contact", text="Contact #")
    tree.heading("Gender", text="Gender")
    tree.heading("Memship_Plan", text="Membership_Plan")
    tree.heading("Cost", text="Cost")
    tree.heading("Edit", text="Edit")
    tree.heading("Delete", text="Delete")
    
    tree.column("ID", width=50, anchor="center")
    tree.column("First_name", width=150, anchor="center")
    tree.column("Last_name", width=150, anchor="center")
    tree.column("Age", width=60, anchor="center")
    tree.column("Contact", width=130, anchor="center")
    tree.column("Gender", width=100, anchor="center")
    tree.column("Memship_Plan", width=150, anchor="center")
    tree.column("Cost", width=60, anchor="center")
    tree.column("Edit", width=50, anchor="center")
    tree.column("Delete", width=50, anchor="center")

    tree.pack(fill="both", expand=True)
    
  
    def on_tree_click(event):
     
        item = tree.identify_row(event.y)
        column = tree.identify_column(event.x)
    
        if not item:
            return
    
        prim_id = tree.item(item, "values")[0]  
    
        if column == "#10":  
            delete_member(prim_id, tree)
    
        elif column == "#9": 
            edit_member(prim_id, tree)
            
    tree.bind("<Button-1>", on_tree_click)

        
    #global value_label
    
    tk.Label(footer_frame, text="MEMBERS", bg="#0308A1", fg="white", font=("Aeonik Typface", 13, "bold")).place(x=20, y=24)
    add_button = tk.Button(footer_frame, fg="black", bg="white", activebackground="white", text="➕ ", command=lambda:open_add(root, tree))
    add_button.place(x=870, y=23)
    tk.Label(footer_frame, text="Add Member", font=("Aeonik Typface", 11, "bold"), fg="white", bg="#0308A1").place(x=905, y=24)
    show(tree)
    
    return root

if __name__ == "__main__":
    menu_form().mainloop()



import tkinter as tk
from tkinter import messagebox
import mysql.connector


def connect_to_db():
    try:
        return mysql.connector.connect(
            host="localhost",
            user="root",
            password="",
            database="gym_management"
        )
    except mysql.connector.Error:
        messagebox.showerror(message="Failed to connect to database")
        return None


def click_green(all_mem):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    cursor.execute("SELECT COUNT(*) FROM member")
    total_m = cursor.fetchone()[0]
    conn.close()
    all_mem.config(text=total_m)


def click_orange(all_inc):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    cursor.execute("SELECT cost FROM member")
    rows = cursor.fetchall()
    conn.close()
    total_c = sum(row[0] for row in rows)
    all_inc.config(text=f"₱{total_c}")


def click_red(total_male):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    query = """
    SELECT
        SUM(CASE WHEN gender = 'Male' THEN 1 ELSE 0 END) AS total_males
    FROM member
    """
    cursor.execute(query)
    sum_males = cursor.fetchone()[0]
    conn.close()
    total_male.config(text=sum_males)


def click_yellow(total_fel):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    query = """
    SELECT
        SUM(CASE WHEN gender = 'Female' THEN 1 ELSE 0 END) AS total_females
    FROM member
    """
    cursor.execute(query)
    sum_females = cursor.fetchone()[0]
    conn.close()
    total_fel.config(text=sum_females)


def click_blue(blu_plan):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    query = """
    SELECT
        SUM(CASE WHEN ship_plan = 'Monthly' THEN 1 ELSE 0 END) AS total_monthly_plans
    FROM member
    """
    cursor.execute(query)
    sum_monthly = cursor.fetchone()[0]
    conn.close()
    blu_plan.config(text=sum_monthly)


def click_gray(gra_plan):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    query = """
    SELECT
        SUM(CASE WHEN ship_plan = 'Weekly' THEN 1 ELSE 0 END) AS total_weekly_plans
    FROM member
    """
    cursor.execute(query)
    sum_weekly = cursor.fetchone()[0]
    conn.close()
    gra_plan.config(text=sum_weekly)


def click_maroon(ma_plan):
    conn = connect_to_db()
    if not conn:
        return
    cursor = conn.cursor()
    query = """
    SELECT
        SUM(CASE WHEN ship_plan = 'Daily' THEN 1 ELSE 0 END) AS total_daily_plans
    FROM member
    """
    cursor.execute(query)
    sum_daily = cursor.fetchone()[0]
    conn.close()
    ma_plan.config(text=sum_daily)


def back(dash_root):
    from main_form import menu_form
    menu_form()
    dash_root.destroy()


def dash_board():
    dash_root = tk.Toplevel()
    dash_root.title("Dashboard")
    dash_root_width = 1024
    dash_root_height = 530
    screen_width = dash_root.winfo_screenwidth()
    screen_height = dash_root.winfo_screenheight()
    x_position = (screen_width - dash_root_width) // 2
    y_position = (screen_height - dash_root_height) // 2
    dash_root.geometry(f"{dash_root_width}x{dash_root_height}+{x_position}+{y_position}")

    header_frame = tk.Frame(dash_root, bd=3, bg="#0308A1", height=40)
    header_frame.pack_propagate(False)
    header_frame.pack(fill="x")

    footer_frame = tk.Frame(dash_root, bd=3, bg="#0308A1", height=70)
    footer_frame.pack_propagate(False)
    footer_frame.pack(side="bottom", fill="x")

    tk.Label(header_frame, text="INDOMITABLE ZENITH GYM FITNESS", font=("Canva Sans", 10, "bold"), bg="#0308A1", fg="white").place(x=43, y=7)
    menu_button = tk.Menubutton(header_frame, text="☰", fg="white", bg="#0308A1", activebackground="#0308A1", activeforeground="black", font=("", 15))
    menu_button.place(x=0, y=0)
    menu = tk.Menu(menu_button, tearoff=0)
    menu.add_command(label="⬅ Back", command=lambda: back(dash_root))
    menu_button.config(menu=menu)

    body_frame = tk.Frame(dash_root, bg="white")
    body_frame.pack(fill="both", expand=True)

    green_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="green")
    green_frame.place(x=30, y=30)
    tk.Label(green_frame, text="Total Members", fg="white", bg="green", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    all_mem = tk.Label(green_frame, text="0", fg="white", bg="green", font=("Canva Sans", 19, "bold"))
    all_mem.place(x=20, y=75)
    click_green(all_mem)

    orange_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="#F96E2A")
    orange_frame.place(x=280, y=30)
    tk.Label(orange_frame, text="Current Income", fg="white", bg="#F96E2A", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    all_inc = tk.Label(orange_frame, text="0", fg="white", bg="#F96E2A", font=("Canva Sans", 19, "bold"))
    all_inc.place(x=20, y=75)
    click_orange(all_inc)

    red_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="#C62E2E")
    red_frame.place(x=530, y=30)
    tk.Label(red_frame, text="Male Members", fg="white", bg="#C62E2E", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    total_male = tk.Label(red_frame, text="0", fg="white", bg="#C62E2E", font=("Canva Sans", 19, "bold"))
    total_male.place(x=20, y=75)
    click_red(total_male)

    yellow_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="#FCC737")
    yellow_frame.place(x=780, y=30)
    tk.Label(yellow_frame, text="Female Members", fg="white", bg="#FCC737", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    total_fel = tk.Label(yellow_frame, text="0", fg="white", bg="#FCC737", font=("Canva Sans", 19, "bold"))
    total_fel.place(x=20, y=75)
    click_yellow(total_fel)

    blue_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="#006BFF")
    blue_frame.place(x=30, y=205)
    tk.Label(blue_frame, text="Monthly Plans", fg="white", bg="#006BFF", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    blu_plan = tk.Label(blue_frame, text="0", fg="white", bg="#006BFF", font=("Canva Sans", 19, "bold"))
    blu_plan.place(x=20, y=75)
    click_blue(blu_plan)

    gray_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="#A6AEBF")
    gray_frame.place(x=280, y=205)
    tk.Label(gray_frame, text="Weekly Plans", fg="white", bg="#A6AEBF", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    gra_plan = tk.Label(gray_frame, text="0", fg="white", bg="#A6AEBF", font=("Canva Sans", 19, "bold"))
    gra_plan.place(x=20, y=75)
    click_gray(gra_plan)

    mar_frame = tk.Frame(body_frame, bd=1, relief="raised", width=210, height=140, bg="#740938")
    mar_frame.place(x=530, y=205)
    tk.Label(mar_frame, text="Daily Plans", fg="white", bg="#740938", font=("Canva Sans", 13, "bold")).place(x=10, y=20)
    ma_plan = tk.Label(mar_frame, text="0", fg="white", bg="#740938", font=("Canva Sans", 19, "bold"))
    ma_plan.place(x=20, y=75)
    click_maroon(ma_plan)

    tk.Label(footer_frame, text="DASHBOARD", bg="#0308A1", fg="white", font=("Canva Sans", 13, "bold")).place(x=20, y=24)
    return dash_root
  
