# Naming to Teach

I have a very simple thing to say about how code is taught. The more advanced you are, the more likely you teach the craft poorly. As you become more advanced in a field, more basic concepts in that field become so familiar that it becomes harder to recall what it was like before you understood those concepts. 

Assuming you actually know how to ride a bike--and have known how to do so for years--imagine the feeling you had of getting on a bike for the very first time. You tried to pedal and keep the handlebars straight, but you _still_ fell down. I can remember this struggle when I was five years old. 

If I got on a bike today, I would have to fight my instincts to try to pedal and fall down. It would actually be _hard_ for me to fall off my bike now. The old saying "it's like riding a bike" isn't just about something being intuitive; it's about how hard it is to recall the feeling of that thing not being intuitive.

Having worked with object-oriented programming languages for over twenty years, to me, the concept of a class constructor is obvious--it's a method which instantiate an instance of the object a class defines. Like riding a bike.

But, when I was first taught about class construction, I was at a complete and utter loss. The concept was one of those fancy italian racing bikes with those clip-on pedals. I was a mermaid. 

Looking back, it was unclear to me because I couldn't differentiate a class definition from an instance of the class--that these were two entirely different things. I couldn't visualize "instantiation". Is this a term ever used outside of programming?

A large part of the problem was how the concept was introduced. I recall seeing one of these example lines of code in my introductory class. While it's easy for me to understand now, I remember vividly the feeling of confusion staring at this kidn of code. I remember having to parse through it a dozen times to finally _get it_. The next day, it became completely foreign again.

```C#
Object myObject = new Object();
myObject.DoSomething();
```

A line of code like this has no relatable concept outside of the programming world to latch onto. The word `Object` appears three times in one line. It creates that dizzying effect I mentioned in _Code Shape_. A knowledgeable programmer knows that they each stand for something different.

* The first `Object` refers to the type of the instance we're instantiating.
* The second `Object` is part of the name of this instance `myObject`.
* The third `Object` is invoking the constructor.
* The fourth `Object` is part of the now-instantiated instance `myObject`, which is now performing the very detailed action of `DoSomething()`.

But, for a programming newbie, this made as much sense as this equally sensible line.

```C#
Xy8lksp0 myXy8lksp0 = new Xy8lksp0();
myXy8lksp0.DoSomething();
```

Some code writers will "improve" upon this example by introducing meaningless variable names--`foo`, `bar`, `baz`, and so forth.

```C#
Object foo = new Object();
foo.DoSomething();
```

This helps rid some of the dizzying effects. At least the instance name is more readily discernable. But, the newbie is still left wondering what this all means. What is a _foo_? I'd come to find that it's intentionally meant to mean nothing specific--a way of ensuring the novice understands that the concept can be pplicable to nearly every kind of a "thing". 

But, at such an early step in the process, concreteness is exactly what the novice needs. The mental bridge to the abstract is far easier to cross this way, than to immediately start with the abstract. 

Then there are the "concrete" examples that are easy to visualize, but unrelatable to the vast majority of real programming tasks. This makes the bridge from the example to something a novice might actually need to do, still, a difficult one to cross. I see this in many writeups when demonstrating how an interface is used, with the classic example always revolving around domesticated household pets.

```C#
interface IAnimal
{
    string Speak();
}

class Dog : IAnimal
{
    public string Speak()
    {
        return "Bark!";
    }
}

class Cat : IAnimal
{
    public string Speak()
    {
        return "Meow!";
    }
}
```
The example is certainly _concrete_, but when's the last time you worked on a video game with anima superheroes or for a pet store supply chain? . Imagine a newbie attempting to parlay the knowledge gained from this example of interfaces into a generic integration engine or a file export tool. 

It may be second nature for a seasoned programmer, but for someone who's just learning these new concept of inheritance and intefaces, it takes a large leap to correlate a method like `Speak()` with animals to `ExportFile()` with various file types. At the same time, it's not as if a newbie doesn't understand the concept that a document can be exported as a PDF, plain-text file, Word doc, and so forth.

So, why not skip the animal stuff and just start with a more relatable example -- you know, the ones you find when you're trying to do things with _software_. Then, it's imperative to name the constructs in our examples as _concretely_ as possible. Let the novice anchor their understanding around something real. This will make the bridge to other similar examples easy to cross. With confidence and momentum behind their newfound knowledge, it will make the further reaching bridges quicker to cross as well.

[Deitel and Deitel example of gradebook...concrete, and more real world -- we might be building some system...but still using myGradebook1 and myGradebook2]

A better example of constructors and class instantiation may be something like this:

```C#
User requester = new User(username, password);
requester.AttemptLogin();
```

Most all software these days require logins and users. This would be an instantly relatable example to the vast majority of novices. 

Nearly every programming book I know will start with an example of a default constructor method that always has no parameters. Now, I know technical authors do this to prove the point that a constructor's unique quality -- the thing that differentiates it from any other method of a class -- is that it instantiates the object. We want to show the newbie that the mere act of calling this thing -- the one with the same name as the class but with two parentheses at the end -- creates an instance. Adding parameters to that method would just get in the way of that point.

But, I remember thinking that for some reason, constructors _couldn't_ have parameters attached to them -- that somehow this made a constructor no longer a constructor. 

Instead, if we showed early examples of constructors with parameters, the whole idea of a constructor makes much more sense. 

```C#
var date_of_birth = new DateTime(1980,1,3);
Person mary = new Person("Mary", date_of_birth);

```

Ah, I see... we're creating a person named Mary born on January 3rd, 1980.

Of course, we know that constructors can have parameters. In fact, most well-written objects will have parameterized constructors that, in particular, accept data that can only be given during the instance's construction. This leads to all sorts of other concepts that bring out the empassioned object-oriented audience like immutability and hiding implementation details.

--

Also, idea of a parameter name neeed not be the name of the passed in thing!!!!

(string myText) doesnt have to pass something called myText. Show SCOPE. Silly things we take for granted but a newbie wouldnt.

[Finish]


