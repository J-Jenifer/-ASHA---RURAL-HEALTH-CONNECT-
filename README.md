# -ASHA---RURAL-HEALTH-CONNECT-
AI-Powered Offline-First EHR for India's ASHA Workers
# ASHA RURAL HEALTH CONNECT: Offline-First EHR for India's Frontline Health Workers

## 📱 About The Project

ASHA Saathi is a mobile app designed for India's 1 million ASHA workers who serve 845 million rural citizens. The app works completely offline and requires no typing—just tap icons and speak.

### The Problem
- ASHA workers spend **3 hours daily** on paper registers
- **65% villages** have no internet
- **40% workers** never used smartphones
- **22 languages** spoken across India
- **35,000 mothers die** annually from preventable causes

### Our Solution
- ✅ **100% offline** - Works in any village
- ✅ **No typing needed** - Just tap icons and speak
- ✅ **12 Indian languages** with voice guidance
- ✅ **Pregnancy tracking** (months 1-9) with danger alerts
- ✅ **Child vaccination** tracker for 8 essential vaccines
- ✅ **AI-powered risk detection** for high-risk pregnancies
- ✅ **Works on ₹4,000 phones** with 1GB RAM

## 🛠️ Built With
- Kotlin + Jetpack Compose
- Room Database (offline storage)
- TensorFlow Lite (on-device AI)
- ML Kit (voice recognition)
- WorkManager (background sync)


## code
plugins {
    id 'com.android.application'
    id 'org.jetbrains.kotlin.android'
    id 'kotlin-kapt'
    id 'kotlin-parcelize'
}

android {
    namespace 'com.asha.saathi'
    compileSdk 34

    defaultConfig {
        applicationId "com.asha.saathi"
        minSdk 24
        versionCode 1
        versionName "1.0"
    }

    buildFeatures {
        viewBinding true
    }

    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
    kotlinOptions {
        jvmTarget = '1.8'
    }
}

dependencies {
    implementation 'androidx.core:core-ktx:1.12.0'
    implementation 'androidx.appcompat:appcompat:1.6.1'
    implementation 'com.google.android.material:material:1.11.0'
    implementation 'androidx.constraintlayout:constraintlayout:2.1.4'
    
    // Room Database
    implementation 'androidx.room:room-runtime:2.6.1'
    implementation 'androidx.room:room-ktx:2.6.1'
    kapt 'androidx.room:room-compiler:2.6.1'
    
    // Lifecycle
    implementation 'androidx.lifecycle:lifecycle-viewmodel-ktx:2.7.0'
    implementation 'androidx.lifecycle:lifecycle-livedata-ktx:2.7.0'
    
    // Navigation
    implementation 'androidx.navigation:navigation-fragment-ktx:2.7.7'
    implementation 'androidx.navigation:navigation-ui-ktx:2.7.7'
    
    // WorkManager
    implementation 'androidx.work:work-runtime-ktx:2.9.0'
    
    // Biometric
    implementation 'androidx.biometric:biometric:1.1.0'
    
    // QR Code
    implementation 'com.google.zxing:core:3.5.2'
    implementation 'com.journeyapps:zxing-android-embedded:4.3.0'
    
    // ML Kit (Voice)
    implementation 'com.google.android.gms:play-services-mlkit-speech:19.0.1'
    
    // TensorFlow Lite
    implementation 'org.tensorflow:tensorflow-lite:2.13.0'
    implementation 'org.tensorflow:tensorflow-lite-support:0.4.4'
    
    // CameraX
    implementation 'androidx.camera:camera-core:1.3.1'
    implementation 'androidx.camera:camera-camera2:1.3.1'
    implementation 'androidx.camera:camera-lifecycle:1.3.1'
    implementation 'androidx.camera:camera-view:1.3.1'
    
    // Coroutines
    implementation 'org.jetbrains.kotlinx:kotlinx-coroutines-android:1.7.3'
    
    // Gson
    implementation 'com.google.code.gson:gson:2.10.1'
}
package com.asha.saathi

import android.os.Bundle
import androidx.appcompat.app.AppCompatActivity
import androidx.navigation.findNavController
import androidx.navigation.ui.AppBarConfiguration
import androidx.navigation.ui.setupActionBarWithNavController
import androidx.navigation.ui.setupWithNavController
import com.asha.saathi.databinding.ActivityMainBinding
import com.asha.saathi.utils.LanguageManager
import com.google.android.material.bottomnavigation.BottomNavigationView

