이 페이지는 ORM(Object Relational Mapper)의 개념을 이해하고, Flask상에서 RDBMS(PostgreSQL)을 ORM으로 구현하고 간단한 활용예제(CRUD 명령문)를 구현하는 페이지입니다.<br/><br/>


## ORM 이란
Object Relational Mapper 의 약자로 개발자가 관계형 데이터베이스에 접근함에 있어 쿼리문 대신, 객체(Class Object)를 이용하여 DB를 관리할 수 있도록 둘을 연결(Mapping)해주는 방법을 말합니다. DB의 쿼리문 대신 객체의 함수를 통해 SQL문을 자동으로 생성하여, DB를 관리할 수 있도록 합니다.<br/>

ORM을 Flask에 적용하는 방법은 두 가지 있습니다.  
    - 하나는 `sqlalchemy` 이고,   
    - 다른 하나는 `Flask-SQLAlchemy` 입니다.<br/><br/>
`Flask-SQLAlchemy` 의 경우 Flask에서 ORM을 보다 쉽게 적용하기 위해서 나온 확장형 패키지인데, 연결적인 측면에서는 쉬운 반면, ORM의 활용성(쿼리 확장성)이 떨어져 `sqlalchemy` 을 더 많이 활용하고 있습니다. 
그렇기 때문에, 아래의 코드에서는 `sqlalchemy` 을 활용하도록 하겠습니다. Flask-SQLAlchemy 의 장단점에 대해서 알고 싶다면 
[Use Flask and SQLalchemy, not Flask-SQLAlchemy!](https://towardsdatascience.com/use-flask-and-sqlalchemy-not-flask-sqlalchemy-5a64fafe22a4)
페이지를 참고하세요..!<br/><br/>

 
---
## DB 접속 using sqlalchemy
아래의 코드는 파이썬에서 sqlalchemy 와 RDBMS(PostgreSQL)와 연결하고, 클래스로 SQL구문을 생성하여 SELECT 결과를 추출하는 예제로, DB가 정상적으로 연결되었는지 확인할 수 있습니다.

``` python
from flask import Flask
import flask

from sqlalchemy import create_engine, inspect, select, MetaData, Table, and_
from sqlalchemy import Column, Integer, String
from sqlalchemy.ext.automap import automap_base
from sqlalchemy.ext.declarative import declarative_base
from sqlalchemy.orm import sessionmaker
import sqlalchemy as db


#DB접속
'''[접속정보] 
  id: postgreSQL 접속계정 ID, 
  pw: postgreSQL 접속계정 PW, 
  port_num: postgreSQL 포트번호, 
  db_name: postgreSQL 데이터베이스명, 
  table: postgreSQL 테이블명, 예제: table_A
'''
engine = db.create_engine('postgresql://{id}:{pw}@localhost:{port_num}/{db_name}')
Session = sessionmaker(autocommit=False, autoflush=False, bind=engine)
Session.configure(bind=engine)
session = Session()

#DB전체 테이블리스트 프린트
print(engine.table_names())

#connection 생성 및 메타데이터 로드 & 예제테이블 SELECT
conn = engine.connect()  #conn 생성
metadata = db.MetaData()

Table_A = db.Table('{table}', metadata, autoload=True, autoload_with=engine)  #테이블객체 생성
query = db.select([Table_A])              #select 구문생성
ResultProxy = conn.execute(query)         #구문 실행
ResultSet = ResultProxy.fetchall()        #결과 저장
```
<br/><br/>


## 매핑선언
ORM에서는 DB와 접속 이후에 개인이 작성한 클래스에 매핑해야 합니다. sqlalchemy에서는 두 가지가 동시에 이뤄지는데 Declarative 를 이용해 클래스를 생성하고 실제 DB 테이블에 연결합니다. 
이후에 연결해야할 클래스를 생성해서, DB 와 클래스를 매핑해보겠습니다. 예제에서는 테이블리스트와 코멘트, 샘플을 담고 있는 table_A 테이블 1개를 기준으로 작성해보겠습니다.

```python
from sqlalchemy.ext.declarative import declarative_base

#DB 매핑선언
Base = automap_base(metadata=metadata)

#DB연결 확인
inspector = inspect(engine)
schemas = inspector.get_schema_names()   #스키마라스트

#전체 테이블리스트
inspector.get_table_names()              #테이블리스트
```
<br/><br/>


## 클래스 생성
automap에 의해서 생긴 메타데이터 정보를 정의하는 과정으로 Base 클래스를 상속받으면서 해당정보를 매핑하는 것으로 추정되고, 컬럼정보 및 클래스표현(repr)정보를 같이 추가하는 코드입니다.
해당클래스는 데이터를 조회(SELECT)하거나, 추가/삭제/조건부조회 등 DataBase의 기본적인 명령문을 수행하는데 필요합니다.  

```python
#클래스 생성 및 컬럼 override And repr 추가

class Table_A(Base):
    __tablename__ = 'table_A' #DB내 테이블명

    #컬럼별 속성정의
    id = Column(Integer, primary_key=True)
    schema = Column(String)
    tables = Column(String)
    table_name = Column(String)
    sample = Column(String)
    useyn = Column(Integer)

    def __init__(self, id, schema, tables, table_name, sample, useyn):
        self.id = id
        self.schema = schema
        self.tables = tables
        self.table_name = table_name
        self.sample = sample
        self.useyn = useyn
    
    #클래스 표현식 정의
    def __repr__(self):
        return "<Table_A('%s', '%s', '%s', '%s', '%s', '%s')>" % (self.id, self.schema, self.tables, self.table_name, self.sample, self.useyn)
 
```
<br/><br/>


## 클래스 매핑

```python
#DB정보 테이블명에 기반하여 initialize
Base.metadata.create_all(engine)
metadata.reflect(engine, only=['table_A','table_B'])

# calling prepare() just sets up mapped classes and relationships.
Base.prepare()
```
<br/><br/>


---
## 수행 예제
아래부터는 매핑이 이루어진 이후, 연결된 정보인 (1)테이블별 클래스, (2)connection 정보, (3)sqlalchemy 을 활용하여 DB를 다루는 과정을 다뤄보는 과정을 담았습니다.

### SQL 쿼리수행 SELECT 방법(1): session.query
이제 DB와 클래스가 매핑이 완료되었기 때문에, session.query 를 활용하여 select문에 해당되는 함수를 바로 추출할 수 있다. 아래 예제는  SELECT * FROM {table} LIMIT 1;  과 같은 구문입니다.  

```python
u1 = session.query(Table_A).first()   #첫번째 행 출력, 테이블: table_A
print(u1)
```
<br/>


### SQL 쿼리수행 SELECT 방법(2): SQL구문 작성 후 conn.execute()
두번째 방법은 sqlalchemy 을 활용하여 쿼리문을 날리는 방식입니다. db(sqlalchemy) 를 통해서 쿼리문을 아래와 같이 작성하고 해당 쿼리를 conn.execute() 함수를 통해 수행하여 결과를 추출할 수 있습니다.  

```python
##connection 연결여부 확인 후 안되어있는 경우 conn추가
conn = engine.connect()

#SQL쿼리수행 예제: 'SELECT * FROM table_A;'
query = db.select([Table_A]) 
ResultProxy = conn.execute(query)
ResultSet = ResultProxy.fetchall()
ResultSet[:3]
```
<br/><br/>


### UPDATE
session.query 이후 filter함수를 통해서 테이블의 WHERE조건을 추가할 수 있습니다.
아래의 예제는 스키마가 'schema_A' 이고, 테이블명이 'tables_B' 인 경우 테이블 코멘트를 '업데이트할 테이블명' 텍스트(string)로 업데이트 하는 코드입니다. 
아래의 코드를 통해 특정조건을 만족하는 행의 정보를 업데이트 할 수 있습니다.   

```python
#'UPDATE WHERE' SQL을 구현한 코드
session.query(Table_A)\
            .filter(Table_A.schema == 'schema_A')\
            .filter(Table_A.tables == 'tables_B')\
            .update({Table_A.table_name:'업데이트할 테이블명'}, synchronize_session=False)
session.commit()
```
<br/><br/>

 
### INSERT
데이터를 추가하고 싶은 경우, (1)테이블 클래스를 활용하여 인스턴스를 생성 후 (2)session.add함수를 통해서 INSERT가 가능합니다.

```python
new_tablecmt =  Table_A(
                        id = session.query(func.max(Table_A.id)).first()[0] + 1,
                        schema = '스키마A'
                        tables = 'table_A'
                        table_name = '테이블 A 설명(코멘트)'
                        sample = 'sample-dateset'
                        useyn = 'y/n value'
                        )
session.add(new_tablecmt)
session.commit()
```
<br/><br/> 


### DELETE
데이터를 지우고 싶은 경우, (1)filter 를 활용하여 지우고 싶은 데이터를 출력한뒤 (2)delete 함수를 통해 삭제할 수 있습니다.

```python
session.query(Table_A)\                    #테이블
        .filter(Table_A.id==etccmt_id)\    #조건
        .delete()
        
```
<br/><br/>



위의 예제를 통해서, RDBMS 성격의 Database와 Flask(Python)을 연결할 수 있게 되었습니다.  
### ORM은 "개발자는 개발에 집중하자"는 취지에서 출발하여 만들어졌습니다.
개발자가 쿼리로 복잡하게 처리하는 것보다 DB를 객체화하여 객체지향형 작업의 장점(가독성, 재생산성)을 극대화하고, 객체전문가인 개발자의 역량을 충분히 활용하자는데 의의가 있는 듯합니다.

이번 페에지는 튜토리얼 수준이지만, ORM을 처음 접근하시는 분들에게 필수적인 정보는 어느정도 담고 있는 듯합니다. 많은 분들에게 도움이 되었으면 좋겠습니다.  

다음 튜토리얼 시리즈에서 뵈어요..!<br/><br/>




