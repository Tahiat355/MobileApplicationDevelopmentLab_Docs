# Weather App - Android Studio Implementation Guide

This app uses Jetpack Compose for the UI and Retrofit for networking, fetching data from WeatherAPI.com (based on the endpoint `/v1/current.json`).

## Step 1: Project Setup & Dependencies

1. Open Android Studio and create a **New Project**.
2. Select **Empty Activity** (make sure it's the one with the Jetpack Compose logo).
3. Name it **"WeatherApp"** and ensure the package name is `com.example.weatherapp`.
4. Open your `build.gradle.kts (Module :app)` file and add the following dependencies for Retrofit, Gson, LiveData, and Coil (for image loading).

```kotlin
dependencies {
    // ... existing core dependencies

    // Retrofit for Networking
    implementation("com.squareup.retrofit2:retrofit:2.9.0")
    implementation("com.squareup.retrofit2:converter-gson:2.9.0")

    // Coil for Image Loading (Using Coil 3 as per your import)
    implementation("io.coil-kt.coil3:coil-compose:3.0.0-rc01")
    implementation("io.coil-kt.coil3:coil-network-okhttp:3.0.0-rc01")

    // LiveData for Compose
    implementation("androidx.compose.runtime:runtime-livedata:1.6.8")
    
    // Icons
    implementation("androidx.compose.material:material-icons-extended:1.6.8")
}
```

**Sync your project** after adding these.

---

## Step 2: Internet Permission

Open `manifests/AndroidManifest.xml` and add the internet permission above the `<application>` tag:

```xml
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools">

    <uses-permission android:name="android.permission.INTERNET"/>

    <application ... >
```

---

## Step 3: Create the Data Layer (API Package)

Create a new package named **`api`** inside `com.example.weatherapp`.

### 3a. Create Data Models (`WeatherModel.kt`)

Since the data model code wasn't explicitly provided but used in WeatherPage, we need to create the data classes to match the API response structure. Create a new kotlin class file `api/WeatherModel.kt`:

```kotlin
package com.example.weatherapp.api

data class WeatherModel(
    val current: Current,
    val location: Location
)

data class Current(
    val temp_c: String,
    val condition: Condition,
    val humidity: String,
    val wind_kph: String,
    val uv: String,
    val precip_in: String
)

data class Condition(
    val text: String,
    val icon: String
)

data class Location(
    val name: String,
    val country: String
)
```

### 3b. Create Constants (`Constant.kt`)

Create `api/Constant.kt`. You will need to sign up at [weatherapi.com](https://www.weatherapi.com/) to get a free API key.

```kotlin
package com.example.weatherapp.api

object Constant {
    // Replace with your actual API Key from WeatherAPI.com
    const val apiKey = "YOUR_API_KEY_HERE" 
}
```

### 3c. Create Network Response (`NetworkResponse.kt`)

Create `api/NetworkResponse.kt`:

```kotlin
package com.example.weatherapp.api

sealed class NetworkResponse<out T> {
    data class Success<out T>(val data: T) : NetworkResponse<T>()
    data class Error(val message: String) : NetworkResponse<Nothing>()
    object Loading : NetworkResponse<Nothing>()
}
```

### 3d. Create API Interface (`WeatherApi.kt`)

Create `api/WeatherApi.kt`:

```kotlin
package com.example.weatherapp.api

import retrofit2.Response
import retrofit2.http.GET
import retrofit2.http.Query

interface WeatherApi {
    @GET("/v1/current.json")
    suspend fun getWeather(
        @Query("key") apikey: String,
        @Query("q") city: String
    ): Response<WeatherModel>
}
```

### 3e. Create Retrofit Instance (`RetrofitInstance.kt`)

Create `api/RetrofitInstance.kt`:

```kotlin
package com.example.weatherapp.api

import retrofit2.Retrofit
import retrofit2.converter.gson.GsonConverterFactory

object RetrofitInstance {

    private const val BASE_URL = "https://api.weatherapi.com"

    private val retrofit: Retrofit by lazy {
        Retrofit.Builder()
            .baseUrl(BASE_URL)
            .addConverterFactory(GsonConverterFactory.create())
            .build()
    }

    val weatherApi: WeatherApi by lazy {
        retrofit.create(WeatherApi::class.java)
    }
}
```

---

## Step 4: Create the ViewModel

Create the file `WeatherViewModel.kt` in the main package. Ensure imports are correct.

```kotlin
package com.example.weatherapp

import androidx.lifecycle.MutableLiveData
import androidx.lifecycle.ViewModel
import androidx.lifecycle.viewModelScope
import com.example.weatherapp.api.Constant
import com.example.weatherapp.api.NetworkResponse
import com.example.weatherapp.api.RetrofitInstance
import com.example.weatherapp.api.WeatherModel
import kotlinx.coroutines.launch

class WeatherViewModel : ViewModel() {

    private val weatherApi = RetrofitInstance.weatherApi
    private val _weatherResult = MutableLiveData<NetworkResponse<WeatherModel>>()
    val weatherResult: MutableLiveData<NetworkResponse<WeatherModel>> = _weatherResult

    fun getData(city: String) {
        _weatherResult.value = NetworkResponse.Loading
        viewModelScope.launch {
            try {
                val response = weatherApi.getWeather(Constant.apiKey, city)
                if (response.isSuccessful) {
                    response.body()?.let {
                        _weatherResult.value = NetworkResponse.Success(it)
                    }
                } else {
                    _weatherResult.value = NetworkResponse.Error("Failed to load data")
                }
            } catch (e: Exception) {
                _weatherResult.value = NetworkResponse.Error("Failed to load data")
            }
        }
    }
}
```

---

## Step 5: Create the UI (`WeatherPage.kt`)

Create `WeatherPage.kt`.

> **Note:** In WeatherModel, I defined fields like humidity as String to match your WeatherKeyVal function which expects `value: String`. If the API returns numbers, Retrofit/Gson is usually smart enough to convert them to Strings automatically.

```kotlin
package com.example.weatherapp

import androidx.compose.foundation.layout.*
import androidx.compose.material.icons.Icons
import androidx.compose.material.icons.filled.LocationOn
import androidx.compose.material.icons.filled.Search
import androidx.compose.material3.*
import androidx.compose.runtime.*
import androidx.compose.runtime.livedata.observeAsState
import androidx.compose.ui.Alignment
import androidx.compose.ui.Modifier
import androidx.compose.ui.graphics.Color
import androidx.compose.ui.text.font.FontWeight
import androidx.compose.ui.text.style.TextAlign
import androidx.compose.ui.unit.dp
import androidx.compose.ui.unit.sp
import coil3.compose.AsyncImage
import com.example.weatherapp.api.NetworkResponse
import com.example.weatherapp.api.WeatherModel

@Composable
fun WeatherPage(viewModel: WeatherViewModel) {
    var city by remember { mutableStateOf("") }
    val weatherResult = viewModel.weatherResult.observeAsState()

    Column(
        modifier = Modifier.fillMaxSize().padding(8.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Row(
            modifier = Modifier.fillMaxWidth().padding(8.dp),
            verticalAlignment = Alignment.CenterVertically,
            horizontalArrangement = Arrangement.SpaceEvenly
        ) {
            OutlinedTextField(
                value = city,
                onValueChange = { city = it },
                label = { Text(text = "Search for any location") }
            )
            IconButton(onClick = { viewModel.getData(city) }) {
                Icon(
                    imageVector = Icons.Default.Search,
                    contentDescription = "Search for any location"
                )
            }
        }

        when (val result = weatherResult.value) {
            is NetworkResponse.Error -> {
                Text(text = result.message)
            }
            NetworkResponse.Loading -> {
                CircularProgressIndicator()
            }
            is NetworkResponse.Success -> {
                WeatherDetails(data = result.data)
            }
            null -> {}
        }
    }
}

@Composable
fun WeatherDetails(data: WeatherModel) {
    Column(
        modifier = Modifier.fillMaxWidth().padding(vertical = 8.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Row(
            modifier = Modifier.fillMaxWidth(),
            horizontalArrangement = Arrangement.Start
        ) {
            Icon(
                imageVector = Icons.Default.LocationOn,
                contentDescription = "Location icon",
                modifier = Modifier.size(40.dp)
            )
            Text(text = data.location.name, fontSize = 30.sp)
            Spacer(modifier = Modifier.width(8.dp))
            Text(text = data.location.country, fontSize = 18.sp, color = Color.Gray)
        }

        Spacer(modifier = Modifier.height(8.dp))
        Text(
            text = "${data.current.temp_c} °C",
            fontSize = 56.sp,
            fontWeight = FontWeight.Bold,
            textAlign = TextAlign.Center
        )

        AsyncImage(
            modifier = Modifier.size(160.dp),
            model = "https:${data.current.condition.icon}".replace("64x64", "128x128"),
            contentDescription = "Condition icon"
        )

        Text(
            text = data.current.condition.text,
            fontSize = 20.sp,
            textAlign = TextAlign.Center,
            color = Color.Gray
        )
        Spacer(modifier = Modifier.height(16.dp))
        Card {
            Column(modifier = Modifier.fillMaxWidth()) {
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceAround
                ) {
                    WeatherKeyVal("Humidity", data.current.humidity)
                    WeatherKeyVal("Wind Speed", data.current.wind_kph)
                }
                Row(
                    modifier = Modifier.fillMaxWidth(),
                    horizontalArrangement = Arrangement.SpaceAround
                ) {
                    WeatherKeyVal("UV", data.current.uv)
                    WeatherKeyVal("Precipitation", data.current.precip_in)
                }
            }
        }
    }
}

@Composable
fun WeatherKeyVal(key: String, value: String) {
    Column(
        modifier = Modifier.padding(16.dp),
        horizontalAlignment = Alignment.CenterHorizontally
    ) {
        Text(text = value, fontSize = 24.sp, fontWeight = FontWeight.Bold)
        Text(text = key, fontWeight = FontWeight.SemiBold, color = Color.Gray)
    }
}
```

---

## Step 6: Main Activity

Finally, update your `MainActivity.kt` to initialize the ViewModel and display the page.

```kotlin
package com.example.weatherapp

import android.os.Bundle
import androidx.activity.ComponentActivity
import androidx.activity.compose.setContent
import androidx.activity.enableEdgeToEdge
import androidx.compose.foundation.layout.fillMaxSize
import androidx.compose.material3.MaterialTheme
import androidx.compose.material3.Surface
import androidx.compose.ui.Modifier
import androidx.lifecycle.ViewModelProvider
import com.example.weatherapp.ui.theme.WeatherAppTheme

class MainActivity : ComponentActivity() {
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        enableEdgeToEdge()
        
        val weatherViewModel = ViewModelProvider(this)[WeatherViewModel::class.java]

        setContent {
            WeatherAppTheme {
                Surface(
                    modifier = Modifier.fillMaxSize(),
                    color = MaterialTheme.colorScheme.background
                ) {
                    WeatherPage(weatherViewModel)
                }
            }
        }
    }
}
```

---

## Final Check

- **API Key:** Ensure you put a valid API key in `Constant.kt`.
- **Imports:** If any imports are red, press `Alt+Enter` to import the correct classes (e.g., `androidx.compose.material3.Card`, `androidx.compose.material3.Text`).
- **Run:** Run the app on an emulator or device with internet access. Type a city name (e.g., "London") and hit search.

---

