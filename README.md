
# SMS Gateway for Android

Turn your Android phone into a local-network SMS gateway. Send SMS messages through your phone's selected SIM using an HTTP API from your PHP website, POS system, eCommerce platform, or other backend application.

**GitHub Repository:** [salahuddin-saif/SMS-gateway](https://github.com/salahuddin-saif/SMS-gateway)

> This project provides the Android application only. PHP scripts, website backends, and databases are not included. Developers can build their own integration using the API documented below.

## Features

- Send SMS through an Android phone's SIM.
- HTTP API for website and backend integration.
- Bearer-token authentication when enabled and required by the app.
- Configurable server port.
- SIM selection for dual-SIM devices.
- Health-check endpoint for connection testing.
- QR-code pairing for connecting a compatible website or client.
- Supports Bangla, English, and other Unicode message text.
- Can be integrated with PHP, POS, eCommerce, inventory, and custom applications.
- Supports website-side bulk SMS workflows when implemented by the client.
- Foreground service for keeping the gateway running while the app is in the background.

## Screenshots

The following screenshots are stored in the repository's `screenshots/` directory.

| Screenshot 1 | Screenshot 2 |
|:---:|:---:|
| ![SMS Gateway Screenshot 1](screenshots/1.png) | ![SMS Gateway Screenshot 2](screenshots/2.png) |

| Screenshot 3 | Screenshot 4 |
|:---:|:---:|
| ![SMS Gateway Screenshot 3](screenshots/3.png) | ![SMS Gateway Screenshot 4](screenshots/4.png) |

| Screenshot 5 | Screenshot 6 |
|:---:|:---:|
| ![SMS Gateway Screenshot 5](screenshots/5.png) | ![SMS Gateway Screenshot 6](screenshots/6.png) |

> Make sure the image filenames and capitalization match the actual files in the `screenshots/` directory.

## How It Works

1. Install and open the SMS Gateway app on your Android phone.
2. Grant the required permissions.
3. Select the SIM for outgoing SMS.
4. Configure the server port and access-token settings.
5. Start the gateway service.
6. Use the QR-code pairing feature to connect a compatible website or client.
7. Obtain the current phone LAN IP address, port, and any required authentication details.
8. Configure your backend with the connection information.
9. Send HTTP requests from your backend to submit SMS messages.

The Android phone must remain powered on, connected to a reachable network, and running the gateway service.

## Requirements

- Android phone with an active SIM.
- SMS permission granted to the app.
- Mobile network signal and sufficient SMS balance.
- A backend capable of sending HTTP requests.
- Network access from the backend to the Android phone.
- A compatible client for the QR-code pairing workflow, if required.

## Getting Started

1. Install the APK.
2. Open the app and grant the requested permissions.
3. Select the desired SIM.
4. Configure the HTTP port. The default port is `8080`, if unchanged in your app.
5. Configure an access token if token authentication is enabled.
6. Start the gateway.
7. Open the QR-code pairing feature and follow the instructions shown in the app.
8. Note the phone's current LAN IP address and port.
9. Configure your website or backend to use the connection details provided by the app.

Example configuration (replace these values with your actual settings):

| Setting | Example |
|---|---|
| Gateway IP | `192.168.1.105` |
| Port | `8080` |
| Health endpoint | `http://192.168.1.105:8080/health` |
| SMS endpoint | `http://192.168.1.105:8080/sms` |

**Important:** These IP and port values are examples only. Use the actual IP and port displayed or configured in your app. The endpoints above apply to app versions that implement the API described below.

## QR-Code Pairing

The QR-code pairing feature is intended to simplify connecting a compatible website or client to the Android SMS Gateway.

### Pairing steps

1. Open the SMS Gateway app.
2. Start the gateway if the app requires it before pairing.
3. Open the QR-code pairing option.
4. Scan the QR code using the compatible website or client.
5. Follow the instructions shown by the app and client to complete pairing.
6. Confirm that the client is using the correct gateway address and authentication settings.
7. Test the connection before sending SMS messages.

**Important:** The QR-code payload, pairing protocol, and authentication behavior depend on the actual app implementation. A client must follow the pairing format supported by the installed app. Do not assume that scanning the QR code alone authorizes a client.

## API Documentation

### 1. Health Check

Checks whether the gateway server is reachable.

- **Method:** `GET`
- **Endpoint:** `/health`
- **Authentication:** No Bearer token required by the implementation described here.

Example URL:

`http://192.168.1.105:8080/health`

Example response:

```json
{
  "ok": true,
  "service": "SMS gateway",
  "simSelected": true
}
```

A successful health check confirms basic connectivity. It does not guarantee that SMS messages can be delivered.

### 2. Send SMS

Submits an SMS message through the selected SIM.

- **Method:** `POST`
- **Endpoint:** `/sms`
- **Content-Type:** `application/json`
- **Authentication:** Bearer token required when token authentication is enabled.

#### Request headers

```http
Content-Type: application/json
Authorization: Bearer YOUR_ACCESS_TOKEN
Accept: application/json
```

Replace `YOUR_ACCESS_TOKEN` with the access token configured in the Android app, if required.

#### Request body

```json
{
  "phone": "+8801XXXXXXXXX",
  "message": "Hello from my website!"
}
```

| Parameter | Type | Required | Description |
|---|---|---|---|
| `phone` | String | Yes | Recipient number with an optional leading `+`; 10–15 digits |
| `message` | String | Yes | SMS message text; maximum 2,000 characters according to the validation described here |

The gateway validates the number's format but does not verify whether the recipient's number is active.

#### Successful response

The implementation described here returns HTTP `202 Accepted` when the message is submitted to Android's SMS subsystem.

```json
{
  "success": true,
  "status": "submitted",
  "note": "Submitted to Android SMS subsystem; delivery not confirmed"
}
```

**Important:** HTTP `202` means the message was submitted for processing. It does not confirm that the recipient received the SMS.

### Common errors

| Status | Meaning | Suggested action |
|---|---|---|
| `400` | Invalid request | Check the JSON body and required fields |
| `401` | Unauthorized | Verify the access token if authentication is enabled |
| `405` | Method not allowed | Use the correct HTTP method |
| `422` | Invalid phone number or message | Validate the supplied data |
| `5xx` | Server or gateway error | Check the app, network, permissions, and logs |

Actual error codes and response formats may vary by app version. Confirm the behavior against the specific release you are integrating.

## PHP Integration Guide

The app does not require a specific PHP script. Website developers can implement their own PHP integration using cURL.

### Requirements

- PHP with the cURL extension enabled.
- The Android phone's reachable LAN IP and gateway port.
- The access token, if token authentication is enabled.
- Network connectivity between the PHP server and Android phone.

### Example PHP code

The following is a minimal server-side integration example. Adapt it to your application's authentication, validation, and error-handling requirements.

```php
<?php

$gatewayIp = getenv('SMS_GATEWAY_IP') ?: '192.168.1.105';
$gatewayPort = (int) (getenv('SMS_GATEWAY_PORT') ?: '8080');
$accessToken = getenv('SMS_GATEWAY_TOKEN') ?: '';

$phone = '+8801XXXXXXXXX';
$message = 'Hello from my PHP website!';

if (!filter_var($gatewayIp, FILTER_VALIDATE_IP, FILTER_FLAG_IPV4)) {
    throw new InvalidArgumentException('Invalid gateway IPv4 address.');
}

if ($gatewayPort < 1 || $gatewayPort > 65535) {
    throw new InvalidArgumentException('Invalid gateway port.');
}

if (!preg_match('/^\+?[0-9]{10,15}$/', $phone)) {
    throw new InvalidArgumentException('Invalid phone number.');
}

if (trim($message) === '') {
    throw new InvalidArgumentException('Message cannot be empty.');
}

$payload = json_encode(
    [
        'phone' => $phone,
        'message' => $message
    ],
    JSON_UNESCAPED_UNICODE | JSON_UNESCAPED_SLASHES
);

if ($payload === false) {
    throw new RuntimeException('Could not encode SMS request.');
}

$url = sprintf(
    'http://%s:%d/sms',
    $gatewayIp,
    $gatewayPort
);

$headers = [
    'Content-Type: application/json; charset=utf-8',
    'Accept: application/json'
];

if ($accessToken !== '') {
    $headers[] = 'Authorization: Bearer ' . $accessToken;
}

$ch = curl_init($url);

curl_setopt_array($ch, [
    CURLOPT_POST => true,
    CURLOPT_POSTFIELDS => $payload,
    CURLOPT_HTTPHEADER => $headers,
    CURLOPT_RETURNTRANSFER => true,
    CURLOPT_CONNECTTIMEOUT => 5,
    CURLOPT_TIMEOUT => 20,
    CURLOPT_FOLLOWLOCATION => false
]);

$response = curl_exec($ch);
$error = curl_error($ch);
$status = (int) curl_getinfo($ch, CURLINFO_HTTP_CODE);

curl_close($ch);

if ($response === false) {
    throw new RuntimeException(
        'Gateway connection failed: ' . $error
    );
}

$result = json_decode($response, true);

if ($status === 202) {
    // Record the message as submitted, not delivered.
    echo 'SMS request submitted successfully.';
} else {
    // Log details securely; never log the access token.
    error_log('SMS gateway returned HTTP status ' . $status);
    throw new RuntimeException(
        'SMS gateway did not accept the request.'
    );
}
```

**Note:** This example assumes that the `/sms` endpoint accepts the JSON payload shown above. If token authentication is mandatory in your app, configure the token and send the Bearer header. If authentication is disabled, follow the app's actual configuration and security requirements.

### PHP integration steps

1. Configure `SMS_GATEWAY_IP`, `SMS_GATEWAY_PORT`, and, when needed, `SMS_GATEWAY_TOKEN` on the server, or load them from a protected configuration file outside the public web directory.
2. Replace the example phone number and message with validated application data.
3. Ensure the PHP server can reach the Android phone's IP and port.
4. Enable the PHP cURL extension.
5. Add authentication and authorization to your own website's SMS endpoint.
6. Log submission status and errors without exposing access tokens.
7. Track submitted messages separately from confirmed deliveries.

**Never expose the access token in frontend JavaScript, HTML, or a public repository.** Send requests from your backend.

## Bulk SMS Integration

Bulk SMS functionality can be implemented in the PHP website or other client application.

Recommended workflow:

1. Validate and prepare the recipient list.
2. Send messages sequentially or through a controlled queue.
3. Display progress on the website.
4. Record each message's submission result.
5. Handle failures and retries carefully to avoid duplicate messages.
6. Apply rate limits and obtain recipient consent where required.

A website-side progress bar reflects the client application's processing progress; it does not necessarily represent confirmed delivery.

## Integration Ideas

Developers can integrate the gateway with:

- POS and inventory systems.
- eCommerce order notifications.
- Customer support systems.
- Appointment and booking applications.
- Payment notifications.
- School or business management systems.
- Custom PHP websites and admin panels.

## Troubleshooting

### Connection timeout

- Make sure the gateway is running.
- Verify the phone's current LAN IP and port.
- Check that the PHP server can reach the phone.
- Check firewall rules and Wi-Fi client isolation.
- Remember that `localhost` refers to the machine running PHP, not the Android phone.

### Unauthorized response

- Verify that the PHP token matches the app token, if authentication is enabled.
- Check the `Authorization` header format: `Bearer YOUR_ACCESS_TOKEN`.
- If authentication is disabled, verify the expected request format in the app.

### SMS is not received

- Check SMS permissions.
- Confirm that the selected SIM is active and has mobile signal.
- Verify the SIM has sufficient balance and can send SMS.
- Remember that submission does not guarantee delivery.

### QR-code pairing fails

- Use a compatible client.
- Follow the pairing instructions displayed by the app.
- Confirm that the client is using the correct gateway details.
- Check the QR-code format and pairing implementation for your app version.

### PHP cURL error

Enable the cURL extension in the active `php.ini` file and restart Apache or the relevant PHP service.

### Gateway IP changes

Use a DHCP reservation in your router or update your backend configuration when the phone's IP changes.

## Security Notes

**The gateway may use plain HTTP on the local network.** Bearer tokens sent over plain HTTP may be intercepted by someone with access to the network.

- Use the gateway on a trusted private network.
- Do not expose the gateway port directly to the public internet.
- Do not configure public router port forwarding to the gateway.
- Keep access tokens secret.
- Add authentication, authorization, rate limiting, and abuse monitoring to your website.
- Consider a secure VPN or a properly secured backend relay if the phone and website are on different networks.
- Send messages only where there is a legitimate basis and comply with applicable laws, carrier policies, and recipient-consent requirements.

## Limitations

- The phone must remain powered on, connected, and available.
- Android background execution and battery restrictions may affect uptime.
- The backend must have network access to the phone's gateway endpoint.
- Carrier charges, delivery, SMS segmentation, and restrictions depend on the SIM operator and message content.
- The basic API does not guarantee SMS delivery receipts.
- A remote hosting provider generally cannot reach a private LAN IP such as `192.168.x.x` without an appropriate secure network connection.

## Contributing

Bug reports, documentation improvements, and code contributions are welcome.

When reporting an issue, include the Android version, app version, steps to reproduce, and sanitized error details. Never post access tokens or private phone numbers in public issues.

## Developer

**Developed by Salah Uddin**

## Repository

[https://github.com/salahuddin-saif/SMS-gateway](https://github.com/salahuddin-saif/SMS-gateway)
