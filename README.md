#main.py


import sqlite3

DATABASE = "students.db"


def connect_db():
    return sqlite3.connect(DATABASE)


def create_table():
    with connect_db() as conn:
        conn.execute("""
            CREATE TABLE IF NOT EXISTS students (
                id INTEGER PRIMARY KEY AUTOINCREMENT,
                name TEXT NOT NULL,
                email TEXT NOT NULL UNIQUE,
                course TEXT NOT NULL
            )
        """)


def add_student():
    name = input("Student name: ").strip()
    email = input("Student email: ").strip()
    course = input("Course: ").strip()

    if not name or not email or not course:
        print("Error: All fields are required.")
        return

    try:
        with connect_db() as conn:
            conn.execute(
                "INSERT INTO students (name, email, course) VALUES (?, ?, ?)",
                (name, email, course)
            )
        print("Student added successfully!")
    except sqlite3.IntegrityError:
        print("Error: This email already exists.")


def view_students():
    with connect_db() as conn:
        students = conn.execute(
            "SELECT id, name, email, course FROM students ORDER BY name"
        ).fetchall()

    if not students:
        print("No students found.")
        return

    print("\n--- Student Records ---")
    for student in students:
        print(
            f"ID: {student[0]} | Name: {student[1]} | "
            f"Email: {student[2]} | Course: {student[3]}"
        )


def search_student():
    name = input("Enter name to search: ").strip()

    with connect_db() as conn:
        students = conn.execute(
            "SELECT id, name, email, course FROM students "
            "WHERE name LIKE ? ORDER BY name",
            (f"%{name}%",)
        ).fetchall()

    for student in students:
        print(student)

    if not students:
        print("No matching students found.")


def delete_student():
    try:
        student_id = int(input("Enter student ID to delete: "))
    except ValueError:
        print("Please enter a valid ID.")
        return

    with connect_db() as conn:
        cursor = conn.execute(
            "DELETE FROM students WHERE id = ?", (student_id,)
        )

    if cursor.rowcount:
        print("Student deleted successfully!")
    else:
        print("Student ID not found.")


def main():
    create_table()

    while True:
        print("\n=== STUDENT MANAGEMENT SYSTEM ===")
        print("1. Add student")
        print("2. View students")
        print("3. Search students")
        print("4. Delete student")
        print("5. Exit")

        choice = input("Choose an option: ").strip()

        if choice == "1":
            add_student()
        elif choice == "2":
            view_students()
        elif choice == "3":
            search_student()
        elif choice == "4":
            delete_student()
        elif choice == "5":
            print("Goodbye!")
            break
        else:
            print("Invalid choice. Select 1-5.")


if __name__ == "__main__":
    main()
