# Client certificate for mutual SSL 
In order to be authenticated to our REST services you have to provide a self-signed client certificate, which can be generated either using makecert.exe or OpenSSL. **Note, that the certificate is valid for 2 years and will have to be regenerated after it expires.**

Generate two certificates for Sandbox and Production environments:

 - Sandbox: set {environment} to Sandbox
 - Production: leave {environment} blank.
 
 Send the generated *.cer (or *.crt, if you use OpenSSL) files to help@mobilepay.dk and store the *.pfx file in a secure private key storage on your end. Note: Please zip the certificate, as our e-mail server is quite sensitive.
 
 ### Using makecert.exe to generate client certificate
 
     makecert.exe ^
     -n "CN={your company name} - MobilePay Online - {environment}" ^
     -sky exchange ^
     -eku 1.3.6.1.5.5.7.3.2 ^
     -r ^
     -pe ^
     -a sha512 ^
     -len 2048 ^
     -m 24 ^
     -sv {environment}MobilePayOnline{your company name}.pvk ^
     {environment}MobilePayOnline{your company name}.cer
     
Export private key to pfx:

    pvk2pfx.exe ^ 
     -pvk {environment}MobilePayOnline{your company name}.pvk ^
     -spc {environment}MobilePayOnline{your company name}.cer ^
     -pfx {environment}MobilePayOnline{your company name}.pfx
     
### Using OpenSSL to generate client certificate

    $ openssl req -x509 -nodes -sha512 -newkey rsa:2048 -keyout {environment}MobilePayOnline{your company name}.pvk -out {environment}MobilePayOnline{your company name}.crt -days 730

Enter `{your company name} - MobilePay Online - {environment}` for Common Name when asked.

Export private key to pfx:

    $ openssl pkcs12 -export -in {environment}MobilePayOnline{your company name}.crt -inkey {environment}MobilePayOnline{your company name}.pvk -CSP "Microsoft Enhanced RSA and AES Cryptographic Provider" -out {environment}MobilePayOnline{your company name}.pfx
