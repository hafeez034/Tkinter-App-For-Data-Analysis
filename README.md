# Tkinter-App-For-Data-Analysis
![image](https://github.com/user-attachments/assets/6313963f-1645-4a71-9b03-f1bba155d42e)

# Code For App

import tkinter as tk
from tkinter import filedialog, messagebox, simpledialog, ttk
from tkinter.scrolledtext import ScrolledText
import pandas as pd
import numpy as np
from io import StringIO

# Global variable to store the dataframe
df = None
selected_column = None



def update_dropdown():
    if df is not None:
        column_dropdown['values'] = list(df.columns)
        if df.columns.size > 0:
            column_dropdown.current(0)  # Set the first column as default selection
            on_column_select(None)  # Update selected column
            

# Load file and display columns
def load_file():
    global df
    file_path = filedialog.askopenfilename(
        filetypes=[("Excel files", "*.xlsx *.xls"), ("CSV files", "*.csv")]
    )
    if file_path:
        try:
            if file_path.endswith(('.xls', '.xlsx')):
                df = pd.read_excel(file_path)
            elif file_path.endswith('.csv'):
                df = pd.read_csv(file_path)
            else:
                messagebox.showerror("Error", "Unsupported file format!")
                return
            
            # Show columns in the dropdown
            column_dropdown['values'] = list(df.columns)
            column_dropdown.current(0)
            on_column_select(None)  # Set the selected column

            # Display a preview of the DataFrame
            update_preview()

            messagebox.showinfo("Success", "File loaded successfully!")
        except Exception as e:
            messagebox.showerror("Error", f"Failed to load file: {str(e)}")

def on_column_select(event):
    global selected_column
    selected_column = column_dropdown.get()
    if selected_column:
        update_preview()  # Update preview when column is selected

def update_preview():
    global df
    if df is not None:
        num_rows = int(row_count_var.get())
        preview_text.delete(1.0, tk.END)
        preview_text.insert(tk.END, df.head(num_rows).to_string(index=False))

# Button functions
def capitalize_first_sentence():
    global df
    if df is not None and selected_column:
        df[selected_column] = df[selected_column].astype(str).str.capitalize()
        update_preview()
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")

def text_to_column():
    global df
    if df is not None and selected_column:
        # Prompt user for delimiter
        delimiter = simpledialog.askstring("Delimiter", f"Enter delimiter to split column '{selected_column}':")
        if delimiter:
            try:
                # Split the column based on the specified delimiter
                new_cols = df[selected_column].str.split(delimiter, expand=True)
                new_col_names = [f"{selected_column}_{i+1}" for i in range(new_cols.shape[1])]
                new_cols.columns = new_col_names
                df = pd.concat([df, new_cols], axis=1)
                update_preview()
            except Exception as e:
                messagebox.showerror("Error", f"Failed to split column: {str(e)}")
        else:
            messagebox.showerror("Error", "No delimiter provided!")
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")


def fill_missing_by_input():
    global df
    if df is not None and selected_column:
        fill_value = simpledialog.askstring("Input", f"Enter value to fill missing in column '{selected_column}':")
        if fill_value is not None:
            df[selected_column].fillna(fill_value, inplace=True)
            update_preview()
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")

def fill_missing_stat(stat):
    global df
    if df is not None and selected_column:
        if stat == 'mean':
            fill_value = df[selected_column].mean()
        elif stat == 'median':
            fill_value = df[selected_column].median()
        elif stat == 'mode':
            fill_value = df[selected_column].mode()[0]
        df[selected_column].fillna(fill_value, inplace=True)
        update_preview()
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")

def fill_string_by_mode():
    global df
    if df is not None and selected_column:
        fill_value = df[selected_column].mode()[0]
        df[selected_column].fillna(fill_value, inplace=True)
        update_preview()
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")

def find_and_replace():
    global df
    if df is not None and selected_column:
        find_value = simpledialog.askstring("Find", f"Enter value to find in column '{selected_column}':")
        replace_value = simpledialog.askstring("Replace", f"Enter value to replace '{find_value}' with:")
        if find_value is not None and replace_value is not None:
            df[selected_column].replace(find_value, replace_value, inplace=True)
            update_preview()
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")

def conditional_formatting():
    global df
    if df is not None and selected_column:
        color = simpledialog.askstring("Color", f"Enter color (e.g., 'red') to highlight in column '{selected_column}':")
        if color:
            formatted_text = df[selected_column].apply(lambda x: f"\033[48;5;{color}m{x}\033[0m")
            preview_text.delete(1.0, tk.END)
            preview_text.insert(tk.END, formatted_text.to_string(index=False))
    else:
        messagebox.showerror("Error", "No file loaded or no column selected!")

def show_info():
    global df
    if df is not None:
        buffer = StringIO()
        df.info(buf=buffer)
        info_text.delete(1.0, tk.END)
        info_text.insert(tk.END, buffer.getvalue())
    else:
        messagebox.showerror("Error", "No file loaded!")

def show_null_info():
    global df
    if df is not None:
        null_counts = df.isnull().sum()
        null_info = null_counts[null_counts > 0]
        null_info_text.delete(1.0, tk.END)
        null_info_text.insert(tk.END, null_info.to_string())
    else:
        messagebox.showerror("Error", "No file loaded!")

# Setup the GUI
root = tk.Tk()
root.title("Advanced File Operations")
root.geometry("1600x800")  # Adjust size to fit your screen
root.configure(bg='#f0f0f0')

# Button Frame
button_frame = tk.Frame(root, bg='#f0f0f0')
button_frame.grid(row=0, column=0, sticky='ns', padx=10, pady=10)

# Load File Button
load_button = tk.Button(button_frame, text="Load File", command=load_file, bg='#4CAF50', fg='white', font=('Arial', 10), width=20)
load_button.grid(row=0, column=0, padx=5, pady=5, sticky='ew')

# Create and place other buttons in a grid layout
buttons = [
    ("Capitalize First Sentence", capitalize_first_sentence),
    ("Text to Column", text_to_column),
    ("Fill Missing by Input", fill_missing_by_input),
    ("Fill Missing by Mean", lambda: fill_missing_stat('mean')),
    ("Fill Missing by Median", lambda: fill_missing_stat('median')),
    ("Fill Missing by Mode", lambda: fill_missing_stat('mode')),
    ("Fill String by Mode", fill_string_by_mode),
    ("Find and Replace", find_and_replace),
    ("Conditional Formatting", conditional_formatting),
    ("Show DataFrame Info", show_info),
    ("Show Missing Values Info", show_null_info)
]

# Arrange buttons in the button_frame
for index, (text, command) in enumerate(buttons):
    row = (index + 1) // 4  # Adjust for additional row due to Load File button
    column = (index + 1) % 4
    btn = tk.Button(button_frame, text=text, command=command, bg='#2196F3', fg='white', font=('Arial', 10), width=20)
    btn.grid(row=row, column=column, padx=5, pady=5, sticky='ew')

# Dropdowns and Labels
column_label = tk.Label(root, text="Select Column:", bg='#f0f0f0', font=('Arial', 12))
column_label.grid(row=1, column=0, padx=10, pady=5, sticky='w')

column_dropdown = ttk.Combobox(root, state="readonly", font=('Arial', 12))
column_dropdown.grid(row=1, column=1, padx=10, pady=5, sticky='ew')

row_count_label = tk.Label(root, text="Number of Rows:", bg='#f0f0f0', font=('Arial', 12))
row_count_label.grid(row=2, column=0, padx=10, pady=5, sticky='w')

row_count_var = tk.StringVar(value='10')
row_count_dropdown = ttk.Combobox(root, textvariable=row_count_var, state="readonly", font=('Arial', 12))
row_count_dropdown['values'] = [10, 20, 50, 100, 200]
row_count_dropdown.grid(row=2, column=1, padx=10, pady=5, sticky='ew')

# Preview Text Widget
preview_frame = tk.Frame(root, bg='#f0f0f0')
preview_frame.grid(row=3, column=0, columnspan=2, padx=10, pady=10, sticky='nsew')

preview_label = tk.Label(preview_frame, text="DataFrame Preview:", bg='#f0f0f0', font=('Arial', 12))
preview_label.pack(anchor='w', padx=5, pady=5)

preview_text = ScrolledText(preview_frame, wrap='none', font=('Arial', 10))
preview_text.pack(expand=True, fill='both')

# Info and Null Info Text Widgets
info_frame = tk.Frame(root, bg='#f0f0f0')
info_frame.grid(row=4, column=0, columnspan=2, padx=10, pady=10, sticky='nsew')

info_label = tk.Label(info_frame, text="DataFrame Info:", bg='#f0f0f0', font=('Arial', 12))
info_label.pack(anchor='w', padx=5, pady=5)

info_text = ScrolledText(info_frame, wrap='none', font=('Arial', 10), height=10)
info_text.pack(expand=True, fill='both')

null_info_frame = tk.Frame(root, bg='#f0f0f0')
null_info_frame.grid(row=5, column=0, columnspan=2, padx=10, pady=10, sticky='nsew')

null_info_label = tk.Label(null_info_frame, text="Missing Values Info:", bg='#f0f0f0', font=('Arial', 12))
null_info_label.pack(anchor='w', padx=5, pady=5)

null_info_text = ScrolledText(null_info_frame, wrap='none', font=('Arial', 10), height=10)
null_info_text.pack(expand=True, fill='both')

# Configure grid weight to allow resizing
root.grid_rowconfigure(3, weight=1)
root.grid_rowconfigure(4, weight=1)
root.grid_rowconfigure(5, weight=1)
root.grid_columnconfigure(1, weight=1)

root.mainloop()

