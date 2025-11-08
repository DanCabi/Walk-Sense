# Walk-Sense

package com.example.walksensee

import android.os.Bundle
import android.widget.Button
import android.widget.Toast
import androidx.appcompat.app.AppCompatActivity
import com.google.android.gms.common.SignInButton

class MainActivity : AppCompatActivity() {

    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)

 
        val googleSignInButton: SignInButton = findViewById(R.id.google_sign_in_button)
        val regularLoginButton: Button = findViewById(R.id.regular_login_button)
        val settingsIcon = findViewById<android.widget.ImageView>(R.id.settings_icon)

      
        googleSignInButton.setOnClickListener {
            Toast.makeText(this, "Intentando Continuar con Google...", Toast.LENGTH_SHORT).show()
        }

      
        regularLoginButton.setOnClickListener {
  
            Toast.makeText(this, "Redirigiendo a Iniciar Sesión...", Toast.LENGTH_SHORT).show()
        }
        
      
        settingsIcon.setOnClickListener {
            Toast.makeText(this, "Abriendo Ajustes Generales...", Toast.LENGTH_SHORT).show()
        }
    }
}





<?xml version="1.0" encoding="utf-8"?>
<androidx.constraintlayout.widget.ConstraintLayout
    xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:app="http://schemas.android.com/apk/res-auto"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context=".MainActivity">

    <ImageView
        android:id="@+id/settings_icon"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="16dp"
        android:layout_marginEnd="16dp"
        android:src="@drawable/ic_settings" 
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintTop_toTopOf="parent"
        android:contentDescription="@string/ajustes_generales" />

    <TextView
        android:id="@+id/app_title"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="WALK\nSENSEE"
        android:textSize="50sp"
        android:textStyle="bold"
        android:gravity="center"
        android:layout_marginTop="100dp"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toTopOf="parent" />

    <com.google.android.gms.common.SignInButton
        android:id="@+id/google_sign_in_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:layout_marginTop="100dp"
        app:layout_constraintBottom_toTopOf="@+id/regular_login_button"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/app_title"
        app:layout_constraintVertical_chainStyle="packed"
        tools:text="Continuar con Google" />

    <Button
        android:id="@+id/regular_login_button"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content"
        android:text="Iniciar Sesión"
        android:layout_marginTop="16dp"
        android:minWidth="200dp"
        app:layout_constraintBottom_toBottomOf="parent"
        app:layout_constraintEnd_toEndOf="parent"
        app:layout_constraintStart_toStartOf="parent"
        app:layout_constraintTop_toBottomOf="@+id/google_sign_in_button" />

</androidx.constraintlayout.widget.ConstraintLayout>