class MainActivity : AppCompatActivity() {

    private lateinit var binding: ActivityMainBinding
    private lateinit var languageManager: LanguageManager

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        
        languageManager = LanguageManager(this)
        languageManager.setAppLocale()
        
        binding = ActivityMainBinding.inflate(layoutInflater)
        setContentView(binding.root)

        setupBottomNavigation()
    }

    private fun setupBottomNavigation() {
        val navView: BottomNavigationView = binding.navView
        val navController = findNavController(R.id.nav_host_fragment_activity_main)

        val appBarConfiguration = AppBarConfiguration(
            setOf(
                R.id.navigation_home,
                R.id.navigation_mothers,
                R.id.navigation_children,
                R.id.navigation_alerts,
                R.id.navigation_household
            )
        )
        setupActionBarWithNavController(navController, appBarConfiguration)
        navView.setupWithNavController(navController)
    }
}
package com.asha.saathi.data

import androidx.room.Entity
import androidx.room.PrimaryKey

@Entity(tableName = "patients")
data class Patient(
    @PrimaryKey
    val patientId: String,
    val name: String,
    val age: Int,
    val gender: String,
    val village: String,
    val isPregnant: Boolean = false,
    val phone: String? = null,
    val createdAt: Long = System.currentTimeMillis(),
    val syncStatus: Int = 0
)

@Entity(tableName = "pregnancies")
data class Pregnancy(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val patientId: String,
    val month: Int,
    val bpSystolic: Int?,
    val bpDiastolic: Int?,
    val weight: Float?,
    val hemoglobin: Float?,
    val dangerSigns: String?,
    val recordedAt: Long = System.currentTimeMillis()
)

@Entity(tableName = "children")
data class Child(
    @PrimaryKey
    val patientId: String,
    val name: String,
    val ageMonths: Int,
    val motherId: String
)

@Entity(tableName = "vaccinations")
data class Vaccination(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val childId: String,
    val vaccineName: String,
    val doseNumber: Int,
    val dueDate: Long,
    val isCompleted: Boolean = false
)

@Entity(tableName = "alerts")
data class Alert(
    @PrimaryKey(autoGenerate = true)
    val id: Long = 0,
    val patientId: String,
    val patientName: String,
    val message: String,
    val severity: String,
    val isActive: Boolean = true,
    val createdAt: Long = System.currentTimeMillis()
)
package com.asha.saathi.utils

import java.text.SimpleDateFormat
import java.util.*

object UniqueIdGenerator {
    private var counter = 0
    private val dateFormat = SimpleDateFormat("yyMMdd", Locale.getDefault())

    fun generatePatientId(): String {
        counter++
        val date = dateFormat.format(Date())
        return "MH24-${date}-${String.format("%05d", counter)}"
    }
}
package com.asha.saathi.utils

import android.content.Context
import android.content.res.Configuration
import java.util.*

class LanguageManager(private val context: Context) {
    
    private val prefs = context.getSharedPreferences("settings", Context.MODE_PRIVATE)
    
    fun setLanguage(languageCode: String) {
        prefs.edit().putString("language", languageCode).apply()
        setAppLocale()
    }
    
    fun getCurrentLanguage(): String {
        return prefs.getString("language", "en") ?: "en"
    }
    
    fun setAppLocale() {
        val locale = Locale(getCurrentLanguage())
        Locale.setDefault(locale)
        val config = Configuration(context.resources.configuration)
        config.setLocale(locale)
        context.resources.updateConfiguration(config, context.resources.displayMetrics)
    }
}
package com.asha.saathi.utils

import android.content.Context
import android.speech.tts.TextToSpeech
import java.util.*

class VoiceHelper(context: Context) : TextToSpeech.OnInitListener {
    
    private var tts: TextToSpeech? = null
    private var isReady = false
    
    init {
        tts = TextToSpeech(context, this)
    }
    
    override fun onInit(status: Int) {
        if (status == TextToSpeech.SUCCESS) {
            isReady = true
            tts?.language = Locale.getDefault()
        }
    }
    
    fun speak(text: String) {
        if (isReady) {
            tts?.speak(text, TextToSpeech.QUEUE_FLUSH, null, null)
        }
    }
    
    fun shutdown() {
        tts?.stop()
        tts?.shutdown()
    }
}
package com.asha.saathi.ui.home

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.fragment.app.Fragment
import androidx.lifecycle.lifecycleScope
import com.asha.saathi.R
import com.asha.saathi.databinding.FragmentHomeBinding
import com.asha.saathi.utils.VoiceHelper
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

