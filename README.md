# Clinic-Appointment-Scheduler
from datetime import datetime, timedelta


def _parse_datetime(date_str, time_str):
    return datetime.strptime(f"{date_str} {time_str}", "%Y-%m-%d %H:%M")


class ClinicScheduler:
    def init(self):  # <-- fixed
        self.calendar = {}

    def schedule_appointment(self, patient_name, date_str, time_str, duration_minutes=30):
        appointment_time = _parse_datetime(date_str, time_str)
        end_time = appointment_time + timedelta(minutes=duration_minutes)

        if self._has_conflict(date_str, appointment_time, end_time):
            print("❌ Conflict: Appointment slot is already taken.")
            return False

        self.calendar.setdefault(date_str, []).append((appointment_time.time(), patient_name))
        print(f"✅ Appointment scheduled for {patient_name} on {date_str} at {time_str}.")
        return True

    def _has_conflict(self, date_str, start_time, end_time):
        for existing_time, _ in self.calendar.get(date_str, []):
            existing_start = datetime.combine(start_time.date(), existing_time)
            existing_end = existing_start + timedelta(minutes=30)
            if start_time < existing_end and end_time > existing_start:
                return True
        return False

    def update_appointment(self, patient_name, old_date, old_time, new_date, new_time):
        if old_date in self.calendar:
            for idx, (appt_time, name) in enumerate(self.calendar[old_date]):
                if name == patient_name and appt_time.strftime("%H:%M") == old_time:
                    del self.calendar[old_date][idx]
                    if not self.calendar[old_date]:
                        del self.calendar[old_date]
                    print("ℹ️ Old appointment removed.")
                    return self.schedule_appointment(patient_name, new_date, new_time)
        print("❌ Appointment not found to update.")
        return False

    def view_day_schedule(self, date_str):
        if date_str not in self.calendar or not self.calendar[date_str]:
            print(f"No appointments on {date_str}.")
            return
        print(f"\n📅 Appointments on {date_str}:")
        for time, name in sorted(self.calendar[date_str]):
            print(f"  {time.strftime('%H:%M')} - {name}")
        print()


def main():
    scheduler = ClinicScheduler()

    while True:
        print("\n--- Clinic Appointment Scheduler ---")
        print("1. Schedule Appointment")
        print("2. Update Appointment")
        print("3. View Day Schedule")
        print("4. Exit")
        choice = input("Choose an option (1-4): ")

        if choice == "1":
            name = input("Patient name: ")
            date = input("Appointment date (YYYY-MM-DD): ")
            time = input("Appointment time (HH:MM in 24hr): ")
            scheduler.schedule_appointment(name, date, time)

        elif choice == "2":
            name = input("Patient name: ")
            old_date = input("Old appointment date (YYYY-MM-DD): ")
            old_time = input("Old appointment time (HH:MM): ")
            new_date = input("New appointment date (YYYY-MM-DD): ")
            new_time = input("New appointment time (HH:MM): ")
            scheduler.update_appointment(name, old_date, old_time, new_date, new_time)

        elif choice == "3":
            date = input("Enter date to view (YYYY-MM-DD): ")
            scheduler.view_day_schedule(date)

        elif choice == "4":
            print("👋 Goodbye!")
            break

        else:
            print("❗️ Invalid choice. Try again.")


main()
