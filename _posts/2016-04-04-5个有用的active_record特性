---
layout: post
title: "5个有用的Active Record特性"
tags: [RoR, 一课吧, 原文翻译]
---

这里介绍Active Record的5个有用的API。

我们假设用“booksandreviews.com"rails应用来演示这些特性。

```ruby
  class Book < ActiveRecord::Base
    belongs_to :author
    has_many :reviews
  end

  class Author < ActiveRecord::Base
    has_many :books
  end

  class Review < ActiveRecord::Base
    belongs_to :book
  end
```

## 1. pluck

Ruby on Rails 4.0中引入的pluck方法。 pluck方法有助于在返回ActiveRecord查询结果时保持内存分配降到最低。一个典型的使用场景是在当数据库表支持的ActiveRecord对象有非常大的列数时，使用pluck方法。在这种情况下，返回对象完整的数据集会引起不必要的内存分配和潜在的昂贵的反序列化(在定制或JSON序列化列的情况下)。

从"fantasy"类型中得到book_ids列表，使用pluck方法很简单：

```ruby
  Book.where(genre: 'fantasy').pluck(:id)
  # SELECT "books"."id" FROM "books" WHERE "books"."genre" = 'fantasy'
  => [1, 3, 45, ...]
```

认识pluck方法的返回值是重要的。它不像select，pluck方法不返回ActiveRecord实例。

pluck允许指定多列，多列的值通过嵌套数组返回。返回的数组将保持请求时的列顺序：

```ruby
  Book.where(genre: 'fantasy').pluck(:id, :title)
  # SELECT "books"."id", "books"."title" FROM "books" WHERE "books"."genre" = 'fantasy'
  => [[1, 'A Title'], [3, 'Another One']]
```

在单个pluck中请求多个列虽然能行，但并不总是最好的办法。太多的列会产生一个难以驾驭的结果以及性能优势的丧失。

## 2. transaction
原子行为是关系型数据库的重要方面。当创建ActiveRecord对象时，通过使用transaction方法来维护原子性。

例如，如果在更新所有书的评论时发生异常，transaction可以帮助减轻有害的副作用：

```ruby
  book = Book.find(1)
  book.reviews.each do |review|
    review.meaningful_update!
  end
```

如果meaningful_update!方法抛出异常，错误之前的review将被更新，错误之后的将不被更新。如果这个方法是有副作用的（比如新增了一个字段），重新遍历所有review是有害的。

通过transaction就不存在这样的问题：

```ruby
  ActiveRecord::Base.transaction do
    book = Book.find(1)
    book.reviews.each do |review|
      review.meaningful_update!
    end
  end
```
添加transaction之后，如果一个异常被抛出，新代码将恢复包含在transaction中的所有更改。它相当于在单个查询中更改所有记录。


## 3. after_commit
不管你爱它们或恨它们，Rails回调是解决很多问题的可行选择。

Rails中使用回调的一个常见场景是围绕一个model被持久化后要做什么。model通过save，create或update进行持久化。概念“保存”是这些方法的共同点，使用 after_save 触发持久化之后的逻辑是合理的。

添加书保存后应该添加新书到ReviewQueue：

```ruby
class Book < ActiveRecord::Base
  after_save :enqueue_for_review

  def enqueue_for_review
    ReviewQueue.add(book_id: self.id)
    Logger.info("Added #{self.id} to ReviewQueue")
  end
end
```

我们假设ReviewQueue是用key/value存储（例如redis）对象。Logger类简单的输出文本到stdout。


当一切顺利时，回调工作的很好:
```ruby
Book.create!(
  title: 'A New Book',
  author_id: 3,
  content: 'Blah blah...'
)
#=> Added 4 to ReviewQueue
#=> <Book id: 4, title: 'A New Book' ..>

ReviewQueue.size
#=> 1
```

