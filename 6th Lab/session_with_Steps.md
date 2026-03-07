# Firebase Realtime Database – Read & Write (Android + Kotlin + Jetpack Compose)

A step-by-step guide to building an Android app that reads and writes data to Firebase Realtime Database using Kotlin and Jetpack Compose.

---

## Step 1: Create a New Android Project

1. Open **Android Studio** → **New Project**
2. Select **Empty Activity**
3. Configure the project:
   - **Name:** `Read_write`
   - **Package:** `com.example.read_write`
   - **Language:** Kotlin
   - **Min SDK:** API 24+
4. Click **Finish**

---

## Step 2: Set Up Firebase

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add Project** → Name it → Click **Continue**
3. Disable Google Analytics *(optional)* → Click **Create Project**
4. On the project dashboard, click the **Android icon** to add an Android app
5. Enter your package name: `com.example.read_write`
6. Click **Register App**
7. Download the `google-services.json` file
8. Place it in your project's `app/` directory *(not the root)*

---

## Step 3: Enable Realtime Database

1. In Firebase Console → **Build** → **Realtime Database**
2. Click **Create Database**
3. Choose a location → Click **Next**
4. Select **Test Mode** *(allows open read/write for development)*
5. Click **Enable**

---

## Step 4: Add Firebase Dependencies

### Project-level `build.gradle`

```kotlin
plugins {
    id("com.google.gms.google-services") version "4.4.0" apply false
}
```

### App-level `build.gradle`

```kotlin
plugins {
    alias(libs.plugins.android.application)
    alias(libs.plugins.kotlin.android)
    alias(libs.plugins.kotlin.compose)
    id("com.google.gms.google-services") // add this line
}
```

```kotlin
dependencies {
    implementation(platform("com.google.firebase:firebase-bom:33.1.0")) // add this line
    implementation("com.google.firebase:firebase-database")              // add this line

    implementation(libs.androidx.core.ktx)
    implementation(libs.androidx.lifecycle.runtime.ktx)
    implementation(libs.androidx.activity.compose)
    implementation(platform(libs.androidx.compose.bom))
    implementation(libs.androidx.compose.ui)
    implementation(libs.androidx.compose.ui.graphics)
    implementation(libs.androidx.compose.ui.tooling.preview)
    implementation(libs.androidx.compose.material3)
    testImplementation(libs.junit)
    androidTestImplementation(libs.androidx.junit)
    androidTestImplementation(libs.androidx.espresso.core)
    androidTestImplementation(platform(libs.androidx.compose.bom))
    androidTestImplementation(libs.androidx.compose.ui.test.junit4)
    debugImplementation(libs.androidx.compose.ui.tooling)
    debugImplementation(libs.androidx.compose.ui.test.manifest)
}
```

> Click **Sync Now** when prompted.

---

## Step 5: Create the Data Class

Create a new Kotlin file: `StudentObj.kt`

```kotlin
package com.example.read_write

data class StudentObj(
    var studentName: String = "",
    var studentRollNumber: String = ""
)
```

> **Note:** Default empty-string values are required so Firebase can deserialize the object.

---

## Step 6: Update `MainActivity.kt`

Replace the contents of `MainActivity.kt` with the following:

