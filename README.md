# Medicine-Vending-Machine
# PBL Project 
from flask import Flask, request, jsonify
import mysql.connector

app = Flask(_name_)

# Connect to MySQL Database
db = mysql.connector.connect(
    host="localhost",
    user="root",          # your MySQL username
    password="yourpassword",  # your MySQL password
    database="medicine_vending_machine"
)
cursor = db.cursor(dictionary=True)

@app.route('/')
def home():
    return "💊 Medicine Vending Machine Backend Running Successfully!"

# Show all medicines
@app.route('/medicines', methods=['GET'])
def get_medicines():
    cursor.execute("SELECT * FROM medicines")
    medicines = cursor.fetchall()
    return jsonify(medicines)

# Buy medicine (reduce stock)
@app.route('/buy', methods=['POST'])
def buy_medicine():
    data = request.json
    name = data.get('name')

    # Check if medicine exists and has stock
    cursor.execute("SELECT * FROM medicines WHERE name=%s", (name,))
    medicine = cursor.fetchone()

    if not medicine:
        return jsonify({"message": "Medicine not found!"}), 404

    if medicine['stock'] > 0:
        cursor.execute("UPDATE medicines SET stock = stock - 1 WHERE name=%s", (name,))
        db.commit()
        return jsonify({
            "message": f"{name} dispensed successfully!",
            "remaining_stock": medicine['stock'] - 1
        })
    else:
        return jsonify({"message": f"{name} is out of stock!"}), 400

# Add new medicine
@app.route('/add', methods=['POST'])
def add_medicine():
    data = request.json
    name = data.get('name')
    price = data.get('price')
    stock = data.get('stock')
    expiry = data.get('expiry_date')

    cursor.execute(
        "INSERT INTO medicines (name, price, stock, expiry_date) VALUES (%s, %s, %s, %s)",
        (name, price, stock, expiry)
    )
    db.commit()
    return jsonify({"message": f"{name} added successfully!"})

if _name_ == '_main_':
    app.run(debug=True)
