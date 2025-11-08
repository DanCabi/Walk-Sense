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
