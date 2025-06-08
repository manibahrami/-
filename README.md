#About this Project 
![Game Screenshot](Guess_Game.PNG)

import tkinter as tk
from tkinter import messagebox, ttk # ttk for modern widgets like Treeview

# --- Main application class ---
class TicketSalesApp:
    def __init__(self, master):
        self.master = master
        master.title("سیستم فروش بلیط پارک") # Set the window title
        master.geometry("700x600") # Set the window size
        master.resizable(False, False) # Prevent window resizing

        # --- Main variables for the application ---
        self.TICKET_TYPES = {
            "بزرگسال": 50000, "کودک": 30000, "سالمند": 25000, "خانواده": 120000 
        }
        self.sold_tickets = [] # List to store all sold tickets
        self.next_ticket_id = 1 # Counter for unique ticket IDs

        self._setup_styles() # Call method to set up visual styles
        self._create_widgets() # Call method to create and arrange all UI elements
        self.update_summary() # Initial update of summary labels and table

    def _setup_styles(self):
        """
        Sets up the visual styles for ttk widgets to make them look modern and appealing.
        """
        s = ttk.Style()
        s.theme_use('clam') # Choose a modern theme ('clam', 'alt', 'default' are common)
        s.configure('TButton', font=('Arial', 12), padding=10) # Configure buttons
        s.configure('TLabel', font=('Arial', 11)) # Configure general labels
        s.configure('TEntry', font=('Arial', 11)) # Configure entry fields
        s.configure('Treeview.Heading', font=('Arial', 10, 'bold')) # Configure Treeview headers
        s.configure('Treeview', font=('Arial', 10)) # Configure Treeview rows

    def _create_widgets(self):
        """
        Creates and arranges all the user interface elements (widgets) on the window.
        """
        # --- Input Frame: For adding and removing tickets ---
        input_frame = tk.LabelFrame(self.master, text="ثبت/حذف بلیط", padx=10, pady=10, font=('Arial', 12, 'bold'))
        input_frame.pack(pady=10, padx=10, fill="x")

        tk.Label(input_frame, text="نام خریدار:", font=('Arial', 11)).grid(row=0, column=0, padx=5, pady=5, sticky="w")
        self.buyer_name_entry = ttk.Entry(input_frame, width=30)
        self.buyer_name_entry.grid(row=0, column=1, padx=5, pady=5, sticky="ew")

        self.add_button = ttk.Button(input_frame, text="✔ افزودن بلیط", command=self.add_ticket)
        self.add_button.grid(row=0, column=2, padx=10, pady=5)
        
        tk.Label(input_frame, text="نوع بلیط:", font=('Arial', 11)).grid(row=1, column=0, padx=5, pady=5, sticky="w")
        # Combobox for selecting ticket type from a predefined list
        self.ticket_type_combobox = ttk.Combobox(input_frame, values=list(self.TICKET_TYPES.keys()), state="readonly", width=28)
        self.ticket_type_combobox.grid(row=1, column=1, padx=5, pady=5, sticky="ew")
        self.ticket_type_combobox.set("بزرگسال") # Set default value

        tk.Label(input_frame, text="ID بلیط برای حذف:", font=('Arial', 11)).grid(row=2, column=0, padx=5, pady=5, sticky="w")
        self.remove_id_entry = ttk.Entry(input_frame, width=30)
        self.remove_id_entry.grid(row=2, column=1, padx=5, pady=5, sticky="ew")

        self.remove_button = ttk.Button(input_frame, text="✖ حذف بلیط", command=self.remove_ticket)
        self.remove_button.grid(row=2, column=2, padx=10, pady=5)

        input_frame.grid_columnconfigure(1, weight=1) # Make the middle column expandable

        # --- Summary Frame: For displaying sales overview ---
        summary_frame = tk.LabelFrame(self.master, text="خلاصه فروش", padx=10, pady=10, font=('Arial', 12, 'bold'))
        summary_frame.pack(pady=10, padx=10, fill="both", expand=True)

        self.total_tickets_label = tk.Label(summary_frame, text="تعداد کل بلیط‌ها: 0", font=('Arial', 12, 'bold'), fg="darkgreen")
        self.total_tickets_label.pack(pady=(0, 5))

        self.total_revenue_label = tk.Label(summary_frame, text="جمع کل درآمد: 0 تومان", font=('Arial', 12, 'bold'), fg="darkblue")
        self.total_revenue_label.pack(pady=(0, 10))

        # Treeview widget to display sold tickets in a table format
        self.tree = ttk.Treeview(summary_frame, columns=("ID", "Buyer", "Type", "Price"), show="headings")
        self.tree.heading("ID", text="ID")
        self.tree.heading("Buyer", text="نام خریدار")
        self.tree.heading("Type", text="نوع بلیط")
        self.tree.heading("Price", text="قیمت (تومان)")
        
        # Set column widths and alignment
        self.tree.column("ID", width=50, anchor="center")
        self.tree.column("Buyer", width=150)
        self.tree.column("Type", width=100, anchor="center")
        self.tree.column("Price", width=100, anchor="e") # 'e' for right-alignment
        self.tree.pack(fill="both", expand=True)

        # Print Summary Button (placed outside frames for distinct positioning)
        self.print_summary_button = ttk.Button(self.master, text="پرینت خلاصه فروش", command=self.print_summary_receipt)
        self.print_summary_button.pack(pady=10)

    def add_ticket(self):
        """
        Adds a new ticket to the sold tickets list based on user input from the GUI.
        Also triggers receipt printing if requested.
        """
        buyer_name = self.buyer_name_entry.get().strip()
        ticket_type = self.ticket_type_combobox.get()

        # Input validation
        if not buyer_name: messagebox.showerror("خطا", "لطفا نام خریدار را وارد کنید."); return
        if not ticket_type: messagebox.showerror("خطا", "لطفا نوع بلیط را انتخاب کنید."); return

        # Create ticket info dictionary
        ticket_info = {"id": self.next_ticket_id, "buyer_name": buyer_name, 
                       "ticket_type": ticket_type, "price": self.TICKET_TYPES[ticket_type]}
        self.sold_tickets.append(ticket_info) # Add to list
        messagebox.showinfo("موفقیت", f"بلیط برای {buyer_name} ({ticket_type}) با موفقیت اضافه شد. ID: {self.next_ticket_id}")
        
        self.next_ticket_id += 1 # Increment ticket ID
        self.update_summary() # Update display
        self.buyer_name_entry.delete(0, tk.END) # Clear buyer name input
        self.ticket_type_combobox.set("بزرگسال") # Reset ticket type selection

        # Ask to print individual receipt
        if messagebox.askyesno("پرینت رسید", "آیا می‌خواهید رسید این بلیط را پرینت کنید؟"):
            self.print_ticket_receipt(ticket_info)

    def remove_ticket(self):
        """
        Removes a ticket from the sold tickets list based on its ID entered by the user.
        """
        ticket_id_str = self.remove_id_entry.get().strip()
        if not ticket_id_str: messagebox.showerror("خطا", "لطفا ID بلیط برای حذف را وارد کنید."); return

        try:
            ticket_id = int(ticket_id_str)
            found = False
            for ticket in self.sold_tickets:
                if ticket['id'] == ticket_id:
                    self.sold_tickets.remove(ticket); found = True; break # Remove and set flag
            if not found: messagebox.showerror("خطا", f"بلیط با ID {ticket_id} یافت نشد.")
        except ValueError: messagebox.showerror("خطا", "ورودی نامعتبر است. لطفاً یک عدد وارد کنید.")
        
        self.update_summary() # Update display after removal
        self.remove_id_entry.delete(0, tk.END) # Clear ID input

    def update_summary(self):
        """
        Updates the total tickets count, total revenue, and the Treeview table display.
        """
        total_tickets = len(self.sold_tickets)
        total_revenue = sum(t['price'] for t in self.sold_tickets)

        self.total_tickets_label.config(text=f"تعداد کل بلیط‌ها: {total_tickets}")
        self.total_revenue_label.config(text=f"جمع کل درآمد: {total_revenue} تومان")

        # Clear existing items in the Treeview
        for item in self.tree.get_children(): self.tree.delete(item)
        # Insert all current sold tickets into the Treeview
        for ticket in self.sold_tickets:
            self.tree.insert("", "end", values=(ticket['id'], ticket['buyer_name'], ticket['ticket_type'], ticket['price']))

    def print_ticket_receipt(self, ticket_info):
        """
        Saves individual ticket details to a text file, simulating a print receipt.
        """
        filename = f"receipt_ticket_{ticket_info['id']}.txt"
        try:
            with open(filename, "w", encoding="utf-8") as f:
                f.write("--- رسید بلیط پارک ---\n"); f.write(f"ID بلیط: {ticket_info['id']}\n")
                f.write(f"نام خریدار: {ticket_info['buyer_name']}\n"); f.write(f"نوع بلیط: {ticket_info['ticket_type']}\n")
                f.write(f"قیمت: {ticket_info['price']} تومان\n"); f.write("\nاز خرید شما متشکریم!\n")
            messagebox.showinfo("پرینت رسید", f"رسید بلیط با ID {ticket_info['id']} در فایل '{filename}' ذخیره شد.")
        except Exception as e: messagebox.showerror("خطا", f"خطا در ذخیره رسید: {e}")

    def print_summary_receipt(self):
        """
        Saves the entire sales summary (total and individual ticket details) to a text file.
        """
        if not self.sold_tickets: messagebox.showinfo("اطلاعات", "لیست فروش خالی است. چیزی برای پرینت نیست."); return

        filename = "sales_summary.txt"
        try:
            with open(filename, "w", encoding="utf-8") as f:
                f.write("--- خلاصه کل فروش بلیط پارک ---\n")
                f.write(f"تعداد کل بلیط‌های فروخته شده: {len(self.sold_tickets)}\n")
                f.write(f"جمع کل درآمد: {sum(t['price'] for t in self.sold_tickets)} تومان\n")
                f.write("\n--- جزئیات بلیط‌ها ---\n")
                for ticket in self.sold_tickets:
                    f.write(f"ID: {ticket['id']}, خریدار: {ticket['buyer_name']}, نوع: {ticket['ticket_type']}, قیمت: {ticket['price']} تومان\n")
                f.write("\n--- پایان گزارش ---\n")
            messagebox.showinfo("پرینت خلاصه", f"خلاصه فروش در فایل '{filename}' ذخیره شد.")
        except Exception as e: messagebox.showerror("خطا", f"خطا در ذخیره خلاصه فروش: {e}")

# --- Program entry point ---
if __name__ == "__main__":
    root = tk.Tk() # Create the main Tkinter window
    app = TicketSalesApp(root) # Create an instance of the TicketSalesApp class
    root.mainloop() # Start the Tkinter event loop, keeping the window open
