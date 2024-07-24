import requests

url = 'https://example.com/api/endpoint'
headers = {
    'Authorization': 'Bearer YOUR_TOKEN_HERE',
    'Content-Type': 'application/json',
    'Custom-Header': 'CustomHeaderValue'
}
data = {
    'key1': 'value1',
    'key2': 'value2'
}

response = requests.post(url, headers=headers, json=data)

if response.status_code == 200:
    print('Success:', response.json())
else:
    print('Error:', response.status_code, response.text)
