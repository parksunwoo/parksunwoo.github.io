---
title: fastapi event driven architecture practice- part2
categories:
  - Dev
tags:
  - event
  - fastapi
  - architecture  
  - eventsourcing
  - cqrs
last_modified_at: 2023-06-16T05:30:00-00:00
---

앞서 정리한 **Event Driven Architecture The Complete Guide - 강의노트** 를 바탕으로 가상 시나리오를 작성하고 event driven architecture 를 연습해보겠습니다.

---

## Event Store

이벤트 스토어는 이벤트 소싱을 활용하는 모든 시스템의 중요한 부분이며 스토리지 메커니즘의 선택은 시스템의 성능, 확장성 및 유연성에 상당한 영향을 미칠 수 있습니다. 이벤트 저장에는 다음 방법을 사용할 수 있습니다.

1. 이벤트 저장소 : 이벤트 저장을 위해 특별히 설계된 데이터 저장소를 사용. 이러한 저장소는 종종 이벤트를 이벤트 스트림에 추가하고 스트림에서 이벤트를 읽기 위한 API를 제공. 여러 데이터 모델 및 데이터 유형을 지원할 수 있으며 고속 데이터를 처리하도록 구축. 이벤트 저장소의 예로는 Event Store DB, Apache Kafka 및 Amazon Kinesis 가 있음
2. NoSQL 데이터베이스 : NoSQL 데이터베이스는 스키마 없는 데이터를 처리하는 기능, 수평적 확장성 및 다양한 데이터 유형을 처리하는 유연성으로 인해 이벤트를 저장하는 데에도 사용. MongoDB와 같은 문서 기반 NoSQL 데이터베이스, Redis와 같은 키-값 저장소, Cassandra와 같은 와이드 컬럼 저장소 또는 InfluxDB와 같은 시계열 데이터베이스는 필요에 따라 적절한 선택이 될 수 있음
3. 관계형 데이터베이스 : 전통적으로 이벤트 소싱과 관련되지는 않았지만 관계형 데이터베이스를 사용하여 이벤트를 저장할 수도 있음. 이벤트는 이벤트 ID, 이벤트 유형, 이벤트 페이로드 및 타임스탬프에 대한 열이 있는 테이블에 저장. PostgreSQL, MySQL 및 Oracle 데이터베이스는 관계형 데이터베이스의 예이다.
4. 메시지 대기열 : 이 외에옫 RabbitMQ 또는 Apache Kafka 와 같은 메시지 대기열을 사용하여 이벤트를 저장할 수 있음. 마이크로 서비스 아키텍쳐의 여러 서비스 간에 이벤트를 분산해야 하고 이벤트를 실시간으로 처리해야 하는 경우에 특히 유용.

이벤트 스토리지 솔루션을 선택할 때 이벤트의 양과 속도, 실시간 처리의 필요성, 이벤트 데이터의 복잡성, 기존 기술 스택의 기능과 같은 요소를 고려하는 것이 중요하다.

```
@app.post("/investors/", response_model=Investor)
async def create_investor(investor: InvestorCreate):
    # Create event
    event = Event(
        id=str(uuid.uuid4()),
        type="InvestorCreated",
        payload=investor.dict(exclude={"password"})
    )

    # Store event
    store_event.delay(event.dict())

    # Process command
    db = SessionLocal()
    hashed_password = hashlib.sha256(investor.password.encode()).hexdigest()
    db_investor = models.Investor(name=investor.name, id=investor.id, password=hashed_password)
    db.add(db_investor)
    db.commit()
    db.refresh(db_investor)
    db.close()
```

주어진 코드에서 `store_event.delay(event.dict())` 부분이 이벤트를 저장하는 부분이며, `delay` 메소드는 Celery의 비동기 작업을 의미합니다. 따라서 이벤트는 `store_event` 작업이 어떻게 구현되었는지에 따라 DB나 메시지 큐에 저장될 것입니다. 

특정 시간의 펀드 자산 가치를 확인할 수 있도록 이벤트 로그를 조합하려면, 이벤트 로그는 시간에 따른 펀드의 가치 변화를 추적할 수 있는 충분한 정보를 포함해야 합니다. 이를 위해서는 펀드 자산 가치 변화에 영향을 미치는 모든 행동(예: 음악저작권 구매/판매, 가치 변동 등)을 이벤트로 기록해야 합니다. 이러한 이벤트를 데이터베이스에 저장하면, 시간 순서대로 이벤트를 가져와서 특정 시간의 펀드 자산 가치를 계산할 수 있습니다.

