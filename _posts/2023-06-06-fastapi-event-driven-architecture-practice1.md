---
title: fastapi event driven architecture practice - ok카우
categories:
  - Dev
tags:
  - event
  - fastapi
  - architecture  
  - eventsourcing
  - cqrs
last_modified_at: 2023-06-06T22:30:00-00:00
---

앞서 정리한 **Event Driven Architecture The Complete Guide - 강의노트** 를 바탕으로 가상 시나리오를 작성하고 event driven architecture 를 연습해보겠습니다.

---

음악 저작권 투자거래 서비스 ok카우 서비스의 투자자 관리시스템 개발을 진행한다.

투자자의 다음 사항을 시스템화하여 ok카우 운영자가 각각의 투자자들에 대한 현황 파악을 돕도록 한다.

- 투자금 납입(입금)
- 투자 시점
    - 시점별 지분가치(음악 저작권 지수 MCPI)를 계산하여, 이에 상응하는 가치로 음악 저작권료 참여 청구권 지분을 발행
- 투자 자산(현황)
- 수익률
    - 투자자가 후속투자자의 투자금 및 수익 변동에 따라, 인식할 음악 저작권료 참여 청구권의 보유 가치 변화분
- 투자금 회수(출금)
    - 위에서 확인된 투자금/수익률(기준가)에 따라 최종적으로 확인되는 투자자의 투자금에 대한 가치(대가)


![MusiCow](/assets/images/musicow-homepage.png)

거래 전 꼭 알아야하는 용어

- 다자간상대매매 : 사고자 하는 사람과 팔고자 하는 사람의 가격이 일치될 때에만 체결되는 거래 방식
- 현재가 : 직전 거래의 체결 가격
- 가격 단위 : 1단위 = 100 캐쉬
- 거래대금 : 당일 체결한 거래금 합산액 (구매액 + 판매액)
- 예치금 : 보유 현금 합산액 (사용가능 금액  + 구매 대기금)
- 평가액 : 보유 저작궈료 참여청구권 현재가의 합산액
- 서킷브레이커 : MCPI 지수가 급락할 경우 마켓 거래를 일시적으로 중단시키는 제도

디자인 패턴으로 CQRS, Eventsourcing 패턴을 사용한다.

모든 이벤트는 이벤트 로그로 기록되며, 이벤트 로그를 조합하면 특정 시간의 펀드 자산 가치 확인이 가능하다

---

언어는 Python FastAPI를 Database는 mysql 을 사용합니다.