然而，如果这段代码被放置事务中，并且事务失败了，那么Book将不会被持久化，但ReviewQueue中的元素仍然存在。
```ruby
ActiveRecord::Base.transaction do
  Book.create!(
    title: 'A New Book',
    author_id: 3,
    content: 'Blah blah...'
  )

  raise StandardError, 'Something Happened'
end

#=> Added 4 to ReviewQueue
#=> Error 'Something Happened'

ReviewQueue.size
#=> 1
```
这段代码为不存在的书生成了ReviewQueue元素；如果使用after_commit回调，这个问题将被很好的解决。

使用after_commit替换after_save:
```ruby
class Book < ActiveRecord::Base
  after_commit :enqueue_for_review

  # ...
end
```
相同的代码，结果更加理想：
```ruby
ActiveRecord::Base.transaction do
  Book.create!(
    title: 'A New Book',
    author_id: 3,
    content: 'Blah blah...'
  )

  raise StandardError, 'Something Happened'
end

#=> Error 'Something Happened'

ReviewQueue.size
#=> 0
```
after_commit回调只在被持久化到数据库后触发，这正是我们需要的。除了after_commit，Rails还提供了更多的[回调钩子](http://guides.rubyonrails.org/active_record_callbacks.html)

## 4. touch

更新ActiveRecord对象的单个时间戳列，touch是很好的选择。这个方法能接收对象要更新的列名，除了:updated_at（如果存在）

当需要跟踪记录最后被查看的时，利用touch方法非常有效。

如果Review表有一个last_viewed_at列，类型为timestamp，touch可以很简单的更新last_viewed_at列：
```ruby
class ReviewsController < ApplicationController
  def show
    @review = Review.find(params[:id])
    @review.touch(:last_viewed_at)
  end
end
```
调用这个方法将导致查询：
```ruby
UPDATE "reviews"
SET "updated_at" = '2016-03-28 00:43:43.616367',
"last_viewed_at" = '2016-03-28 00:43:43.616367'
WHERE "reviews"."id" = 1
```
不像设置其它属性，touch方法调用一个更新查询。

## 5. changes
当更新时，ActiveRecord保留了对象相当多的信息。通过访问changes方法获得包含所有更新的信息的hash。另外，ActiveRecord为每个属性公开一系列辅助方法，提供更多的可见性。

changes hash中的每个键对应记录的属性。每个键的值是一个数组，包含两个元素：原来的属性值，现在的属性值：
```ruby
review = Review.find(1)
review.book_id = 5
review.changes
# => {"book_id"=>[4, 5]}
```

### a. changed?
  changed?方法用来确定对象自从找回后是否被更改。
```ruby
review = Review.find(1)
review.book_id = 5
review.changed?
# => true
```

不管怎样，如果设置的值和原来的值相同，changed? 返回false:
```ruby
book = Book.find(1)
book.title
# => 'A Book'
book.title.object_id
# => 2158279020
book.title = 'A Book'
book.changed?
# => false
book.title.object_id
# => 2158274600
```

两个“不同”的字符串，由于有相同的字符，对象就没有改变。

当使用PostgreSQL，如果对象没有改变，changed?方法可以节省不必要的 begin 和 commit 事务:

```ruby
class BooksController < ApplicationController
  def update
    @book = Book.find(sanitized_params[:id])
    @book.assign_attributes(sanitized_params)
    @book.save! if @book.changed?
  end
end
```

### b. &lt;attribute> _was
&lt;attribute>_was 返回属性被赋值前的值：
```ruby
review = Review.find(1)
review.status = 'Approved'
review.status_was
# => 'Pending'
```

如果Review的状态从"Pending"变成"Approved",应该从批准队列中移除，在回调中可以利用status_was方法
```ruby
class Review
  before_save :maybe_remove_from_review_queue

  def maybe_remove_from_review_queue
    if status == 'Approved' &&
        status_was == 'Pending'
      ReviewQueue.remove(id)
    end
  end
end
```

为了验证结果，我们可以检查ReviewQueue的size:
```ruby
ReviewQueue.size
#=> 1
review = Review.find(1)
review.status
# => 'Pending'
review.status = 'Approved'
review.save!
# => true
ReviewQueue.size
#=> 0
```

原文地址： http://jakeyesbeck.com/2016/03/27/five-more-active-record-features-you-should-be-using/