또한, SQLAlchemy와 같은 ORM을 사용하는 경우, 이벤트를 데이터베이스에 저장하기 전에 이벤트 데이터를 유효성 검사하고 필요한 경우 변환하는 등의 작업을 수행할 수 있는 이벤트 모델을 만드는 것이 좋습니다. 이렇게 하면 이벤트 데이터의 일관성과 신뢰성을 보장할 수 있습니다.

아래는 이를 반영한 코드입니다:

## store_event_async() celery 처리

해당 코드는 Celery 비동기 작업으로 이벤트를 저장하는 역할을 하고 있습니다. 현재 코드는 이벤트를 단순히 `events` 리스트에 추가하고 있는데, 이것을 SQLAlchemy를 사용해 데이터베이스에 저장하도록 변경하겠습니다.

```
from core.db import SessionLocal

class Event(BaseModel):
    id: str
    type: str
    payload: dict

@celery_app.task
def store_event_async(event_dict):
    db = SessionLocal()

    # Convert event_dict to Event instance
    event = Event(
        id=event_dict["id"],
        type=event_dict["type"],
        payload=event_dict["payload"]
    )

    # Add the event to the session and commit
    db.add(event)
    db.commit()
    db.close()
```

위 코드는 이벤트 사전을 `models.Event` 인스턴스로 변환하고, 이를 데이터베이스 세션에 추가한 다음 커밋합니다. 이렇게 하면 데이터베이스에 이벤트가 저장됩니다.

이렇게 이벤트를 데이터베이스에 저장하면, 나중에 특정 시점의 시스템 상태를 재구성하기 위해 이벤트 로그를 조회할 수 있습니다.

## 펀드의 가치를 계산하는 compute_fund_value()

투자자가 음악 저작권을 수익화하거나 보유한 음악 저작권의 현재가치를 조회할때 모두 사용됩니다.

아래 코드는 투자자가 현재 보유한 음악 저자권의 현재가치를 조회하는 메서드입니다.

```python
@app.get("/investment/{investor_id}", response_model=InvestmentDetails)
@Transactional()
async def get_investment_details(investor_id: str):
    db = session

    investor = await db.query(models.Investor).filter(models.Investor.id == investor_id).first()
    if not investor:
        raise HTTPException(status_code=404, detail="Investor not found")

    # Fetch all investments of the investor
    investments = await db.query(models.Investment).filter(models.Investment.investor_id == investor_id).all()
    current_value = 0.0

    for investment in investments:
        fund_value = await compute_fund_value(db, investment.fund_id)

        # Compute the current value of the investment
        current_value += fund_value * investment.investment_amount / investment.fund.total_supply

    return {"investment_amount": investor.investment_amount, "current_value": current_value}
```

그럼 compute_fund_value() 부분을 살펴보겠습니다

```python
async def compute_fund_value(db: Session, fund_id: int):
    # Fetch all 'McpBought' and 'McpSold' events of the fund
    buy_events = await db.query(models.Event).filter(
        models.Event.type == "McpPurchased",
        models.Event.payload["fund_id"] == fund_id).all()
    sell_events = await db.query(models.Event).filter(
        models.Event.type == "McpSold",
        models.Event.payload["fund_id"] == fund_id).all()

    # Compute the current value of the fund
    fund_value = sum(event.payload["price_cash"] for event in buy_events) - sum(
        event.payload["price_cash"] for event in sell_events)

    return fund_value
```

데이터베이스에 저장되어있는 event 에서 mcp를 구매한 내역과 판매한 내역을 조회한 후

price_cash (캐시가격)을 계산합니다. ok 카우에서는 1단위 = 100 캐쉬 입니다.

현재의 방식은 모든 McpPurchased, McpSold를 가져와서 직접 연산을 수행하는 것입니다. 이 방법은 절대적인 이벤트 수가 많아지면 성능이 문제가 될 수 있습니다. 

이를 개선하기 위해선 이벤트가 발생할 때마다 ‘McpPurchased’ 와 ‘McpSold’ 이벤트에 대한 누적 값을 계산하고 이를 데이터베이스에 저장하는 방식을 사용할 수 있습니다. 이렇게 하면 매번 직접 연산을 수행하지않고 누적된 값만 조회합니다.

예를들어 Fund 모델에 **`accumulated_purchase_value`** 와 **`accumulated_sold_value`** 두 개의 새로운 필드를 추가합니다. 위 compute_fund_value() 는 다음과 같이 간단하게 수정될 수 있습니다.

```python
async def compute_fund_value(db: Session, fund_id: int):
    fund = await db.query(models.Fund).filter(models.Fund.id == fund_id).first()

    # Compute the current value of the fund
    fund_value = fund.accumulated_purchase_value - fund.accumulated_sold_value

    return fund_value
```

## 음악저작권을 구매하는 `purchase_mcp`()

