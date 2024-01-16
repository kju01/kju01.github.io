---
title: fastapi tutorial
layout: backend
tags: [backend, fastapi, bitfumes]

date: 2024-01-04
last_modified_at: 2024-01-16
---
# fastapi tutorial
[강의 링크](https://www.youtube.com/watch?v=7t2alSnE2-I&t=1050s "FastAPI - A python framework | Full Course") FastAPI - A python framework | Full Course by bitfumes
[github](https://github.com/kju01/fastapi_tutorial "fastapi tutorial source code") - 'fastapi tutorial' source code
  ### 개발환경과 실습 도중 발생한 이슈
  영상과 같이 __vscode__ 를 이용하여 진행하였다.

  __pipenv__ 를 통해 가상환경 생성하였다.

  가상환경내에는 __fastapi, uvicorn, sqlalchemy, passlib, brypt, python-jose__ 모듈이 있어야 한다. 자세한 내용은 github 소스코드에서 Pipfile에 나와있다.

  실습과정에서 splite db를 사용함. 이를 위해 tableplus설치하거나 vscode로 하는 경우 splite viewer 확장프로그램을 설치하여 db를 확인할 수도 있다.

  __코드 실행 중 발생하는 error__ 01:25:37 Create Model and Tables 
  (pipenv를 통해 가상환경을 설정하였을 때 기준)
  위의 강의를 듣다가  create model and tables 과정에서 'ImportError: DLL load failed while importing _sqlite3: 시스템에서 파일에 액세스할 수 없습니다.' error가 발생하였다.
  해결방법 : sqlite3.def와 sqlite3.dll 파일을 다운 받은 후 자신의 pipenv 가상환경에 압축을 풀어주면 해결된다. (소스코드 기준 blog폴더에도 넣어야 실행이 가능했다.)
  Cf. anaconda3를 통해 conda 가상환경을 생성한 경우 anaconda3/DLLs 폴더에 압축을 풀어주면 해결된다고 한다.

  ### 주요 모듈 소개

  #### fastapi
  - __Depends__
    
  - HTTPException
  - status
  - security.OAuth2PasswordBearer
  - FastAPI
  - APIRouter

  #### pydantic
  - BaseModel

  #### uvicorn
  server를 실행하기 위해 필요한 모듈로 blog폴더에서 터미널을 연 후  ```uvicorn main:app --reload``` 를 입력하면 실행이 된다. 
  여기서 main:app은 FastAPI()가 정의된 py파일에서 FastAPI()를 정의한 변수명과 관련있다. 이 소스코드의 경우 main.py에 app=FastAPI()로 정의되어 있으므로 main:app으로 입력하면 된다.
  --reload의 경우 소스코드를 변경할 때마다 바로바로 반영되게 해준다.

  #### sqlalchemy
  - Column
  - Integer
  - String
  - ForeignKey
  - orm.relationship
  - ext.declarative.declarative_base
  - orm.sessionmaker
  - create_engine
  - orm.Session

  #### passlib
  - context.CryptContext

  #### brypt
  - 

  #### python-jose
  - JWTError
  - jwt

  ### 각 py파일 설명(blog 폴더 내 기준)

  #### database.py

  ```python
  from sqlalchemy import create_engine
  from sqlalchemy.ext.declarative import declarative_base
  from sqlalchemy.orm import sessionmaker

  def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
        
  sqlalchemay_database_url = 'sqlite:///./blog.db'

  engine = create_engine(sqlalchemay_database_url, connect_args={'check_same_thread': False})

  SessionLocal = sessionmaker(bind=engine, autocommit=False, autoflush=False,)

  Base = declarative_base()
  ```

  #### hashing.py

  ```python
  from passlib.context import CryptContext

  pwd_cxt = CryptContext(schemes=["bcrypt"], deprecated='auto')

  class Hash():
    def bcrypt(password: str):
        return pwd_cxt.hash(password)
    
    def verify(hashed_password, plain_password):
        return pwd_cxt.verify(plain_password, hashed_password)
  ```

  #### JWTtoken.py

  ```python
  from datetime import datetime, timedelta
  from jose import JWTError, jwt
  import schemas


  SECRET_KEY =  "09d25e094faa6ca2556c818166b7a9563b93f7099f6f0f4caa6cf63b88e8d3e7"
  ALGORITHM = "HS256"
  ACCESS_TOKEN_EXPIRE_MINUTES = 30

  def create_access_token(data: dict):
    to_encode = data.copy()
    expire = datetime.utcnow() + timedelta(minutes=ACCESS_TOKEN_EXPIRE_MINUTES)
    to_encode.update({"exp": expire})
    encoded_jwt = jwt.encode(to_encode, SECRET_KEY, algorithm=ALGORITHM)
    return encoded_jwt

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

  #### models.py

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

  #### oauth2.py
  ```python
  from fastapi import Depends, HTTPException, status
  from fastapi.security import OAuth2PasswordBearer
  import JWTtoken

  oauth2_scheme = OAuth2PasswordBearer(tokenUrl="login")

  def get_current_user(data: str = Depends(oauth2_scheme)):
    credentials_exception = HTTPException(
        status_code=status.HTTP_401_UNAUTHORIZED,
        detail="Could not validate credentials",
        headers={"WWW-Authenticate": "Bearer"},
    )

    return JWTtoken.verify_token(data, credentials_exception)
  ```

  #### schemas.py

  ```python
  from pydantic import BaseModel
  from typing import List, Optional

  class BlogBase(BaseModel):
    title: str
    body: str

  class Blog(BlogBase):
    class Config():
        orm_mode = True

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

  #### routers/authentication.py
  ```python
  from fastapi import APIRouter, Depends, HTTPException,status
  from fastapi.security import OAuth2PasswordRequestForm
  import database, models, JWTtoken
  from hashing import Hash
  from sqlalchemy.orm import Session

  router = APIRouter(
      tags=['Authentication']
  )

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

  #### routers/blog.py

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

  @router.get('/', response_model=List[schemas.ShowBlog])
  def all(db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):`
    return blog.get_all(db)

  @router.post('/' , status_code=status.HTTP_201_CREATED) # status_code에 따라 뜻이 다름, 201 = create fastapi.status에서 뭔지 확인 가능
  def create(request : schemas.Blog, db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
    return blog.create(request,db)
    

  @router.delete('/{id}',status_code=status.HTTP_204_NO_CONTENT)
  def destroy(id, db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
    return blog.destroy(id,db)
    

  @router.put('/{id}',status_code=status.HTTP_202_ACCEPTED)
  def update(id, request: schemas.Blog, db : Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
    return blog.update(id,request,db)

  @router.get('/{id}', status_code=200,response_model=schemas.ShowBlog) # in sql, where / schemas로 데이터 불러오는 형태 지정
  def show(id, db: Session = Depends(database.get_db),get_current_user: schemas.User = Depends(oauth2.get_current_user)):
    return blog.show(id,db)

  ```

  #### routers/user.py

  ```python
  from fastapi import APIRouter, Depends
  import schemas, database
  from sqlalchemy.orm import Session
  from repository import user


  router = APIRouter(
      prefix="/user",
      tags=['User']
  )


  @router.post('/', response_model=schemas.ShowUser)
  def create_user(request: schemas.User, db : Session = Depends(database.get_db)):
    return user.create_user(request,db)

  @router.get('/{id}', response_model=schemas.ShowUser)
  def get_user(id:int, db:Session = Depends(database.get_db)):
    return user.get_user(id,db)
  ```

  #### repository/blog.py

  ```python
  from sqlalchemy.orm import Session
  import models, schemas, database
  from fastapi import Depends, status, HTTPException

  def get_all(db):
    blogs = db.query(models.Blog).all()
    return blogs

  def create(request:schemas.Blog, db : Session = Depends(database.get_db)):
    new_blog = models.Blog(title = request.title, body = request.body, user_id=1)
    db.add(new_blog)
    db.commit() # db update?
    db.refresh(new_blog)
    return new_blog

  def destroy(id:int, db:Session = Depends(database.get_db)):
    blog = db.query(models.Blog).filter(models.Blog.id == id)
    if not blog.first():
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"Blog with id {id} not found")
    blog.delete(synchronize_session=False)
    db.commit()
    return 'done'

  def update(id:int, request:schemas.Blog, db : Session = Depends(database.get_db)):
    blog = db.query(models.Blog).filter(models.Blog.id == id)
    if not blog.first():
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND, detail=f"Blog with id {id} not found")
    blog.update(request)
    db.commit()
    return 'updated'

  def show(id:int, db: Session = Depends(database.get_db)):
    blog = db.query(models.Blog).filter(models.Blog.id == id).first()
    if not blog:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,detail=f"Blog with the id {id} is not available")
    return blog

  ```

  #### repository/user.py

  ```python
  from fastapi import Depends, status, HTTPException
  import schemas, database, models, hashing
  from sqlalchemy.orm import Session

  def create_user(request: schemas.User, db : Session = Depends(database.get_db)):
    new_user = models.User(name=request.name,email = request.email, password = hashing.Hash.bcrypt(request.password))
    db.add(new_user)
    db.commit()
    db.refresh(new_user)
    return new_user

  def get_user(id:int, db:Session = Depends(database.get_db)):
    user = db.query(models.User).filter(models.User.id == id).first()
    if not user:
        raise HTTPException(status_code=status.HTTP_404_NOT_FOUND,detail=f"User with the id {id} is not available")
    return user
  ```