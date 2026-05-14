# [OCP] Replacing the default ingress certificate, link Reference

Owner: Nam Tran
Last edited time: September 9, 2025 10:28 AM

Renew the default ingress certificate

```bash
# create folder ssl and move all config from ingress-operator to folder
cd ssl
# remove old wildcard certificate 
rm -f wildcard.apps.vvt1.*
# gen new key
openssl genrsa -out wildcard.apps.vvt1.ocp4.unit.local.key 2048
# gen cert signed request
openssl req -new -key wildcard.apps.vvt1.ocp4.unit.local.key -out wildcard.apps.vvt1.ocp4.unit.local.csr -config sanCSR_wildcard.apps.vvt1.ocp4.unit.local.conf
# using root key and cert to sign
openssl x509 -req -in wildcard.apps.vvt1.ocp4.unit.local.csr -CA rootCA_unit.local.crt -CAkey rootCA_unit.local.key -CAcreateserial -out wildcard.apps.vvt1.ocp4.unit.local.crt -days 825 -sha256 -extfile sanCA_wildcard.apps.vvt1.ocp4.unit.local.conf

# create new secret ssl for wildcard *.apps
oc create secret tls unit-local-ssl --cert=ssl/wildcard.apps.vvt1.ocp4.unit.local.crt --key=ssl/wildcard.apps.vvt1.ocp4.unit.local.key -n openshift-ingress
# Update the Ingress Controller configuration with the newly created secret
oc patch ingresscontroller.operator default --type=merge -p '{"spec":{"defaultCertificate": {"name": "unit-l
```