class HomeFragment : Fragment() {

    private var _binding: FragmentHomeBinding? = null
    private val binding get() = _binding!!
    private lateinit var voiceHelper: VoiceHelper

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentHomeBinding.inflate(inflater, container, false)
        voiceHelper = VoiceHelper(requireContext())
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)
        
        // Set dummy data for demo
        binding.tvMothersCount.text = "24"
        binding.tvChildrenCount.text = "187"
        binding.tvHouseholdsCount.text = "428"
        binding.tvTasksCount.text = "12"
        binding.tvHighRisk.text = "2 HIGH RISK"
        binding.tvVaccineDue.text = "5 DUE"
        
        binding.btnVoice.setOnClickListener {
            voiceHelper.speak("Home screen. 24 mothers, 187 children, 2 high risk alerts")
        }
        
        binding.cardMothers.setOnClickListener {
            // Navigate to mothers fragment
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        voiceHelper.shutdown()
        _binding = null
    }
}
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="#FFFFFF"
    android:padding="16dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical">

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="🌤️ Today"
            android:textSize="28sp"
            android:textStyle="bold"
            android:layout_marginBottom="24dp"/>

        <GridLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:columnCount="2"
            android:rowCount="2"
            android:alignmentMode="alignMargins"
            android:columnOrderPreserved="false"
            android:padding="4dp">

            <!-- Mothers Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/cardMothers"
                android:layout_width="0dp"
                android:layout_height="180dp"
                android:layout_columnWeight="1"
                android:layout_rowWeight="1"
                android:layout_margin="8dp"
                app:cardCornerRadius="16dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical"
                    android:gravity="center"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="🤰"
                        android:textSize="48sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="MOTHERS"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginTop="8dp" />

                    <TextView
                        android:id="@+id/tvMothersCount"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="0"
                        android:textSize="32sp"
                        android:textStyle="bold"
                        android:textColor="#4CAF50"
                        android:layout_marginTop="4dp" />

                    <TextView
                        android:id="@+id/tvHighRisk"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="0 HIGH RISK"
                        android:textSize="14sp"
                        android:textColor="#F44336" />
                </LinearLayout>
            </androidx.cardview.widget.CardView>

            <!-- Children Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/cardChildren"
                android:layout_width="0dp"
                android:layout_height="180dp"
                android:layout_columnWeight="1"
                android:layout_rowWeight="1"
                android:layout_margin="8dp"
                app:cardCornerRadius="16dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical"
                    android:gravity="center"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="👶"
                        android:textSize="48sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="CHILDREN"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginTop="8dp" />

                    <TextView
                        android:id="@+id/tvChildrenCount"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="0"
                        android:textSize="32sp"
                        android:textStyle="bold"
                        android:textColor="#2196F3"
                        android:layout_marginTop="4dp" />

                    <TextView
                        android:id="@+id/tvVaccineDue"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="0 DUE"
                        android:textSize="14sp"
                        android:textColor="#FFC107" />
                </LinearLayout>
            </androidx.cardview.widget.CardView>

            <!-- Households Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/cardHouseholds"
                android:layout_width="0dp"
                android:layout_height="180dp"
                android:layout_columnWeight="1"
                android:layout_rowWeight="1"
                android:layout_margin="8dp"
                app:cardCornerRadius="16dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical"
                    android:gravity="center"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="🏠"
                        android:textSize="48sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="HOUSEHOLDS"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginTop="8dp" />

                    <TextView
                        android:id="@+id/tvHouseholdsCount"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="0"
                        android:textSize="32sp"
                        android:textStyle="bold"
                        android:textColor="#9C27B0"
                        android:layout_marginTop="4dp" />
                </LinearLayout>
            </androidx.cardview.widget.CardView>

            <!-- Tasks Card -->
            <androidx.cardview.widget.CardView
                android:id="@+id/cardTasks"
                android:layout_width="0dp"
                android:layout_height="180dp"
                android:layout_columnWeight="1"
                android:layout_rowWeight="1"
                android:layout_margin="8dp"
                app:cardCornerRadius="16dp"
                app:cardElevation="4dp">

                <LinearLayout
                    android:layout_width="match_parent"
                    android:layout_height="match_parent"
                    android:orientation="vertical"
                    android:gravity="center"
                    android:padding="16dp">

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="⚕️"
                        android:textSize="48sp" />

                    <TextView
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="TODAY'S TASKS"
                        android:textSize="18sp"
                        android:textStyle="bold"
                        android:layout_marginTop="8dp" />

                    <TextView
                        android:id="@+id/tvTasksCount"
                        android:layout_width="wrap_content"
                        android:layout_height="wrap_content"
                        android:text="0"
                        android:textSize="32sp"
                        android:textStyle="bold"
                        android:textColor="#FF9800"
                        android:layout_marginTop="4dp" />
                </LinearLayout>
            </androidx.cardview.widget.CardView>
        </GridLayout>

        <TextView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:text="⚠️ URGENT ALERTS"
            android:textSize="20sp"
            android:textStyle="bold"
            android:textColor="#F44336"
            android:layout_marginTop="24dp"
            android:layout_marginBottom="8dp"/>

        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="8dp"
            app:cardCornerRadius="12dp"
            app:cardElevation="2dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp"
                android:background="#FFEBEE">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🔴 Savita - High BP (150/100)"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#F44336"/>

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Month 7 • Immediate referral needed"
                    android:textSize="14sp"
                    android:layout_marginTop="4dp"/>
            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <androidx.cardview.widget.CardView
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginBottom="16dp"
            app:cardCornerRadius="12dp"
            app:cardElevation="2dp">

            <LinearLayout
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:orientation="vertical"
                android:padding="16dp"
                android:background="#FFF3E0">

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="🟡 Arjun - Vaccine Due (MR-1)"
                    android:textSize="16sp"
                    android:textStyle="bold"
                    android:textColor="#FF9800"/>

                <TextView
                    android:layout_width="match_parent"
                    android:layout_height="wrap_content"
                    android:text="Due date: 15 Mar 2024"
                    android:textSize="14sp"
                    android:layout_marginTop="4dp"/>
            </LinearLayout>
        </androidx.cardview.widget.CardView>

        <Button
            android:id="@+id/btnVoice"
            android:layout_width="match_parent"
            android:layout_height="60dp"
            android:text="🎤 Voice Command"
            android:textSize="18sp"
            android:backgroundTint="#6200EE"
            android:layout_marginTop="8dp"
            android:layout_marginBottom="24dp"/>

    </LinearLayout>
