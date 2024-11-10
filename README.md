# Simple YubiKey Authenticode code-signing service

The purpose of this project is to perform remote code-signing on a Linux host
running Docker where a USB YubiKey (or other device) containing a code-signing
certificate is attached.
This way other hosts (e.g. Jenkins agent nodes) can send a file to be signed
to this service, receiving a signed file as a response, without the need of
having physical access to the YubiKey.

## Build the signsvc service

```console
go build
```

## Build the Docker container image

Create the *.env* file containing the variables related to your setup:

```console
PIN="12345678"
CERT="certificate.cer"
TS_URL="http://timestamp.digicert.com"
```

Build the Docker image:

```console
docker build -t signsvc -f docker/Dockerfile .
```

## Run the Docker container

```console
docker run -d \
  --device /dev/bus/usb \
  --device /dev/usb \
  -p 9115:80 \
  --restart unless-stopped \
  --name signsvc signsvc
```

## Send a signature request

The following *curl* command will send a binary file with a POST API request,
receiving the signed file in the response.

```console
curl -sSL -F file=@file.msi http://remote_addr:9115/sign -o file_signed.msi --fail
```
