## Database : Finders

<ul id="topic-list">
		<li><a href="/docs/database/finders#single-record-result">Single record result</a></li>
		<li><a href="/docs/database/finders#multiple-records-result">Multiple records result</a></li>
		<li><a href="/docs/database/finders#finder-options">Finder options</a></li>
		<li><a href="/docs/database/finders#conditions">Conditions</a></li>
		<li><a href="/docs/database/finders#limit-offset">Limit & Offset</a></li>
		<li><a href="/docs/database/finders#order">Order</a></li>
		<li><a href="/docs/database/finders#select">Select</a></li>
		<li><a href="/docs/database/finders#from">From</a></li>
		<li><a href="/docs/database/finders#group">Group</a></li>
		<li><a href="/docs/database/finders#having">Having</a></li>
		<li><a href="/docs/database/finders#read-only">Read only</a></li>
		<li><a href="/docs/database/finders#dynamic-finders">Dynamic finders</a></li>
		<li><a href="/docs/database/finders#joins">Joins</a></li>
		<li><a href="/docs/database/finders#find-by-custom-sql">Find by custom SQL</a></li>
		<li><a href="/docs/database/finders#eager-loading">Eager loading associations</a></li>
	</ul>


ActiveRecord supports a number of methods by which you can find records such as: via primary key, dynamic field name finders. It has the ability to fetch all the records in a table with a simple call, or you can make use of options like order, limit, select, and group.


There are essentially two groups of finders you will be working with: <a href="/docs/database/finders#single-record-result">a single record result</a> and <a href="/docs/database/finders#multiple-records-result">multiple records result</a>. Sometimes there will be little transparency for the method calls, meaning you may use the same method to get either one, but you will pass an option to that method to signify which type of result you will fetch.


All methods used to fetch records from your database will go through <strong>Model::find()</strong>, with one exception, custom sql can be passed to <a href="/docs/database/finders#find-by-custom-sql">Model::find_by_sql()</a>. In all cases, the finder methods in ActiveRecord are statically invoked. This means you will always use the following type of syntax.


```php
1 class Book extends ActiveRecord\Model {}
2 
3 Book::find('all');
4 Book::find('last');
5 Book::first();
6 Book::last();
7 Book::all();

```

#### Single record result


Whenever you invoke a method which produces a single result, that method will return an instance of your model class. There are 3 different ways to fetch a single record result. We'll start with one of the most basic forms.


#### Find by primary key


You can grab a record by passing a primary key to the find method. You may pass an <a href="/docs/database/finders#finder-options">options array</a> as the second argument for creating specific queries. If no record is found, a RecordNotFound exception will be thrown.


```php
1 # Grab the book with the primary key of 2
2 Book::find(2);
3 # sql => SELECT * FROM `books` WHERE id = 2

```

#### Find first


You can get the first record back from your database two ways. If you do not pass conditions as the second argument, then this method will fetch all the results from your database, but will only return the very first result back to you. Null will be returned if no records are found.


```php
1 # Grab all books, but only return the first result back as your model object.
2 $book = Book::first();
3 echo "the first id is: {$book->id}" # => the first id is: 1
4 # sql => SELECT * FROM `books`
5 
6 # this produces the same sql/result as above
7 Book::find('first');

```

#### Find last


If you haven't yet fallen asleep reading this guide, you should've guessed this is the same as "find first", except that it will return the last result. Null will be returned if no records are found.


```php
1 # Grab all books, but only return the last result back as your model object.
2 $book = Book::last();
3 echo "the last id is: {$book->id}" # => the last id is: 32
4 # sql => SELECT * FROM `books`
5 
6 # this produces the same sql/result as above
7 Book::find('last');

```

#### Multiple records result


This type of result will always return an array of model objects. If your table holds no records, or your query yields no results, then an empty array will be given.


#### Find by primary key


Just like the single record result for find by primary key, you can pass an array to the find method for multiple primary keys. Again, you may pass an <a href="/docs/database/finders#finder-options">options array</a> as the last argument for creating specific queries. Every key which you use as an argument must produce a corresponding record, otherwise, a RecordNotFound exception will be thrown.


