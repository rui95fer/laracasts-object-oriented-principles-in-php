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
