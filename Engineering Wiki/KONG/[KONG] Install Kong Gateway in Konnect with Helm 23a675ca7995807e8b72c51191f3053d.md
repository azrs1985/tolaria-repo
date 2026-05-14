# [KONG] Install Kong Gateway in Konnect with Helm

Owner: Nam Tran
Last edited time: October 13, 2025 10:19 PM

To install Kong Gateway in Konnect with Helm, follow these steps:

1. **Set Up Konnect and Obtain a Personal Access Token (PAT):**
    - Register for a Konnect account if you don’t have one.
    - Generate a PAT from the [Konnect PAT page](https://cloud.konghq.com/global/account/tokens).
    - Export your token:
        
        ```bash
        export KONNECT_TOKEN='YOUR KONNECT TOKEN'
        ```
        
2. **Helm Setup:**
    - Add the Kong Helm chart repository and update:
        
        ```bash
        helm repo add kong <https://charts.konghq.com>
        helm repo update
        ```
        
3. **Create Certificates:**
    - Generate a certificate and key for mTLS:
        
        ```bash
        openssl req -new -x509 -nodes -newkey ec:<(openssl ecparam -name secp384r1) \\
          -keyout ./tls.key -out ./tls.crt -days 1095 -subj "/CN=kong_clustering"
        ```
        
    - Create a Kubernetes secret with the certificate:
        
        ```bash
        kubectl create namespace kong
        kubectl create secret tls kong-cluster-cert --cert=./tls.crt --key=./tls.key -n kong
        ```
        
4. **Create a Control Plane in Konnect:**
    - Create the control plane and capture its details:
        
        ```bash
        CONTROL_PLANE_DETAILS=$( curl -X POST "https://sg.api.konghq.com/v2/control-planes" \
          -H "Authorization: Bearer $KONNECT_TOKEN" \
          -d '{ "name": "demo-control-plane" }')
        ```
        
    - Extract the Control Plane ID and endpoints:
        
        ```bash
        CONTROL_PLANE_ID=$(echo $CONTROL_PLANE_DETAILS | jq -r .id)
        CONTROL_PLANE_ENDPOINT=$(echo $CONTROL_PLANE_DETAILS | jq -r '.config.control_plane_endpoint | sub("https://";"")')
        CONTROL_PLANE_TELEMETRY=$(echo $CONTROL_PLANE_DETAILS | jq -r '.config.telemetry_endpoint | sub("https://";"")')
        ```
        
    - Upload the certificate to the Control Plane:
        
        ```bash
        CERT=$(awk 'NF {sub(/\\r/ ""); printf "%s\\\\n",$0;}' tls.crt)
        curl -X POST "<https://sg.api.konghq.com/v2/control-planes/0b0b27d9-56bc-4d12-98bc-3747472aa8b4/dp-client-certificates>" \
          -H "Authorization: Bearer $KONNECT_TOKEN" \
          --json '{ "cert": "'$CERT'" }'
        ```
        
5. **Create a `values-dp.yaml` File for the Data Plane:**
    - Example content:
        
        ```yaml
        ingressController:
          enabled: false
        
        image:
          repository: kong/kong-gateway
          tag: ""
        
        secretVolumes:
          - kong-cluster-cert
        
        env:
          role: data_plane
          database: "off"
          konnect_mode: 'on'
          vitals: "off"
          cluster_mtls: pki
          cluster_control_plane: "<CONTROL_PLANE_ENDPOINT>"
          cluster_telemetry_endpoint: "<CONTROL_PLANE_ENDPOINT>:443"
          cluster_telemetry_server_name: "<CONTROL_PLANE_ENDPOINT>"
          cluster_cert: /etc/secrets/kong-cluster-cert/tls.crt
          cluster_cert_key: /etc/secrets/kong-cluster-cert/tls.key
        
        ```
        
    - Replace `<CONTROL_PLANE_ENDPOINT>` with the value from the previous step.
6. **Deploy the Data Plane with Helm:**
    
    ```bash
    helm install kong kong/kong --values ./values-dp.yaml -n kong --create-namespace
    
    ```
    

This process will connect your Kong Gateway data plane to the Konnect control plane using Helm and mTLS certificates [Install Kong Gateway in Konnect with Helm](https://developer.konghq.com/gateway/install/kubernetes/konnect/#install-site-base-gateway-in-konnect-with-helm).