```php
1 # Grab books with the primary key of 2 or 3
2 Book::find(2,3);
3 # sql => SELECT * FROM `books` WHERE id IN (2,3)
4 
5 # same as above
6 Book::find(array(2,3));

```

#### Find all


There are 2 more ways which you can use to get multiple records back from your database. They use different methods; however, they are basically the same thing. If you do not pass an <a href="/docs/database/finders#finder-options">options array</a>, then it will fetch all records.


```php
1 # Grab all books from the database
2 Book::all();
3 # sql => SELECT * FROM `books`
4 
5 # same as above
6 Book::find('all');

```

Here we pass some options to the same method so that we don't fetch <strong>every</strong> record.


```php
1 $options = array('limit' => 2);
2 Book::all($options);
3 # sql => SELECT * FROM `books` LIMIT 0,2
4 
5 # same as above
6 Book::find('all', $options);

```

#### Finder options


There are a number of options available to pass to one of the finder methods for granularity. Let's start with one of the most important options: conditions.


#### Conditions


This is the "WHERE" of a SQL statement. By creating conditions, ActiveRecord will parse them into a corresponding "WHERE" SQL statement to filter out your results. Conditions can be extremely simple by only supplying a string. They can also be as complex as you'd like by creating a conditions string that uses ? marks as placeholders for values. Let's start with a simple example of a conditions string.


```php
1 # fetch all the cheap books!
2 Book::all(array('conditions' => 'price < 15.00'));
3 # sql => SELECT * FROM `books` WHERE price < 15.00
4 
5 # fetch all books that have "war" somewhere in the title
6 Book::find('all', array('conditions' => "title LIKE '%war%'"));
7 # sql => SELECT * FROM `books` WHERE title LIKE '%war%'

```

As stated, you can use *?* marks as placeholders for values which ActiveRecord will replace with your supplied values. The benefit of using this process is that ActiveRecord will escape your string in the backend with your database's native function to prevent SQL injection.


```php
 1 # fetch all the cheap books!
 2 Book::all(array('conditions' => array('price < ?', 15.00)));
 3 # sql => SELECT * FROM `books` WHERE price < 15.00
 4 
 5 # fetch all lousy romance novels
 6 Book::find('all', array('conditions' => array('genre = ?', 'Romance')));
 7 # sql => SELECT * FROM `books` WHERE genre = 'Romance'
 8 
 9 # fetch all books with these authors
10 Book::find('all', array('conditions' => array('author_id in (?)', array(1,2,3))));
11 # sql => SELECT * FROM `books` WHERE author_id in (1,2,3)
12 
13 # fetch all lousy romance novels which are cheap
14 Book::all(array('conditions' => array('genre = ? AND price < ?', 'Romance', 15.00)));
15 # sql => SELECT * FROM `books` WHERE genre = 'Romance' AND price < 15.00

```

Here is a more complicated example. Again, the first index of the conditions array are the condition strings. The values in the array after that are the values which replace their corresponding ? marks.


```php
1 # fetch all cheap books by these authors
2 $cond =array('conditions'=>array('author_id in(?) AND price < ?', array(1,2,3), 15.00));
3 Book::all($cond);
4 # sql => SELECT * FROM `books` WHERE author_id in(1,2,3) AND price < 15.00

```

#### Limit & Offset


This one should be fairly obvious. A limit option will produce a SQL limit clause for supported databases. It can be used in conjunction with the <strong>offset</strong> option.


```php
1 # fetch all but limit to 10 total books
2 Book::find('all', array('limit' => 10));
3 # sql => SELECT * FROM `books` LIMIT 0,10
4 
5 # fetch all but limit to 10 total books starting at the 6th book
6 Book::find('all', array('limit' => 10, 'offset' => 5));
7 # sql => SELECT * FROM `books` LIMIT 5,10

```

#### Order


Produces an ORDERY BY SQL clause.