</ScrollView>
<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout 
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/container"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <fragment
        android:id="@+id/nav_host_fragment_activity_main"
        android:name="androidx.navigation.fragment.NavHostFragment"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        app:layout_constraintBottom_toTopOf="@+id/nav_view"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        app:defaultNavHost="true"
        app:navGraph="@navigation/mobile_navigation" />

    <com.google.android.material.bottomnavigation.BottomNavigationView
        android:id="@+id/nav_view"
        android:layout_width="0dp"
        android:layout_height="wrap_content"
        android:background="?android:attr/windowBackground"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintLeft_toLeftOf="parent"
        app:layout_constraintRight_toRightOf="parent"
        app:menu="@menu/bottom_nav_menu" />

</androidx.constraintlayout.widget.ConstraintLayout>
<?xml version="1.0" encoding="utf-8"?>
<menu xmlns:android="http://schemas.android.com/apk/res/android">
    <item
        android:id="@+id/navigation_home"
        android:icon="@drawable/ic_home"
        android:title="Home" />
    <item
        android:id="@+id/navigation_mothers"
        android:icon="@drawable/ic_mother"
        android:title="Mothers" />
    <item
        android:id="@+id/navigation_children"
        android:icon="@drawable/ic_child"
        android:title="Children" />
    <item
        android:id="@+id/navigation_alerts"
        android:icon="@drawable/ic_alert"
        android:title="Alerts" />
    <item
        android:id="@+id/navigation_household"
        android:icon="@drawable/ic_household"
        android:title="Household" />
</menu>
      <?xml version="1.0" encoding="utf-8"?>
<resources>
    <color name="purple_500">#6200EE</color>
    <color name="teal_200">#03DAC5</color>
    
    <!-- Alert Colors -->
    <color name="red_alert">#F44336</color>
    <color name="yellow_alert">#FFC107</color>
    <color name="green_normal">#4CAF50</color>
    
    <!-- Status Colors -->
    <color name="status_green">#4CAF50</color>
    <color name="status_yellow">#FFC107</color>
    <color name="status_red">#F44336</color>
