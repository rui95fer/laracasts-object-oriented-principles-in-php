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

## Episode 04 — Dependencies, Coupling, and Interfaces

- **What is a dependency?** It is something a method or class needs to do its job. The `User` parameter declares that `subscribe()` needs a user; this initial example uses constructor property promotion from PHP 8.
  ```php
  class User
  {
      public function __construct(public string $emailAddress)
      {
      }
  }

  class Newsletter
  {
      public function subscribe(User $user)
      {
          // Subscription logic goes here.
      }
  }

  $newsletter = new Newsletter();
  $newsletter->subscribe(new User('jane@example.com'));
  ```

- **What does coupling mean?** Coupled code is linked to another type or implementation. Creating a specific provider inside `Newsletter` ties it to that provider; this may be acceptable, but switching services requires changing the class.
  ```php
  // Illustrative pseudocode: these are made-up SDK and database APIs.
  public function subscribe(User $user)
  {
      $api = new CampaignMonitorAPI();
      $api->addApiKey('example-api-key');
      $list = $api->findList('default');
      $list->addToList($user->emailAddress);

      $user->update(['newsletterSubscribed' => true]);
      return true;
  }
  ```

- **How does a provider wrapper hide complexity?** Move the service-specific code into a class with a simple method. This gives `Newsletter` a clearer API to call, although it still depends on the concrete provider until we introduce an interface.
  ```php
  // After: extract the SDK operations from the method above.
  class CampaignMonitorProvider
  {
      public function addToList(string $list, string $emailAddress): void
      {
          // Illustrative pseudocode: not a real CampaignMonitor API.
          $api = new CampaignMonitorAPI();
          $api->addApiKey('example-api-key');
          $subscriberList = $api->findList($list);
          $subscriberList->addToList($emailAddress);
      }
  }

  // Inside Newsletter::subscribe():
  $provider = new CampaignMonitorProvider();
  $provider->addToList('default', $user->emailAddress);
  // The illustrative user update and return follow as before.
  ```

- **What does an interface require?** An interface declares a contract: implementing classes must provide its methods with compatible signatures. Here, each provider must accept a list and email address, and `void` means the method returns no value; the interface supplies no method body.
  ```php
  interface NewsletterProvider
  {
      public function addToList(string $list, string $emailAddress): void;
  }

  // Replace the earlier provider definition with this implementation.
  class CampaignMonitorProvider implements NewsletterProvider
  {
      public function addToList(string $list, string $emailAddress): void
      {
          // The CampaignMonitor SDK operations from above go here.
      }
  }
  // Omitting addToList(), or declaring an incompatible signature,
  // makes the implementation invalid.
  ```

- **How does method injection remove the specific provider dependency?** Pass the dependency into the method rather than create it inside. A `NewsletterProvider` type accepts an object implementing that interface; it does not create an interface instance.
  ```php
  // Replace the earlier Newsletter; use User and the provider above.
  class Newsletter
  {
      public function subscribe(User $user, NewsletterProvider $provider): bool
      {
          $provider->addToList('default', $user->emailAddress);
          // Illustrative database update omitted:
          // $user->update(['newsletterSubscribed' => true]);
          return true;
      }
  }

  $newsletter = new Newsletter();
  $newsletter->subscribe(
      new User('jane@example.com'),
      new CampaignMonitorProvider(),
  );
  ```

- **When is constructor injection useful?** Pass a dependency into the constructor when multiple methods need the same object, then store it as a property. This PHP 8 example uses a public promoted property to match the lesson; visibility and encapsulation come later.
  ```php
  // Replace the method-injection version of Newsletter.
  class Newsletter
  {
      public function __construct(public NewsletterProvider $provider)
      {
      }

      public function subscribe(User $user): bool
      {
          $this->provider->addToList('default', $user->emailAddress);
          // The illustrative database update is omitted here too.
          return true;
      }
  }

  $newsletter = new Newsletter(new CampaignMonitorProvider());
  $newsletter->subscribe(new User('jane@example.com'));
  ```

- **How can you switch services without editing `Newsletter`?** Supply another object implementing `NewsletterProvider`. Each provider translates the shared `addToList()` method into its own service's API calls; the implementation body below is intentionally omitted.
  ```php
  class PostmarkProvider implements NewsletterProvider
  {
      public function addToList(string $list, string $emailAddress): void
      {
          // Service-specific integration would go here.
      }
  }

  // Use the constructor-injection Newsletter and User above.
  $user = new User('jane@example.com');

  // Choose one provider when constructing the newsletter.
  $newsletter = new Newsletter(new CampaignMonitorProvider());
  // Or replace that construction with:
  $newsletter = new Newsletter(new PostmarkProvider());

  $newsletter->subscribe($user);
  ```

> **Takeaway:** Isolate service details in provider classes and inject an object that follows a shared interface, so `Newsletter` can work with different providers through the same contract.