```php
1 # order all books by title desc
2 Book::find('all', array('order' => 'title desc'));
3 # sql => SELECT * FROM `books` ORDER BY title desc
4 
5 # order by most expensive and title
6 Book::find('all', array('order' => 'price desc, title asc'));
7 # sql => SELECT * FROM `books` ORDER BY price desc, title asc

```

#### Select


Passing a select key in your <a href="/docs/database/finders#finder-options">options array</a> will allow you to specify which columns you want back from the database. This is helpful when you have a table with too many columns, but you might only want 3 columns back for 50 records. It is also helpful when used with a group statement.


```php
1 # fetch all books, but only the id and title columns
2 Book::find('all', array('select' => 'id, title'));
3 # sql => SELECT id, title FROM `books`
4 
5 # custom sql to feed some report
6 Book::find('all', array('select' => 'avg(price) as avg_price, avg(tax) as avg_tax'));
7 # sql => SELECT avg(price) as avg_price, avg(tax) as avg_tax FROM `books` LIMIT 5,10

```

#### From


This designates the table you are selecting from. This can come in handy if you do a <a href="/docs/database/finders#joins">join</a> or require finer control.


```php
1 # fetch the first book by aliasing the table name
2 Book::first(array('select'=> 'b.*', 'from' => 'books as b'));
3 # sql => SELECT b.* FROM books as b LIMIT 0,1

```

#### Group


Generate a GROUP BY clause.


```php
1 # group all books by prices
2 Book::all(array('group' => 'price'));
3 # sql => SELECT * FROM `books` GROUP BY price

```

#### Having


Generate a HAVING clause to add conditions to your GROUP BY.


```php
1 # group all books by prices greater than $45
2 Book::all(array('group' => 'price', 'having' => 'price > 45.00'));
3 # sql => SELECT * FROM `books` GROUP BY price HAVING price > 45.00

```

#### Read only


Readonly models are just that: readonly. If you try to save a readonly model, then a ReadOnlyException will be thrown.


```php
 1 # specify the object is readonly and cannot be saved
 2 $book = Book::first(array('readonly' => true));
 3 
 4 try {
 5   $book->save();
 6 } catch (ActiveRecord\ReadOnlyException $e) {
 7   echo $e->getMessage();
 8   # => Book::save() cannot be invoked because this model is set to read only
 9 }

```

#### Dynamic finders


These offer a quick and easy way to construct conditions without having to pass in a bloated array option. This option makes use of PHP 5.3's <a href="http://www.php.net/lsb" class="external">late static binding</a> combined with <a href="http://www.php.net/__callstatic" class="external">__callStatic()</a> allowing you to invoke undefined static methods on your model. You can either use YourModel::find_by which returns a <a href="/docs/database/finders#single-record-result">single record result</a> and YourModel::find_all_by returns <a href="/docs/database/finders#multiple-records-result">multiple records result</a>. All you have to do is add an underscore and another field name after either of those two methods. Let's take a look.


```php
 1 # find a single book by the title of War and Peace
 2 $book = Book::find_by_title('War and Peace');
 3 #sql => SELECT * FROM `books` WHERE title = 'War and Peace'
 4 
 5 # find all discounted books
 6 $book = Book::find_all_by_discounted(1);
 7 #sql => SELECT * FROM `books` WHERE discounted = 1
 8 
 9 # find all discounted books by given author
10 $book = Book::find_all_by_discounted_and_author_id(1, 5);
11 #sql => SELECT * FROM `books` WHERE discounted = 1 AND author_id = 5
12 
13 # find all discounted books or those which cost 5 bux
14 $book = Book::find_by_discounted_or_price(1, 5.00);
15 #sql => SELECT * FROM `books` WHERE discounted = 1 OR price = 5.00

```

#### Joins


A join option may be passed to specify SQL JOINS. There are two ways to produce a JOIN. You may pass custom SQL to perform a join as a simple string. By default, the joins option will not <a href="/docs/database/finders#select">select</a> the attributes from the joined table; instead, it will only select the attributes from your model's table. You can pass a select option to specify the fields.


