# [UNIT] Running the Jenkins agent as a Windows service is the best way to avoid manually starting it after every reboot

Owner: Nam Tran
Last edited time: November 18, 2025 11:02 AM

## 🛠 Using **NSSM (Non-Sucking Service Manager)**

1. **Download NSSM**
    - Get it from [nssm.cc](https://nssm.cc/download).
    - Extract and place `nssm.exe` somewhere accessible (e.g., `C:\\nssm\\nssm.exe`).
2. **Install Jenkins agent as a service**
    
    Run:
    
    ```bash
    nssm install JenkinsAgent
    
    ```
    
    - In the GUI:
        - **Path**: `C:\Program Files\Java\jdk-17\bin\java.exe`
        - **Arguments**:
            
            ```
            java -jar C:\Users\Administrator\Downloads\agent.jar -url https://jenkins.vvt1.ops.unit.local/ -secret 27720b4b85729d5088170638ddb4966c3f25126da5dc1e69e1d98be734ca95a3 -name "jenkins-agent-04-win" -webSocket -workDir "D:\jenkins"
            ```
            
        - **Startup directory**: `D:\jenkins`
3. **Start the service**
    
    ```bash
    nssm start JenkinsAgent
    
    ```