# Baza_danych


dependencies {
    implementation 'androidx.appcompat:appcompat:1.2.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.0.4'
    implementation 'androidx.sqlite:sqlite:2.1.0'
}


package com.example.czolko;

import android.content.Context;
import android.database.Cursor;
import android.database.sqlite.SQLiteDatabase;
import android.database.sqlite.SQLiteOpenHelper;

import java.util.ArrayList;
import java.util.List;

public class DatabaseHelper extends SQLiteOpenHelper {

    private static final String DATABASE_NAME = "czolko.db";
    private static final int DATABASE_VERSION = 1;

    public DatabaseHelper(Context context) {
        super(context, DATABASE_NAME, null, DATABASE_VERSION);
    }

    @Override
    public void onCreate(SQLiteDatabase db) {
        db.execSQL("CREATE TABLE categories (id INTEGER PRIMARY KEY AUTOINCREMENT, name TEXT)");
        db.execSQL("CREATE TABLE words (id INTEGER PRIMARY KEY AUTOINCREMENT, category_id INTEGER, word TEXT)");
        
        // Dodaj kategorie i słowa
        db.execSQL("INSERT INTO categories (name) VALUES ('Zwierzęta'), ('Państwa'), ('Jedzenie'), ('Kolory'), ('Zawody'), ('Owoce'), ('Samochody'), ('Sporty'), ('Filmy'), ('Napoje')");
        db.execSQL("INSERT INTO words (category_id, word) VALUES " +
            "(1, 'kot'), (1, 'pies'), (1, 'słoń'), (1, 'żyrafa'), (1, 'jeleń'), (1, 'tygrys'), " +
            "(2, 'Polska'), (2, 'Niemcy'), (2, 'Francja'), (2, 'Włochy'), (2, 'Hiszpania'), (2, 'Japonia'), " +
            "(3, 'pizza'), (3, 'sushi'), (3, 'hamburger'), (3, 'spaghetti'), (3, 'sałatka'), (3, 'kebab'), " +
            "(4, 'czerwony'), (4, 'niebieski'), (4, 'zielony'), (4, 'żółty'), (4, 'fioletowy'), (4, 'pomarańczowy'), " +
            "(5, 'lekarz'), (5, 'nauczyciel'), (5, 'informatyk'), (5, 'policjant'), (5, 'dziennikarz'), (5, 'artysta'), " +
            "(6, 'jabłko'), (6, 'banan'), (6, 'truskawka'), (6, 'winogrono'), (6, 'pomarańcza'), (6, 'ananas'), " +
            "(7, 'BMW'), (7, 'Audi'), (7, 'Mercedes'), (7, 'Toyota'), (7, 'Honda'), (7, 'Volkswagen'), " +
            "(8, 'piłka nożna'), (8, 'koszykówka'), (8, 'siatkówka'), (8, 'bieganie'), (8, 'pływanie'), (8, 'rower'), " +
            "(9, 'Forrest Gump'), (9, 'Gwiezdne wojny'), (9, 'Titanic'), (9, 'Harry Potter'), (9, 'Avengers'), (9, 'Piraci z Karaibów'), " +
            "(10, 'kawa'), (10, 'herbata'), (10, 'cola'), (10, 'sok'), (10, 'woda'), (10, 'piwo')");
    }

    @Override
    public void onUpgrade(SQLiteDatabase db, int oldVersion, int newVersion) {
        db.execSQL("DROP TABLE IF EXISTS categories");
        db.execSQL("DROP TABLE IF EXISTS words");
        onCreate(db);
    }

    public List<String> getAllCategories() {
        List<String> categories = new ArrayList<>();
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery("SELECT name FROM categories", null);
        if (cursor.moveToFirst()) {
            do {
                categories.add(cursor.getString(0));
            } while (cursor.moveToNext());
        }
        cursor.close();
        return categories;
    }

    public List<String> getWordsForCategory(String categoryName) {
        List<String> words = new ArrayList<>();
        SQLiteDatabase db = this.getReadableDatabase();
        Cursor cursor = db.rawQuery("SELECT words.word FROM words JOIN categories ON words.category_id = categories.id WHERE categories.name = ?", new String[]{categoryName});
        if (cursor.moveToFirst()) {
            do {
                words.add(cursor.getString(0));
            } while (cursor.moveToNext());
        }
        cursor.close();
        return words;
    }
}



package com.example.czolko;

import androidx.appcompat.app.AppCompatActivity;

import android.content.res.Configuration;
import android.os.Bundle;
import android.view.View;
import android.widget.TextView;

import java.util.List;
import java.util.Random;

public class MainActivity extends AppCompatActivity {

    private DatabaseHelper dbHelper;
    private List<String> currentWords;
    private int currentWordIndex = 0;
    private TextView currentWordTextView;
    private List<String> categories;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        dbHelper = new DatabaseHelper(this);
        currentWordTextView = findViewById(R.id.currentWordTextView);

        // Przywrócenie stanu gry po zmianie orientacji ekranu
        if (savedInstanceState != null) {
            currentWords = savedInstanceState.getStringArrayList("currentWords");
            currentWordIndex = savedInstanceState.getInt("currentWordIndex");
            displayCurrentWord();
        }
    }

    // Metoda wywoływana po kliknięciu przycisku "Następne hasło"
    public void nextWord(View view) {
        if (currentWords == null || currentWords.isEmpty()) {
            return;
        }
        currentWordIndex = (currentWordIndex + 1) % currentWords.size();
        displayCurrentWord();
    }

    // Metoda wywoływana po kliknięciu przycisku "Wybierz kategorię"
    public void chooseCategory(View view) {
        if (categories == null) {
            categories = dbHelper.getAllCategories();
        }
        Random random = new Random();
        String category = categories.get(random.nextInt(categories.size()));
        currentWords = dbHelper.getWordsForCategory(category);
        currentWordIndex = 0;
        displayCurrentWord();
    }

    // Wyświetla aktualne hasło w TextView
    private void displayCurrentWord() {
        if (currentWords == null || currentWords.isEmpty()) {
            return;
        }
        currentWordTextView.setText(currentWords.get(currentWordIndex));
    }

    @Override
    protected void onSaveInstanceState(Bundle outState) {
        super.onSaveInstanceState(outState);
        // Zapisanie stanu gry
        outState.putStringArrayList("currentWords", new ArrayList<>(currentWords));
        outState.putInt("currentWordIndex", currentWordIndex);
    }

    @Override
    public void onConfigurationChanged(Configuration newConfig) {
        super.onConfigurationChanged(newConfig);
        // Zablokowanie resetowania gry po zmianie orientacji ekranu
    }
}


<!-- activity_main.xml -->
<?xml version="1.0" encoding="utf-8"?>
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:padding="16dp"
    tools:context=".MainActivity">

    <TextView
        android:id="@+id/currentWordTextView"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text=""
        android:textSize="24sp"
        android:textStyle="bold"
        android:layout_centerInParent="true"/>

    <Button
        android:id="@+id/nextButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Następne hasło"
        android:layout_alignParentBottom="true"
        android:layout_centerHorizontal="true"
        android:layout_marginBottom="24dp"
        android:onClick="nextWord"/>

    <Button
        android:id="@+id/categoryButton"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Wybierz kategorię"
        android:layout_above="@id/nextButton"
        android:layout_centerHorizontal="true
        
