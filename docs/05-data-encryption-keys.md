# Generating the Data Encryption Config and Key

## Encryption Key

```
ENCRYPTION_KEY=$(head -c 32 /dev/urandom | base64)
```

## Encryption Config File

```
cat > encryption-config.yaml <<EOF
kind: EncryptionConfig
apiVersion: v1
resources:
  - resources:
      - secrets
    providers:
      - aescbc:
          keys:
            - name: key1
              secret: ${ENCRYPTION_KEY}
      - identity: {}
EOF
```

## Distribute encryption config file

```
for i in k8s-controller1 k8s-controller2 k8s-controller3; do
scp encryption-config.yaml defo@${i}:~/
done
```