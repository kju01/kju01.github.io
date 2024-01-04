---
title: backend-fastapi tutorial
author: kju
layout: post
---
<html>
  <head>
    <title>fastapi tutorial</title>
    <meta charset="utf-8">
  </head>
<body>
  <h2>본 글은 bitfumes youtube채널의 fastapi 영상을 follow up 하는 글입니다.</h2>
  <p><span style="color: blue;"><a href="https://www.youtube.com/watch?v=7t2alSnE2-I&t=1050s">강의 링크</a></span> - FastAPI - A python framework | Full Course</p>
  <ol>
    <li>
      개발환경
      <p>
        영상과 같이 vscode를 이용하여 진행함.
      </p>
      <p>
        pipenv를 통해 가상환경 생성하였음.
      </p>
      <p>
        가상환경내에는 fastapi, uvicorn, pydantic모듈이 있어야 함. 이러한 과정은 영상에도 나와있으니 참고바람.
      </p>
      <p>
        실습과정에서 splite db를 사용함. 이를 위해 tableplus설치가 필요함.
        <br>vscode로 하는 경우 splite viewer 확장프로그램을 설치해야 blog.db를 생성하였을때 database파일로 표시가 됨.
      </p>
    </li>
    <li>
      00:22:33 break it down
      <p>server 활성화 시 터미널에 'uvicorn main:app --reload'로 실행이 가능하다.</p>
    </li>
    <li>
      01:25:37 Create Model and Tables
      <ul>
        <li></li>
        <li>
        <p>코드 실행 중 발생하는 error</p>
        <p>
        pipenv를 통해 가상환경을 설정하였을 때 기준<br>
        위의 강의를 듣다가  create model and tables 과정에서 'ImportError: DLL load failed while importing _sqlite3: 시스템에서 파일에 액세스할 수 없습니다.' error가 발생하였다.
        <br>해결방법 : sqlite3.def와 sqlite3.dll 파일을 다운 받은 후 자신의 pipenv 가상환경에 압축을 풀어주면 해결된다.
        <br>Cf.anaconda3를 통해 conda 가상환경을 생성한 경우 anaconda3/DLLs 폴더에 압축을 풀어주면 해결된다고 한다.
        </li>
      </p>
      </ul>
    </li>
  </ol>
  



</body>
</html>
