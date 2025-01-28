<div align="center">
  <picture>
      <img alt="Shows the Memobase Workflow" src="./assets/images/what.png" width="80%">
    </picture>
  <h1>Aichemy</h1>
  <p><strong>🚧AI Hooks for your Database🚧</strong></p>
  <p>
    <img src="https://img.shields.io/badge/python->=3.9-blue">
    <a href="https://pypi.org/project/aichemy/">
      <img src="https://img.shields.io/pypi/v/aichemy.svg">
    </a>
  </p>
</div>



***Aichemy is still in development and everything in this readme is a possible concept yet.***

Aichemy makes it easy to use AI in your database. Consider this as an observation layer that watch the changes in DB and act when possible. With powerful AI, many seemingly complex update logics can be implemented:

- When `age` is updated, combining with `title` to guess `interested`(bool, if user wants to buy our services).
- When `bio` is updated, combing with `name` to generate a personalized `sale_dm`(string, content of DM)
- ...

All you need to do is focusing on the business logic, and write the instructions clearly.

## Install

```
pip install aichemy SQLAlchemy
```

> aichemy only supports SQLAlchemy for now. 



## QuickStart

#### 1. Design your DB

```python
from sqlalchemy import create_engine, Column, Integer, String
from sqlalchemy.orm import declarative_base, sessionmaker


class User(Base):
    __tablename__ = "users"
    id = Column(Integer, primary_key=True)
    name = Column(String)
    title = Column(String)
    intro = Column(String)
    dm_message = Column(String, nullable=True)
    
    
engine = create_engine("sqlite:///test.db")
Base.metadata.create_all(engine)
Session = sessionmaker(bind=engine)
```



#### 2. Write `aichemy functions`

```python
import aichemy as ac

@ac.watch(User.intro).with(User.name, User.title).to(User.dm_message)
def write_dm(row: dict, results: dict):
  """Based on the user information, write a thank-you DM for our AI product.
### Information
Name: {users.name}
Title: {users.title}
Self-Intro: {users.intro}
"""
  
  # pre-processes
  row['users.name'] = row['users.name'].capitalize()
  yield
  # post-process
  results['users.dm_message'] = results['users.dm_message'][:200]
```

- `row` is a dict containing the values of (`User.intro`, `User.name`, `User.title`)
- You can use `docstring` of the function to write your instructions
- Before `yield`, you can modify the values inside `row` for pre-process. At this time, `results` is always empty and any changes to `results` is meaningless.
- After `yield`, you can modify the return values inside `results` before they're committed to Database



#### 3. Start watching

```python
write_dm.start()

with Session() as session:
    user = User(name="John", title="Software Engineer", intro="I love AI and basketball")
    session.add(user)
    session.commit()
```



#### 4. Debuging

Debug the prompt is easy in aichemy:

```python
write_dm.debug({
  "users.name": "Gus",
  "title": "Founder of Memobase",
	"users.intor": "I'm a programmer",
})
```

