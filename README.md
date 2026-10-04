# Object-Oriented Principles in PHP

## Episode 01 — Classes

- **A class is a blueprint; an object (or instance) is one specific thing made from it.** A `Person` can have a name and age, while each person has their own values.
  ```php
  class Person
  {
      public $name;
      public $age;
  }

  $jeffrey = new Person();
  $jeffrey->name = 'Jeffrey';

  $jane = new Person();
  $jane->name = 'Jane';
  ```

- **Look for nouns when choosing possible classes.** In a blog, nouns such as `Post`, `User`, `Comment`, `Tag`, and `Category` can represent different concepts. PHP class names conventionally start with a capital letter.
  ```php
  class Post
  {
  }
  ```

- **Look for verbs when choosing methods for an object's behavior.** A post might be archived or shared, and a comment might respond to another comment. Noun and verb naming is a useful starting point, not a strict rule; method bodies below are intentionally empty to focus on their names.
  ```php
  class Post
  {
      public function archive()
      {
      }

      public function share()
      {
      }
  }

  class Comment
  {
      public function respond()
      {
      }
  }
  ```

> **Takeaway:** Classes describe a kind of thing, objects hold particular instances of it, and methods describe behavior.

## Episode 02 — Objects

- **How do classes, objects, and properties relate?** A class is a blueprint; each object is an instance with its own property values. A `public` property can be read or changed from outside the class.
  ```php
  class Playlist
  {
      public $name;
  }

  $headbangers = new Playlist();
  $headbangers->name = '80s headbangers';

  $depressing = new Playlist();
  $depressing->name = 'Depressing 90s';
  ```

- **How does a constructor initialize an object?** PHP calls the special `__construct` method when `new` creates an object. Here, `$this` refers to the object being initialized, and the optional songs argument defaults to an empty array.
  ```php
  class Playlist
  {
      public $name;
      public $songs;

      public function __construct($name, $songs = [])
      {
          $this->name = $name;
          $this->songs = $songs;
      }
  }

  $playlist = new Playlist('80s headbangers', ['Back in Black', 'Hells Bells']);
  ```

- **How can a class provide behavior?** Add a method to the class; `shuffle()` delegates to PHP's built-in `shuffle()`, which changes the songs array in place.
  ```php
  class Playlist
  {
      public $songs;

      public function __construct($songs = [])
      {
          $this->songs = $songs;
      }

      public function shuffle()
      {
          shuffle($this->songs);
      }
  }

  $playlist = new Playlist(['Back in Black', 'Hells Bells', 'Highway to Hell']);
  $playlist->shuffle();
  ```

- **What does constructor property promotion change?** In PHP 8, visibility on a constructor parameter declares and assigns the property for you, replacing the separate declarations and assignments shown above.
  ```php
  class Playlist
  {
      public function __construct(
          public $name,
          public $songs = [],
      ) {
      }
  }

  $playlist = new Playlist('80s headbangers', ['Back in Black', 'Hells Bells']);
  ```

> **Takeaway:** A class defines the properties and behavior its objects share, while each object keeps its own data.
