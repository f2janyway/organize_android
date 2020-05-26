2020-05-26
==
google
```
해당 hyundaioil.keystore 있는 directory에서 release key
keytool -list -v -keystore hyundaioil.keystore -alias hyundaioilbank
```

kakao
```
-kakao (.android/oil 에서)
keytool -exportcert -alias androiddebugkey -keystore debug.keystore -storepass android -keypass android | openssl sha1 -binary | openssl base64


keytool -exportcert -alias hyundaioilbank -keystore hyundaioil.keystore | openssl sha1 -binary | openssl base64
Enter keystore password: 
```