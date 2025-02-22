# Albums Model and Repository Classes Design Recipe

_Copy this recipe template to design and implement Model and Repository classes for a database table._

## 1. Design and create the Table

If the table is already created in the database, you can skip this step.

Otherwise, [follow this recipe to design and create the SQL schema for your table](./single_table_design_recipe_template.md).

*In this template, we'll use an example table `students`*

```
# EXAMPLE

Table: students

Columns:
id | name | cohort_name
```

## 2. Create Test SQL seeds

Your tests will depend on data stored in PostgreSQL to run.

If seed data is provided (or you already created it), you can skip this step.

```sql
-- EXAMPLE
-- (file: spec/seeds_{table_name}.sql)

-- Write your SQL seed here. 

-- First, you'd need to truncate the table - this is so our table is emptied between each test run,
-- so we can start with a fresh state.
-- (RESTART IDENTITY resets the primary key)

TRUNCATE TABLE items RESTART IDENTITY; -- replace with your own table name.
TRUNCATE TABLE orders RESTART IDENTITY;
-- Below this line there should only be `INSERT` statements.
-- Replace these statements with your own seed data.

INSERT INTO items (name, unit_price, quantity_unit) VALUES ('Super Shark Vacuum Cleaner', '99', '30');
INSERT INTO items (name, unit_price, quantity_unit ) VALUES ('Makerspresso Coffee Machine', '69', '15');
```

Run this SQL file on the database to truncate (empty) the table, and insert the seed data. Be mindful of the fact any existing records in the table will be deleted.

```bash
psql -h 127.0.0.1 your_database_name < seeds_{table_name}.sql
```

## 3. Define the class names

Usually, the Model class name will be the capitalised table name (single instead of plural). The same name is then suffixed by `Repository` for the Repository class name.

```ruby
# EXAMPLE
# Table name: students

# Model class
# (in lib/student.rb)
class Items
end

class Orders
end

# Repository class
# (in lib/student_repository.rb)
class ItemsRepository
end

class OrdersRepository
end
```

## 4. Implement the Model class

Define the attributes of your Model class. You can usually map the table columns to the attributes of the class, including primary and foreign keys.

```ruby
# EXAMPLE
# Table name: students

# Model class
# (in lib/student.rb)

class Items
    attr_accessor :name, :unit_price, :quantity_unit
end

class Orders
attr_accessor :item_name, :order_date, :customer_name
end

# The keyword attr_accessor is a special Ruby feature
# which allows us to set and get attributes on an object,
# here's an example:
#
# student = Student.new
# student.name = 'Jo'
# student.name
```

*You may choose to test-drive this class, but unless it contains any more logic than the example above, it is probably not needed.*

## 5. Define the Repository Class interface

Your Repository class will need to implement methods for each "read" or "write" operation you'd like to run against the database.

Using comments, define the method signatures (arguments and return value) and what they do - write up the SQL queries that will be used by each method.

```ruby
# EXAMPLE
# Table name: students

# Repository class
# (in lib/artist_repository.rb)

class ItemsRepository

  # Selecting all records
  # No arguments
  def all
    # Executes the SQL query:
    # SELECT name, unit_price, quantity_unit_order_id FROM items;

    # Returns an array of Items objects.


    # Executes the SQL query:
    # SELECT item_name, order_date, customer_name FROM orders;

    # Returns an array of Orders objects.

  end
  def find
     # Executes the SQL query:
     # SELECT name, unit_price, quantity_unit, order_id FROM albums where id = $1;

     #Returns a single Item objects.

     # Executes the SQL query:
     # SELECT item_name, order_date, customer_name FROM albums where id = $1;
     
     #Returns a single Order objects.
  end

  # Insert a new Item
  # Item is a new Item object
   def create(Item)
     #INSERT INTO items table(name, unit_price, quantity_unit, order_id) VALUES($1, $2, $3, $4);

     # returns nothing
   end

   # Insert a new Order
   # Item is a new order object
   def create(Order)
     # INSERT INTO orders table (item_name, order_date, customer_name ) values($1, $2, $3);

     # returns nothing
   end
end
```

## 6. Write Test Examples

Write Ruby code that defines the expected behaviour of the Repository class, following your design from the table written in step 5.

These examples will later be encoded as RSpec tests.

```ruby
# EXAMPLES

# 1
# Get all items

repo = ItemRepository.new

items = repo.all 
items.length # => 4
items.find.name # => 'Super Shark Vacuum Cleaner'
items.find.unit_price # => '99'
items.find.quantity_unit # => '30'
items.find.order_id # => ' '
# Add more examples for each method

# 2
# Get single Item
repo = ItemRepository.new

items = repo.find(1)
items.name # => 'Super Shark Vacuum Cleaner'
items.unit_price # => '99'
items.quantity_unit # => '30'
items.order_id # => ' '
# 3
# Get single Item
repo = ItemRepository.new

items.name # => 'Makerspresso Coffee Machine'
items.unit_price # => '69'
items.quantity_unit # => '15'
items.order_id # => ' '

# 4 
# Insert a new item
repo = ItemRepository.new
item = Item.new
items.name = 'philips'
items.unit_price = 100
items.quantity_unit = 20
items.order_id = ' '

repo.create(item)

all_items = repo.all

# all_items should contain the new item

```

Encode this example as a test.

## 7. Reload the SQL seeds before each test run

Running the SQL code present in the seed file will empty the table and re-insert the seed data.

This is so you get a fresh table contents every time you run the test suite.

```ruby
# EXAMPLE

# file: spec/student_repository_spec.rb

def reset_albums_table
  seed_sql = File.read('spec/seed_albums.sql')
  connection = PG.connect({ host: '127.0.0.1', dbname: 'music_library_test' })
  connection.exec(seed_sql)
end

describe AlbumRepository do
  before(:each) do 
    reset_albums_table
  end

  # (your tests will go here).
  
end
```

## 8. Test-drive and implement the Repository class behaviour

_After each test you write, follow the test-driving process of red, green, refactor to implement the behaviour._