</resources>
<resources>
    <string name="app_name">ASHA Saathi</string>
    <string name="title_home">Home</string>
    <string name="title_mothers">Mothers</string>
    <string name="title_children">Children</string>
    <string name="title_alerts">Alerts</string>
    <string name="title_household">Household</string>
</resources>
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/mobile_navigation"
    app:startDestination="@+id/navigation_home">

    <fragment
        android:id="@+id/navigation_home"
        android:name="com.asha.saathi.ui.home.HomeFragment"
        android:label="@string/title_home" />

    <fragment
        android:id="@+id/navigation_mothers"
        android:name="com.asha.saathi.ui.mothers.MothersFragment"
        android:label="@string/title_mothers" />

    <fragment
        android:id="@+id/navigation_children"
        android:name="com.asha.saathi.ui.children.ChildrenFragment"
        android:label="@string/title_children" />

    <fragment
        android:id="@+id/navigation_alerts"
        android:name="com.asha.saathi.ui.alerts.AlertsFragment"
        android:label="@string/title_alerts" />

    <fragment
        android:id="@+id/navigation_household"
        android:name="com.asha.saathi.ui.household.HouseholdFragment"
        android:label="@string/title_household" />
</navigation>
package com.asha.saathi.ui.mothers

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.cardview.widget.CardView
import androidx.fragment.app.Fragment
import androidx.lifecycle.lifecycleScope
import androidx.navigation.findNavController
import androidx.recyclerview.widget.LinearLayoutManager
import androidx.recyclerview.widget.RecyclerView
import com.asha.saathi.R
import com.asha.saathi.databinding.FragmentMothersBinding
import com.asha.saathi.utils.VoiceHelper
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

class MothersFragment : Fragment() {

    private var _binding: FragmentMothersBinding? = null
    private val binding get() = _binding!!
    private lateinit var voiceHelper: VoiceHelper
    private lateinit var motherAdapter: MotherAdapter

    // Sample data - replace with Room database queries
    private val motherList = listOf(
        MotherItem(
            id = "1",
            name = "Savita Devi",
            age = 28,
            month = 7,
            riskLevel = "HIGH",
            bpStatus = "HIGH",
            hbStatus = "LOW"
        ),
        MotherItem(
            id = "2",
            name = "Laxmi Bai",
            age = 24,
            month = 4,
            riskLevel = "NORMAL",
            bpStatus = "NORMAL",
            hbStatus = "NORMAL"
        ),
        MotherItem(
            id = "3",
            name = "Rani Sharma",
            age = 32,
            month = 2,
            riskLevel = "LOW",
            bpStatus = "NORMAL",
            hbStatus = "LOW"
        ),
        MotherItem(
            id = "4",
            name = "Geeta Kumari",
            age = 26,
            month = 8,
            riskLevel = "HIGH",
            bpStatus = "CRITICAL",
            hbStatus = "NORMAL"
        ),
        MotherItem(
            id = "5",
            name = "Anita Yadav",
            age = 22,
            month = 5,
            riskLevel = "NORMAL",
            bpStatus = "NORMAL",
            hbStatus = "NORMAL"
        )
    )

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentMothersBinding.inflate(inflater, container, false)
        voiceHelper = VoiceHelper(requireContext())
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        setupRecyclerView()
        setupClickListeners()
        updateStats()
    }

    private fun setupRecyclerView() {
        motherAdapter = MotherAdapter(motherList) { mother ->
            // Navigate to mother detail
            val action = MothersFragmentDirections.actionMothersToMotherDetail(mother.id)
            view?.findNavController()?.navigate(R.id.action_mothers_to_mother_detail)
        }
        binding.recyclerMothers.layoutManager = LinearLayoutManager(requireContext())
        binding.recyclerMothers.adapter = motherAdapter
    }

    private fun setupClickListeners() {
        binding.btnVoice.setOnClickListener {
            voiceHelper.speak("Mothers screen. Total ${motherList.size} pregnant women. ${motherList.count { it.riskLevel == "HIGH" }} high risk cases.")
        }

        binding.btnAddMother.setOnClickListener {
            // Navigate to add mother screen
            voiceHelper.speak("Open add mother form")
        }

        binding.btnBack.setOnClickListener {
            view?.findNavController()?.popBackStack()
        }
    }

    private fun updateStats() {
        val totalMothers = motherList.size
        val highRisk = motherList.count { it.riskLevel == "HIGH" }
        
        binding.tvTotalMothers.text = totalMothers.toString()
        binding.tvHighRiskCount.text = "$highRisk HIGH RISK"
    }

    override fun onDestroyView() {
        super.onDestroyView()
        voiceHelper.shutdown()
        _binding = null
    }
}