```php
1 # fetch all books joining their corresponding authors
2 $join = 'LEFT JOIN authors a ON(books.author_id = a.author_id)';
3 $book = Book::all(array('joins' => $join));
4 # sql => SELECT `books`.* FROM `books`
5 #      LEFT JOIN authors a ON(books.author_id = a.author_id)

```

Or, you may specify a join via an <a href="/docs/database/relationships">related</a> model.


```php
 1 class Book extends ActiveRecord\Model
 2 {
 3   static $belongs_to = array(array('author'),array('publisher'));
 4 }
 5 
 6 # fetch all books joining their corresponding author
 7 $book = Book::all(array('joins' => array('author')));
 8 # sql => SELECT `books`.* FROM `books`
 9 #      INNER JOIN `authors` ON(`books`.author_id = `authors`.id)
10 
11 # here's a compound join
12 $book = Book::all(array('joins' => array('author', 'publisher')));
13 # sql => SELECT `books`.* FROM `books`
14 #      INNER JOIN `authors` ON(`books`.author_id = `authors`.id)
15 #         INNER JOIN `publishers` ON(`books`.publisher_id = `publishers`.id)

```

Joins can be combined with strings and associated models.


```php
 1 class Book extends ActiveRecord\Model
 2 {
 3   static $belongs_to = array(array('publisher'));
 4 }
 5 
 6 $join = 'LEFT JOIN authors a ON(books.author_id = a.author_id)';
 7 # here we use our $join string and the association publisher
 8 $book = Book::all(array('joins' => $join, 'publisher'));
 9 # sql => SELECT `books`.* FROM `books`
10 #      LEFT JOIN authors a ON(books.author_id = a.author_id)
11 #         INNER JOIN `publishers` ON(`books`.publisher_id = `publishers`.id)

```

#### Find by custom SQL


If, for some reason, you need to create a complicated SQL query beyond the capacity of <a href="/docs/database/finders#finder-options">finder options</a>, then you can pass a custom SQL query through Model::find_by_sql(). This will render your model as <a href="/docs/database/finders#read-only">readonly</a> so that you cannot use any write methods on your returned model(s).


<strong>Caution:</strong> find_by_sql() will NOT prevent SQL injection like all other finder methods. The burden to secure your custom find_by_sql() query is on you. You can use the Model::connection()->escape() method to escape SQL strings.


```php
1 # this will return a single result of a book model with only the title as an attirubte
2 $book = Book::find_by_sql('select title from `books`');
3 
4 # you can even select from another table
5 $cached_book = Book::find_by_sql('select * from books_cache');
6 # this will give you the attributes from the books_cache table

```

#### Eager loading associations


Eager loading retrieves the base model and its associations using only a few queries. This avoids the N + 1 problem.


Imagine you have this code which finds 10 posts and then displays each post's author's first name.
```php
1 $posts = Post::find('all', array('limit' => 10));
2 foreach ($posts as $post)
3   echo $post->author->first_name;

```


What happens here is the we get 11 queries, 1 to find 10 posts, + 10 (one per each post to get the first name from the authors table).


We solve this problem by using the <strong>include</strong> option which would only issue two queries instead of 11. Here's how this would be done:


```php
1 $posts = Post::find('all', array('limit' => 10, 'include' => array('author')));
2 foreach ($posts as $post)
3   echo $post->author->first_name;
4 
5 SELECT * FROM `posts` LIMIT 10
6 SELECT * FROM `authors` WHERE `post_id` IN (1,2,3,4,5,6,7,8,9,10)

```

Since <strong>include</strong> uses an array, you can specify more than one association:
```php
1 $posts = Post::find('all', array('limit' => 10, 'include' => array('author', 'comments')));

```


You can also <strong>nest</strong> the <strong>include</strong> option to eager load associations of associations. The following would find the first post, eager load the first post's category, its associated comments and the associated comments' author:


```php
1 $posts = Post::find('first', array('include' => array('category', 'comments' => array('author'))));

```