```python
@app.post("/mcp/purchase", response_model=schemas.McpPurchase, status_code=status.HTTP_200_OK)
@Transactional()
def purchase_mcp(mcp_purchase: McpPurchase, db: Session = Depends(session)):
    # 펀드를 가져옵니다.
    fund = db.query(models.Fund).first()

    # 펀드 자금이 구매 가격보다 많은지 확인
    if fund.capital_cash < mcp_purchase.price_cash:
        raise HTTPException(status_code=400, detail="Insufficient fund")

    # 펀드 자금에서 구매 가격을 차감
    fund.capital_cash -= mcp_purchase.price_cash

    # Mcp를 inventory에 등록
    mcp = models.Mcp(copyright_code=mcp_purchase.copyright_code,
                      registration_number=mcp_purchase.registration_number,
                      music_id=mcp_purchase.music_id,
                      estimated_value_cash=mcp_purchase.estimated_value_cash,
                      acquisition_date=mcp_purchase.acquisition_date)
    db.add(mcp)

    # Event 생성 및 저장
    event = Event(
        id=str(uuid.uuid4()),
        type="McpPurchased",
        payload=mcp_purchase.dict()
    )
    store_event_async.delay(event.dict())

    db.refresh(mcp)

    return {"asset_id": mcp.id}
```

`purchase_mcp` 는 음악저작권을 구매하는 메서드입니다. 펀드를 조회한후 해당 펀드의 자금이 구매하려는 음악저작권 이상인지 먼저체크합니다. 구매 가능하다면 펀드 자금에서 음악 저작권의 가격을 차감하고 

구매한 음악저작권을 inventory에 등록합니다. 

이어서 event를 생성하고 저장하는 코드가 나오고 구매한 음악저작권의 id를 return 하는 것으로 종료됩니다.

전체 메서드는 @Transactional() 데코레이터로 감싸져있으며 메서드 중간에서 실패한다면 롤백됩니다.

## celery를 사용한 update_mcp_asset_value() 주기적 실행

```python
@celery_app.on_after_configure.connect
def setup_periodic_tasks(sender, **kwargs):
    # Calls update_mpc_asset_value() every day at midnight
    sender.add_periodic_task(crontab(minute=0, hour=0), update_mcp_asset_value.s())
```

이 코드는 매일 자정에 **`update_mcp_asset_value`** 함수를 수행하도록 스케줄링합니다. 이 함수는 모든 mcp의 가치를 추정하고, 이를 합산하여 펀드의 자산 가치를 업데이트합니다.

```python
@celery_app.task
def update_mcp_asset_value():
    db = SessionLocal()  # Create a new DB session
    try:
        funds = db.query(models.Fund).all()  # Retrieve all funds from DB
        for fund in funds:
            total_value = 0
            for mcp in fund.mcps:  # Retrieve all MCPs belong to the fund
                estimated_price = get_mcp_index(mcp.copyright_code, mcp.registration_number, mcp.music_id)
                if estimated_price is not None:
                    total_value += estimated_price

            fund.asset_value = total_value
            fund.capital_cash = fund.capital_cash + total_value  # Update the fund's capital with the new asset value

            # Update the base price of the fund's shares
            if fund.total_supply != 0:  # Prevent division by zero
                fund.base_price = fund.capital_cash / fund.total_supply

        db.commit()
    except Exception as e:
        print(f"An error occurred: {e}")
    finally:
        db.close()
```

조금 부연설명을 하면, mcp의 개별 가격이 MCPI (MUSIC COPYRIGHT PROPERTY INDEX : 음악 저작권 지수)를 기반으로 예측된다고 가정했습니다.

실제 MCPI 지수는 뮤직카우 옥션을 통해 공유된 저작권을 구성종목으로 산출한 총수익지수로, 코스콤과 협의하여 개발되었으며, 매월 저작권료가 발생되는 음원 저작권의 특성과 해당 수익이 재투자되는 것을 고려하고 있습니다. 2019년 1월 1일을 기준시점으로 산출하고 있습니다.

주의할 점은 Celery 작업이 별도의 프로세스에서 수행되므로, DB 연결이 공유되어 있지 않는다는 점입니다. 
따라서 각 Celery 작업 내에서 DB 세션을 직접 관리해야 합니다. 

처음 시작부분에서

`db = SessionLocal()`

해당 코드를 통해 새로운 DB세션을 생성하고 이후 펀드 자산 가치를 업데이트하는 코드가 실행됩니다.


## 참고자료


[뮤직카우-MCPI](https://www.musicow.com/mcpi)

[MSA의 트랜잭션 이야기3 - 이벤트 소싱과 CQRS](https://blog.neonkid.xyz/267)