data class MotherItem(
    val id: String,
    val name: String,
    val age: Int,
    val month: Int,
    val riskLevel: String,
    val bpStatus: String,
    val hbStatus: String
)

class MotherAdapter(
    private val mothers: List<MotherItem>,
    private val onItemClick: (MotherItem) -> Unit
) : RecyclerView.Adapter<MotherAdapter.MotherViewHolder>() {

    class MotherViewHolder(itemView: View) : RecyclerView.ViewHolder(itemView) {
        private val tvName: TextView = itemView.findViewById(R.id.tvMotherName)
        private val tvAge: TextView = itemView.findViewById(R.id.tvMotherAge)
        private val tvMonth: TextView = itemView.findViewById(R.id.tvMonth)
        private val tvRiskBadge: TextView = itemView.findViewById(R.id.tvRiskBadge)
        private val tvBpStatus: TextView = itemView.findViewById(R.id.tvBpStatus)
        private val tvHbStatus: TextView = itemView.findViewById(R.id.tvHbStatus)
        private val cardView: CardView = itemView.findViewById(R.id.cardMother)

        fun bind(mother: MotherItem, clickListener: (MotherItem) -> Unit) {
            tvName.text = mother.name
            tvAge.text = "${mother.age} years"
            tvMonth.text = "Month ${mother.month} of 9"
            
            // Set risk badge
            tvRiskBadge.text = mother.riskLevel
            when (mother.riskLevel) {
                "HIGH" -> {
                    tvRiskBadge.setBackgroundColor(itemView.context.getColor(android.R.color.holo_red_dark))
                    tvRiskBadge.setTextColor(itemView.context.getColor(android.R.color.white))
                }
                "LOW" -> {
                    tvRiskBadge.setBackgroundColor(itemView.context.getColor(android.R.color.holo_orange_light))
                    tvRiskBadge.setTextColor(itemView.context.getColor(android.R.color.black))
                }
                else -> {
                    tvRiskBadge.setBackgroundColor(itemView.context.getColor(android.R.color.holo_green_light))
                    tvRiskBadge.setTextColor(itemView.context.getColor(android.R.color.black))
                }
            }

            // Set BP status
            tvBpStatus.text = mother.bpStatus
            tvBpStatus.setTextColor(
                when (mother.bpStatus) {
                    "CRITICAL" -> itemView.context.getColor(android.R.color.holo_red_dark)
                    "HIGH" -> itemView.context.getColor(android.R.color.holo_orange_dark)
                    else -> itemView.context.getColor(android.R.color.holo_green_dark)
                }
            )

            // Set HB status
            tvHbStatus.text = mother.hbStatus
            tvHbStatus.setTextColor(
                when (mother.hbStatus) {
                    "LOW" -> itemView.context.getColor(android.R.color.holo_red_dark)
                    else -> itemView.context.getColor(android.R.color.holo_green_dark)
                }
            )

            cardView.setOnClickListener { clickListener(mother) }
        }
    }

    override fun onCreateViewHolder(parent: ViewGroup, viewType: Int): MotherViewHolder {
        val view = LayoutInflater.from(parent.context)
            .inflate(R.layout.item_mother, parent, false)
        return MotherViewHolder(view)
    }

    override fun onBindViewHolder(holder: MotherViewHolder, position: Int) {
        holder.bind(mothers[position], onItemClick)
    }

    override fun getItemCount() = mothers.size
}
<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:orientation="vertical"
    android:background="#FFFFFF">

    <!-- Header -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp"
        android:background="#6200EE">

        <ImageView
            android:id="@+id/btnBack"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:src="@drawable/ic_back"
            android:padding="12dp"
            android:background="?selectableItemBackgroundBorderless"
            android:tint="#FFFFFF"/>

        <TextView
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:text="🤰 Mothers"
            android:textSize="24sp"
            android:textStyle="bold"
            android:textColor="#FFFFFF"
            android:gravity="center"
            android:layout_gravity="center"/>

        <ImageView
            android:id="@+id/btnAddMother"
            android:layout_width="48dp"
            android:layout_height="48dp"
            android:src="@drawable/ic_add"
            android:padding="12dp"
            android:background="?selectableItemBackgroundBorderless"
            android:tint="#FFFFFF"/>
    </LinearLayout>

    <!-- Stats -->
    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="horizontal"
        android:padding="16dp"
        android:background="#F5F5F5">

        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center">

            <TextView
                android:id="@+id/tvTotalMothers"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="24"
                android:textSize="32sp"
                android:textStyle="bold"
                android:textColor="#6200EE"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Total Mothers"
                android:textSize="14sp"/>
        </LinearLayout>

        <LinearLayout
            android:layout_width="0dp"
            android:layout_height="wrap_content"
            android:layout_weight="1"
            android:orientation="vertical"
            android:gravity="center">

            <TextView
                android:id="@+id/tvHighRiskCount"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="2 HIGH RISK"
                android:textSize="16sp"
                android:textStyle="bold"
                android:textColor="#F44336"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Needs attention"
                android:textSize="14sp"/>
        </LinearLayout>
    </LinearLayout>

    <!-- Mothers List -->
    <androidx.recyclerview.widget.RecyclerView
        android:id="@+id/recyclerMothers"
        android:layout_width="match_parent"
        android:layout_height="0dp"
        android:layout_weight="1"
        android:padding="8dp"/>

    <!-- Voice Button -->
    <Button
        android:id="@+id/btnVoice"
        android:layout_width="match_parent"
        android:layout_height="60dp"
        android:text="🎤 Voice Command"
        android:textSize="18sp"
        android:backgroundTint="#6200EE"
        android:layout_margin="16dp"/>

