# My-personal-daily-planner-app

import json
import os

from kivy.app import App
from kivy.metrics import dp
from kivy.uix.boxlayout import BoxLayout
from kivy.uix.gridlayout import GridLayout
from kivy.uix.label import Label
from kivy.uix.button import Button
from kivy.uix.textinput import TextInput
from kivy.uix.scrollview import ScrollView


class PlannerApp(App):
    def build(self):
        self.title = "My Daily Planner"
        self.tasks = []
        self.file_path = os.path.join(self.user_data_dir, "tasks.json")
        self.load_tasks()

        root = BoxLayout(
            orientation="vertical",
            padding=dp(15),
            spacing=dp(10)
        )

        title = Label(
            text="MY DAILY PLANNER",
            font_size="25sp",
            bold=True,
            size_hint_y=None,
            height=dp(65)
        )
        root.add_widget(title)

        self.progress = Label(
            text="Your progress: 0%",
            font_size="18sp",
            size_hint_y=None,
            height=dp(40)
        )
        root.add_widget(self.progress)

        self.task_input = TextInput(
            hint_text="Enter a new task...",
            multiline=False,
            size_hint_y=None,
            height=dp(50),
            font_size="17sp"
        )
        root.add_widget(self.task_input)

        add_button = Button(
            text="ADD TASK",
            size_hint_y=None,
            height=dp(50),
            background_color=(0.15, 0.55, 0.85, 1)
        )
        add_button.bind(on_press=self.add_task)
        root.add_widget(add_button)

        scroll = ScrollView()
        self.task_list = GridLayout(
            cols=1,
            spacing=dp(8),
            size_hint_y=None
        )
        self.task_list.bind(
            minimum_height=self.task_list.setter("height")
        )
        scroll.add_widget(self.task_list)
        root.add_widget(scroll)

        self.refresh()
        return root

    def load_tasks(self):
        try:
            with open(self.file_path, "r", encoding="utf-8") as f:
                self.tasks = json.load(f)
        except:
            self.tasks = []

    def save_tasks(self):
        os.makedirs(self.user_data_dir, exist_ok=True)
        with open(self.file_path, "w", encoding="utf-8") as f:
            json.dump(self.tasks, f, ensure_ascii=False)

    def add_task(self, instance):
        name = self.task_input.text.strip()
        if not name:
            return

        self.tasks.append({
            "name": name,
            "done": False
        })
        self.task_input.text = ""
        self.save_tasks()
        self.refresh()

    def toggle_task(self, index):
        self.tasks[index]["done"] = not self.tasks[index]["done"]
        self.save_tasks()
        self.refresh()

    def delete_task(self, index):
        self.tasks.pop(index)
        self.save_tasks()
        self.refresh()

    def refresh(self):
        self.task_list.clear_widgets()

        total = len(self.tasks)
        completed = sum(1 for t in self.tasks if t["done"])
        percent = int(completed / total * 100) if total else 0
        self.progress.text = f"Your progress: {percent}%"

        for i, task in enumerate(self.tasks):
            row = BoxLayout(
                size_hint_y=None,
                height=dp(58),
                spacing=dp(5)
            )

            status = "DONE" if task["done"] else "TODO"
            name = task["name"]

            label = Label(
                text=f"{status}  {name}",
                font_size="15sp",
                halign="left",
                valign="middle"
            )
            label.bind(size=lambda obj, size: setattr(
                obj, "text_size", size
            ))
            row.add_widget(label)

            done_btn = Button(
                text="✓",
                size_hint_x=None,
                width=dp(48)
            )
            done_btn.bind(
                on_press=lambda btn, x=i: self.toggle_task(x)
            )
            row.add_widget(done_btn)

            delete_btn = Button(
                text="X",
                size_hint_x=None,
                width=dp(48),
                background_color=(0.8, 0.2, 0.2, 1)
            )
            delete_btn.bind(
                on_press=lambda btn, x=i: self.delete_task(x)
            )
            row.add_widget(delete_btn)

            self.task_list.add_widget(row)


if __name__ == "__main__":
    PlannerApp().run()
    
