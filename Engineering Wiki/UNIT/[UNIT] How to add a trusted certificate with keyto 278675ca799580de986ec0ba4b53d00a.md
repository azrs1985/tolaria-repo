# [UNIT] How to add a trusted certificate with keytool on Windows 🖥️

Owner: Nam Tran
Last edited time: November 18, 2025 10:14 AM

## 🔑 Steps on Window

1. **Locate your Java keystore**
    - Default keystore:
        
        `C:\Program Files\Java\jdk-<version>\lib\security\cacerts`
        
    - Or wherever your JDK/JRE is installed.
    - You can also use a custom keystore if your application defines one.
2. **Open Command Prompt as Administrator**
    - Press **Win + R**, type `cmd`, then right-click → **Run as administrator**.
    - This ensures you have permission to modify the keystore.
3. **Run the `keytool -importcert` command**
Example:
    
    ```bash
     keytool.exe -importcert -trustcacerts -alias jenkins -file "C:\Users\Administrator\Downloads\jenkins.vvt1.ops.unit.download.crt" -keystore "C:\Program Files\Java\jdk-17\lib\security\cacerts" -storepass changeit
    ```
    
    **🔑 Parts of the command**
    
    - `keytool.exe`
    
    The Java utility used to manage keystores and certificates.
    
    - `-importcert`
    
    Tells keytool you want to import a certificate into a keystore.
    
    - `-trustcacerts`
    
    Marks the certificate as trusted, treating it like a CA (Certificate Authority) certificate.
    
    - `-alias jenkins`
    
    A unique name (alias) under which the certificate will be stored in the keystore.
    
    **⁠◦**  Here, the alias is jenkins, so you can later reference this certificate by that name.
    
    - `-file "C:\Users\Administrator\Downloads\jenkins.vvt1.ops.unit.download.crt"`
    
    Path to the certificate file you want to import.
    
    **⁠◦**  In this case, it’s a .crt file downloaded for Jenkins.
    
    - `-keystore "C:\Program Files\Java\jdk-17\lib\security\cacerts"`
    
    Specifies the keystore file where the certificate will be added.
    
    **⁠◦**  cacerts is the default Java keystore that stores trusted root certificates.
    
    **⁠◦**  Located inside your JDK installation.
    
    - `-storepass changeit`
    
    The password for the keystore.
    
    **⁠◦**  By default, the Java cacerts keystore uses the password changeit (unless it has been changed).
    
4. **Verify the certificate was added**
    
    ```powershell
    keytool -list -keystore "C:\Program Files\Java\jdk-17\lib\security\cacerts" -storepass changeit | findstr jenkins
    ```
    
    - `findstr` is the Windows equivalent of `grep`.

---

## ⚠️ Notes

- Always **back up `cacerts`** before modifying it.
- If you’re using **Java installed via PATH**, you can check your active JDK with:
Then navigate to `%JAVA_HOME%\lib\security\cacerts`.
    
    ```bash
    echo %JAVA_HOME%
    ```
    

---

👉 Do you want me to show you how to **add the certificate only for one specific Java application** (using a custom keystore), or do you want it **system-wide** so all Java apps on Windows trust it?