</LinearLayout>
<?xml version="1.0" encoding="utf-8"?>
<androidx.cardview.widget.CardView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    android:id="@+id/cardMother"
    android:layout_width="match_parent"
    android:layout_height="wrap_content"
    android:layout_margin="8dp"
    app:cardCornerRadius="12dp"
    app:cardElevation="2dp">

    <LinearLayout
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:orientation="vertical"
        android:padding="16dp">

        <!-- Row 1: Name and Risk -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal">

            <TextView
                android:id="@+id/tvMotherName"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="Savita Devi"
                android:textSize="18sp"
                android:textStyle="bold"/>

            <TextView
                android:id="@+id/tvRiskBadge"
                android:layout_width="wrap_content"
                android:layout_height="24dp"
                android:paddingStart="12dp"
                android:paddingEnd="12dp"
                android:gravity="center"
                android:text="HIGH"
                android:textSize="12sp"
                android:textStyle="bold"
                android:textColor="#FFFFFF"
                android:background="@android:color/holo_red_dark"/>
        </LinearLayout>

        <!-- Row 2: Age and Month -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="4dp">

            <TextView
                android:id="@+id/tvMotherAge"
                android:layout_width="0dp"
                android:layout_height="wrap_content"
                android:layout_weight="1"
                android:text="28 years"
                android:textSize="14sp"
                android:textColor="#666666"/>

            <TextView
                android:id="@+id/tvMonth"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Month 7 of 9"
                android:textSize="14sp"
                android:textColor="#6200EE"
                android:textStyle="bold"/>
        </LinearLayout>

        <!-- Row 3: Vital Indicators -->
        <LinearLayout
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:orientation="horizontal"
            android:layout_marginTop="12dp">

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="BP: "
                android:textSize="14sp"/>

            <TextView
                android:id="@+id/tvBpStatus"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="HIGH"
                android:textSize="14sp"
                android:textStyle="bold"
                android:textColor="#FF9800"
                android:layout_marginEnd="16dp"/>

            <TextView
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="Hb: "
                android:textSize="14sp"/>

            <TextView
                android:id="@+id/tvHbStatus"
                android:layout_width="wrap_content"
                android:layout_height="wrap_content"
                android:text="LOW"
                android:textSize="14sp"
                android:textStyle="bold"
                android:textColor="#F44336"/>
        </LinearLayout>
    </LinearLayout>
</androidx.cardview.widget.CardView>
package com.asha.saathi.ui.mothers

import android.os.Bundle
import android.view.LayoutInflater
import android.view.View
import android.view.ViewGroup
import android.widget.TextView
import androidx.fragment.app.Fragment
import androidx.lifecycle.lifecycleScope
import androidx.navigation.findNavController
import com.asha.saathi.R
import com.asha.saathi.databinding.FragmentMotherDetailBinding
import com.asha.saathi.utils.VoiceHelper
import kotlinx.coroutines.delay
import kotlinx.coroutines.launch

