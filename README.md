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

## Episode 03 — DTOs, Types, and Static Analysis

- **How do property types catch invalid data earlier?** Without types, a playlist can accept `false` for its songs and only warn when you try to use it as an array. Adding types to the promoted properties makes PHP reject an invalid songs argument during construction with a `TypeError`.
  ```php
  // Before: no type checks on the arguments.
  class Playlist
  {
      public function __construct(
          public $name,
          public $songs = [],
      ) {
      }
  }

  $playlist = new Playlist('90s movie soundtracks', false);
  echo $playlist->songs[0]; // Warning: accessing an array offset on false.
  ```
  ```php
  // After: replace the class above with this typed version (PHP 8).
  class Playlist
  {
      public function __construct(
          public string $name,
          public array $songs = [],
      ) {
      }
  }

  $playlist = new Playlist('90s movie soundtracks', false);
  // TypeError: argument 2 ($songs) must be an array.
  ```

- **What is a data transfer object (DTO)?** It is an object that groups related data for passing around. A `Song` DTO gives each song named, typed properties and requires both values at construction, avoiding the missing-key problem of an associative array.
  ```php
  // Before: an associative array can omit an expected key.
  $song = ['name' => 'My Heart Will Go On'];
  echo $song['artist']; // Warning: undefined array key "artist".
  ```
  ```php
  // After: both constructor arguments are required.
  class Song
  {
      public function __construct(
          public string $name,
          public string $artist,
      ) {
      }
  }

  $song = new Song('My Heart Will Go On', 'Celine Dion');
  echo $song->artist; // Celine Dion
  ```

- **How do you access data from a playlist of song objects?** Each array item is now a `Song` instance, so use `->` to read its public properties. This example uses the typed `Playlist` and `Song` classes above.
  ```php
  $playlist = new Playlist('90s movie soundtracks', [
      new Song('My Heart Will Go On', 'Celine Dion'),
  ]);

  $song = $playlist->songs[0];
  echo $song->name . PHP_EOL;
  echo $song->artist . PHP_EOL;
  ```
  ```text
  My Heart Will Go On
  Celine Dion
  ```

- **Does an `array` type guarantee that every item is a `Song`?** It checks the container, not its contents: mixed elements still pass the constructor's runtime check. Using the classes above, the invalid items below cause warnings when the loop tries to read their artist.
  ```php
  $playlist = new Playlist('90s movie soundtracks', [
      new Song('My Heart Will Go On', 'Celine Dion'),
      false,
      'Just a title',
      null,
  ]);

  foreach ($playlist->songs as $song) {
      echo $song->artist . PHP_EOL;
  }
  ```

- **How can static analysis check an array's item types?** Static analysis checks code without executing it; tools such as PHPStan or Psalm understand `@param Song[] $songs` as an array of `Song` objects. PHP does not enforce this annotation at runtime; the episode's November 2024 recording notes that native generics were unavailable.
  ```php
  // Replace the earlier Playlist class; use the Song class above.
  class Playlist
  {
      /**
       * @param Song[] $songs
       */
      public function __construct(
          public string $name,
          public array $songs = [],
      ) {
      }
  }

  // A static analyzer can flag this argument: some items are not Song objects.
  $invalid = new Playlist('90s movie soundtracks', [
      new Song('My Heart Will Go On', 'Celine Dion'),
      false,
      'Just a title',
      null,
  ]);

  // Corrected: every item matches the documented type.
  $playlist = new Playlist('90s movie soundtracks', [
      new Song('My Heart Will Go On', 'Celine Dion'),
  ]);
  ```

> **Takeaway:** DTOs give related data a consistent structure, PHP types catch invalid arguments at runtime, and static analysis can check the item types inside arrays before execution.
