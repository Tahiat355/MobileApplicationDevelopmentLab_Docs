
# 📝 Todo App Setup Guide (Jetpack Compose + ViewModel)

This guide walks through adding dependencies, creating a ViewModel, setting up the UI, and connecting everything in `MainActivity`.

---

## Step 1: Add Dependencies

Open `build.gradle.kts (Module :app)`

Find the `dependencies` block and add these lines:

```kotlin
implementation("androidx.lifecycle:lifecycle-viewmodel-compose:2.6.1")
implementation("androidx.activity:activity-compose:1.7.2")
```

Click **"Sync Now"** at the top right when the yellow banner appears.

---

## Step 2: Create the ViewModel Class

Right-click on `com.example.apppractice` (in `kotlin+java` folder)
**New → Kotlin Class/File**
Name it: `TodoViewModel`
Select **"Class"**

Paste this code:

```kotlin
package com.example.apppractice

import androidx.lifecycle.ViewModel
import androidx.compose.runtime.mutableStateListOf

class TodoViewModel : ViewModel() {
    var tasks = mutableStateListOf<String>()
        private set

    fun addTask(task: String) {
        tasks.add(task)
    }
}
```

---

## Step 3: Create Separate File

Right-click on `com.example.apppractice`
**New → Kotlin Class/File**
Name it: `TodoApp`
Select **"File"**

Paste this code:

```kotlin
package com.example.apppractice

import androidx.compose.foundation.layout.Column
import androidx.compose.foundation.layout.Spacer
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.foundation.layout.fillMaxWidth
import androidx.compose.foundation.layout.height
import androidx.compose.foundation.layout.padding
import androidx.compose.foundation.text.KeyboardActions
import androidx.compose.foundation.text.KeyboardOptions
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.OutlinedTextField
import androidx.compose.material3.Text
import androidx.compose.runtime.Composable
import androidx.compose.runtime.getValue
import androidx.compose.runtime.mutableStateOf
import androidx.compose.runtime.remember
import androidx.compose.runtime.setValue
import androidx.compose.ui.Modifier
import androidx.compose.ui.text.input.ImeAction
import androidx.compose.ui.unit.dp

@Composable
fun TodoApp(viewModel: TodoViewModel) {
    var text by remember { mutableStateOf("") }

    Column(
        modifier = Modifier
            .fillMaxSize()
            .padding(16.dp)
    ) {
        OutlinedTextField(
            value = text,
            onValueChange = { text = it },
            label = { Text("Enter a task") },
            modifier = Modifier.fillMaxWidth(),
            keyboardOptions = KeyboardOptions.Default.copy(imeAction = ImeAction.Done),
            keyboardActions = KeyboardActions(
                onDone = {
                    if (text.isNotBlank()) {
                        viewModel.addTask(text)
                        text = ""
                    }
                }
            )
        )

        Spacer(modifier = Modifier.height(16.dp))
        Text("Tasks:", style = MaterialTheme.typography.titleMedium)
        Spacer(modifier = Modifier.height(8.dp))

        for (task in viewModel.tasks) {
            Text("• $task")
        }
    }
}
```

---

## Step 4: Update `MainActivity.kt`

Replace your `MainActivity.kt` with this:

```kotlin
package com.example.apppractice

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.lifecycle.viewmodel.compose.viewModel
import com.example.apppractice.ui.theme.APPPRACTICETheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContent {
            val todoViewModel: TodoViewModel = viewModel()
            TodoApp(viewModel = todoViewModel)
        }
    }
}
```

---

If you want, I can also add a **preview section, screenshots section, and project structure** to make your README look even more professional for GitHub ⭐