참고로 FastAPI template 은 해당 Repo(https://github.com/teamhide/fastapi-boilerplate)를 사용하였습니다 

먼저 주어진 요구사항을 해결하기 위해 데이터베이스 스키마 설계부터 시작해보겠습니다.

Investor 는 투자자에 해당하고 Fund는 펀드, Investment 는 투자에, MusicCopyrightProperty 는 음악 저작권을 나타냅니다.

마지막에 있는 Event 테이블은 중간에 event 처리부분에서 설명해보겠습니다.

```python
from sqlalchemy import Column, Integer, String, DateTime, Float, ForeignKey
from sqlalchemy.orm import relationship
from sqlalchemy.sql import func
from database import Base

class Investor(Base):
    __tablename__ = "investors"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, unique=True, index=True)

    investments = relationship("Investment", back_populates="investor")

class Fund(Base):
    __tablename__ = "funds"

    id = Column(Integer, primary_key=True, index=True)
    name = Column(String, unique=True, index=True)
    total_supply = Column(Float)
    base_price = Column(Float)
    capital_cash = Column(Float)  # 펀드의 총자산

    investments = relationship("Investment", back_populates="fund")
    mcps = relationship("MusicCopyrightProperty", back_populates="fund")  # 펀드가 보유한 MCP

class Investment(Base):
    __tablename__ = "investments"

    id = Column(Integer, primary_key=True, index=True)
    investor_id = Column(Integer, ForeignKey("investors.id"))
    fund_id = Column(Integer, ForeignKey("funds.id"))
    investment_amount = Column(Float)
    investment_time = Column(DateTime, default=func.now())

    investor = relationship("Investor", back_populates="investments")
    fund = relationship("Fund", back_populates="investments")

class MusicCopyrightProperty(Base):  # Music Copyright Property 모델
    __tablename__ = "music_copyright_properties"

    id = Column(Integer, primary_key=True, index=True)
    copyright_code = Column(String)  # 저작권 코드
    registration_number = Column(String)  # 등록 번호
    music_id = Column(String)  # 고유 음악 ID
    price_cash = Column(Float)  # 추정 가치
    buy_at = Column(DateTime)  # 취득 일자
    fund_id = Column(Integer, ForeignKey("funds.id"))  # 해당 MCP가 속한 펀드 ID

    fund = relationship("Fund", back_populates="music_copyright_properties")

class Event(Base):
    __tablename__ = "events"

    id = Column(UUID(as_uuid=True), primary_key=True, server_default=text("(gen_random_uuid())"))
    type = Column(String, nullable=False)
    payload = Column(JSON)
```

위에서 만든 데이터베이스 스키마를 기반으로 API를 작성해보겠습니다.

가장 처음 작성할 것은 투자자를 등록하는 API 입니다

```python
from fastapi import FastAPI
from fastapi.middleware.cors import CORSMiddleware
from typing import List

app = FastAPI()

app.add_middleware(
    CORSMiddleware,
    allow_origins=["*"],
    allow_credentials=True,
    allow_methods=["*"],
    allow_headers=["*"],
)

from pydantic import BaseModel
from sqlalchemy.orm import Session
import models
from database import SessionLocal, engine

models.Base.metadata.create_all(bind=engine)

class Investor(BaseModel):
    name: str

class InvestorCreate(BaseModel):
    name: str
    id: str
    password: str

@app.post("/investors/", response_model=Investor)
def create_investor(investor: InvestorCreate):
    db = SessionLocal()
    hashed_password = hashlib.sha256(investor.password.encode()).hexdigest()  # 비밀번호는 보안을 위해 해시화하여 저장
    db_investor = models.Investor(name=investor.name, id=investor.id, password=hashed_password)
    db.add(db_investor)
    db.commit()
    db.refresh(db_investor)
    db.close()
    return {"id": db_investor.id}
```

투자자 등록 API를 구현하기 위해, 먼저 사용자 이름, 아이디, 비밀번호를 받을 수 있는 **`InvestorCreate`** 모델을 정의해보았습니다. 이어서 create_investor() 에 기본적인 구현사항을 작성했습니다.

다음은 위 코드에 앞서 배운 CQRS와 Event Sourcing을 적용하는 간단한 예시입니다.

투자자 등록과 등록된 투자자를 조회하는 2개 API를 가정했습니다.

```python
from fastapi import FastAPI, HTTPException
from pydantic import BaseModel
import uuid

class Event(BaseModel):
    id: str
    type: str
    payload: dict

# In-memory event store
events = []

app = FastAPI()

# Command handler
@app.post("/investors/", response_model=Investor)
async def create_investor(investor: InvestorCreate):
    # Create event
    event = Event(
        id=str(uuid.uuid4()),
        type="InvestorCreated",
        payload=investor.dict(exclude={"password"})
    )
    # Store event
    events.append(event.dict())

    # Process command
    db = SessionLocal()
    hashed_password = hashlib.sha256(investor.password.encode()).hexdigest()
    db_investor = models.Investor(name=investor.name, id=investor.id, password=hashed_password)
    db.add(db_investor)
    db.commit()
    db.refresh(db_investor)
    db.close()

    return {"id": db_investor.id, "status": "Investor Created"}

# Query handler
@app.get("/investor/{investor_id}")
async def get_investor(investor_id: str):
    # Reconstruct state from events
    for event in events:
        if event["type"] == "InvestorCreated" and event["payload"]["id"] == investor_id:
            return event["payload"]
    raise HTTPException(status_code=404, detail="Investor not found")
```

CQRS & 이벤트 소싱은 언제 왜 사용하는 것일까?

분산 데이터베이스 환경이 가진 문제 중 하나가 데이터 결합이다. 여러 데이터베이스에 나누어져 있는 데이터를 하나의 뷰(view)로 구성해서 클라이언트에 제공하려면 어떻게 해야할까?

마이크로 서비스에서 고려할 수 있는 기법에는 두 가지가 있다. 하나는 API 컴포지션이고 다른 한 가지 기법은 CQRS&이벤트소싱이다. 

CQRS(명령 질의 책임 분리)란, 데이터 접근 처리를 갱신형 처리 (명령, 즉 데이터 삽입/변경/삭제) 와 참조형 처리 (질의, 즉 데이터 검색/취득) 로 구분하고 각각을 구현하기 위해 독립된 서비스 컴포넌트와 데이터 저장소를 두는 디자인 패턴이다. 

갱신형 처리와 참조형 처리용으로 분리된 프로그램과 데이터 저장소를 사용하자는 발상이 CQRS다. 갱신형 처리에는 트랜잭션 기능과 신뢰성 높은 영구적 데이터 저장소를 만들고, 참조형 처리에는 고속 검색 기능을 가진 데이터 스토어를 배치하므로 최적화된 설계를 실현할 수 있는 것이다.

이벤트 소싱에서는 비즈니스 데이터를 분할하지 않고 그대로 모아서 하나의 데이터 저장소에 저장한다. 이 데이터 저장소를 이벤트 소스(event source)라고 한다. 

---

다시 작성된 코드를 살펴보면, 
이 코드에서 **`create_investor`** 함수는 투자자를 생성하는 명령을 처리하고, **`get_investor`** 함수는 투자자 정보를 조회하는 쿼리를 처리합니다. 이렇게 라우터를 명령 처리와 쿼리 처리로 분리함으로써 CQRS를 적용했습니다.

또한, **`create_investor`** 함수에서는 투자자를 생성하는 이벤트를 만들고 이를 이벤트 저장소에 저장함으로써 Event Sourcing을 적용했습니다. 먼저 투자자를 생성하는 이벤트를 만들어 이벤트 저장소에 저장하고, 그 다음으로 데이터베이스에 투자자 정보를 입력하게 됩니다. 이렇게 하면 이벤트 저장소와 데이터베이스가 항상 동기화됩니다.

새 투자자를 등록했으니 투자자가 투자금을 납입하는 API를 작성해보겠습니다.

투자자의 id정보와 투자금액을 입력값으로 정의하고,

```python
class InvestmentCreate(BaseModel):
    investor_id: str
    investment_amount: float
```

```python
@app.post("/investment/")
async def create_investment(investment: InvestmentCreate):
    # Create event
    event = Event(
        id=str(uuid.uuid4()),
        type="InvestmentCreated",
        payload=investment.dict()
    )

    # Store event
    events.append(event.dict())

    # Update the investor's investment
    db = SessionLocal()
    investor = db.query(models.Investor).filter(models.Investor.id == investment.investor_id).first()
    if not investor:
        raise HTTPException(status_code=400, detail="Investor not found")

    investor.investment_amount += investment.investment_amount
    db.commit()
    db.refresh(investor)
    db.close()

    return {"status": "Investment Created"}
```

위 방식으로 API를 구현하면, 투자자의 투자금을 업데이트하면서 동시에 투자금 납입에 대한 이벤트도 기록하게 됩니다. 이를 통해 이벤트 저장소와 데이터베이스를 동기화시킬 수 있습니다

투자자가 음악 저작권에 투자를 진행했고 이후에 수익화를 위해 구매한 저작권료 참여 청구권을 마켓에서 판매 시 구매가와 판매가의 차액만큼 이익 혹은 손실이 발생할 것입니다. 

아래는 투자자가 음악 저작권을 수익화하는 투자금 회수 API 부분입니다.

```python
# Pydantic 모델을 정의하여 API 요청 형식을 설정합니다.
class WithdrawInvestment(BaseModel):
    investor_id: str
    stock_quantity: int

@app.post("/investment/withdraw", status_code=status.HTTP_200_OK)
async def withdraw_investment(withdraw: WithdrawInvestment):
    db = SessionLocal()

    investor = db.query(models.Investor).filter(models.Investor.id == withdraw.investor_id).first()
    if not investor:
        raise HTTPException(status_code=400, detail="Investor not found")

    if withdraw.stock_quantity < 0:
        raise HTTPException(status_code=400, detail="Invalid stock quantity")

    # 투자자가 보유한 주식수를 체크
    if investor.stock_quantity < withdraw.stock_quantity:
        raise HTTPException(status_code=400, detail="Not enough stock quantity")

    # 가치 변동에 따른 투자자의 회수액을 계산
    # 이때, 가치는 아래 compute_fund_value()를 사용 - 현재 미구현
		fund_value = await compute_fund_value(db, investment.fund_id)

    # Compute the withdrawal amount
    withdraw_amount = fund_value * withdraw.stock_quantity / investment.fund.total_supply

    # Event 생성
    event = Event(
        id=str(uuid.uuid4()),
        type="InvestmentWithdrawn",
        payload={
            **withdraw.dict(),
            "withdrawn_amount": withdrawn_amount,
        }
    )

    # Event 저장
    events.append(event.dict())

    # 주식 회수 처리
    investor.stock_quantity -= withdraw.stock_quantity
    db.commit()
    db.close()

    return {"responseCode": 200, "withdrawn_amount": withdrawn_amount}  # 정상 응답 코드
```

위 API는 투자자 ID와 주식 수를 입력으로 받아 투자금을 회수하는 이벤트를 생성하고 저장합니다.

Event Sourcing을 사용하는 경우, 이벤트를 생성하고 저장하는 것이 주된 작업이지만, 실제 상태 변경은 이벤트를 처리하는 별도의 로직에서 수행되어야 합니다

이쯤에서 고민해봐야하는게 사용자수가 늘어나 대규모 인원이 되었을때 시스템 상황을 가정해 보는 것입니다.
지금처럼 한 두명만 기능을 사용하는게 아니라 투자금 납입과 투자금 회수 등 여러 사용자가 동시에 시스템을 사용한다면 fund를 안정적으로 운영하기 위해 동시성 처리를 어떻게 할 수 있을까요?

대규모 사용자를 관리하는 시스템에서는 데이터의 동시성 처리가 매우 중요합니다. 여러 사용자가 동시에 같은 데이터에 접근하려고 할 때, 데이터 무결성이 깨질 수 있기 때문입니다.

이 문제를 해결하기 위한 방법 중 하나는 데이터베이스의 트랜잭션을 활용하는 것입니다. 트랜잭션을 사용하면 데이터의 무결성을 보장할 수 있습니다.

Python의 SQLAlchemy ORM을 사용한다면, sessionmaker의 **`autocommit=False`** 설정으로 이를 해결할 수 있습니다. 이 설정은 각 세션에서 트랜잭션을 자동으로 시작하고, 명시적으로 세션을 커밋하거나 롤백하기 전까지 변경사항을 DB에 커밋하지 않습니다.

```python
@app.post("/investment/withdraw", status_code=status.HTTP_200_OK)
async def withdraw_investment(withdraw: WithdrawInvestment):
    db = SessionLocal()

    investor = db.query(models.Investor).filter(models.Investor.id == withdraw.investor_id).first()
    if not investor:
        raise HTTPException(status_code=400, detail="Investor not found")

    if withdraw.stock_quantity < 0:
        raise HTTPException(status_code=400, detail="Invalid stock quantity")

    # Event 생성
    event = Event(
        id=str(uuid.uuid4()),
        type="InvestmentWithdrawn",
        payload={
            **withdraw.dict(),
            "withdrawn_amount": withdrawn_amount,
        }
    )

    # Event 저장
    events.append(event.dict())

    try:
        # 투자자의 주식 수를 감소시키고, 이를 DB에 저장합니다.
        if investor.stock_quantity < withdraw.stock_quantity:
            raise HTTPException(status_code=400, detail="Insufficient stock quantity")
        investor_stock.quantity -= withdraw.stock_quantity
        db.commit()  # Commit the transaction.
    except:
        db.rollback()  # If an error occurred, rollback the transaction.
        raise HTTPException(status_code=500, detail="An error occurred while processing the transaction")
    finally:
        db.close()

    return {"responseCode": 200}  # 정상 응답 코드
```

이전 코드와는 다르게 — 투자자가 주식을 출금할때 출금하려는 주식 수량이 투자자가 보유한 주식 수량보다 많지 않은지 확인하고 출금하는 수량만큼 투자자의 주식수를 감소 시키는 — 해당 부분에서 발생할 수 있는 동시성 문제를 트랜잭션을 사용하여 해결합니다. 트랜잭션 내에서 수행되는 모든 작업은 원자적으로 처리되며, 에러가 발생하면 롤백되므로 데이터의 무결성이 보장됩니다.

이러한 방식을 통해 동시에 여러 사용자가 투자금을 회수하려 하더라도 각 트랜잭션은 순차적으로 처리되므로 펀드의 총량이 음수가 되는 등의 문제를 방지할 수 있습니다.

또 다른 방법으로는, 락(locking)을 사용하여 특정 자원에 대한 동시 접근을 제한할 수도 있습니다. 예를 들어, 데이터베이스 레벨에서 제공하는 행 레벨 락이나, 어플리케이션 레벨에서 구현하는 뮤텍스(mutex) 등의 도구를 사용할 수 있습니다.

조금더 간편하게 트랜잭션 처리를 할 수있는 방법을 찾다보니

처음 언급했던 https://github.com/teamhide/fastapi-boilerplate 를 보면 아래와 같은 부분이 [READ](http://README.md)ME 에 나옵니다.

## SQLAlchemy for asyncio context

```python
from core.db import Transactional, session

@Transactional()
async def create_user(self):
    session.add(User(email="padocon@naver.com"))

#Do not use explicit commit(). Transactional class automatically do.
```

위 예시 코드는 명시적으로 commit 을 하지않아도 되고 @Transactional() 데코레이터를 사용하면 동시성처리와 트랜잭션 관리를 수행하는 방식입니다.

현재까지 작성된 create_investor(), create_investment(), withdraw_investment() 메서드에 @Transactional() 데코레이터를 추가하고  db.commit() 부분은 지웁니다. - @Transactional() 데코레이터가 알아서 commit 해주기 떄문, 그럼 가장 마지막에 이야기한 withdraw_investment() 메서드의 실제 변경사항을 살펴보겠습니다.

```python
from core.db import Transactional, session

@app.post("/investment/withdraw", status_code=status.HTTP_200_OK)
@Transactional()
async def withdraw_investment(withdraw: WithdrawInvestment):
    db = session

    investor = await db.query(models.Investor).filter(models.Investor.id == withdraw.investor_id).first()
    if not investor:
        raise HTTPException(status_code=400, detail="Investor not found")

    if withdraw.stock_quantity < 0:
        raise HTTPException(status_code=400, detail="Invalid stock quantity")

    # Fetch the investment
    investment = await db.query(models.Investment).filter(models.Investment.investor_id == withdraw.investor_id, models.Investment.fund_id == withdraw.fund_id).first()
    if not investment:
        raise HTTPException(status_code=400, detail="Investment not found")

    # Check if the investor has enough shares
    if investment.investment_amount < withdraw.stock_quantity:
        raise HTTPException(status_code=400, detail="Not enough stock quantity")

    fund_value = await compute_fund_value(db, investment.fund_id)

    # Compute the withdrawal amount
    withdrawn_amount = fund_value * withdraw.stock_quantity / investment.fund.total_supply

    # Create the 'InvestmentWithdrawn' event
    event = Event(
        id=str(uuid.uuid4()),
        type="InvestmentWithdrawn",
        payload={
            **withdraw.dict(),
            "withdrawn_amount": withdrawn_amount,
        }
    )

    # Store the event
    events.append(event.dict())

    # Update the investment
    investment.investment_amount -= withdraw.stock_quantity

    return {"responseCode": 200}  # 정상 응답 코드
```

@Transactional() 데코레이터를 사용함으로써 db.commit() 와 같은 코드를 사용하지않고 조금더 간결하게 비즈니스 로직만 작성할 수 있습니다. 

투자자를 등록하는 API로도 비교해보겠습니다.

이벤트 생성 및 저장 부분은 생략했고, @Transactional() 데고레이터 사용으로 db.commit()과 db.close() 가 삭제되었습니다.

```python
#ASIS
@app.post("/investors/", response_model=Investor)
async def create_investor(investor: InvestorCreate):
...

    # Process command
    db = SessionLocal()
    hashed_password = hashlib.sha256(investor.password.encode()).hexdigest()
    db_investor = models.Investor(name=investor.name, id=investor.id, password=hashed_password)
    db.add(db_investor)
    db.commit()
    db.refresh(db_investor)
    db.close()

    return {"id": db_investor.id, "status": "Investor Created"}

#TOBE
@app.post("/investors/", response_model=Investor)
@Transactional()
async def create_investor(investor: InvestorCreate):
...

    # Process command
    db = session
    hashed_password = bcrypt.hashpw(investor.password.encode(), bcrypt.gensalt())
    db_investor = models.Investor(name=investor.name, id=investor.id, password=hashed_password)
    db.add(db_investor)
    db.refresh(db_investor)

    return {"id": db_investor.id, "status": "Investor Created"}
```


## 참고자료


[뮤직카우](https://www.musicow.com/about/guide)

[도서-그림으로 공부하는 마이크로서비스 구조](https://www.aladin.co.kr/shop/wproduct.aspx?ItemId=298604016)

