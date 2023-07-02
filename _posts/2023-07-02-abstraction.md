---
layout: post
title: Abstract, Abstract, Abstract
---

- [Intro](#intro)
- [What is abstraction?](#what-is-abstraction)
  - [In life](#in-life)
  - [Wasn't this a programming blog?](#wasnt-this-a-programming-blog)
  - [Object Oriented Programming](#object-oriented-programming)
  - [But, but performance!](#but-but-performance)
  - [So am I right?](#so-am-i-right)
- [Examples](#examples)
  - [Bevy](#bevy)
  - [Computers in general.](#computers-in-general)

## Intro

So first blog, go easy :P

I am going to be using both [Python](https://www.python.org/) and [Rust](https://www.rust-lang.org/) in this blog, but you don't need to understand them to understand my point. I will mainly try to use Python but will use Rust for some parts.

Also why two langs? Well, Python is the easiest to read, hence using that the most, but some examples are easier in Rust, also I have a lot of examples from real Rust applications.

## What is abstraction?

### In life

Abstraction is really about ignoring complexity, once you understand the underlying concepts, if you even do, then you can generally just ignore it and use the abstraction.

Abstractions are used all the time in our daily lives, ever had those moments in science class where last year's information is for some reason now wrong? Well it is not wrong, it is just too abstract for this year.
But it was still useful.

A good example is electricity, you might have seen some [youtube videos](https://youtu.be/bHIhgxav9LY) talking about how electricity works, but who cares? Am I gonna think about that when I want to get a LED to blink? No, I use an abstraction because it is easier to think about. But you also have to know about the limits, like the "laws" of electricity breakdown at high voltages.

### Wasn't this a programming blog?

Oh right! The same thing goes in programming, I dare say programming is one of the fields with the most abstractions (maybe besides physics/science).

Abstractions help us build projects step by step, once we finish a part we can just ignore the implementation and just know it works. This works for planning as well, we can write code that uses functions we haven't defined and worry about them later. For example,

```python
def login(username: str, password: str):
    user_auth_info = Database.get_info(username)
    if user_auth_info.password_hash == hash_password(password):
        if user_auth_info.has_2fa:
            do_2fa_auth(user_auth_info)
        else:
            not_allowed()
    else:
        not_allowed()
```
here we could plan out how login would work in general, without worrying about how any of those used functions work, we don't worry about how we will do the hash, how we will store the user info, etc. Ofc we will deal with the latter, but for now, we don't care, meaning we can get a plan out quickly by just assuming we already have a secure hash function and database setup.

But, I _don't_ do this, why? well, it is hard to test! assume we did this, we couldn't run our program until it was finished. This is fine to do as pseudo-code, but as real code? just don't. Instead, you should start with the small stuff, yes plan it out like this. but start at the bottom and build up.

start at the simplest stuff, figure out password hashing, write [tests](https://docs.pytest.org/en/7.3.x/) for it, and when you know it works, forget about it, now it is abstracted away and you don't have to worry about how it works anymore (until the new dev breaks it :/).

### Object Oriented Programming

[OOP](https://en.wikipedia.org/wiki/Object-oriented_programming) is basically the holy grail of abstraction, yes it has issues, yes [functional](https://en.wikipedia.org/wiki/Functional_programming) code does abstraction really well as well. But the idea of packing values, methods and similar into its own little package is what abstraction is all about!

```python
class Enemy:
    def __init__(self, health: int, name: str):
        self.health = health
        self.name = name
    
    def deal_dmg(self, amount: int):
        self.health -= amount

    def is_dead(self) -> bool:
        return self.health <= 0
```

Here the code using this `Enemy` class doesn't have to care about how dealing damage works, or how death is calculated (I will touch more closely on this in the [Bevy Section](#bevy))

And here Python has even more abstraction to give us, [dataclasses](https://docs.python.org/3/library/dataclasses.html)! Do you see that `__init__` function? I would dare say 90% of `__init__` functions look like that, and that is what dataclasses solve.

```python
from dataclasses import dataclass

@dataclass
class Enemy:
    health: int
    name: str

    # ...
```

Here the `dataclass` decorator abstracts away the constructor (And a lot more!). [Decorators](https://realpython.com/primer-on-python-decorators/) are also another one of pythons abstractions, or syntatic sugars as it is more known.

### But, but performance! 

Well, it depends on the abstractions and the lang, I would argue the kind of abstraction I showed above is very normal already, splitting up your functions is not a new thing, even if your function is only called from one place, it is still fine to be a function. 

Functions don't only make reuse easier, it makes thinking about your code easier. And honestly, computers are so freaking fast now, is one function call really gonna matter? 

But also as I said it depends on the kind of abstraction, Rust for example (I said it would popup) has very good compile-time optimization, meaning they have [zero cost abstractions](https://stackoverflow.com/questions/69178380/what-does-zero-cost-abstraction-mean), meaning you are much more encouraged to abstract away.

> And a tip, if somebody wont shut up about performance, just link them this [video](https://youtu.be/tKbV6BpH-C8)

### So am I right?
> _Maybe I should have started with this section?_

Making stuff more abstract makes it easy to add new features, but it also wastes time if you never use that freedom!
> I spent 4 hours makeing the enemy system abstract enough to allow for basically any enemy we can imagine
> 
> _You did what? Dave, we only have one enemy in the game, just hardcode that \*\*\*\*!_

Maybe a bit extreme, but you get the point. If you wanna go more in-depth into that discussion, here is an excellent youtube video on the subject: 

> God this is the second video of theirs I linked in this blog, maybe you should just watch their whole channel, they have some great stuff
<iframe width="560" height="315" src="https://www.youtube.com/embed/rQlMtztiAoA" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" allowfullscreen></iframe>


## Examples

### Bevy

So [Bevy](https://bevyengine.org/) is a game engine for Rust, and it uses the [ECS](https://bevyengine.org/learn/book/getting-started/ecs/) (Entity, Component, System) pattern. This makes abstractions really easy and fluent.
> Also bevy internals are not nice to read, so dont event try to understand the black magic below :P

bevy lets us write code in an independent way, take this system to snap the currently held item to the cursor.
```rust
fn snap_to_cursor(
    cursor_location: Res<CursorLocation>,
    mut entities: Query<(&mut Transform, &CurrentlyHeld)>,
) // ...
```

Let's just look at the signature, what does this function work on? Well 2 main things

- `CursorLocation` The current location of the cursor
- `entities` Now this is where some bevy magic happens, we are getting a list of all entities that have a `Transform` and a `CurrentlyHeld` component, we don't care about what kind of entity it is, just that it is held (and that it has a location :P).

The rest is just Rust syntax, and I won't bore you with those details. 

Another way Bevy helps is with events, imagine we are making a roguelike or another game where stuff needs to react to a bunch of stuff. How do you model that? Do you keep a central list of all the different events, something like this? 
```python
events = {
    "enemy_dead": [...],
    "player_take_dmg": [...],
    ...
}
```

Well, that gets messy fast, instead in Bevy each system can either write or read. I shall give an example from one of my games, here the player can shoot enemies.

you might have logic like:
```rust
if bullet.hit(enemy) {
    enemy.health -= bullet.dmg
    if enemy.health <= 0 {
        ...
    }
}
```

but now all that logic is in one place? What if we want an enemy that only takes dmg from fire attacks or an enemy that spawns more enemies on death? this is going to get very complex

```rust
if bullet.hit(enemy) {
    match enemy.type {
        EnemyType::Normal => ...
        EnemyType::FireGod => ...
        ...
    }

    if enemy.health <= 0 {
        match enemy.type {
            EnemyType::Normal => ...
            EnemyType::FireGod => ...
            ...
        }
    }
}
```

To be fair it doesn't look too bad here, but still, this is all in the bullets logic, that doesn't feel right anymore with all the enemy logic we have here. you could make methods, but that gets clunky as you need to pass around a lot of values (like references to enemy assets, again something that feels out of place in a bullet script). 

The solution is events, we can split this into 3 functions
```rust
if bullet.hit(enemy) {
    event_write.send(EnemyHit(enemy, bullet.damage))
}
```

And then different functions can listen to the event to react, like 

```Rust
fn normal_enemy(mut enemies: Query<..., With<NormalEnemy>>, event_reader: ...) { ... }
fn normal_enemy(mut enemies: Query<..., With<SpecialEnemy>>, event_reader: ...) { ... }
```

Similar to death, this makes the code much easier. And also allows for easily adding those items I mentioned, because these functions can again dispatch events for when an enemy dies, or when an enemy takes dmg, etc. these systems won't care who reads the events, and the readers won't care why an enemy died, this is a very powerful form of abstraction. 

### Computers in general.

So that language you are using to write code? that is an abstraction over assembly code, which abstracts over machine code, which abstracts over the hardware itself! Building stuff from scratch is a great way to learn how all those layers work, I have personally designed a CPU and written [my own compiler](https://github.com/vivax3794/viv_script), even made an OS (don't get your hopes up, I am not replacing Windows any time soon :P)

if you want to do something similar, I highly recommend the game [Turing Complete](https://store.steampowered.com/app/1444480/Turing_Complete/), for lang dev there is [Crafting Interpreters](https://craftinginterpreters.com/), and for everything else, I highly recommend [Build your own X](https://github.com/codecrafters-io/build-your-own-x).