```kotlin
package com.example.read_write

import android.content.Context
import android.os.Bundle
import android.widget.Toast
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.background
import androidx.compose.foundation.layout.Arrangement
import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxHeight
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.material3.Button
import androidx.compose.material3.Surface
import androidx.compose.material3.Text
import androidx.compose.material3.TextField
import androidx.compose.runtime.Composable
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.platform.LocalContext
import androidx.compose.ui.text.TextStyle
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.input.TextFieldValue
import androidx.compose.ui.tooling.preview.Preview
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import com.example.read_write.ui.theme.Read_writeTheme
import com.google.firebase.database.DataSnapshot
import com.google.firebase.database.DatabaseError
import com.google.firebase.database.DatabaseReference
import com.google.firebase.database.FirebaseDatabase
import com.google.firebase.database.ValueEventListener

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        setContent {
            Read_writeTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                ) {
                    val firebaseDatabase = FirebaseDatabase.getInstance()
                    val databaseReference = firebaseDatabase.getReference("StudentInfo")

                    firebaseUI(LocalContext.current, databaseReference)
                }
            }
        }
    }
}

@Composable
fun firebaseUI(context: Context, databaseReference: DatabaseReference) {
    val name = remember { mutableStateOf(TextFieldValue()) }
    val rollNo = remember { mutableStateOf(TextFieldValue()) }

    Column(
        modifier = Modifier
            .fillMaxHeight()
            .fillMaxWidth()
            .background(Color.White),
        verticalArrangement = Arrangement.Top,
        horizontalAlignment = Alignment.CenterHorizontally,
    ) {

        Text(
            text = "Firebase Database",
            modifier = Modifier.padding(15.dp),
            style = TextStyle(color = Color.Black, fontSize = 25.sp),
            fontWeight = FontWeight.Bold
        )

        TextField(
            value = name.value,
            onValueChange = { name.value = it },
            placeholder = { Text(text = "Enter your name") },
            modifier = Modifier
                .padding(15.dp)
                .fillMaxWidth(),
            textStyle = TextStyle(color = Color.Black, fontSize = 15.sp),
            singleLine = true
        )

        Spacer(modifier = Modifier.height(10.dp))

        TextField(
            value = rollNo.value,
            onValueChange = { rollNo.value = it },
            placeholder = { Text(text = "Enter your roll number") },
            modifier = Modifier
                .padding(15.dp)
                .fillMaxWidth(),
            textStyle = TextStyle(color = Color.Black, fontSize = 15.sp),
            singleLine = true
        )

        Spacer(modifier = Modifier.height(10.dp))

        // ── WRITE ──────────────────────────────────────────────────────────
        Button(
            onClick = {
                val studobj = StudentObj(name.value.text, rollNo.value.text)

                databaseReference.addValueEventListener(object : ValueEventListener {
                    override fun onDataChange(snapshot: DataSnapshot) {
                        databaseReference.setValue(studobj)
                        val dummyName = name.value.text
                        val dummyRoll = rollNo.value.text
                        Toast.makeText(context, "$dummyName $dummyRoll", Toast.LENGTH_LONG).show()
                    }

                    override fun onCancelled(error: DatabaseError) {
                        Toast.makeText(context, "Fail to add data to Firebase", Toast.LENGTH_LONG).show()
                    }
                })
            },
            modifier = Modifier
                .padding(18.dp)
                .fillMaxWidth()
        ) {
            Text(text = "Add data to Firebase", modifier = Modifier.padding(7.dp))
        }

        Spacer(modifier = Modifier.height(10.dp))

        // ── READ ───────────────────────────────────────────────────────────
        Button(
            onClick = {
                databaseReference.addValueEventListener(object : ValueEventListener {
                    override fun onDataChange(snapshot: DataSnapshot) {
                        val name = snapshot.child("studentName").getValue(String::class.java)
                        val rollNo = snapshot.child("studentRollNumber").getValue(String::class.java)
                        Toast.makeText(context, "Your data: $name $rollNo", Toast.LENGTH_LONG).show()
                    }

                    override fun onCancelled(error: DatabaseError) {
                        Toast.makeText(context, "Fail to read data from Firebase", Toast.LENGTH_LONG).show()
                    }
                })
            },
            modifier = Modifier
                .padding(18.dp)
                .fillMaxWidth()
        ) {
            Text(text = "Read data from Firebase", modifier = Modifier.padding(7.dp))
        }
    }
}

@Preview(showBackground = true)
@Composable
fun GreetingPreview() {
    Read_writeTheme {}
}
```

---

## Step 7: Add Internet Permission

In `AndroidManifest.xml`, add the following line **before** the `<application>` tag:

```xml
<uses-permission android:name="android.permission.INTERNET"/>
```

Your `AndroidManifest.xml` should look similar to this:

```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.example.read_write">

    <uses-permission android:name="android.permission.INTERNET"/>

    <application
        android:allowBackup="true"
        android:label="@string/app_name"
        android:theme="@style/Theme.Read_write"
        ...>
        <activity android:name=".MainActivity">
            ...
        </activity>
    </application>

</manifest>
```

---

## Step 8: Run the App

1. Connect a physical device or start an emulator
2. Click **Run ▶** in Android Studio
3. Enter a name and roll number in the text fields
4. Tap **Add data to Firebase** to write the record
5. Tap **Read data from Firebase** to retrieve and display it via a Toast

> You can also verify the data in real time on the **Firebase Console → Realtime Database** dashboard.

---

## Project Structure

```
app/
├── src/main/
│   ├── java/com/example/read_write/
│   │   ├── MainActivity.kt       ← UI + Firebase logic
│   │   └── StudentObj.kt         ← Data class
│   ├── res/
│   └── AndroidManifest.xml       ← Internet permission
├── google-services.json          ← Firebase config (place here)
└── build.gradle                  ← Firebase dependencies
```

---

## Troubleshooting

| Problem | Solution |
|---|---|
| `google-services.json` not found | Ensure the file is in `app/`, not the project root |
| Sync fails | Check that the `google-services` plugin version matches BOM version |
| Data not appearing | Confirm Test Mode is enabled in Firebase Realtime Database rules |
| Toast not showing | Verify Internet permission is added to `AndroidManifest.xml` |
