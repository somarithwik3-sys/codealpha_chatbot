# codealpha_chatbot
import datetime
import json
import os
import re

DATA_FILE = "daily_assistant_data.json"


class DailyAssistant:
    def __init__(self):
        self.data = self.load_data()

    def load_data(self):
        """Loads tasks and notes from a local file, or creates new storage."""
        if os.path.exists(DATA_FILE):
            try:
                with open(DATA_FILE, "r") as f:
                    return json.load(f)
            except Exception:
                pass
        return {"tasks": [], "notes": []}

    def save_data(self):
        """Saves current state to JSON."""
        with open(DATA_FILE, "w") as f:
            json.dump(self.data, f, indent=4)

    # ----------------- TASK MANAGER -----------------
    def add_task(self, task_text):
        if not task_text:
            return "Please specify a task to add. Example: 'add task buy groceries'"
        self.data["tasks"].append({"task": task_text, "done": False})
        self.save_data()
        return f" Added task: '{task_text}'"

    def list_tasks(self):
        tasks = self.data["tasks"]
        if not tasks:
            return " You have no tasks on your list!"
        output = [" Your To-Do List:"]
        for idx, item in enumerate(tasks, 1):
            status = "✓" if item["done"] else " "
            output.append(f"  [{status}] {idx}. {item['task']}")
        return "\n".join(output)

    def complete_task(self, index_str):
        try:
            idx = int(index_str) - 1
            if 0 <= idx < len(self.data["tasks"]):
                self.data["tasks"][idx]["done"] = True
                self.save_data()
                return f" Marked task #{idx + 1} as completed!"
            return " Invalid task number."
        except ValueError:
            return "Please provide a valid number. Example: 'done 1'"

    def clear_completed(self):
        initial_count = len(self.data["tasks"])
        self.data["tasks"] = [t for t in self.data["tasks"] if not t["done"]]
        self.save_data()
        removed = initial_count - len(self.data["tasks"])
        return f" Cleaned up {removed} completed task(s)."

    # ----------------- QUICK NOTES -----------------
    def add_note(self, note_text):
        if not note_text:
            return "Please provide text for your note. Example: 'note meeting at 4 PM'"
        timestamp = datetime.datetime.now().strftime("%Y-%m-%d %H:%M")
        self.data["notes"].append({"text": note_text, "time": timestamp})
        self.save_data()
        return f" Saved note: '{note_text}'"

    def list_notes(self):
        notes = self.data["notes"]
        if not notes:
            return " No saved notes found."
        output = [" Your Saved Notes:"]
        for idx, item in enumerate(notes, 1):
            output.append(f"  {idx}. [{item['time']}] {item['text']}")
        return "\n".join(output)

    # ----------------- CALCULATOR -----------------
    def calculate(self, expression):
        # Allow only safe arithmetic characters: digits, spaces, +, -, *, /, %, (, ), .
        if not re.match(r"^[\d\s\+\-\*\/\%\(\)\.]+$", expression):
            return "Invalid expression. Only numbers and operators (+, -, *, /, %, parenthesis) are allowed."
        try:
            # Evaluate using restricted globals/locals for safety
            result = eval(expression, {"__builtins__": None}, {})
            return f" Result: {expression} = {result}"
        except ZeroDivisionError:
            return "Error: Cannot divide by zero."
        except Exception:
            return "Could not calculate expression. Please check the format."

    # ----------------- TIME & DATE -----------------
    def get_time(self):
        now = datetime.datetime.now()
        return f" Current Date & Time: {now.strftime('%A, %B %d, %Y - %I:%M %p')}"

    # ----------------- HELP MENU -----------------
    def help_menu(self):
        return """
 Available Commands:
  • To-Do:
      - 'add task <text>'      : Add a new task
      - 'tasks' or 'list'      : View all tasks
      - 'done <number>'        : Mark a task as finished
      - 'clear done'           : Remove completed tasks
  • Notes:
      - 'note <text>'          : Save a quick note
      - 'notes'                : View all notes
  • Utility:
      - 'calc <expression>'    : Calculate math (e.g. 'calc (25 * 4) + 10')
      - 'time' or 'date'       : View current date and time
  • General:
      - 'help'                 : Display this help message
      - 'exit' or 'quit'       : Close assistant
"""

    # ----------------- INTENT DISPATCHER -----------------
    def respond(self, user_msg):
        text = user_msg.strip()
        lower = text.lower()

        if not text:
            return "How can I help you today? Type 'help' to see what I can do."

        # Exit
        if lower in ["exit", "quit", "bye"]:
            return None

        # Greetings
        if lower in ["hi", "hello", "hey"]:
            return "Hello! How can I assist your daily tasks today? (Type 'help' for commands)"

        # Help
        if lower in ["help", "commands"]:
            return self.help_menu()

        # Date & Time
        if any(keyword in lower for keyword in ["time", "date", "day", "clock"]):
            return self.get_time()

        # Tasks
        if lower.startswith("add task "):
            return self.add_task(text[9:].strip())
        elif lower.startswith("todo "):
            return self.add_task(text[5:].strip())
        elif lower in ["tasks", "list", "todo", "show tasks"]:
            return self.list_tasks()
        elif lower.startswith("done "):
            return self.complete_task(text[5:].strip())
        elif lower == "clear done":
            return self.clear_completed()

        # Notes
        if lower.startswith("note "):
            return self.add_note(text[5:].strip())
        elif lower in ["notes", "show notes", "list notes"]:
            return self.list_notes()

        # Calculator
        if lower.startswith("calc "):
            return self.calculate(text[5:].strip())
        elif lower.startswith("calculate "):
            return self.calculate(text[10:].strip())

        # Fallback response
        return "I didn't quite catch that. Type 'help' to see my supported commands, or try 'tasks', 'note', or 'time'."


def main():
    bot = DailyAssistant()
    print("=" * 55)
    print("      DAILY TASK ASSISTANT CHATBOT")
    print("=" * 55)
    print("Ready to help! Type 'help' for commands or 'exit' to quit.\n")

    while True:
        try:
            user_input = input("You > ")
            response = bot.respond(user_input)
            if response is None:
                print("Bot > Goodbye! Have a productive day!")
                break
            print(f"Bot > {response}\n")
        except (KeyboardInterrupt, EOFError):
            print("\nBot > Session ended. Goodbye!")
            break


if __name__ == "__main__":
    main()
    
