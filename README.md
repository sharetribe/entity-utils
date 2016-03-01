# EntityUtils

EntityUtils gives you tools to add a schemas to your data structures, namely Ruby hashes. It provides you means to perform validations and transformations to the hash structures.

## Motivation

Ruby is dynamically typed language. One of the drawbacks of this is that by looking your method signature, you can not be sure about the type and the structure of the parameters. For simple methods and simple types such as number, string, booleans etc. this may not be big issue, but as soon as you start passing hashes to methods, you easily loose track what data should be inside the hash.

## Installation

With bundler:

```ruby
# Add to Gemfile

gem 'entity_utils', git: 'https://github.com/sharetribe/entity-utils.git'
```

## Usage

You start by defining a builder, which will build the Entity hashes.

The builder takes the schema as a parameter. Each field specification in the schema is represented as an array.

For example, the following array describes a "name" field, which needs to be present (mandatory validator) and it needs to be string:

```ruby
[:name, :string, :mandatory]
```

A complete schema is a collection of these field specifications.

The following example defines a builder for Person entity:

```ruby
Person = EntityUtils.define_builder(
  [:name, :string, :mandatory],
  [:sex, :to_symbol, one_of: [:f, :m]]
)

Person.call({
  name: "John",
  sex: "m",
  age: 30
})

#=> { name: "John", sex: :m }
```

The Person builder schema contains to fields, `name` and `sex`. `name` needs to be present and it needs to be string. `sex` is first transformed to symbol and then validated that it is either `:m` of `:f`. `age` is ignored because it's not part of the schema.

## Transformers

Transformers transform the content of the hash. They are run before validators.

### List of predefined transformers

TODO

### Custom transformers

You can also define custom transformers. The custom transformer needs to be a lambda that returns ???. In addition, the transformer needs to be idempotent, which is a fancy way of saying that running the transformer once should return the same result as running it twice.

TODO Improve this

```ruby

to_time = ->(v) {
  if (v.is_a? Time) {
    # This is important check to ensure idempotency
    v
  } elsif (v.nil?) {
    v
  } else {
    Time.at(v)
  }
}

[:time, transform_with: to_time]
```

## Validators

Validators validate the content of the hash.

Validators are run:

* Before after transformers
* In the order they are defined

### Predefined validators

TODO

### Custom validators

You can also define custom validators. The custom validator needs to be a lambda that returns ???

TODO Improve this

```ruby

even_number = ->(v) { v.nil? && v.even? }

[:even_number, validate_with: even_number]
```

### Receive validation errors instead of exceptions

### Skip validations for production

Skipping validations in production will give you a slight performance gain. You can do this by adding an initializer:

```ruby
# config/initializer/entity_utils.rb

if Rails.env == "production"
  EntityUtils.configure!(
    validate: false
  )
end
```

Transformers will be run even if validations are skipped:

```ruby
EntityUtils.configure!(
   validate: false
)

person = EntityUtils.define_builder(
  [:name, :string, :mandatory],
  [:sex, :to_symbol, one_of: [:f, :m]]
)

person.call(name: "John", sex: "male")

#=> {name: "John", sex: :male}
```

If you want to make sure that validations are always run (despite the global configuration), use the `define_builder_validate_always` method:

```ruby
EntityUtils.configure!(
   validate: false
)

person = EntityUtils.define_builder_validate_always(
  [:name, :string, :mandatory],
  [:sex, :to_symbol, one_of: [:f, :m]]
)

person.call(name: "John", sex: "male")

#=> Exception
```

## Nested entities

You can also nest entities.

Use `entity` to nest a single entity:

```ruby

Address = EntityUtils.define_builder(
  [:street, :string, :mandatory],
  [:city, :string, :mandatory],
  [:postal_code, :string, :mandatory]
)

Person = EntityUtils.define_builder(
  [:name, :string, :mandatory],
  [:address, entity: Address]
)

Person.call({
  name: "John",
  address: {
    street: "Bulevardi 14",
    city: "Helsinki",
    postal_code: "00100"
  }
})
```

Use `collection` to nest a collection of entities:

```ruby
Item = EntityUtils.define_builder(
  [:name, :string, :mandatory],
  [:price_cents, :fixnum, default: 0]
)

ShoppingCart = EntityUtils.define_builder(
  [:items, collection: Item]
)

cart = ShoppingCart.new(
  items: [
    {name: "Shirt", price_cents: 5000},
    {name: "Shoes", price_cents: 7500},
    {name: "Free delivery!"}
  ]
)
```

## Inspired by

pluratic/schema

## Sponsored by

Sharetribe - An Open Source platrom for creating marketplaces

Create an online marketplace with our hosted solutin, it only takes a minute.
