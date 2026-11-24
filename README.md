
import sys

from PyQt6.QtCore import Qt, pyqtSignal
from PyQt6.QtGui import QAction
from PyQt6.QtWidgets import (
    QApplication,
    QMainWindow,
    QWidget,
    QLabel,
    QLineEdit,
    QPushButton,
    QSpinBox,
    QComboBox,
    QCheckBox,
    QRadioButton,
    QSlider,
    QProgressBar,
    QTextEdit,
    QGroupBox,
    QTabWidget,
    QVBoxLayout,
    QHBoxLayout,
    QFormLayout,
    QDialog,
    QDialogButtonBox,
    QFileDialog,
    QMessageBox,
    QInputDialog,
)


# ==========================================================
# 1. Власний віджет для введення даних
# ==========================================================

class StudentForm(QWidget):
    # Власні сигнали з різними типами параметрів
    name_changed = pyqtSignal(str)
    student_submitted = pyqtSignal(int, str)

    def __init__(self):
        super().__init__()

        self.setObjectName("studentForm")

        self.name_edit = QLineEdit()
        self.name_edit.setPlaceholderText("Введіть ім'я та прізвище")
        self.name_edit.setToolTip("Поле для введення ПІБ студента")

        self.age_spin = QSpinBox()
        self.age_spin.setRange(16, 100)
        self.age_spin.setValue(19)
        self.age_spin.setToolTip("Вкажіть вік студента")

        self.course_combo = QComboBox()
        self.course_combo.addItems([
            "Python",
            "Java",
            "C++",
            "C#",
            "JavaScript"
        ])

        self.level_slider = QSlider(Qt.Orientation.Horizontal)
        self.level_slider.setRange(1, 10)
        self.level_slider.setValue(5)

        self.level_label = QLabel("Рівень знань: 5/10")

        self.progress = QProgressBar()
        self.progress.setRange(1, 10)
        self.progress.setValue(5)
        self.progress.setFormat("%v з 10")

        self.agree_check = QCheckBox(
            "Підтверджую правильність введених даних"
        )

        self.first_radio = QRadioButton("Початківець")
        self.second_radio = QRadioButton("Середній рівень")
        self.third_radio = QRadioButton("Впевнений користувач")

        self.first_radio.setChecked(True)

        self.preview_label = QLabel("Ваше ім'я з'явиться тут")
        self.preview_label.setObjectName("previewLabel")

        self.submit_button = QPushButton("Надіслати дані")
        self.submit_button.setObjectName("submitButton")

        self.reset_button = QPushButton("Очистити форму")

        # Сигнали та слоти
        self.name_edit.textChanged.connect(self.preview_label.setText)
        self.name_edit.textChanged.connect(self.name_changed.emit)

        self.level_slider.valueChanged.connect(
            self.update_level
        )

        self.level_slider.valueChanged.connect(
            self.progress.setValue
        )

        self.agree_check.toggled.connect(
            self.submit_button.setEnabled
        )

        self.submit_button.clicked.connect(
            self.send_data
        )

        self.reset_button.clicked.connect(
            self.reset_form
        )

        # Розміщення елементів
        form_layout = QFormLayout()
        form_layout.addRow("ПІБ студента:", self.name_edit)
        form_layout.addRow("Вік:", self.age_spin)
        form_layout.addRow("Мова програмування:", self.course_combo)

        level_layout = QVBoxLayout()
        level_layout.addWidget(self.level_label)
        level_layout.addWidget(self.level_slider)
        level_layout.addWidget(self.progress)

        level_widget = QWidget()
        level_widget.setLayout(level_layout)

        form_layout.addRow("Рівень знань:", level_widget)

        radio_layout = QVBoxLayout()
        radio_layout.addWidget(self.first_radio)
        radio_layout.addWidget(self.second_radio)
        radio_layout.addWidget(self.third_radio)

        radio_widget = QWidget()
        radio_widget.setLayout(radio_layout)

        form_layout.addRow("Підготовка:", radio_widget)

        buttons_layout = QHBoxLayout()
        buttons_layout.addWidget(self.submit_button)
        buttons_layout.addWidget(self.reset_button)

        main_layout = QVBoxLayout()
        main_layout.addLayout(form_layout)
        main_layout.addWidget(self.agree_check)
        main_layout.addWidget(self.preview_label)
        main_layout.addLayout(buttons_layout)

        self.setLayout(main_layout)

    def update_level(self, value):
        self.level_label.setText(
            f"Рівень знань: {value}/10"
        )

    # Окрема функція перевірки введених даних
    def validate_form(self):
        name = self.name_edit.text().strip()

        if len(name) < 2:
            return False, "Введіть ім'я та прізвище."

        if not self.agree_check.isChecked():
            return False, "Підтвердіть правильність введених даних."

        return True, ""

    def send_data(self):
        valid, message = self.validate_form()

        if not valid:
            QMessageBox.warning(
                self,
                "Помилка введення",
                message
            )
            return

        name = self.name_edit.text().strip()
        age = self.age_spin.value()

        self.student_submitted.emit(age, name)

    def reset_form(self):
        self.name_edit.clear()
        self.age_spin.setValue(19)
        self.course_combo.setCurrentIndex(0)
        self.level_slider.setValue(5)
        self.agree_check.setChecked(False)
        self.first_radio.setChecked(True)
        self.preview_label.setText(
            "Ваше ім'я з'явиться тут"
        )


