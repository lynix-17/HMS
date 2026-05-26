# HMS
Hospital Management System built using Python and MySQL. The project manages patients, doctors, appointments, and billing through a menu-driven interface with CRUD operations, relational database integration, and CSV import/export support for data backup and recovery.
import mysql.connector
import csv
import os
print("*********HOSPITAL MANAGEMENT SYSTEM by Abhinav Singh and Shone Manoj of class XII B of KV RWF Yelahanka, 2025-26.*********")
pwd=input('enter password: ')

#DATABASE CREATION IF NOT EXISTS

def create_database():
    con=mysql.connector.connect(host='localhost',user='root',password=pwd)
    cur=con.cursor()
    cur.execute("show databases")
    databases = [db[0] for db in cur.fetchall()]
    if "hospital_db" not in databases:
        cur.execute("CREATE DATABASE hospital_db")
        cur.execute("use hospital_db")
        cur.execute("CREATE TABLE if not exists patients (patient_id INT NOT NULL AUTO_INCREMENT, name VARCHAR(50), age INT, gender VARCHAR(10), contact VARCHAR(15), disease VARCHAR(100), PRIMARY KEY (patient_id))")
        cur.execute("CREATE TABLE if not exists doctors (doctor_id INT NOT NULL AUTO_INCREMENT, name VARCHAR(50), specialization VARCHAR(50), contact  VARCHAR(15), PRIMARY KEY (doctor_id))")
        cur.execute("CREATE TABLE if not exists appointments (appointment_id INT NOT NULL AUTO_INCREMENT,patient_id INT NOT NULL,doctor_id INT NOT NULL,appointment_date DATE NOT NULL,PRIMARY KEY (appointment_id),KEY patient_id (patient_id),KEY doctor_id (doctor_id),CONSTRAINT appointments_ibfk_1 FOREIGN KEY (patient_id) REFERENCES patients(patient_id) ON DELETE CASCADE ON UPDATE CASCADE, CONSTRAINT appointments_ibfk_2 FOREIGN KEY (doctor_id) REFERENCES doctors(doctor_id) ON DELETE CASCADE ON UPDATE CASCADE);")
        cur.execute("CREATE TABLE if not exists billing ( bill_id INT NOT NULL AUTO_INCREMENT, patient_id INT NOT NULL, amount DECIMAL(10,2) NOT NULL, status VARCHAR(20) NOT NULL, PRIMARY KEY (bill_id), KEY patient_id (patient_id), CONSTRAINT billing_ibfk_1 FOREIGN KEY (patient_id) REFERENCES patients(patient_id) ON DELETE CASCADE ON UPDATE CASCADE );")
        print("Database created!")
    else:
        print("Database is present!")
create_database() 

# DATABASE CONNECTION
db = mysql.connector.connect(
    host="localhost",
    user="root",
    password=pwd,
    database="hospital_db"
)

cursor = db.cursor()


def table_has_data(table):
    cursor.execute(f"SELECT COUNT(*) FROM {table}")
    return cursor.fetchone()[0] > 0


def export_to_csv(query, filename, headers):
    cursor.execute(query)
    rows = cursor.fetchall()
    with open(filename, 'w', newline='') as file:
        writer = csv.writer(file)
        writer.writerow(headers)
        writer.writerows(rows)
    print(f"Data exported to {filename}")


def import_from_csv(filename, table, columns):
    if not os.path.exists(filename):
        return

    with open(filename, 'r') as file:
        reader = csv.reader(file)
        next(reader)  # skip header

        for row in reader:
            values = row  
            placeholders = ",".join(["%s"] * len(values))
            col_string = ",".join(columns)

            query = f"INSERT INTO {table} ({col_string}) VALUES ({placeholders})"
            cursor.execute(query, values)

    db.commit()


if not table_has_data("patients"):
    import_from_csv("patients.csv", "patients",
                    ["patient_id","name", "age", "gender", "contact","disease"])

if not table_has_data("doctors"):
    import_from_csv("doctors.csv", "doctors",
                    ["doctor_id","name", "specialization", "contact"])

if not table_has_data("appointments"):
    import_from_csv("appointments.csv", "appointments",
                    ["appointment_id","patient_id", "doctor_id", "appointment_date"])

if not table_has_data("billing"):
    import_from_csv("billing.csv", "billing",
                    ["bill_id","patient_id", "amount", "status"])


