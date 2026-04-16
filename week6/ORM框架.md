# 一、什么是ORM

- 用python的类表述数据库表，用python的对象表述表中的行映射
  - 用class定义数据库表结构（不用写CREATE TABLE）
  - 用类的实例代表表中的一行记录（不用构造INSERT语句）
  - 用方法调用执行增删改查

# 二、ORM代码结构解析

## 2.1基础ORM结构

### 2.1.1 创建数据库引擎

```python
engine = create_engine('sqlite:///library_orm.db', echo=True)
```

- 作用：告诉SQLAIchemy使用哪个数据库（SQLite 文件 `library_orm.db`），并连接。
- 相当于SQL中的 qlite3.connect('library_orm.db')
- echo=True：在控制台打印生成的 SQL 语句

### 2.1.2 创建ORM基类

```python
Base = declarative_base()
```

- 作用：提供元数据（Metadata）容器。每个继承Base的模型类，其表结构信息会被自动收集到 Base.metadata中。Base.metadata是一个 MetaData 对象，记录了所有已定义的表。

### 2.1.3 定义模型类

```python
class Author(Base):
    __tablename__ = 'authors'  # authors是映射到数据库的表名字
    author_id = Column(Integer,primary_key=True)
    name = Column(String,nullable=False)
    nationality = Column(String)

class Book(Base):
    __tablename__ = 'books'
    book_id = Column(Integer, primary_key=True)
    title = Column(String, nullable=False)
    published_year = Column(Integer)
```

- 每个模型类对应数据库中的一张表
- 相当于CREATE TABLE
- author_id、name对应表的列，Column定义列的类型、约束

### 2.1.4 创建表

```python
Base.metadata.create_all(engine)
```

- 作用：自动根据上述Base继承的模型类（Author、Book）在数据库中创建相应的表

### 2.1.5 关系映射

```python
class Author(Base):
    books = relationship("Book",back_populates="author")

class Book(Base):
    author_id = Column(Integer,ForeignKey('authors.author_id'))
    author = relationship("Author",back_populates="books")
```

- 在Author.books 中写 back_populates="author"：Author类中的books属性对应Book类中的author属性
- 在Book.author 中写 back_populates="books"：Book类中的author属性对应Author类中的books属性
- ForeignKey('authors.author_id')：在Book类中的author_id属性列的值必须等于Author类中的author_id属性列中的值

### 2.1.6 创建会话

```python
Session = sessionmaker(bind=engine)
session = Session()
```

- 会话是 ORM 的核心工作区，所有对数据库的操作（增删改查）都必须通过会话进行

## 2.2 ORM的增删改查
### 2.2.1 插入

```python
jk_rowling = Author(name='J.K.Rowling, nationality='British')
session.add(jk_rowling)
session.commit()
```

- 先创建jk_rowling的python对象，然后用session.add()把该对象加入队列，最后commit()提交
	- 相当于SQL的INSERT
- 可以连续add多个对象，最后统一commit()
### 2.2.2 查询

```python
results = session.query(Book).join(Author).all()
for book in results:
	print(f"书籍：{book.title}，作者：{book.author.name}")
```

- query(Book)：查询Book这张表
	- 相当于SQL的SELECT
- .join(Author)：检查Book和Author之间是否定义了外键关系，如果找到关系则自动关联
	- 这里的关系指的是：books.author_id = authors.author_id
	- 此时返回一个Query对象
- .all()：立即执行该语句并返回所有匹配的行，返回的是python列表

```python
book = session.query(Book).filter_by(title='Harry Potter').first()
```

- filter_by(title='Harry Potter') 相当于 WHERE title = 'Harry Potter'
- .first()返回第一条结果，如果没有则返回 None
### 2.2.3 更新

```python
book = session.quert(Book).filter_by(title='Harry Potter').dirst()
book.published_year = 1998
session.commit()
```

### 2.2.4 删除

```python
borrower = session.query(Borrower).filter_by(name='Bob').first()
session.delete(borrower)
session.commit()
```