# ==========================================================
# 2. Другий власний віджет для відображення результату
# ==========================================================

class StudentSummary(QWidget):

    def __init__(self):
        super().__init__()

        self.setObjectName("studentSummary")

        self.title_label = QLabel("Результат заповнення форми")
        self.title_label.setObjectName("sectionTitle")

        self.result_text = QTextEdit()
        self.result_text.setReadOnly(True)
        self.result_text.setPlaceholderText(
            "Тут відображатимуться введені дані..."
        )

        self.clear_button = QPushButton("Очистити результат")
        self.clear_button.clicked.connect(
            self.clear_result
        )

        layout = QVBoxLayout()
        layout.addWidget(self.title_label)
        layout.addWidget(self.result_text)
        layout.addWidget(self.clear_button)

        self.setLayout(layout)

    # Метод приймає два параметри: ціле число та рядок
    def show_student(self, age, name):
        self.result_text.append(
            f"Ім'я студента: {name}\n"
            f"Вік: {age} років\n"
            "Дані успішно отримано.\n"
            "-----------------------------"
        )

    def clear_result(self):
        self.result_text.clear()


# ==========================================================
# 3. Власне модальне діалогове вікно
# ==========================================================

class StudentDialog(QDialog):

    def __init__(self, parent=None):
        super().__init__(parent)

        self.setWindowTitle("Додаткові дані студента")
        self.setMinimumWidth(350)

        self.name_edit = QLineEdit()
        self.name_edit.setPlaceholderText("Введіть ім'я")

        self.specialty_edit = QLineEdit()
        self.specialty_edit.setPlaceholderText(
            "Введіть спеціальність"
        )

        self.year_spin = QSpinBox()
        self.year_spin.setRange(1, 6)
        self.year_spin.setValue(1)

        form_layout = QFormLayout()
        form_layout.addRow("Ім'я:", self.name_edit)
        form_layout.addRow("Спеціальність:", self.specialty_edit)
        form_layout.addRow("Курс навчання:", self.year_spin)

        buttons = QDialogButtonBox(
            QDialogButtonBox.StandardButton.Ok
            | QDialogButtonBox.StandardButton.Cancel
        )

        buttons.accepted.connect(self.accept)
        buttons.rejected.connect(self.reject)

        layout = QVBoxLayout()
        layout.addLayout(form_layout)
        layout.addWidget(buttons)

        self.setLayout(layout)

    def get_data(self):
        return {
            "name": self.name_edit.text().strip(),
            "specialty": self.specialty_edit.text().strip(),
            "year": self.year_spin.value()
        }


# ==========================================================
# 4. Головне вікно програми
# ==========================================================

