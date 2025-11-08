KT 1

package com.example.walksense

import android.Manifest
import android.content.Intent
import android.content.pm.PackageManager
import android.os.Bundle
import android.speech.RecognizerIntent
import android.widget.Button
import android.widget.TextView
import android.widget.Toast
import androidx.activity.result.contract.ActivityResultContracts
import androidx.appcompat.app.AppCompatActivity
import androidx.core.app.ActivityCompat
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationServices
import java.util.*

class MainActivity : AppCompatActivity() {

    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var textUbicacion: TextView
    private lateinit var textDestino: TextView
    private lateinit var botonVoz: Button
    private lateinit var botonNavegar: Button

    private var destino: String? = null

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)
        textUbicacion = findViewById(R.id.textUbicacion)
        textDestino = findViewById(R.id.textDestino)
        botonVoz = findViewById(R.id.botonVoz)
        botonNavegar = findViewById(R.id.botonNavegar)

        verificarPermisos()

        botonVoz.setOnClickListener { escucharDestino() }
        botonNavegar.setOnClickListener { iniciarNavegacion() }

        obtenerUbicacionActual()
    }

    private fun verificarPermisos() {
        if (ActivityCompat.checkSelfPermission(
                this, Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            ActivityCompat.requestPermissions(
                this, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), 100
            )
        }
    }

    private fun obtenerUbicacionActual() {
        if (ActivityCompat.checkSelfPermission(
                this, Manifest.permission.ACCESS_FINE_LOCATION
            ) != PackageManager.PERMISSION_GRANTED
        ) {
            return
        }

        fusedLocationClient.lastLocation.addOnSuccessListener { location ->
            if (location != null) {
                val texto =
                    "Ubicación actual:\nLatitud: ${location.latitude}\nLongitud: ${location.longitude}"
                textUbicacion.text = texto
            } else {
                textUbicacion.text = "No se pudo obtener la ubicación actual"
            }
        }
    }

    private val escucharVozLauncher =
        registerForActivityResult(ActivityResultContracts.StartActivityForResult()) { result ->
            if (result.resultCode == RESULT_OK && result.data != null) {
                val resultados = result.data!!.getStringArrayListExtra(RecognizerIntent.EXTRA_RESULTS)
                if (!resultados.isNullOrEmpty()) {
                    destino = resultados[0]
                    textDestino.text = "Destino: $destino"
                    Toast.makeText(this, "Destino reconocido: $destino", Toast.LENGTH_SHORT).show()
                }
            }
        }

    private fun escucharDestino() {
        val intent = Intent(RecognizerIntent.ACTION_RECOGNIZE_SPEECH)
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE_MODEL, RecognizerIntent.LANGUAGE_MODEL_FREE_FORM)
        intent.putExtra(RecognizerIntent.EXTRA_LANGUAGE, Locale("es", "MX"))
        intent.putExtra(RecognizerIntent.EXTRA_PROMPT, "Di tu destino...")
        escucharVozLauncher.launch(intent)
    }

    private fun iniciarNavegacion() {
        if (destino.isNullOrEmpty()) {
            Toast.makeText(this, "Primero di tu destino por voz", Toast.LENGTH_SHORT).show()
            return
        }

        val intent = Intent(this, NavigationService::class.java)
        intent.putExtra("destino", destino)
        startService(intent)
        Toast.makeText(this, "Iniciando navegación hacia $destino", Toast.LENGTH_LONG).show()
    }
}

XML

<?xml version="1.0" encoding="utf-8"?>
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:orientation="vertical"
    android:gravity="center_horizontal"
    android:padding="20dp"
    android:background="#F2F2F2"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TextView
        android:id="@+id/textUbicacion"
        android:text="Ubicación actual: desconocida"
        android:textSize="16sp"
        android:textColor="#000000"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="10dp" />

    <TextView
        android:id="@+id/textDestino"
        android:text="Destino no definido"
        android:textSize="16sp"
        android:textColor="#000000"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:padding="10dp" />

    <Button
        android:id="@+id/botonVoz"
        android:text="Hablar destino"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="20dp"
        android:backgroundTint="#3F51B5"
        android:textColor="#FFFFFF" />

    <Button
        android:id="@+id/botonNavegar"
        android:text="Iniciar navegación"
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:layout_marginTop="10dp"
        android:backgroundTint="#009688"
        android:textColor="#FFFFFF" />
</LinearLayout>

Navigator service

package com.example.walksense

import android.app.Service
import android.content.Intent
import android.location.Location
import android.os.IBinder
import android.os.Vibrator
import android.speech.tts.TextToSpeech
import android.widget.Toast
import com.google.android.gms.location.FusedLocationProviderClient
import com.google.android.gms.location.LocationCallback
import com.google.android.gms.location.LocationRequest
import com.google.android.gms.location.LocationResult
import com.google.android.gms.location.LocationServices
import com.google.android.gms.location.Priority
import java.util.*

class NavigationService : Service(), TextToSpeech.OnInitListener {

    private lateinit var fusedLocationClient: FusedLocationProviderClient
    private lateinit var locationCallback: LocationCallback
    private lateinit var tts: TextToSpeech
    private var destino: String? = null

    override fun onCreate() {
        super.onCreate()
        fusedLocationClient = LocationServices.getFusedLocationProviderClient(this)
        tts = TextToSpeech(this, this)
    }

    override fun onStartCommand(intent: Intent?, flags: Int, startId: Int): Int {
        destino = intent?.getStringExtra("destino")
        Toast.makeText(this, "Navegando hacia: $destino", Toast.LENGTH_LONG).show()
        hablar("Iniciando navegación hacia $destino")
        iniciarActualizacionUbicacion()
        return START_STICKY
    }

    private fun iniciarActualizacionUbicacion() {
        val request = LocationRequest.Builder(Priority.PRIORITY_HIGH_ACCURACY, 5000L)
            .setMinUpdateIntervalMillis(3000L)
            .build()

        locationCallback = object : LocationCallback() {
            override fun onLocationResult(locationResult: LocationResult) {
                val location: Location? = locationResult.lastLocation
                if (location != null) {
                    val texto = "Ubicación actual: ${location.latitude}, ${location.longitude}"
                    hablar(texto)
                    val vibrator = getSystemService(VIBRATOR_SERVICE) as Vibrator
                    vibrator.vibrate(300)
                }
            }
        }

        fusedLocationClient.requestLocationUpdates(request, locationCallback, mainLooper)
    }

    private fun hablar(texto: String) {
        tts.speak(texto, TextToSpeech.QUEUE_FLUSH, null, null)
    }

    override fun onInit(status: Int) {
        if (status == TextToSpeech.SUCCESS) {
            tts.language = Locale("es", "MX")
        }
    }

    override fun onDestroy() {
        fusedLocationClient.removeLocationUpdates(locationCallback)
        tts.stop()
        tts.shutdown()
        super.onDestroy()
    }

    override fun onBind(intent: Intent?): IBinder? = null
}

