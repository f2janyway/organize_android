2020-05-11
===
기본적인 바이오 구현
<br>

[Android-doc](https://developer.android.com/training/sign-in/biometric-auth)

```
    implementation "androidx.biometric:biometric:1.0.1"

```

```
/**
    * bio auth anroid 9 -> no face or iris in BiometricPrompt yet
    */
private fun bioAuth(){
    val biometricManager = BiometricManager.from(this)
    when (biometricManager.canAuthenticate()) {
        BiometricManager.BIOMETRIC_SUCCESS ->{
            Log.d("MY_APP_TAG", "App can authenticate using biometrics.")
            val biometricPrompt = instanceOfBiometricPrompt()
            val promptInfo = getPromptInfo()

            val canAuthenticate = biometricManager.canAuthenticate()
            if (canAuthenticate == BiometricManager.BIOMETRIC_SUCCESS) {
                biometricPrompt.authenticate(promptInfo)
            } else {
                Log.e(TAG, "could not authenticate because: $canAuthenticate")
            }
        }
        BiometricManager.BIOMETRIC_ERROR_NO_HARDWARE ->
            Log.e(TAG, "No biometric features available on this device.")
        BiometricManager.BIOMETRIC_ERROR_HW_UNAVAILABLE ->
            Log.e(TAG, "Biometric features are currently unavailable.")
        BiometricManager.BIOMETRIC_ERROR_NONE_ENROLLED ->
            Log.e(TAG, "The user hasn't associated " +
                    "any biometric credentials with their account.")
    }
}

private fun instanceOfBiometricPrompt(): BiometricPrompt {
    val executor = ContextCompat.getMainExecutor(this)

    val callback = object: BiometricPrompt.AuthenticationCallback() {
        override fun onAuthenticationError(errorCode: Int, errString: CharSequence) {
            super.onAuthenticationError(errorCode, errString)
            Log.e(TAG, "$errorCode :: $errString")
        }

        override fun onAuthenticationFailed() {
            super.onAuthenticationFailed()
            Log.e(TAG, "Authentication failed for an unknown reason");
        }

        override fun onAuthenticationSucceeded(result: BiometricPrompt.AuthenticationResult) {
            super.onAuthenticationSucceeded(result)
            Log.e(TAG, "Authentication was successful");
        }
    }

    return BiometricPrompt(this@MainActivity, executor, callback)
}
private fun getPromptInfo(): BiometricPrompt.PromptInfo {
    return BiometricPrompt.PromptInfo.Builder()
        .setTitle(getString(R.string.bio))
        .setSubtitle("Please login to get access")
        .setDescription("My App is using Android biometric authentication")
        .setDeviceCredentialAllowed(true)
//            .setNegativeButtonText("Use account password")
//            .setConfirmationRequired(false)
//            .setConfirmationRequired(false)
        .build()
}
```