class MainWindow(QMainWindow):

    def __init__(self):
        super().__init__()

        self.setWindowTitle(
            "Лабораторна робота №4 — PyQt6"
        )
        self.resize(1000, 750)
        self.setMinimumSize(800, 600)

        self.form = StudentForm()
        self.summary = StudentSummary()

        self.create_interface()
        self.create_actions()
        self.create_menus()
        self.create_toolbar()
        self.connect_custom_signals()
        self.apply_styles()

        self.statusBar().showMessage(
            "Програму запущено. Заповніть форму."
        )

    # Створення центрального віджета та вкладок
    def create_interface(self):
        self.tabs = QTabWidget()

        self.tabs.addTab(
            self.form,
            "Форма студента"
        )

        self.tabs.addTab(
            self.summary,
            "Результат"
        )

        self.signal_checkbox = QCheckBox(
            "Блокувати власні сигнали форми"
        )

        self.signal_checkbox.toggled.connect(
            self.set_signals_blocked
        )

        self.signal_info = QLabel(
            "Власні сигнали форми активні."
        )

        self.signal_info.setObjectName("signalInfo")

        central_widget = QWidget()

        layout = QVBoxLayout()
        layout.addWidget(self.tabs)
        layout.addWidget(self.signal_checkbox)
        layout.addWidget(self.signal_info)

        central_widget.setLayout(layout)

        self.setCentralWidget(central_widget)

    # Підключення власних сигналів між віджетами
    def connect_custom_signals(self):
        self.form.student_submitted.connect(
            self.summary.show_student
        )

        self.form.name_changed.connect(
            self.show_name_status
        )

    def show_name_status(self, name):
        if name.strip():
            self.statusBar().showMessage(
                f"Введене ім'я: {name}"
            )
        else:
            self.statusBar().showMessage(
                "Поле імені очищено."
            )

    # Демонстрація blockSignals()
    def set_signals_blocked(self, blocked):
        self.form.blockSignals(blocked)

        if blocked:
            self.signal_info.setText(
                "Власні сигнали форми заблоковані."
            )
        else:
            self.signal_info.setText(
                "Власні сигнали форми активні."
            )

    # Створення дій меню
    def create_actions(self):
        self.open_action = QAction("Відкрити файл", self)
        self.open_action.setShortcut("Ctrl+O")
        self.open_action.setStatusTip(
            "Відкрити текстовий файл"
        )
        self.open_action.triggered.connect(
            lambda checked=False: self.open_file()
        )

        self.save_action = QAction("Зберегти результат", self)
        self.save_action.setShortcut("Ctrl+S")
        self.save_action.setStatusTip(
            "Зберегти результат у файл"
        )
        self.save_action.triggered.connect(
            lambda checked=False: self.save_file()
        )

        self.exit_action = QAction("Вихід", self)
        self.exit_action.setShortcut("Ctrl+Q")
        self.exit_action.triggered.connect(
            lambda checked=False: self.close()
        )

        self.input_action = QAction(
            "Ввести ім'я через діалог", self
        )
        self.input_action.triggered.connect(
            lambda checked=False: self.input_dialog()
        )

        self.custom_dialog_action = QAction(
            "Додаткові дані", self
        )
        self.custom_dialog_action.triggered.connect(
            lambda checked=False: self.show_custom_dialog()
        )

        self.clear_action = QAction(
            "Очистити результат", self
        )
        self.clear_action.triggered.connect(
            lambda checked=False: self.summary.clear_result()
        )

        self.about_action = QAction("Про програму", self)
        self.about_action.triggered.connect(
            lambda checked=False: self.show_about()
        )

        self.help_action = QAction("Інструкція", self)
        self.help_action.triggered.connect(
            lambda checked=False: self.show_help()
        )

    # Створення меню
    def create_menus(self):
        file_menu = self.menuBar().addMenu("Файл")
        file_menu.addAction(self.open_action)
        file_menu.addAction(self.save_action)
        file_menu.addSeparator()
        file_menu.addAction(self.exit_action)

        tools_menu = self.menuBar().addMenu("Інструменти")
        tools_menu.addAction(self.input_action)
        tools_menu.addAction(self.custom_dialog_action)
        tools_menu.addAction(self.clear_action)

        help_menu = self.menuBar().addMenu("Довідка")
        help_menu.addAction(self.help_action)
        help_menu.addAction(self.about_action)

    # Панель інструментів використовує ті самі QAction
    def create_toolbar(self):
        toolbar = self.addToolBar("Панель інструментів")
        toolbar.addAction(self.open_action)
        toolbar.addAction(self.save_action)
        toolbar.addAction(self.input_action)
        toolbar.addAction(self.custom_dialog_action)

    # Стандартний діалог відкриття файлу
    def open_file(self):
        file_name, _ = QFileDialog.getOpenFileName(
            self,
            "Відкрити файл",
            "",
            "Текстові файли (*.txt);;Усі файли (*)"
        )

        if not file_name:
            return

        try:
            with open(file_name, "r", encoding="utf-8") as file:
                content = file.read()

            self.summary.result_text.setPlainText(content)
            self.tabs.setCurrentWidget(self.summary)

            self.statusBar().showMessage(
                "Файл успішно відкрито."
            )

        except OSError as error:
            QMessageBox.critical(
                self,
                "Помилка",
                f"Не вдалося відкрити файл:\n{error}"
            )

    # Стандартний діалог збереження файлу
    def save_file(self):
        file_name, _ = QFileDialog.getSaveFileName(
            self,
            "Зберегти результат",
            "result.txt",
            "Текстові файли (*.txt)"
        )

        if not file_name:
            return

        try:
            with open(file_name, "w", encoding="utf-8") as file:
                file.write(
                    self.summary.result_text.toPlainText()
                )

            QMessageBox.information(
                self,
                "Збереження",
                "Результат успішно збережено."
            )

        except OSError as error:
            QMessageBox.critical(
                self,
                "Помилка",
                f"Не вдалося зберегти файл:\n{error}"
            )

    # Діалог введення даних QInputDialog
    def input_dialog(self):
        name, accepted = QInputDialog.getText(
            self,
            "Введення імені",
            "Введіть ім'я студента:"
        )

        if accepted and name.strip():
            self.form.name_edit.setText(name.strip())
            self.statusBar().showMessage(
                "Ім'я введено через діалогове вікно."
            )

    # Власне модальне діалогове вікно
    def show_custom_dialog(self):
        dialog = StudentDialog(self)

        if dialog.exec() == QDialog.DialogCode.Accepted:
            data = dialog.get_data()

            if not data["name"] or not data["specialty"]:
                QMessageBox.warning(
                    self,
                    "Помилка",
                    "Заповніть ім'я та спеціальність."
                )
                return

            self.summary.result_text.append(
                "\nДОДАТКОВІ ДАНІ\n"
                f"Ім'я: {data['name']}\n"
                f"Спеціальність: {data['specialty']}\n"
                f"Курс: {data['year']}\n"
                "-----------------------------"
            )

            self.tabs.setCurrentWidget(self.summary)

    def show_about(self):
        QMessageBox.information(
            self,
            "Про програму",
            "Навчальна програма створена на Python "
            "з використанням бібліотеки PyQt6.\n\n"
            "Лабораторна робота №4."
        )

    def show_help(self):
        QMessageBox.information(
            self,
            "Інструкція",
            "1. Заповніть поля форми.\n"
            "2. Підтвердіть правильність даних.\n"
            "3. Натисніть «Надіслати дані».\n"
            "4. Перегляньте результат у відповідній вкладці.\n"
            "5. Для збереження скористайтеся меню «Файл»."
        )

    # Глобальне оформлення QSS
    def apply_styles(self):
        self.setStyleSheet("""
            QWidget {
                font-family: Arial;
                font-size: 14px;
            }

            QMainWindow {
                background-color: #f2f4f8;
            }

            QGroupBox {
                border: 1px solid #aeb8c5;
                border-radius: 7px;
                margin-top: 10px;
                padding: 10px;
            }

            QLabel {
                color: #263445;
            }

            QLineEdit, QSpinBox, QComboBox, QTextEdit {
                background-color: white;
                border: 1px solid #aeb8c5;
                border-radius: 5px;
                padding: 6px;
            }

            QLineEdit:focus, QTextEdit:focus {
                border: 2px solid #3978c5;
            }

            QPushButton {
                background-color: #3978c5;
                color: white;
                border: none;
                border-radius: 5px;
                padding: 8px 14px;
            }

            QPushButton:hover {
                background-color: #285f9f;
            }

            QPushButton:pressed {
                background-color: #1d4778;
            }

            QPushButton:disabled {
                background-color: #aeb8c5;
                color: #eeeeee;
            }

            #submitButton {
                font-weight: bold;
                background-color: #26845b;
            }

            #submitButton:hover {
                background-color: #1d6847;
            }

            #previewLabel {
                color: #3978c5;
                font-weight: bold;
                padding: 7px;
            }

            #sectionTitle {
                font-size: 18px;
                font-weight: bold;
                padding: 5px;
            }

            #signalInfo {
                color: #555f6b;
                font-style: italic;
            }

            QTabWidget::pane {
                border: 1px solid #aeb8c5;
                border-radius: 5px;
            }

            QTabBar::tab {
                padding: 8px 15px;
                background-color: #dfe5ed;
            }

            QTabBar::tab:selected {
                background-color: white;
                font-weight: bold;
            }
        """)


# ==========================================================
# 5. Запуск програми
# ==========================================================

def main():
    app = QApplication(sys.argv)
    app.setStyle("Fusion")

    window = MainWindow()
    window.show()

    sys.exit(app.exec())


if __name__ == "__main__":
    main()
