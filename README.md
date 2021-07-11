# Private Methods

## Learning Goals

- Define private methods
- Understand the context in which you will use such a method
- See how private methods are defined

## Background: Instance and Class Methods

As we dive deeper into object orientation in Ruby, we've been working with
different types of methods: instance and class methods. Remember that instance
methods are called on instances of a class. Let's make the following class
called `Bartender`.

```ruby
class Bartender
  attr_accessor :name

  def initialize(name)
    @name = name
  end

  def intro
    "Hello, my name is #{self.name}!"
  end
end

phil = Bartender.new("Phil")
phil.intro
#=> "Hello, my name is Phil!"
```

`intro` is an instance method because we can call it on an instance of the
`Bartender` class.

And class methods we call on the entire class itself, not the instance. Like if
we had a method that kept track of all of the new instances of `Bartender`:

```ruby
class Bartender
  attr_accessor :name

  @@all = []

  def initialize(name)
    @name = name
    @@all << self
  end

  def self.all
    @@all
  end

  def intro
    "Hello, my name is #{self.name}!"
  end
end
```

Here we're shoveling in every new instance of `Bartender` into a class variable
that holds onto all bartenders. Then we have a class method `self.all`, which
we'll call on the class itself to return all of the bartenders.

```ruby
phil = Bartender.new("Phil")
nancy = Bartender.new("Nancy")

Bartender.all
#=> [#<Bartender:0x007f94cb16bbd0 @name="Phil">, #<Bartender:0x007f94cb16bb58 @name="Nancy">]
```

## Public vs. Private Methods

### Defining Public Methods

We've already been writing public methods: `intro` and `self.all`. We can call
them from outside the scope of the class declaration, like on the instance of
the class or the class itself. Public methods are called by an explicit
receiver: the instance of `phil` explicitly receives the method `intro`.

### Defining Private Methods

Private methods cannot be called by an explicit receiver. What does that mean?
It means we can only call private methods **within** the context of the defining
class: the receiver of a private method is always `self`.

### Why Use Private Methods?

Private methods are a way of encapsulating functionality within a class. For
example, a bartender can make a drink at a customer's request. Part of the
process of making a drink includes choosing liquors, mixers, garnish, and
stirring everything together. As a customer, you're permitted to ask the
bartender for a drink (from a menu of options), but you can't instruct him or
her on each step. The smaller steps that make up the bartender's job can be
considered private methods.

Private methods also signal to other developers that this method is _depended_
on by other methods in your program. It signals that they should beware of
removing such a method for fear of breaking other parts of the program that they
may not realize rely on it.

### Building Private Methods

We've already written a private method in our `Bartender` class: `#initialize`.

```ruby
phil.initialize
#=>NoMethodError: private method `initialize' called for #<Bartender:0x007fafb4257dd8 @name="Phil">
```

The `#initialize` method is special for a couple reasons:

- We don't invoke it directly; we use `ClassName.new` to create a new instance
  and then invoke the `#initialize` method on that instance.
- It is a private method by default.

Private methods, aside from initialize, are usually written with the word
`private` above them. Let's make some private methods: `#choose_liquor`,
`#choose_mixer`, and `#choose_garnish`.

We'll also create a public method `#make_drink` that calls on those private
methods.

```ruby
class Bartender
  attr_accessor :name

  @@all = []

  def initialize(name)
    @name = name
    @@all << self
  end

  def self.all
    @@all
  end

  def intro
    "Hello, my name is #{name}!"
  end

  def make_drink
    @cocktail_ingredients = []
    choose_liquor
    choose_mixer
    choose_garnish
    "Here is your drink. It contains #{@cocktail_ingredients}"
  end

  private

  def choose_liquor
    @cocktail_ingredients << "whiskey"
  end

  def choose_mixer
    @cocktail_ingredients << "vermouth"
  end

  def choose_garnish
    @cocktail_ingredients << "olives"
  end

end
```

If we try to call `#choose_liquor` with an instance of a bartender, we get an
error:

```ruby
phil.choose_liquor
# NoMethodError: private method `choose_liquor' called for #<Bartender:0x007f9f5b03d318 @name="Phil">
```

Again, private methods cannot be called by an explicit receiver. Because `phil`
is the explicit receiver of `choose_liquor`, the method errors out.

However, if we call `make_drink`, the `choose_liquor` method works. What gives?

```ruby
phil.make_drink
# Here is your drink. It contains ["whiskey", "vermouth", "olives"]
```

The `#choose_liquor` method was called without any receiver. Ruby sees the
missing receiver and assumes it to be self, or the current object. When
`#choose_liquor` is called, self is an instance of Bartender. Only a Bartender
object can tell itself to choose a liquor, a mixer, and a garnish. Phil can tell
himself to choose a liquor, garnish, etc., but we cannot instruct Phil to do so.

Private methods restrict an outsider from calling methods that belong to an
object. However, we, as customers, are free to ask a bartender to make us a
drink (`#make_drink`).

**Note:** Prior to version 2.7 of Ruby, if we tried to call our private methods
from `#make_drink` using `self` (e.g., `self.choose_liquor` rather than just
`choose_liquor`), we would get that same 'private method' error. However,
beginning with version 2.7 of Ruby, private methods can be called from other
methods in the Class with or without the `self` keyword.

## Conclusion

When defining a class in Ruby, it's important to consider what kind of interface
the class exposes to the rest of your code base. By using a mix of private and
public methods, you can more carefully choose what methods are callable from
outside the class, and what methods are only callable from within the class.
Methods that are meant to be helper methods within your class, for example, are
good candidates to be private methods.

## Resources

- [Ruby Method Visibility](https://www.rubyguides.com/2018/10/method-visibility/)
