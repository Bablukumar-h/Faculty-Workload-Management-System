csv_data = """Faculty,Subject,Hours
Dr. Sharma,DAA,6
Dr. Sharma,DBMS,5
Dr. Verma,OOP,6
Dr. Verma,Maths,6
Dr. Verma,Lab,6
Dr. Iyer,Networks,4
"""
with open("workload.csv", "w") as f:
    f.write(csv_data)
print("workload.csv created")

%%writefile workload_system.py
"""
Faculty Workload Management System
Concepts: Pandas, Dictionaries, Functions, Scheduling
Weekly limit per faculty: 16 hours
"""

import pandas as pd

LIMIT = 16
FILE = "workload.csv"
COLUMNS = ["Faculty", "Subject", "Hours"]


# ---------- Load data ----------
def load_data(path=FILE):
    """Read the CSV with Pandas. Returns None if the file is missing/invalid."""
    try:
        df = pd.read_csv(path)
    except FileNotFoundError:
        print(f"Error: '{path}' not found. Place it in the same folder.")
        return None
    except pd.errors.EmptyDataError:
        return pd.DataFrame(columns=COLUMNS)

    if not all(col in df.columns for col in COLUMNS):
        print(f"Error: CSV must contain the columns {COLUMNS}.")
        return None

    df["Faculty"] = df["Faculty"].astype(str).str.strip()
    df["Hours"] = pd.to_numeric(df["Hours"], errors="coerce").fillna(0).astype(int)
    return df


# ---------- Helpers ----------
def get_totals(df):
    """Dictionary: faculty name -> total weekly hours."""
    totals = {}
    for _, row in df.iterrows():
        name = row["Faculty"]
        totals[name] = totals.get(name, 0) + int(row["Hours"])
    return totals


def parse_hours(value):
    """Return a positive whole number, or None if invalid."""
    try:
        hours = int(str(value).strip())
    except (ValueError, TypeError):
        return None
    return hours if hours > 0 else None


def match_faculty(totals, name):
    """Case-insensitive lookup. Returns the stored name or None."""
    for stored in totals:
        if stored.lower() == name.strip().lower():
            return stored
    return None


# ---------- Feature 1: Workload summary ----------
def show_summary(df):
    totals = get_totals(df)
    if not totals:
        print("No workload data available.")
        return
    for name, hrs in totals.items():
        if hrs > LIMIT:
            print(f"{name} : {hrs} hours - OVERLOADED by {hrs - LIMIT} hours")
        else:
            print(f"{name} : {hrs} hours ({LIMIT - hrs} free)")


# ---------- Feature 2: Assign subject ----------
def assign_subject(df, faculty, subject, hours):
    hours = parse_hours(hours)
    if hours is None:
        print("Hours must be a positive whole number.")
        return df

    if not subject.strip():
        print("Subject name cannot be empty.")
        return df

    totals = get_totals(df)
    stored_name = match_faculty(totals, faculty)
    if stored_name is None:
        print(f"Faculty '{faculty}' not found.")
        return df

    new_total = totals[stored_name] + hours
    if new_total > LIMIT:
        print(f"Cannot assign: {stored_name} would have {new_total} hours, "
              f"exceeding the {LIMIT}-hour limit.")
        return df

    new_row = pd.DataFrame(
        [{"Faculty": stored_name, "Subject": subject.strip(), "Hours": hours}]
    )
    df = pd.concat([df, new_row], ignore_index=True)

    msg = f"Assigned. {stored_name} now has {new_total} hours"
    msg += " (limit reached)." if new_total == LIMIT else "."
    print(msg)
    return df


# ---------- Feature 3: Find faculty with free hours ----------
def find_free_faculty(df, needed):
    needed = parse_hours(needed)
    if needed is None:
        print("Hours must be a positive whole number.")
        return

    totals = get_totals(df)
    available = {n: LIMIT - h for n, h in totals.items() if LIMIT - h >= needed}

    if not available:
        print(f"No faculty available for {needed} hours.")
        return

    print(f"Faculty available for {needed} hours:")
    for name, free in available.items():
        print(f"{name} : {free} hours free")


# ---------- Menu ----------
def main():
    df = load_data()
    if df is None:
        return

    while True:
        print("\n===== Faculty Workload Management System =====")
        print("1. Show workload summary")
        print("2. Assign subject")
        print("3. Find faculty with free hours")
        print("4. Exit")
        choice = input("Menu: ").strip()

        if choice == "1":
            show_summary(df)
        elif choice == "2":
            faculty = input("Faculty: ")
            subject = input("Subject: ")
            hours = input("Hours: ")
            df = assign_subject(df, faculty, subject, hours)
        elif choice == "3":
            needed = input("Hours needed: ")
            find_free_faculty(df, needed)
        elif choice == "4":
            print("Goodbye!")
            break
        else:
            print("Invalid choice. Enter a number from 1 to 4.")


if __name__ == "__main__":
    main()

    %run workload_system.py

