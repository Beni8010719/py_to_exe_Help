# py_to_exe_Help
I need help my Computer don´t make it pls
i need this in a exe

import sys
import pandas as pd
from PyQt5.QtWidgets import QApplication, QMainWindow, QPushButton, QFileDialog, QVBoxLayout, QWidget, QTableWidget, QTableWidgetItem
from datetime import datetime

class CSVEditorApp(QMainWindow):
    def __init__(self):
        super().__init__()
        self.initUI()

    def initUI(self):
        self.setWindowTitle('CSV Editor')
        self.setGeometry(100, 100, 600, 400)

        self.open_button = QPushButton('CSV-Datei öffnen', self)
        self.open_button.clicked.connect(self.open_csv)

        self.save_button = QPushButton('CSV-Datei speichern', self)
        self.save_button.clicked.connect(self.save_csv)
        self.save_button.setEnabled(False)

        self.table = QTableWidget(self)

        layout = QVBoxLayout()
        layout.addWidget(self.open_button)
        layout.addWidget(self.save_button)
        layout.addWidget(self.table)

        container = QWidget(self)
        container.setLayout(layout)
        self.setCentralWidget(container)

    def open_csv(self):
        options = QFileDialog.Options()
        file_name, _ = QFileDialog.getOpenFileName(self, "CSV-Datei öffnen", "", "CSV-Dateien (*.csv);;Alle Dateien (*)", options=options)
        if file_name:
            # Datei mit Zeichensatz 'ISO-8859-1' (Latin-1) öffnen und nur Zeile 2 und folgende Zeilen lesen
            self.df = pd.read_csv(file_name, sep=';', encoding='ISO-8859-1', skiprows=1)

            # Überprüfen und Umwandeln der Spalten 'Konto' und 'Gegenkonto (ohne BU-Schlüssel)' in Zeichenketten
            if 'Konto' in self.df.columns:
                self.df['Konto'] = self.df['Konto'].astype(str)
            if 'Gegenkonto (ohne BU-Schlüssel)' in self.df.columns:
                self.df['Gegenkonto (ohne BU-Schlüssel)'] = self.df['Gegenkonto (ohne BU-Schlüssel)'].astype(str)

            # Überprüfen und Umwandeln der Spalte 'Belegdatum'
            if 'Belegdatum' in self.df.columns:
                self.df['Belegdatum'] = self.df['Belegdatum'].apply(self.convert_date)

            # Überprüfen und Umwandeln der Spalte 'Belegdatum'
            if 'Belegfeld 2' in self.df.columns:
                self.df['Belegfeld 2'] = self.df['Belegfeld 2'].apply(self.convert_datee)

            # Fehlende Werte (NaN) durch leere Zeichenketten ersetzen
            self.df.fillna('', inplace=True)

            # Hier können Sie die Daten bearbeiten (Beispiel: Ersetzen der Werte in den Spalten 'Konto' und 'Gegenkonto (ohne BU-Schlüssel)')
            if 'Konto' in self.df.columns:
                self.df['Konto'] = self.df['Konto'].apply(lambda x: self.replace_first_chars(x))
            if 'Gegenkonto (ohne BU-Schlüssel)' in self.df.columns:
                self.df['Gegenkonto (ohne BU-Schlüssel)'] = self.df['Gegenkonto (ohne BU-Schlüssel)'].apply(lambda x: self.replace_first_chars(x))

            # Daten in der Tabelle darstellen
            self.display_data()

            self.save_button.setEnabled(True)

            # Druckausgabe zum Überprüfen der Spalten nach dem Ersetzen
            print(self.df.head())

    def save_csv(self):
        options = QFileDialog.Options()
        options |= QFileDialog.DontUseNativeDialog
        save_file_name, _ = QFileDialog.getSaveFileName(self, "CSV-Datei speichern", "", "CSV-Dateien (*.csv);;Alle Dateien (*)", options=options)
        if save_file_name:
            # Entfernen der ersten Zeile aus dem DataFrame (index 0 entspricht der ersten Zeile)
            self.df = self.df.iloc[1:]

            # Bearbeitete Daten in eine neue CSV-Datei speichern mit Semikolon als Trennzeichen und Zeichensatz 'ISO-8859-1'
            self.df.to_csv(save_file_name, sep=';', index=False, encoding='ISO-8859-1')
            print("CSV-Datei erfolgreich bearbeitet und gespeichert.")

    def display_data(self):
        # Daten in der Tabelle darstellen
        self.table.setRowCount(self.df.shape[0])
        self.table.setColumnCount(self.df.shape[1])
        self.table.setHorizontalHeaderLabels(self.df.columns)

        for row in range(self.df.shape[0]):
            for col in range(self.df.shape[1]):
                item = QTableWidgetItem(str(self.df.iat[row, col]))
                self.table.setItem(row, col, item)

    def replace_first_chars(self, value):
        replacements = {
            '2': 'K',
            '1': 'D',
            '3': 'S'
        }
        # Ersatz der ersten beiden Zeichen, die restlichen Zeichen bleiben unverändert
        return ''.join(replacements.get(char, char) if i < 2 else char for i, char in enumerate(value))

    def convert_date(self, value):
        # Funktion zum Konvertieren des Datumsformats (z. B. '3107' -> '31.07.2023')
        try:
            date_str = str(value)
            year = '2023'
            day = date_str[:-2].zfill(2)  # Fügt führende Null hinzu, wenn nötig
            month = date_str[-2:]
            return f'{day}.{month}.{year}'
        except ValueError:
            return value

    def convert_datee(self, value):
        # Funktion zum Konvertieren des Datumsformats (z. B. '310723' -> '31.07.2023')
        try:
            date_str = str(value)
            year = '20' + date_str[-2:]  # Annahme, dass es immer um das Jahr 20XX geht
            day = date_str[:-4].zfill(2)  # Fügt führende Null hinzu, wenn nötig
            month = date_str[-4:-2]
            return f'{day}.{month}.{year}'
        except ValueError:
            return value

if __name__ == "__main__":
    app = QApplication(sys.argv)
    window = CSVEditorApp()
    window.show()
    sys.exit(app.exec_())
