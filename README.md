# switch-bot
Control Switch Bot Bot in Python

Here the code to login into the API with your secret and token, get the deviceID for your Bot and press the button. 

```
import requests
import json
import uuid
import time
import hmac
import hashlib
import base64

# Your token and secret
token = 'YOUR_TOKEN'
secret = 'YOUR_SECRET'

# Generate nonce and timestamp for GET request
nonce_get = str(uuid.uuid4())
t_get = int(round(time.time() * 1000))

# Create string to sign for GET request
string_to_sign_get = '{}{}{}'.format(token, t_get, nonce_get)
string_to_sign_get = bytes(string_to_sign_get, 'utf-8')
secret_bytes = bytes(secret, 'utf-8')

# Generate signature for GET request
sign_get = base64.b64encode(hmac.new(secret_bytes, msg=string_to_sign_get, digestmod=hashlib.sha256).digest()).decode('utf-8').upper()

# Headers for GET request
apiHeader_get = {
    'Authorization': token,
    'Content-Type': 'application/json',
    'charset': 'utf8',
    't': str(t_get),
    'sign': sign_get,
    'nonce': str(nonce_get)
}

# GET request URL
get_url = 'https://api.switch-bot.com/v1.0/devices'

# Making the GET request
response_get = requests.get(get_url, headers=apiHeader_get)
devices_response = response_get.json()

# Extracting the Bot device ID
bot_device_id = None
for device in devices_response.get('body', {}).get('deviceList', []):
    if device.get('deviceType') == 'Bot':
        bot_device_id = device.get('deviceId')
        break

if bot_device_id is None:
    print("No Bot device found.")
    exit()

print("Bot Device ID:", bot_device_id)

# Generate nonce and timestamp for POST request
nonce_post = str(uuid.uuid4())
t_post = int(round(time.time() * 1000))

# Create string to sign for POST request
string_to_sign_post = '{}{}{}'.format(token, t_post, nonce_post)
string_to_sign_post = bytes(string_to_sign_post, 'utf-8')

# Generate signature for POST request
sign_post = base64.b64encode(hmac.new(secret_bytes, msg=string_to_sign_post, digestmod=hashlib.sha256).digest()).decode('utf-8').upper()

# Headers for POST request
apiHeader_post = {
    'Authorization': token,
    'Content-Type': 'application/json',
    'charset': 'utf8',
    't': str(t_post),
    'sign': sign_post,
    'nonce': str(nonce_post)
}

# POST request URL for v1.1 API
device_id = bot_device_id
post_url = f'https://api.switch-bot.com/v1.1/devices/{device_id}/commands'

# Data for POST request (command to press the Bot)
data = {
    'command': 'press',
    'parameter': 'default',
    'commandType': 'command'
}

# Making the POST request
response_post = requests.post(post_url, json=data, headers=apiHeader_post)
response_data = response_post.json()

# Check if the response is successful
if response_data.get('statusCode') == 100:
    for item in response_data.get('body', {}).get('items', []):
        if item.get('deviceID') == bot_device_id:
            print(f"Command sent successfully to device {item.get('deviceID')}.")
            print(f"Status: Battery {item['status']['battery']}%, Power {item['status']['power']}.")
            print("Message:", item.get('message'))
else:
    print("Failed to send command. Status Code:", response_data.get('statusCode'))
    print("Response:", response_data)
```