def print_patients_table(rows):
    print("\n{:<5} {:<20} {:<5} {:<8} {:<15} {:<20}".format(
        "ID", "Name", "Age", "Gender", "Contact", "Disease"
    ))
    print("-" * 75)

    for row in rows:
        print("{:<5} {:<20} {:<5} {:<8} {:<15} {:<20}".format(
            row[0], row[1], row[2], row[3], row[4], row[5]
        ))


# PATIENT MODULE
def patient_menu():
    while True:
        print("""
========= PATIENT MENU =========
1. View Patients               |
2. Add Patient                 |
3. Update Patient              |
4. Delete Patient              |
5. Export to CSV               |
6. Back                        |
================================
""")

        choice = input("Enter choice: ")

        if choice == '1':
            cursor.execute("SELECT * FROM patients")
            rows = cursor.fetchall()
    
            if not rows:
                print("No patient records found.")
            else:
                print_patients_table(rows)

        elif choice == '2':
            name = input("Patient name: ")
            age = int(input("Age: "))
            gender = input("Gender: ")
            contact = input("Contact: ")
            disease = input("Disease: ")

            cursor.execute(
                "INSERT INTO patients (name, age, gender, contact,disease) VALUES (%s,%s,%s,%s,%s)",
                (name, age, gender, contact,disease)
            )
            db.commit()
            print("Patient added.")

        elif choice == '3':
            pid = int(input("Patient idno: "))

            print("""
Update:
1. Name
2. Age
3. Gender
4. Contact
5. Disease
6. Back
""")

            opt = input("Choice: ")
            if opt==6:
                break

            fields = {
                '1': 'name',
                '2': 'age',
                '3': 'gender',
                '4': 'contact',
                '5': 'disease'
            }

            if opt in fields:
                new_val = input("Enter new value: ")
                cursor.execute(
                    f"UPDATE patients SET {fields[opt]}=%s WHERE patient_id=%s",
                    (new_val, pid)
                )
                db.commit()
                print("Patient updated.")
            else:
                print("Invalid option.")

        elif choice == '4':
            pid = int(input("Patient ID: "))
            cursor.execute("DELETE FROM patients WHERE patient_id=%s", (pid,))
            db.commit()
            print("Patient deleted.")

        elif choice == '5':
            export_to_csv(
                "SELECT * FROM patients",
                "patients.csv",
                ["ID", "Name", "Age", "Gender", "Contact"]
            )

        elif choice == '6':
            break

def print_doctors_table(rows):
    print("\n{:<5} {:<20} {:<25} {:<15}".format(
        "ID", "Name", "Specialization", "Contact"
    ))
    print("-" * 70)

    for row in rows:
        print("{:<5} {:<20} {:<25} {:<15}".format(
            row[0], row[1], row[2], row[3]
        ))


#DOCTOR MODULE
def doctor_menu():
    while True:
        print("""
========= DOCTOR MENU =========
1. View Doctors               |
2. Add Doctor                 |
3. Update Doctor              |
4. Delete Doctor              |
5. Export to CSV              |
6. Back                       |
===============================
""")

        choice = input("Enter choice: ")

        if choice == '1':
            cursor.execute("SELECT * FROM doctors")
            rows = cursor.fetchall()

            if not rows:
                print("No doctor records found.")
            else:
                print_doctors_table(rows)

            
        elif choice == '2':
            name = input("Name of the doctor: ")
            spec = input("Specialization: ")
            contact = input("Contact: ")

            cursor.execute(
                "INSERT INTO doctors (name, specialization, contact) VALUES (%s,%s,%s)",
                (name, spec, contact)
            )
            db.commit()
            print("Doctor added.")

        elif choice == '3':
            did = int(input("Doctor ID: "))
            print("""
Update:
1. Name
2. Specialization
3. Contact
4. Back
""")
            opt=input("Choice: ")
            if opt==4:
                break
            fields={
                '1':'name',
                '2':'Specialization',
                '3':'Contact'}
            if opt in fields:
                new_val=input("Enter new value: ")
                cursor.execute(
                    f"UPDATE doctors SET {fields[opt]}=%s WHERE doctor_id=%s",
                    (new_val,did))
                
                db.commit()
                print("Doctor updated.")
            else:
                print("Invalid option")

        elif choice == '4':
            did = int(input("Doctor ID: "))
            cursor.execute("DELETE FROM doctors WHERE doctor_id=%s", (did,))
            db.commit()
            print("Doctor deleted.")

        elif choice == '5':
            export_to_csv(
                "SELECT * FROM doctors",
                "doctors.csv",
                ["ID", "Name", "Specialization", "Contact"]
            )

        elif choice == '6':
            break