## Episode 05 — Inheritance and Abstract Classes

- **When does inheritance fit?** Use it when the child class is a kind of the parent class: a `Cart` is a `Vehicle`. The child, also called a subclass, uses `extends` to inherit properties and methods from its parent.
  ```php
  class Vehicle
  {
      public function accelerate(): void
      {
          echo 'Accelerating';
      }
  }

  class Cart extends Vehicle
  {
  }

  (new Cart())->accelerate(); // Accelerating
  ```

- **How does a child change inherited behavior?** Override the method by defining it in the child with a compatible signature. Calling that method on the child now uses its implementation; without the override, it uses the inherited one.
  ```php
  // Replace Cart above; keep Vehicle.
  class Cart extends Vehicle
  {
      public function accelerate(): void
      {
          echo 'Rolling';
      }
  }

  (new Cart())->accelerate(); // Rolling
  ```

- **How can subclasses share initialization but behave differently?** A child inherits its parent's constructor when it does not define its own. Here, `EmailNotification` inherits the message property and initialization, then overrides `send()`; the echoed messages only simulate delivery. Constructor property promotion requires PHP 8.
  ```php
  class Notification
  {
      public function __construct(public string $message)
      {
      }

      public function send(): void
      {
          echo 'Show pop-up flash message: ' . $this->message;
      }
  }

  class EmailNotification extends Notification
  {
      public function send(): void
      {
          echo 'Send email: ' . $this->message;
      }
  }

  $notification = new EmailNotification('Your subscription renewal failed.');
  echo $notification->message . PHP_EOL;
  $notification->send();
  ```
  ```text
  Your subscription renewal failed.
  Send email: Your subscription renewal failed.
  ```

- **How can achievements share data while using different qualification rules?** Put the name, description, and icon in the parent, then give each subclass its own `qualifier()` method returning whether a user qualifies. This illustrative PHP 8 example uses counts stored on a dummy user; a real application would obtain them from its data.
  ```php
  // Separate example: this User replaces Episode 04's User.
  class User
  {
      public function __construct(
          public int $postCount,
          public int $commentCount,
      ) {
      }
  }

  class Achievement
  {
      public function __construct(
          public string $name,
          public string $description,
          public string $icon,
      ) {
      }
  }

  class FirstPostAchievement extends Achievement
  {
      public function qualifier(User $user): bool
      {
          return $user->postCount > 0;
      }
  }

  $firstPost = new FirstPostAchievement(
      'First Post',
      'Granted when you create your first post.',
      'first-post.svg',
  );

  $user = new User(postCount: 1, commentCount: 0);
  echo $firstPost->qualifier($user) ? 'They qualify' : 'They do not qualify';
  // They qualify
  ```

- **What does an abstract class prevent?** Declaring a class `abstract` prevents creating it directly while still letting subclasses inherit its data and implemented methods. Use it for a shared base such as `Achievement`, where you intend to create specific achievements.
  ```php
  // Replace Achievement above; keep User and FirstPostAchievement.
  abstract class Achievement
  {
      public function __construct(
          public string $name,
          public string $description,
          public string $icon,
      ) {
      }
  }

  // Invalid: new Achievement('First Post', 'Your first post.', 'first-post.svg');
  // Error: cannot instantiate abstract class Achievement.

  // Correct: create a concrete subclass using the inherited constructor.
  $firstPost = new FirstPostAchievement(
      'First Post',
      'Granted when you create your first post.',
      'first-post.svg',
  );
  ```

- **How does an abstract method require subclass behavior?** Declare the method without a body, ending it with a semicolon. Every concrete subclass must provide a compatible implementation, directly or through inheritance; a subclass that leaves it unimplemented must also be abstract. This ensures every usable achievement has a qualification rule.
  ```php
  // Replace Achievement again; keep User and FirstPostAchievement above.
  abstract class Achievement
  {
      public function __construct(
          public string $name,
          public string $description,
          public string $icon,
      ) {
      }

      abstract public function qualifier(User $user): bool;
  }

  // FirstPostAchievement already satisfies this requirement.
  class TalkativeAchievement extends Achievement
  {
      public function qualifier(User $user): bool
      {
          return $user->commentCount >= 200;
      }
  }

  $talkative = new TalkativeAchievement(
      'Talkative',
      'Granted when you write at least 200 comments.',
      'talkative.svg',
  );

  var_dump($talkative->qualifier(new User(postCount: 0, commentCount: 199)));
  var_dump($talkative->qualifier(new User(postCount: 0, commentCount: 200)));
  // Omitting qualifier() from this concrete subclass would cause a fatal error.
  ```
  ```text
  bool(false)
  bool(true)
  ```

> **Takeaway:** Inheritance lets specialized classes share data and behavior; abstract classes prevent direct instantiation, and abstract methods require concrete subclasses to supply the missing behavior.
