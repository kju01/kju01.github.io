---
title: Backend - fastapi basic
subtitle: 간단한 blog 예제를 통한 fastapi 배워보기
author: kju
layout: post
categories: backend
---

fastapi basic
=============

### 본 포스팅은 fastapi 공식문서에서 blog를 만드는 예제를 통해 fastapi의 기본적인 함수를 알아보는 포스팅입니다.


[강의 링크](https://www.youtube.com/watch?v=7t2alSnE2-I&t=1050s "FastAPI - A python framework | Full Course") FastAPI - A python framework | Full Course by bitfumes   
[github](https://github.com/kju01/fastapi_tutorial "fastapi tutorial source code") - 'fastapi tutorial' source code
## 개발환경과 실습 도중 발생한 이슈    
 영상과 같이 __vscode__ 를 이용하여 진행하였다.   

__pipenv__ 를 통해 가상환경 생성하였다.    

가상환경내에는 __fastapi, uvicorn, sqlalchemy, passlib, brypt, python-jose__ 모듈이 있어야 한다. 자세한 내용은 github 소스코드에서 Pipfile에 나와있다.    

실습과정에서 splite db를 사용함. 이를 위해 tableplus설치하거나 vscode로 하는 경우 splite viewer 확장프로그램을 설치하여 db를 확인할 수도 있다.   

#### 코드 실행 중 발생하는 error 01:25:37 Create Model and Tables (pipenv를 통해 가상환경을 설정하였을 때 기준)   
위의 강의를 듣다가 create model and tables 과정에서 'ImportError: DLL load failed while importing _sqlite3: 시스템에서 파일에 액세스할 수 없습니다.' error가 발생하였다.   

해결방법 : sqlite3.def와 sqlite3.dll 파일을 다운 받은 후 자신의 pipenv 가상환경에 압축을 풀어주면 해결된다. (소스코드 기준 blog폴더에도 넣어야 실행이 가능했다.)   

Cf. anaconda3를 통해 conda 가상환경을 생성한 경우 anaconda3/DLLs 폴더에 압축을 풀어주면 해결된다고 한다.

## 주요 모듈 소개

### fastapi   

- __Depends__  [-상세정보-](https://fastapi.tiangolo.com/tutorial/dependencies/ "Dependencies")   
종속성(dependency)를 선언하기 위한 class로 함수를 실행하기 위해 필요한 인수를 정의하기 위해 사용된다.
- __status__  [-상세정보-](https://fastapi.tiangolo.com/tutorial/response-status-code/ "response status code")   
   http의 상태를 입력할 수 있는 코드로 ```status.HTTP_204_NO_CONTENT``` 와 같이 넘버링과 함께 그 넘버의 의미가 같이 포함되어 있어 넘버의 의미를 굳이 외울 필요가 없다.   
- __HTTPException__ [-상세정보-](https://fastapi.tiangolo.com/tutorial/handling-errors/ "handing errors")   
  http 예외처리를 작성해주는 함수로 입력 예시는 ```raise HTTPException(status.HTTP_404_NOT_FOUND, detail=f"(에러 내용)")``` 이다.   
- __security.OAuth2PasswordBearer__ [-상세정보-](https://fastapi.tiangolo.com/tutorial/security/first-steps/ "security")   
OAuth2 표준의 비밀번호 인증 플로우를 구현하는데 사용되는 클래스로 이 클래스는 사용자의 아이디와 비밀번호를 받아 인증 토큰을 발급하고, 이를 통해 API에 접근하는 것을 지원한다.   
- __FastAPI__ [-상세정보-](https://fastapi.tiangolo.com/reference/fastapi/ "FastAPI")   
FastAPI 프레임워크에서 API 엔드포인트를 정의하기 위해 사용되는 함수이다.
서버의 제일 큰 틀이라고 생각하면 될 것같다.   
- __APIRouter__ [-상세정보-](https://fastapi.tiangolo.com/reference/apirouter/ "APIRouter")  
router를 작성하기 위해 필요한 함수로 API 엔드포인트를 그룹화 및 모듈화를 하게 해준다. 이를 통해 API를 역할에 따라 분리하여 저장함으로서 코드를 조직화하고 유지보수하기 쉽게 만드는 역할을 한다.
tutorial의 경우 user, blog, authentication과 같이 분리하여 router를 정의하였다.

### pydantic   

- __BaseModel__ [-상세정보-](https://fastapi.tiangolo.com/tutorial/body/ "Request Body")    
Request Body(schemas)를 정의하기위해 사용되는 class이다. 자세한 내용은 schemas.py를 참조

### uvicorn 
[-상세정보-](https://fastapi.tiangolo.com/deployment/manually/ "run a server manually - Uvicorn")   

server를 실행하기 위해 필요한 모듈로 blog폴더에서 터미널을 연 후  ```uvicorn main:app --reload``` 를 입력하면 실행이 된다. 
여기서 main:app은 FastAPI()가 정의된 py파일에서 FastAPI()를 정의한 변수명과 관련있다. 이 소스코드의 경우 main.py에 app=FastAPI()로 정의되어 있으므로 main:app으로 입력하면 된다.
--reload의 경우 소스코드를 변경할 때마다 바로바로 반영되게 해준다.

### sqlalchemy
[-상세정보-](https://fastapi.tiangolo.com/tutorial/sql-databases/ "SQL (Relational) Databases")   

- Column
default값으로 사용되는 값으로 ```id = Column(Integer, primary_key=True)```와 같은 식으로 column을 정의해야한다.
- Integer, String
각 데이터의 형태 중 하나
- ForeignKey
model을 서로 연결하기 위한 값
- orm   
ORM : object-relational mapping

- orm.relationship
다른 table과 관계를 정의하기 위한 class로 
```items = relationship("Item"(table의 class명), back_populates="owner"(table내에 관계를 정의한 변수)) ```
```... ```
```owner = relationship("User", back_populates="items")```
- ext.declarative.declarative_base
database models 또는 classes를 각각 만들기 위한 class
본 예제에서의 경우 ```class User(Base):``` 와 같이 database를 정의할 때 사용된다.
정의할 때 ```__tablename__ = "users```와 같은 방식으로 모델에서 사용될 tablename을 정의해야한다.
- orm.sessionmaker
database session 이 될 class를 생성하기 위해 사용된다.
- create_engine
engine을 정의하기 위한 함수로 저장하기위한 database파일의 경로가 필요하다.
- orm.Session
db의 매개변수이다.

### passlib[bcrypt]   
- context.CryptContext
password를 hash, verify(확인)할 때 사용된다. password가 들어오면 이를 hashing하고 password가 맞는지 verify하는 역할을 한다.

### python-jose   
- JWTError, jwt
random한 secret key(token)를 생성하기 위함.

## 각 py파일 설명(blog 폴더 내 기준)   


### database.py      

```python
from sqlalchemy import create_engine
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker

#db를 얻는 dependency를 작성
def get_db():
  db = SessionLocal()
  try:
      yield db
  finally:
      db.close()

#db의 형태 정의 및 경로 설정(sqlite, ./blog.db)
sqlalchemay_database_url = 'sqlite:///./blog.db'

engine = create_engine(sqlalchemay_database_url, connect_args={'check_same_thread': False})

SessionLocal = sessionmaker(bind=engine, autocommit=False, autoflush=False,)

Base = declarative_base()
```

[cf. yield를 이용한 dependency 작성](https://fastapi.tiangolo.com/tutorial/dependencies/dependencies-with-yield/ "Dependencies with yield")


### hashing.py    

```python
from passlib.context import CryptContext

pwd_cxt = CryptContext(schemes=["bcrypt"], deprecated='auto')

class Hash():
  def bcrypt(password: str):
      return pwd_cxt.hash(password) # hashing
  
  def verify(hashed_password, plain_password):
      return pwd_cxt.verify(plain_password, hashed_password) # 검증
```

### JWTtoken.py    

```python
from datetime import datetime, timedelta
from jose import JWTError, jwt
import schemas

# $ openssl rand -hex 32
SECRET_KEY =  "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
ALGORITHM = "HS256"
ACCESS_TOKEN_EXPIRE_MINUTES = 30

# 새로운 엑세스 토큰을 만들기 위한 함수
def create_access_token(data: dict):
  to_encode = data.copy()
  expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
  to_encode.update({"exp": expire})
  encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
  return encoded_jwt

#token을 받고 확인한 후 user를 반환
def verify_token(token:str,credentials_exception):
  try:
      payload = jwt.decode(token, SECRET_KEY, algorithms=[ALGORITHM])
      email: str = payload.get("sub")
      if email is None:
          raise credentials_exception
      token_data = schemas.TokenData(email=email)
  except JWTError:
      raise credentials_exception
```

### models.py    

```python
from database import Base
from sqlalchemy import Column, Integer, String, ForeignKey
from sqlalchemy.orm import relationship

class Blog(Base):
  __tablename__ = 'blogs'

  id = Column(Integer,primary_key=True,index=True)
  title = Column(String)
  body = Column(String)
  user_id = Column(Integer, ForeignKey('users.id'))

  creator = relationship("User", back_populates="blogs")
    

class User(Base):
  __tablename__ = 'users'

  id = Column(Integer,primary_key=True,index=True)
  name = Column(String)
  email = Column(String)
  password = Column(String)

  blogs = relationship('Blog', back_populates="creator")
```

### oauth2.py    

```python
from fastapi import Depends, HTTPException, status
from fastapi.security import OAuth2PasswordBearer
import JWTtoken

oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

# 만약 token이 거부되면 http error 반환, 아니면 token을 확인 후 user반환
def get_current_user(data: str = Depends(oauth2_scheme)):
  credentials_exception = HTTPException(
      status_code=status.HTTP_401_UNAUTHORIZED,
      detail="Could not validate credentials",
      headers={"WWW-Authenticate": "Bearer"},
  )

  return JWTtoken.verify_token(data, credentials_exception)
```

### schemas.py     

```python
from pydantic import BaseModel
from typing import List, Optional

class BlogBase(BaseModel):
  title: str
  body: str

class Blog(BlogBase):
  class Config():
      orm_mode = True # 데이터베이스와의 상호작용 더욱 편리

class User(BaseModel):
  name : str
  email: str
  password : str

class ShowUser(BaseModel):
  name : str
  email: str
  blogs: List[Blog] = []

class ShowBlog(BaseModel):
  title: str
  body: str
  creator: ShowUser

  class Config():
      orm_mode = True

class Login(BaseModel):
  username: str
  password:str

class Token(BaseModel):
  access_token: str
  token_type: str

class TokenData(BaseModel):
  username: Optional[str]= None

```

### routers/authentication.py    

```python
from fastapi import APIRouter, Depends, HTTPException,status
from fastapi.security import OAuth2PasswordRequestForm
import database, models, JWTtoken
from hashing import Hash
from sqlalchemy.orm import Session

router = APIRouter(
    tags=['Authentication']
)

# user를 확인하기 위한 부분, 만약 확인이 되면 권한을 얻게 됨
@router.post('/login')
def login(request:OAuth2PasswordRequestForm = Depends(), db: Session = Depends(database.get_db)):
  user = db.query(models.User).filter(models.User.email == request.username).first()
  if not user:
      raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, 
                          detail=f'invalid Credentials')
  if not Hash.verify(user.password, request.password):
      raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,
                          detail=f"Incorrect password") 
  
  access_token = JWTtoken.create_access_token(data={"sub": user.email})
  return {"access_token": access_token, "token_type": "bearer"}
```

### routers/blog.py    

```python
from fastapi import APIRouter, Depends, status
import schemas, database, oauth2
import repository.blog as blog
from typing import List
from sqlalchemy.orm import Session

router = APIRouter(
    prefix="/blog",
    tags=['blog']
)
# get : /blog/로 접속시 바로 뜨게 되는 내용들이다.
@router.get('/', response_model=List[schemas.ShowBlog])
def all(db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):`
  return blog.get_all(db)

#blog를 만드는 함수
@router.post('/' , status_code=status.HTTP_201_CREATED) # status_code에 따라 뜻이 다름, 201 = create fastapi.status에서 뭔지 확인 가능
def create(request : schemas.Blog, db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
  return blog.create(request,db)
  
#blog를 삭제하는 함수
@router.delete('/{id}',status_code=status.HTTP_204_NO_CONTENT)
def destroy(id, db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
  return blog.destroy(id,db)
  
#blog를 업데이트하는 함수
@router.put('/{id}',status_code=status.HTTP_202_ACCEPTED)
def update(id, request: schemas.Blog, db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
  return blog.update(id,request,db)

#특정 id의 블로그를 보여준다
@router.get('/{id}', status_code=200,response_model=schemas.ShowBlog) # in sql, where / schemas로 데이터 불러오는 형태 지정
def show(id, db: Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
  return blog.show(id,db)

```

### routers/user.py     

```python
from fastapi import APIRouter, Depends
import schemas, database
from sqlalchemy.orm import Session
from repository import user


router = APIRouter(
    prefix="/user",
    tags=['User']
)

#user를 만드는 함수
@router.post('/', response_model=schemas.ShowUser)
def create_user(request: schemas.User, db : Session = Depends(database.get_db)):
  return user.create_user(request,db)

#특정 id의 user를 불러오는 함수
@router.get('/{id}', response_model=schemas.ShowUser)
def get_user(id:int, db:Session = Depends(database.get_db)):
  return user.get_user(id,db)
```

### repository/blog.py     

```python
from sqlalchemy.orm import Session
import models, schemas, database
from fastapi import Depends, status, HTTPException

def get_all(db):
  blogs = db.query(models.Blog).all() # blog db에서의 query를 전부 보여준다.
  return blogs

def create(request:schemas.Blog, db : Session = Depends(database.get_db)):
  new_blog = models.Blog(title = request.title, body = request.body, user_id=1)
  db.add(new_blog)
  db.commit() # db에 반영
  db.refresh(new_blog)
  return new_blog

def destroy(id:int, db:Session = Depends(database.get_db)):
  blog = db.query(models.Blog).filter(models.Blog.id == id) #삭제하길 원하는 blog의 id 를 통해 접근
  if not blog.first(): # 삭제하길 원하는 id의 blog가 없다면 error반환
      raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"Blog with id {id} not found")
  blog.delete(synchronize_session=False)
  db.commit()
  return 'done'

def update(id:int, request:schemas.Blog, db : Session = Depends(database.get_db)):
  blog = db.query(models.Blog).filter(models.Blog.id == id) # 업데이트하길 원하는 blog의 id를 통해 접근
  if not blog.first(): # 업데이트하길 원하는 id의 blog가 없다면 error반환
      raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"Blog with id {id} not found")
  blog.update(request)
  db.commit()
  return 'updated'

def show(id:int, db: Session = Depends(database.get_db)):
  blog = db.query(models.Blog).filter(models.Blog.id == id).first() # 보기를 원하는 blog의 id를 통해 접근
  if not blog:
      raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,detail=f"Blog with the id {id} is not available")
  return blog

```

### repository/user.py      

```python
from fastapi import Depends, status, HTTPException
import schemas, database, models, hashing
from sqlalchemy.orm import Session

#user 만드는 함수
def create_user(request: schemas.User, db : Session = Depends(database.get_db)):
  new_user = models.User(name=request.name,email = request.email, password = hashing.Hash.bcrypt(request.password))
  db.add(new_user)
  db.commit()
  db.refresh(new_user)
  return new_user

#user를 불러오는 함수
def get_user(id:int, db:Session = Depends(database.get_db)):
  user = db.query(models.User).filter(models.User.id == id).first()
  if not user:
      raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,detail=f"User with the id {id} is not available")
  return user
```