class MotherDetailFragment : Fragment() {

    private var _binding: FragmentMotherDetailBinding? = null
    private val binding get() = _binding!!
    private lateinit var voiceHelper: VoiceHelper

    override fun onCreateView(
        inflater: LayoutInflater,
        container: ViewGroup?,
        savedInstanceState: Bundle?
    ): View {
        _binding = FragmentMotherDetailBinding.inflate(inflater, container, false)
        voiceHelper = VoiceHelper(requireContext())
        return binding.root
    }

    override fun onViewCreated(view: View, savedInstanceState: Bundle?) {
        super.onViewCreated(view, savedInstanceState)

        setupPatientData()
        setupMonthWheel()
        setupClickListeners()
    }

    private fun setupPatientData() {
        binding.tvPatientName.text = "Savita Devi"
        binding.tvPatientAge.text = "28 years"
        binding.tvPatientId.text = "ID: MH24-240315-00123"
        binding.tvHusbandName.text = "Husband: Ramesh"
        binding.tvMonth.text = "Month 7 of 9"

        // Vital signs
        binding.tvBpValue.text = "130/85"
        binding.tvBpStatus.text = "HIGH"
        binding.tvBpStatus.setTextColor(requireContext().getColor(android.R.color.holo_orange_dark))

        binding.tvWeightValue.text = "58.5 kg"
        binding.tvWeightStatus.text = "NORMAL"
        binding.tvWeightStatus.setTextColor(requireContext().getColor(android.R.color.holo_green_dark))

        binding.tvHbValue.text = "10.2"
        binding.tvHbStatus.text = "LOW"
        binding.tvHbStatus.setTextColor(requireContext().getColor(android.R.color.holo_red_dark))

        binding.tvIronValue.text = "15/30"
        binding.tvIronStatus.text = "ON TRACK"

        // Danger signs
        binding.tvDanger1.text = "• 🔴 Swelling in feet"
        binding.tvDanger2.text = "• 🟡 Severe headache"
        binding.tvDanger3.text = "• 🔴 Blurred vision"
    }

    private fun setupMonthWheel() {
        // Highlight month 7
        val months = listOf(
            binding.tvMonth1, binding.tvMonth2, binding.tvMonth3,
            binding.tvMonth4, binding.tvMonth5, binding.tvMonth6,
            binding.tvMonth7, binding.tvMonth8, binding.tvMonth9
        )

        for (i in 0..8) {
            if (i < 6) {
                months[i].setBackgroundResource(R.drawable.circle_completed)
                months[i].setTextColor(android.graphics.Color.WHITE)
            } else if (i == 6) {
                months[i].setBackgroundResource(R.drawable.circle_current)
                months[i].setTextColor(android.graphics.Color.WHITE)
            } else {
                months[i].setBackgroundResource(R.drawable.circle_future)
                months[i].setTextColor(android.graphics.Color.BLACK)
            }
        }
    }

    private fun setupClickListeners() {
        binding.btnBack.setOnClickListener {
            view?.findNavController()?.popBackStack()
        }

        binding.btnVoice.setOnClickListener {
            voiceHelper.speak("Savita Devi, month 7 of 9. Blood pressure 130 over 85, high. Hemoglobin 10 point 2, low. Danger signs: swelling in feet, severe headache, blurred vision.")
        }

        binding.btnAddReading.setOnClickListener {
            voiceHelper.speak("Open add reading")
        }

        binding.btnHistory.setOnClickListener {
            voiceHelper.speak("Open history")
        }
    }

    override fun onDestroyView() {
        super.onDestroyView()
        voiceHelper.shutdown()
        _binding = null
    }
}
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#4CAF50"/>
    <size android:width="40dp" android:height="40dp"/>
</shape>
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#FFC107"/>
    <size android:width="40dp" android:height="40dp"/>
</shape>
<?xml version="1.0" encoding="utf-8"?>
<shape xmlns:android="http://schemas.android.com/apk/res/android"
    android:shape="oval">
    <solid android:color="#E0E0E0"/>
    <size android:width="40dp" android:height="40dp"/>
</shape>
<?xml version="1.0" encoding="utf-8"?>
<navigation xmlns:android

## 👥 Team
- KAVITHA M Android Developer & Backend Developer
- JENIFER J - AI/ML Engineer  &  UI/UX Designer

**College:** CSI COLLEGE OF ENGINEERING KETTI

## 📄 License
MIT License
