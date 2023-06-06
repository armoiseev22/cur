import requests
import tkinter as tk


def choose_from_currency():
    app = ListApp("FROM")
    app.resizable(True, False)
    app.mainloop()


def choose_to_currency():
    app = ListApp("TO")
    app.resizable(True, False)
    app.mainloop()


def enter_amount():
    app = AmountApp()
    app.resizable(True, False)
    app.mainloop()


def clear_entries():
    global from_currency, to_currency, amount, result_label
    from_currency = None
    to_currency = None
    amount = None
    btn1.config(text="FROM", fg="#fff")
    btn2.config(text="TO", fg="#fff")
    btn3.config(text="AMOUNT", fg="#fff")
    result_label.config(text="Waiting for conversion data...")
    root.update()


def finish_program():
    root.destroy()


class ListApp(tk.Tk):
    def __init__(self, type):
        super().__init__()
        self.type = type

        self.title(f"{type}")

        self.label = tk.Label(self,
                              text=f"Choose the currency you want to convert {type}",
                              font=("Helvetica", 17), bg="#363636", fg="#ffc35b",
                              anchor="n")

        self.list = tk.Listbox(self,
                               width=30, height=20, bg="#363636", fg="#fff",
                               selectbackground="#ffc35b",
                               font=("Helvetica", 15))

        self.print_btn = tk.Button(self, text="CONFIRM",
                                   width=15, height=2, bg="#363636", fg="#fff",
                                   font=("Helvetica", 17),
                                   command=self.print_selection)

        self.label.pack(fill=tk.BOTH)
        self.list.pack(fill=tk.BOTH)
        self.print_btn.pack(fill=tk.BOTH)

        response = requests.get('http://data.fixer.io/api/latest?access_key=67188d5aac8d3181c9b6e59118c3af17')
        if response.status_code == 200:
            data = response.json()
            rates = data['rates']
            for rate in rates:
                self.list.insert(tk.END, rate)

    def print_selection(self):
        selection = self.list.curselection()
        currency = self.list.get(selection)
        print(currency)

        if self.type == "FROM":
            global from_currency
            from_currency = currency
            btn1.config(text=currency, fg="#ffc35b")

        elif self.type == "TO":
            global to_currency
            to_currency = currency
            btn2.config(text=currency, fg="#ffc35b")

        self.destroy()


class AmountApp(tk.Tk):
    def __init__(self):
        super().__init__()

        self.title("AMOUNT")

        self.label = tk.Label(self,
                              text="Enter the amount you want to convert",
                              font=("Helvetica", 17), bg="#363636", fg="#ffc35b",
                              anchor="n")

        self.amount_entry = tk.Entry(self,
                                     font=("Helvetica", 15),
                                     bg="#363636", fg="#fff")

        self.print_btn = tk.Button(self,
                                   text="CONFIRM",
                                   font=("Helvetica", 17),
                                   bg="#363636", fg="#fff",
                                   command=self.print_amount)

        self.label.pack(fill=tk.BOTH)
        self.amount_entry.pack(fill=tk.BOTH)
        self.print_btn.pack(fill=tk.BOTH)

    def print_amount(self):
        global amount
        amount = float(self.amount_entry.get())
        print(amount)

        btn3.config(text=str(amount), fg="#ffc35b")

        self.destroy()


root = tk.Tk()
root.title("CurrencyConverter.de")
from_currency = None
to_currency = None
amount = None
root.resizable(True, False)

btn1 = tk.Button(root,
                 text="FROM",
                 width=30, height=3,
                 bg="#363636", fg="#fff",
                 command=choose_from_currency, font=("Helvetica", 20))
btn1.pack(fill=tk.BOTH)

btn2 = tk.Button(root,
                 text="TO",
                 width=30, height=3,
                 bg="#363636", fg="#fff",
                 command=choose_to_currency, font=("Helvetica", 20))
btn2.pack(fill=tk.BOTH)

btn3 = tk.Button(root,
                 text="AMOUNT",
                 width=30, height=3,
                 bg="#363636", fg="#fff",
                 command=enter_amount, font=("Helvetica", 20))
btn3.pack(fill=tk.BOTH)

result_label = tk.Label(root,
                        text="Waiting for conversion data...",
                        width=40, height=3, bg="#363636",
                        fg="#ffc35b", font=("Helvetica", 20))
result_label.pack(fill=tk.BOTH)


def currency_conversion():
    if from_currency is None or to_currency is None or (amount is None or (amount < 0.0)):
        return result_label.config(text='ERROR',
                                   width=40, height=3,
                                   bg="#363636", fg="#ffc35b",
                                   font=("Helvetica", 20))

    url = f"http://data.fixer.io/api/latest?access_key=67188d5aac8d3181c9b6e59118c3af17"
    response = requests.get(url)
    rate = response.json()["rates"][from_currency]
    amount_koeff = (amount) / (rate)
    converted_amount = amount_koeff * response.json()["rates"][to_currency]
    converted_amount = round(converted_amount, 2)
    print (from_currency, to_currency, amount, converted_amount)

    result_label.config(text='Result: '+str(converted_amount)+' '+str(to_currency),
                        width=40, height=3,
                        bg="#363636", fg="#ffc35b",
                        font=("Helvetica", 20))

    root.update()


result_btn = tk.Button(root,
                      text="CONVERT",
                      width=30, height=3,
                      bg="#363636", fg="#fff",
                      command=currency_conversion, font=("Helvetica", 20))
result_btn.pack(fill=tk.BOTH)

clear_btn = tk.Button(root,
                      text="CLEAR",
                      width=30, height=3,
                      bg="#363636", fg="#fff",
                      command=clear_entries, font=("Helvetica", 20))
clear_btn.pack(fill=tk.BOTH)

finish_btn = tk.Button(root,
                       text="FINISH",
                       width=30, height=3,
                       bg="#363636", fg="#fff",
                       command=finish_program, font=("Helvetica", 20))
finish_btn.pack(fill=tk.BOTH)

root.mainloop()