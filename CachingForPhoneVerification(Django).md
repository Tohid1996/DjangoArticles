# django - implement login SMS verification
## Function demonstration

![CachingForPhoneVerificationDjango](https://files.virgool.io/upload/users/7548/posts/paytetrm088z/ng70fgu4vu33.png)

Core tasks

- Front end function:

1. Click the button Ajax to call the send verification code function
2. Ajax calls the verification function after entering the verification code

- Back end features:

1. Function 1: send verification code function
2. Function 2: verification code check

- Background core logic (no handwriting required)

1. Function 3: send SMS
2. Function 4: generate SMS verification code (randomly generate 6 digits)

- Integrated Redis

1. Use Redis instead of session cache to store data!
2. Redis is integrated into Django!

- Extended features:

1. Unified interface return result specification method!

Function 1:Django integrated Redis
Because our SMS verification code life cycle control is very strict! And the data does not need to be stored after use. So it is recommended to directly store the data in the cache / memory!

1. Scheme 1: use session or cookie storage! Session is valid in the current browser! Cookie storage is not safe in the user's local area! Session and cookie operations are complex, time control is not accurate! The amount of data stored is very limited!
2. Scheme 2: use Redis/Mongdb and other key:value databases! They can read and write very fast, store a large amount of data, and control the validity period very accurately!


Step 1: Download Django redis module

```
 pip install django-redis
 ```
## Step 2: configure write configuration in setting.py
Configure Redis as Django cache and replace the original session 
 
```python
 #Configure Redis as Django cache
CACHES = {
"default": {
"BACKEND": "django_redis.cache.RedisCache",
"LOCATION": "redis://127.0.0.1: 6379 / 0 ", address
"OPTIONS": {
"CLIENT_CLASS": "django_redis.client.DefaultClient",
}
}
}
# Cache session in Redis
SESSION_ENGINE = "django.contrib.sessions.backends.cache"
SESSION_CACHE_ALIAS = "default"
# session settings (can not be written)
SESSION_COOKIE_AGE = 60 * 60 * 12 # 12 hours
SESSION_SAVE_EVERY_REQUEST = True
SESSION_EXPIRE_AT_BROWSER_CLOSE = True # If the browser is closed, COOKIE fails
``` 
## Step 3: views import cache module 
``` 
from django.core.cache import cache 
``` 
## Step 4: use
```python
def test_redis(request):
	# Store data
	chache.set("name", "tom", 20) # The value is valid for 20s
	# Judge whether there is Redis
	print(cache.has_kay("name"))  # Include: true
	# Obtain
	print(cache.get("name"))  # Return: tom no return null
	return HttpResponse("test Redis")
```
## Function 2: SMS sending
## Step 1: apply for SMS service

Reference document application Alibaba cloud SMS service.pdf document
## Step 2: module of sending SMS and generating verification code independently
```python
#!/usr/bin/env python
#coding=utf-8
import random
from aliyunsdkcore.client import AcsClient
from aliyunsdkcore.request import CommonRequest
from utils import restful
def send_sms(phone,code):
	client = AcsClient('mxTYXZ4QDQecJQDN', 'znxNezmm4zfA9kPyqx1WrpznjCaJFT', 'cnhangzhou')
	#phone = '17600950805'
	#aa= '222222'
	code = "{'code':%s}"%(code)
	request = CommonRequest()
	request.set_accept_format('json')
	request.set_domain('dysmsapi.aliyuncs.com')
	request.set_method('POST')
	request.set_protocol_type('https') # https | http
	request.set_version('2017-05-25')
	request.set_action_name('SendSms')
	request.add_query_param('RegionId', 'cn-hangzhou')
	request.add_query_param('PhoneNumbers', phone)
	request.add_query_param('SignName', 'North net')
	request.add_query_param('TemplateCode', 			'SMS_162738723')
	request.add_query_param('TemplateParam',code )
	response = client.do_action(request)
	# python2: print(response)
	print(str(response, encoding = 'utf-8'))
	return str(response, encoding = 'utf-8')
#The number indicates how many digits are generated, and the True indicates that the False with letters is generated without letters
def get_code(n=6,alpha=True):
	s = '' # Create a string variable to store the generated verification code
	for i in range(n): # Verification code number by for loop control
			num = random.randint(0,9) # Generate random numbers 0-9
			if alpha: # The letter verification code is required, and the parameter is not needed. If the letter is not needed, the keyword alpha=False
				upper_alpha = chr(random.randint(65,90))
				lower_alpha = chr(random.randint(97,122))
				num = random.choice([num,upper_alpha,lower_alpha])
			s = s + str(num)
	return s
if __name__ == '__main__':
	send_sms('18434288349', get_code(6,False))
	print(get_code(6,False)) # Print 6-digit verification code
	print(get_code(6,True)) # Print 6-digit alphanumeric verification code
	print(get_code(4,False)) # Print 4-digit verification code
	print(get_code(4,True)) # Print 4-digit alphanumeric verification code
```
## Function 3: background function: send SMS interface

Technological process:
Get mobile number - > generate 6-digit verification code - > cache verification code to Redis - > send SMS - > return status
```python
# SMS interface
def sms_send(request):
	# http://localhost:8000/duanxin/duanxin/sms_send/?phone=18434288349
	# 1 get mobile number
	phone = request.GET.get('phone')
	# 2 generate 6-digit verification code
	code = aliyunsms.get_code(6, False)
	# 3 cache to Redis
	cache.set(phone,code,60) #60s validity period
	print('Determine whether there is any in the cache:',cache.has_key(phone))
	print('Obtain Redis Verification Code:',cache.get(phone))
	# 4 text messaging
	result = aliyunsms.send_sms(phone, code)
	return HttpResponse(result)
  ```
## Function 4: verification of SMS verification code

Technological process:
Get the front desk phone and verification code - > get the verification code stored in Redis - > whether the comparison is equal - > return the result
```python
# Verification of SMS verification code
def sms_check(request):
	# /duanxin/sms_check/?phone=xxx&code=xxx
	# 1. Verification code for telephone and manual input
	phone = request.GET.get('phone')
	code = request.GET.get('code')
	# 2. Get the code saved in redis
	print('Whether the cache contains:',cache.has_key(phone))
	print('Value:',cache.get(phone))
	cache_code = cache.get(phone)
	# 3. judgement
	if code == cache_code:
		return HttpResponse(json.dumps({'result':'OK'}))
	else:
		return HttpResponse(json.dumps({'result':'False'}))
```
Manually test the assumed parameters on the browser:
Http: / / localhost: 8000 / Duanxin / SMS? Send /? Phone = phone number

Http: / / localhost: 8000 / Duanxin / SMS? Check /? Phone = mobile number & code = verification code

## Function 5: unified interface data format:
Independent restful.py interface module!

Unified interface module restful.py
```python
#encoding: utf-8
from django.http import JsonResponse
class HttpCode(object):
	ok = 200
	pageerror = 404
	methoderror = 405
	servererror = 500
# {"code":400,"message":"","data":{}}
def result(code=HttpCode.ok,message="",data=None,kwargs=None):
	json_dict = {"code":code,"message":message,"result":data}
	if kwargs and isinstance(kwargs,dict) and kwargs.keys():
		json_dict.update(kwargs)
	return JsonResponse(json_dict,json_dumps_params={'ensure_ascii': False})
def ok(message='OK',data=None):
	return result(code=HttpCode.ok,message=message,data=data)
def page_error(message="",data=None):
	return result(code=HttpCode.pageerror,message=message,data=data)
def method_error(message='',data=None):
	return result(code=HttpCode.methoderror,message=message,data=data)
def server_error(message='',data=None):
	return result(code=HttpCode.servererror,message=message,data=data)
```
The return results of any interface are normalized with the restful.py method
```python
# Verification of SMS verification code
def sms_check(request):
	# /duanxin/sms_check/?phone=xxx&code=xxx
	# 1. Verification code for telephone and manual input
	phone = request.GET.get('phone')
	code = request.GET.get('code')
	# 2. Get the code saved in redis
	print('Whether the cache contains:',cache.has_key(phone))
	print('Value:',cache.get(phone))
	cache_code = cache.get(phone)
	# 3. judgement
	if code == cache_code:
		#After unified format adjustment
		return restful.ok("OK",data=None)
	else:
		#After unified format adjustment
		return restful.params_error("Verification code error", data=None)
```
## Function 6: two front-end SMS Ajax
```javascript
<script type="text/javascript">
            //Count down
            var countdown=60;
            function sendemail(){
                var obj = $("#btn");
                settime(obj);

                }
            function settime(obj) { //Send verification code countdown
                if (countdown == 0) {
                    obj.attr('disabled',false);
                    //obj.removeattr("disabled");
                    obj.val("Get verification code for free");
                    countdown = 60;
                    return;
                } else {
                    obj.attr('disabled',true);
                    obj.val("Resend(" + countdown + ")");
                    countdown--;
                }
            setTimeout(function() {
                settime(obj) }
                ,1000)
            }
        </script>
     <script>
            //Verification Code
            $('#btn').click(function () {
                phone = $('#phone').val();
                if (phone == "") {
                    alert("Please enter your mobile number");
                    return false;
                }
                //ajax
                $.ajax({
                    type: "get",
                    url: "/user/sms_send",
                    data: "phone="+phone,
                    success: function (msg) {
                        console.log("Data Saved: "+msg);
                        //Convert to json
                        obj = eval("("+msg+")");
                        console.log('Result: '+obj.Message);
                        if (obj.Message == "OK") {
                            $('#msg').html("message sent successfully")
                        }else{
                            $('#msg').html("message sending failed")
                        }
                    },
                    error: function (res) {
                        //Status code
                        console.log(res.status)
                    }
                });
            })
        </script>
<script>
          //Verification of SMS verification code
            $('#smscode').change(function () {
                phone = $('#phone').val();
                code = $('#smscode').val();
                $.ajax({
                    type: "get",
                    url: "/user/sms_check",
                    data: "phone="+phone+"&code="+code,
                    success: function (msg) {
                        console.log("Data Saved: "+msg);
                        if (msg.code == '200') {
                            alert('Success');
                        }else{
                            $('#msg').html("mobile phone verification code error")
                        }
                    },
                    error: function (res) {
                        console.log(res.status)
                    }
                });

            })
        </script>
```