def print_appointments_table(rows):
    print("\n{:<5} {:<10} {:<10} {:<15}".format(
        "ID", "Patient ID", "Doctor ID", "Date"
    ))
    print("-" * 45)

    for row in rows:
        print("{:<5} {:<10} {:<10} {:<15}".format(
            row[0], row[1], row[2], str(row[3])
        ))


#APPOINTMENT MODULE
def appointment_menu():
    while True:
        print("""
======= APPOINTMENT MENU =======
1. View Appointments           |
2. Add Appointment             |
3. Delete Appointment          |
4. Export to CSV               |
5. Back                        |
================================
""")

        choice = input("Enter choice: ")

        if choice == '1':
            cursor.execute("SELECT * FROM appointments")
            rows = cursor.fetchall()
            if not rows:
                print("No appointments.")
            else:
                print_appointments_table(rows)               

        elif choice == '2':
            pid = int(input("Patient ID: "))
            did = int(input("Doctor ID: "))
            date = input("Date (YYYY-MM-DD): ")

            cursor.execute(
                "INSERT INTO appointments (patient_id, doctor_id, appointment_date) VALUES (%s,%s,%s)",
                (pid, did, date)
            )
            db.commit()
            print("Appointments added.")

        elif choice == '3':
            aid = int(input("Appointment ID: "))
            cursor.execute("DELETE FROM appointments WHERE appointment_id=%s", (aid,))
            db.commit()
            print("Appointments deleted.")

        elif choice == '4':
            export_to_csv(
                "SELECT * FROM appointments",
                "appointments.csv",
                ["ID", "Patient ID", "Doctor ID", "Date"]
            )

        elif choice == '5':
            break

def print_billing_table(rows):
    print("\n{:<7} {:<10} {:<10} {:<10}".format(
        "Bill ID", "Patient ID", "Amount", "Status"
    ))
    print("-" * 45)

    for row in rows:
        print("{:<7} {:<10} {:<10.2f} {:<10}".format(
            row[0], row[1], row[2], row[3]
        ))


#BILLING MODULE
def billing_menu():
    while True:
        print("""
========= BILLING MENU =========
1. View Bills                  |
2. Add Bill                    |
3. Update Status               |
4. Export to CSV               |
5. Back                        |
================================
""")

        choice = input("Enter choice: ")

        if choice == '1':
            cursor.execute("SELECT * FROM billing")
            rows = cursor.fetchall()
            if not rows:
                print("No bills.")
            else:
                print_billing_table(rows)

        elif choice == '2':
            pid = int(input("Patient ID: "))
            amount = float(input("Amount: "))
            status = input("Status (Paid/Unpaid): ")

            cursor.execute(
                "INSERT INTO billing (patient_id, amount, status) VALUES (%s,%s,%s)",
                (pid, amount, status)
            )
            db.commit()
            print("Bill generated.")

        elif choice == '3':
            bid = int(input("Bill ID: "))
            status = input("New Status: ")
            cursor.execute(
                "UPDATE billing SET status=%s WHERE bill_id=%s",
                (status, bid)
            )
            db.commit()
            print("Status updated.")

        elif choice == '4':
            export_to_csv(
                "SELECT * FROM billing",
                "billing.csv",
                ["Bill ID", "Patient ID", "Amount", "Status"]
            )

        elif choice == '5':
            break


#MAIN MENU
def main_menu():
    while True:
        print("""
====== HOSPITAL MANAGEMENT SYSTEM ======
1. Patients                            |
2. Doctors                             |
3. Appointments                        |
4. Billing                             |
5. Exit                                |
========================================
""")

        choice = input("Enter choice: ")

        if choice == '1':
            patient_menu()
        elif choice == '2':
            doctor_menu()
        elif choice == '3':
            appointment_menu()
        elif choice == '4':
            billing_menu()
        elif choice == '5':
            print("System closed.")
            break


main_menu()
db.close()
this is the code just review it and describe how it actually works

