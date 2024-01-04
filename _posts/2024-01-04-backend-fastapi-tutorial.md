---
title: backend-fastapi tutorial
author: kju
layout: post
---
fastapi
<html>
  <head>
    <title>fastapi tutorial</title>
    <meta charset="utf-8">
  </head>
<body>

  <h3>fastapi에 관한 강의를 듣는 도중 발생한 error</h3>
  <p><a href="https://www.youtube.com/watch?v=7t2alSnE2-I&t=1050s">강의 링크</a> - FastAPI - A python framework | Full Course</p>


<p>
  pipenv를 통해 가상환경을 설정하였을 때 기준
  위의 강의를 듣다가 01:25:37 create model and tables 과정에서 'ImportError: DLL load failed while importing _sqlite3: 시스템에서 파일에 액세스할 수 없습니다.' error가 발생하였다.
  <br>sqlite3.def와 sqlite3.dll 파일을 다운 받은 후 자신의 pipenv 가상환경에 압축을 풀어주면 해결된다.
  <br>cf.anaconda3를 통해 conda 가상환경을 생성한 경우 anaconda3/DLLs 폴더에 압축을 풀어주면 해결된다고 한다.
</p>
</body>
</html>
