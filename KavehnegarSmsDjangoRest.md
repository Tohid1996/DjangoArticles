# Kavehnegar SMS service in Django Rest Framework
Sometimes it happens to us that we have to log in and register the user by sending an SMS code to the mobile number and an SMS code. Numerous services are used to perform this process, which Kavehnegar uses

![Kavehnegar SMS service](https://miro.medium.com/max/1600/1*Mtu6FbQ4EvEFGiW66okNug.png)

## Execution steps:
1. We install the Kavehnegar package
```
pip install kavenegar
```
2. Define the package 
```python
from kavenegar import *
```
3. API_KEY definition 

### Is the identity key of your service, which will be provided to you after entering the Kavehnegar panel and launching the SMS service.

### **api_key** It is confidential and should not be available to the public 
```python
API_KEY = 'your_api_key'
```
4. Define the method for sending the code
```python
def kave_negar_token_send(receptor, token):
    try:
        api = KavenegarAPI(API_KEY)
        params = {
            'receptor': receptor,
            'template': 'your_template',
            'token': token
        }
        response = api.verify_lookup(params)
    except APIException as e:
        print(e)
    except HTTPException as e:
        print(e)
```
**This method is created from two inputs:**

The user's mobile number is used by the recipient
There is a four-, five-, or six-digit code that you use for the token
Your service type is that you see the driver service from Kaveh panel in a visible way and define the existing pattern
A class to connect to your service and a brief package is installed KavenegarAPI (API_KEY)

### verify_lookup (Parameters):

By using this method to authenticate users by sending a validation code or essential information such as sending a password, member approval code, invoice number, purchase and discount codes and using the screen

5. How to make a four, five and six digit code
```python
token = randint(1000, 9999) //4 digits number
token = randint(10000, 99999) //5 digits number
token = randint(100000, 999999) //6 digits number
```

6. Use method and send code
```python
token = randint(1000, 9999) //4 digits number
kave_negar_token_send(ser.data['phone_number'], token)
```
7. How to validate the code sent by the user

**There are several ways to validate:**

For example, you can save the code along with the user's mobile number in the database when sending the code to the user, and then where the user sends the code to you, fetch the mobile number and check with the sent, the database code Fetch the data. Complete the operation code

8. The desired class for generating and sending code
```python
class GenerateOTP(APIView):
    serializer_class = GetOTP

    def post(self, request, format=None):
        ser = self.serializer_class(
            data=request.data,
            context={'request': request}
        )
        if ser.is_valid():
            ...
            token = randint(100000, 999999) //6 digits number
            kave_negar_token_send(ser.data['phone_number'], token)
            ...
```
