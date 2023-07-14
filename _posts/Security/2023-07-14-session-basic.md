---
title: "[Dreamhack] session-basic"
categories:
- Security

toc: true
toc_sticky: true
---
<https://dreamhack.io/wargame/challenges/409/>

## 문제 정보
쿠키와 세션으로 인증 상태를 관리하는 간단한 로그인 서비스입니다.

admin 계정으로 로그인에 성공하면 플래그를 획득할 수 있습니다.

플래그 형식은 DH{…} 입니다.

## 문제 풀이

문제 코드를 한번 살펴보자
```python
#!/usr/bin/python3
from flask import Flask, request, render_template, make_response, redirect, url_for

app = Flask(__name__)

try:
    FLAG = open('./flag.txt', 'r').read()
except:
    FLAG = '[**FLAG**]'

users = {
    'guest': 'guest',
    'user': 'user1234',
    'admin': FLAG
}


# this is our session storage
session_storage = {
}


@app.route('/')
def index():
    session_id = request.cookies.get('sessionid', None)
    try:
        # get username from session_storage
        username = session_storage[session_id]
    except KeyError:
        return render_template('index.html')

    return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')


@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'GET':
        return render_template('login.html')
    elif request.method == 'POST':
        username = request.form.get('username')
        password = request.form.get('password')
        try:
            # you cannot know admin's pw
            pw = users[username]
        except:
            return '<script>alert("not found user");history.go(-1);</script>'
        if pw == password:
            resp = make_response(redirect(url_for('index')) )
            session_id = os.urandom(32).hex()
            session_storage[session_id] = username
            resp.set_cookie('sessionid', session_id)
            return resp
        return '<script>alert("wrong password");history.go(-1);</script>'


@app.route('/admin')
def admin():
    # developer's note: review below commented code and uncomment it (TODO)

    #session_id = request.cookies.get('sessionid', None)
    #username = session_storage[session_id]
    #if username != 'admin':
    #    return render_template('index.html')

    return session_storage


if __name__ == '__main__':
    import os
    # create admin sessionid and save it to our storage
    # and also you cannot reveal admin's sesseionid by brute forcing!!! haha
    session_storage[os.urandom(32).hex()] = 'admin'
    print(session_storage)
    app.run(host='0.0.0.0', port=8000)

```

이 코드에서 
```

users = {
    'guest': 'guest',
    'user': 'user1234',
    'admin': FLAG
}

 return render_template('index.html', text=f'Hello {username}, {"flag is " + FLAG if username == "admin" else "you are not admin"}')

```
라는 부분으로 admin 의 비밀번호가 flag인것을 유추할수 있다.  
그렇다면 어떻게 admin의 비멀번호를 구해야할까??

코드를 다시 보도록 하자  
```
@app.route('/admin')
def admin():
    # developer's note: review below commented code and uncomment it (TODO)

    #session_id = request.cookies.get('sessionid', None)
    #username = session_storage[session_id]
    #if username != 'admin':
    #    return render_template('index.html')

    return session_storage
```
![K-166](https://github.com/freeman3427/freeman3427.github.io/assets/92138609/898da6d3-43f3-4c16-8560-842596d74895)
![K-165](https://github.com/freeman3427/freeman3427.github.io/assets/92138609/1634fd3d-857f-48a1-8f71-c3959cdd69a5)

<br>
코드를 보면 주소 뒤에 /admin 으로 한 뒤 접속하면 세션 저장소가 나온다<br/>
admin의 세션을 복사하도록 하자  <br/>
<br>
<img src="https://img1.daumcdn.net/thumb/R1280x0/?scode=mtistory2&fname=https%3A%2F%2Fblog.kakaocdn.net%2Fdn%2FcbJcnJ%2FbtrHI32i3d9%2FkH6sEsvnS0tLLwZ9U7Gmr1%2Fimg.png">
<br>

브라우저 개발자 도구를 열고 복사한 세션 값을 붙여넣자.

그리고 새로고침 
<br>
![K-164](https://github.com/freeman3427/freeman3427.github.io/assets/92138609/b7df0118-fc60-42d5-915c-3e680fca7d25)


